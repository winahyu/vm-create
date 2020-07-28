
### prepare Director VM
```bash
### Install required packages 
yum -y install virt-tools virt-resize libguestfs-tools libguestfs-xfs libvirt-client virt-install

### prepare IP address of Director VM
cat > ifcfg-eth0 <<EOF
DEVICE="eth0"
BOOTPROTO="static"
ONBOOT="yes"
TYPE="Ethernet"
PEERDNS="yes"
IPADDR=10.5.132.12 
NETMASK=255.255.255.0
GATEWAY=10.5.132.1
DNS1=10.16.129.90
DNS2=10.16.129.91
EOF

### Create QCOW2 file for Director VM
VM_IMAGE="/opt/source-images/rhel-guest-image-7.8-41.x86_64.qcow2"
VM=undercloud
VMPASSWD=password
qemu-img create -f qcow2 $VM.qcow2 100G
virt-resize --expand /dev/sda1 $VM_IMAGE $VM.qcow2
virt-customize -a $VM.qcow2 --hostname $VM --run-command "sed -i -e '/^PermitRootLogin/s/^.*$/PermitRootLogin yes/' /etc/ssh/sshd_config" --copy-in ifcfg-eth0:/etc/sysconfig/network-scripts --root-password password:$VMPASSWD --uninstall cloud-init --selinux-relabel
virt-install --ram 16384 --vcpus 4 --os-variant rhel7   --disk path=$VM.qcow2,device=disk,bus=virtio,format=qcow2   --import --noautoconsole --graphics vnc,listen=0.0.0.0  --network bridge=br0 --network bridge:br1 --name $VM  --cpu host,+vmx   --dry-run --print-xml > /tmp/$VM.xml
virsh define /tmp/$VM.xml
virsh start $VM
```

## Deploy undercloud
### Copy SSH key to director VM and login to director VM as root
```bash
ssh-copy-id director
ssh root@director
```

### Create stack user 
```bash
useradd stack
mkdir /home/stack/.ssh
cp /root/.ssh/authorized_keys /home/stack/.ssh/
chown -R stack:stack /home/stack/.ssh
echo 'stack ALL=(root) NOPASSWD:ALL' | tee -a /etc/sudoers.d/stack
chmod 0440 /etc/sudoers.d/stack
```



### Register Director VM and Download required packages
```bash
subscription-manager register --username=<RHN-USERNAME> --password=<RHN-PASSWORD>
subscription-manager attach --pool=<POOL_ID>
subscription-manager repos --enable rhel-7-server-extras-rpms
subscription-manager repos --enable rhel-7-server-rh-common-rpms
subscription-manager repos --enable rhel-ha-for-rhel-7-server-rpms
subscription-manager repos --enable rhel-7-server-openstack-13-rpms
yum update -y
reboot

```

### For Environment with Redhat Satellite only, Register Director VM and Download required packages from the Satellite
```bash
curl --insecure --output katello-ca-consumer-latest.noarch.rpm https://<SATELLITE>/pub/katello-ca-consumer-latest.noarch.rpm
yum localinstall katello-ca-consumer-latest.noarch.rpm
subscription-manager register --org="<ORG>" --activationkey="<KEY>"
subscription-manager attach --pool=<POOL_ID>
subscription-manager repos --enable rhel-7-server-extras-rpms
subscription-manager repos --enable rhel-7-server-rh-common-rpms
subscription-manager repos --enable rhel-ha-for-rhel-7-server-rpms
subscription-manager repos --enable rhel-7-server-openstack-13-rpms
yum update -y
reboot


### Install Openstack TripleOClient
```bash
Login back as stack user
ssh stack@director
sudo yum -y install python-tripleoclient
```

### Deploy Director VM services
```bash
cat > undercloud.conf <<EOF
[DEFAULT]
undercloud_hostname = undercloud.corp.wan
local_ip = 10.5.136.21/24
undercloud_public_host = 10.5.136.22
undercloud_admin_host = 10.5.136.23
undercloud_nameservers = 10.16.129.90
undercloud_ntp_servers = 10.16.229.90
subnets = ctlplane-subnet
local_subnet = ctlplane-subnet
#undercloud_service_certificate =
generate_service_certificate = true
certificate_generation_ca = local
local_interface = eth1
inspection_extras = false
undercloud_debug = false
enable_tempest = false
enable_ui = false

[auth]

[ctlplane-subnet]
cidr = 10.5.136.0/24
dhcp_start = 10.5.136.50
dhcp_end = 10.5.136.149
inspection_iprange = 10.5.136.150,10.5.136.200
gateway = 10.5.136.21
masquerade = true
EOF

openstack undercloud install

### review two files created by installation process

cat ~/stackrc
cat ~/undercloud-passwords.conf

```

### Director VM is ready
### Overcloud and Container Images preparation
```bash
sudo yum -y install rhosp-director-images
mkdir ~/images
tar -C ~/images -xvf /usr/share/rhosp-director-images/overcloud-full-latest.tar
tar -C ~/images -xvf /usr/share/rhosp-director-images/ironic-python-agent-latest.tar
openstack overcloud image upload --image-path ~/images
```
### Registry using rhn
```bash
REGISTRY=registry.access.redhat.com
LOCAL_REGISTRY=10.5.130.21:8787
openstack overcloud container image tag discover --image $REGISTRY/rhosp13/openstack-base --tag-from-label {version}-{release}
sudo openstack overcloud container image prepare --namespace=$REGISTRY/rhosp13 --push-destination=$LOCAL_REGISTRY  --tag=13.0 --tag-from-label {version}-{release} -e /usr/share/openstack-tripleo-heat-templates/environments/services/neutron-sriov.yaml -e /usr/share/openstack-tripleo-heat-templates/environments/docker.yaml -e /usr/share/openstack-tripleo-heat-templates/environments/docker-ha.yaml -e /usr/share/openstack-tripleo-heat-templates/environments/services/neutron-ovs-dpdk.yaml --output-env-file overcloud_images.yaml --output-images-file local_registry_images.yaml
sudo openstack overcloud container image upload --config-file local_registry_images.yaml --verbose
```
### Registry using Satellite
```bash
REGISTRY=local_satellite:5000
LOCAL_REGISTRY=10.5.130.21:8787
openstack overcloud container image tag discover --image $REGISTRY/default_organization-rhosp13-rhosp13_openstack-base --tag-from-label {version}-{release}
openstack overcloud container image prepare --namespace=$REGISTRY --push-destination=$LOCAL_REGISTRY  --prefix=default_organization-rhosp13-rhosp13_openstack- --tag-from-label {version}-{release}  -e /usr/share/openstack-tripleo-heat-templates/environments/services/neutron-sriov.yaml -e /usr/share/openstack-tripleo-heat-templates/environments/docker.yaml -e /usr/share/openstack-tripleo-heat-templates/environments/docker-ha.yaml -e /usr/share/openstack-tripleo-heat-templates/environments/services/neutron-ovs-dpdk.yaml --output-env-file overcloud_images.yaml --output-images-file local_registry_images.yaml
```
### Upload the container to local registry
```bash
sudo openstack overcloud container image upload --config-file local_registry_images.yaml --verbose

### verify the local registry
curl -s -H "Accept: application/json" http://$LOCAL_REGISTRY/v2/_catalog | jq .
```

