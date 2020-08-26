```
 VM=ceph01
 IMAGE=/home/CentOS-7-x86_64-GenericCloud-1802.qcow2
 qemu-img create -f qcow2 $VM.qcow2 60G
 virt-resize --expand /dev/sda1 $IMAGE $VM.qcow2
 virt-customize -a $VM.qcow2 --hostname $VM --run-command "sed -i -e '/^PermitRootLogin/s/^.*$/PermitRootLogin yes/' /etc/ssh/sshd_config" --root-password password:password --selinux-relabel --run-command "yum -y remove cloud-init"
 virt-install --virt-type kvm --name $VM --vcpu 2 --ram 8192 --disk $VM.qcow2,format=qcow2 --network network:default --network network:provision --graphics vnc,listen=0.0.0.0 --noautoconsole --os-type=linux --import

 VM=ceph02
 qemu-img create -f qcow2 $VM.qcow2 60G
 virt-resize --expand /dev/sda1 $IMAGE $VM.qcow2
 virt-customize -a $VM.qcow2 --hostname $VM --run-command "sed -i -e '/^PermitRootLogin/s/^.*$/PermitRootLogin yes/' /etc/ssh/sshd_config" --root-password password:password --selinux-relabel --run-command "yum -y remove cloud-init"
 virt-install --virt-type kvm --name $VM --vcpu 2 --ram 8192 --disk $VM.qcow2,format=qcow2 --network network:default --network network:provision --graphics vnc,listen=0.0.0.0 --noautoconsole --os-type=linux --import
 
 VM=ceph03
 qemu-img create -f qcow2 $VM.qcow2 60G
 virt-resize --expand /dev/sda1 $IMAGE $VM.qcow2
 virt-customize -a $VM.qcow2 --hostname $VM --run-command "sed -i -e '/^PermitRootLogin/s/^.*$/PermitRootLogin yes/' /etc/ssh/sshd_config" --root-password password:password --selinux-relabel --run-command "yum -y remove cloud-init"
 virt-install --virt-type kvm --name $VM --vcpu 2 --ram 8192 --disk $VM.qcow2,format=qcow2 --network network:default --network network:provision --graphics vnc,listen=0.0.0.0 --noautoconsole --os-type=linux --import
 ```
 
