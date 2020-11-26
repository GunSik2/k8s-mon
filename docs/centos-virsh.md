## Libvirt & virt-install
```
# yum install net-tools virt-viewer wget
# vi /etc/selinux/config
SELINUX=disabled
# reboot
# getenforce
Disabled
# yum -y install qemu-kvm libvirt virt-install bridge-utils
# lsmod |grep kvm
kvm_intel             170181  0
kvm                   554609  1 kvm_intel
irqbypass              13503  1 kvm
# systemctl start libvirtd
# systemctl enable libvirtd
# virsh net-destroy default
```
## Bridge Config
```
# cat /etc/sysconfig/network-scripts/ifcfg-enp0s25 
TYPE=Ethernet
NAME="enp0s25"
DEVICE=enp0s25
ONBOOT=yes
BRIDGE=br0

# cat /etc/sysconfig/network-scripts/ifcfg-br0 
TYPE=Bridge
BOOTPROTO=static
IPV4_FAILURE_FATAL=no
NAME=br0
DEVICE=br0
ONBOOT=yes
BOOTPROTO=dhcp
#IPADDR=192.168.0.23
#PREFIX=24
#GATEWAY=192.168.0.1

# systemctl restart network
# brctl show
bridge name	bridge id		STP enabled	interfaces
br0		8000.28802300cc50	no		enp0s25
```

## VM Install
```
wget http://ftp.kaist.ac.kr/CentOS/7.9.2009/isos/x86_64/CentOS-7-x86_64-Minimal-2009.iso

mkdir /home/data/ && chmod a+rwx /home/data -R

virt-install \
--virt-type kvm \
--name test \
--vcpus 1 \
--memory 2048 \
--os-variant debian8 \
--cdrom ./debian-8.10.0-i386-netinst.iso \
--network bridge=br0,model=virtio \
--boot cdrom,hd \
--graphics vnc,listen=0.0.0.0,password=choi \
--disk path=/home/data/test-vm.qcow2,size=10,format=qcow2 
```

## CentOS VNC
```
wget https://mirrors.kernel.org/centos/7.4.1708/isos/x86_64/CentOS-7-x86_64-Minimal-1708.iso

virt-install \
--virt-type=kvm \
--name centos7 \
--vcpus=1 \
--ram 2048 \
--os-variant=centos7.0 \
--cdrom=./CentOS-7-x86_64-Minimal-2009.iso \
--network=bridge=br0,model=virtio \
--graphics vnc \
--boot cdrom,hd \
--disk path=/home/data/centos7.qcow2,size=10,bus=virtio,format=qcow2
```
## CentOS Console
```
# Test only 
virt-install \
--virt-type=kvm \
--name centos7 \
--vcpus=1 \
--ram 2048 \
--os-variant=centos7.0 \
--cdrom=./CentOS-7-x86_64-Minimal-2009.iso \
--network=bridge=br0,model=virtio \
--graphics none \
--console pty,target_type=serial \
--extra-args 'console=ttyS0,115200n8 serial' \
--disk path=/home/data/centos7.qcow2,size=10,bus=virtio,format=qcow2
```
## Virsh
```
virsh vncdisplay 1
virsh start centos7
virsh shutdown centos7
virsh reboot centos7
virsh console centos7
virsh undefine centos7
```

## Reference
- https://pathcre8or.tistory.com/14
- https://www.cyberciti.biz/faq/how-to-install-kvm-on-centos-7-rhel-7-headless-server/
