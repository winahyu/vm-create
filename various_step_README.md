```bash

### bare metal instrospection
openstack overcloud node import --introspect --provide instackenv.json

### create flavor
openstack flavor create --id auto --ram 4096 --disk 40 --vcpus 1 computeovsdpdk
openstack flavor set --property "resources:CUSTOM_BAREMETAL"="1" --property "resources:DISK_GB"="0" --property "resources:MEMORY_MB"="0" --property "resources:VCPU"="0" --property "capabilities:boot_option"="local" --property "capabilities:profile"="computeovsdpdk" computeovsdpdk

###
