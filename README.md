```bash
# Manually Create a VM
## Step by step in RHEL/Centos/Fedora

### install the required tools

yum -y install virt-tools virt-resize libguestfs-xfs libvirt-client virt-install

yum -y install libguestfs-tools

### Download cloud image : ie rhel-guest-image-7.8-41.x86_64.qcow2


IMAGE=rhel-guest-image-7.8-41.x86_64.qcow2
VM=vm_name
VMPASSWD=vm_password

### Create base QCOW image.
qemu-img create -f qcow2 $VM.qcow2 100G

### resize and customize it with cloud image

virt-resize --expand /dev/sda1 $IMAGE $VM.qcow2

virt-customize -a $VM.qcow2 --hostname $VM --run-command "sed -i -e '/^PermitRootLogin/s/^.*$/PermitRootLogin yes/' /etc/ssh/sshd_config" --root-password password:$VMPASSWD --uninstall cloud-init --selinux-relabel

virt-install --ram 16384 --vcpus 4 --os-variant rhel7   --disk path=$VM.qcow2,device=disk,bus=virtio,format=qcow2   --import --noautoconsole --graphics vnc,listen=0.0.0.0  --network bridge=br-ex --network bridge:br-ctlplane --name $VM  --cpu host,+vmx   --dry-run --print-xml > /tmp/$VM.xml

### start the VM

virsh define /tmp/$VM.xml
virsh start $VM
```
