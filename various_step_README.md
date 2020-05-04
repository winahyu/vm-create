```bash

### bare metal instrospection
openstack overcloud node import --introspect --provide instackenv.json
### review the registered nodes
openstack baremetal node list

### create flavor and set additional profile for computeovsdpdk
openstack flavor create --id auto --ram 4096 --disk 40 --vcpus 1 computeovsdpdk
openstack flavor set --property "resources:CUSTOM_BAREMETAL"="1" --property "resources:DISK_GB"="0" --property "resources:MEMORY_MB"="0" --property "resources:VCPU"="0" --property "capabilities:boot_option"="local" --property "capabilities:profile"="computeovsdpdk" computeovsdpdk

### assigned profiles to the registered bare-metal nodes
openstack baremetal node set --property capabilities='profile:computeovsdpdk,boot_option:local,boot_mode:uefi' compute01
openstack baremetal node set --property capabilities='profile:control,boot_option:local,boot_mode:uefi' controller01

### review the node profiles
openstack overcloud profiles list
```
