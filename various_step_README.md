```bash

### bare metal instrospection
openstack overcloud node import --introspect --provide instackenv.json
### review the registered nodes
openstack baremetal node list

### create flavor and set additional profile for computeovsdpdk
openstack flavor create --id auto --ram 4096 --disk 40 --vcpus 1 computeovsdpdk
openstack flavor set --property "resources:CUSTOM_BAREMETAL"="1" --property "resources:DISK_GB"="0" --property "resources:MEMORY_MB"="0" --property "resources:VCPU"="0" --property "capabilities:boot_option"="local" --property "capabilities:profile"="computeovsdpdk" computeovsdpdk

### assigned profiles to the registered bare-metal nodes
openstack baremetal node set --property capabilities='profile:computeovsdpdk,cpu_vt:true,cpu_hugepages:true,cpu_hugepages_1g:true,boot_option:local,boot_mode:uefi' compute01 openstack baremetal node set --property capabilities='profile:control,boot_option:local,boot_mode:uefi' controller01

### review the node profiles
openstack overcloud profiles list


## Overcloud 
### set flavor to support hugepages
openstack flavor set <FLAVOR> --property hw:mem_page_size=1048576

### create network segment
openstack network create --provider-network-type vlan --provider-physical-network datacentre1  --provider-segment 30 net30
openstack subnet create subnet30 --network net30  --subnet-range 192.168.30.1/24 --ip-version 4
```
