### Compared the local registry with redhat registry for the version differences.
```bash
LOCAL_REGISTRY=10.5.130.21:8787

curl http://$LOCAL_REGISTRY/v2/rhosp13/openstack-nova-libvirt/tags/list | jq .tags
curl https://access.redhat.com/webassets/docker/content/dist/rhel/server/7/7Server/multiarch/openstack/13/containers/rhosp13/openstack-nova-libvirt/tags/list | jq .tags
```

### Overcloud and Container Images update preparation
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

edit local_registry_images.yaml if only some containers that need to be updated
```

### Running the overcloud registry update preparation
```bash
sudo openstack overcloud container image upload --config-file local_registry_images.yaml --verbose

### verify the local registry
curl -s -H "Accept: application/json" http://$LOCAL_REGISTRY/v2/_catalog | jq .
```

### Updating all the nodes
```bash
source ~/stackrc

#Updating all Controller nodes
openstack overcloud update run --nodes Controller


#Updating all Compute nodes
openstack overcloud update run --nodes Compute

#finalizing.
openstack overcloud update converge \
    --templates \
    -e /home/stack/templates/overcloud_images.yaml \
    -e /usr/share/openstack-tripleo-heat-templates/environments/services/neutron-sriov.yaml \
    -e /usr/share/openstack-tripleo-heat-templates/environments/docker.yaml \
    -e /usr/share/openstack-tripleo-heat-templates/environments/docker-ha.yaml \
    -e /usr/share/openstack-tripleo-heat-templates/environments/services/neutron-ovs-dpdk.yaml \

```
