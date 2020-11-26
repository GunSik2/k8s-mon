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
# virsh net-list
# virsh net-destroy default
```
## Host Bridge Config
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
mkdir /home/data/ && cd /home/data && chmod a+rwx /home/data -R

wget http://ftp.kaist.ac.kr/CentOS/7.9.2009/isos/x86_64/CentOS-7-x86_64-DVD-2009.iso

virt-install \
--virt-type=kvm \
--name centos7-tp \
--vcpus=1 \
--ram 2048 \
--os-variant=centos7.0 \
--cdrom=./CentOS-7-x86_64-DVD-2009.iso \
--network=bridge=br0,model=virtio \
--graphics vnc \
--boot cdrom,hd \
--disk path=/home/data/centos7-tp.qcow2,size=10,bus=virtio,format=qcow2
```

## VM network config
```
# vi /etc/sysconfig/network-scripts/ifcfg-eth0
TYPE=Ethernet
NAME=eth0
DEVICE=eth0
ONBOOT=yes
IPADDR=192.168.0.101
PREFIX=24
GATEWAY=192.168.0.1
# sudo systemctl restart network
# cat /etc/resolv.conf
nameserver 8.8.8.8
```


## VM Clone
```
sudo virt-clone --original centos7-tp --name kube-master --file /home/data/kube-master.qcow2
sudo virt-clone --original centos7-tp --name kube-worker1 --file /home/data/kube-worker1.qcow2
sudo virt-clone --original centos7-tp --name kube-worker2 --file /home/data/kube-worker2.qcow2

virsh list
```

## Virsh
```
# 가상머신 list 
virsh list
# 가상머신 시작
virsh start debian-6.0.10
# 가상 머신 종료
virsh shutdown debian-6.0.10
# 가상 머신 재부팅
virsh reboot debian-6.0.10
# 가상 머신 강제 종료
virsh destroy debian-6.0.10
# 가상머신 삭제
virsh undefine --domain debian-6.0.10
# 가상 머신 저장 -- 스냅샷 가상 머신이 일시 정지후 자동 종료되버림 주의 라이브 아님
virsh save debian-6.0.10 debian-6.0.10_20200205
# 가상 머신 복원 -- 복원 후 가상 머신 실행
virsh restore debian-6.0.10_20200205
# 가상머신 아미지 파일 삭제 
rm data/vm/debian-6.0.10.img
# 가상머신 설정 변경
virsh edit debian-6.0.10
# 또는 vi /etc/libvirt/qemu/
# virsh로 접속해있기 
virsh
# Hypervisor 접속하기
virsh -c qemu:///system
# 모든 VM 리스트 확인
virsh list --all
# 설정 백업
virsh dumpxml source_vm > .../vm-definition-repository/source_vm.xml
```

## Reference
- https://pathcre8or.tistory.com/14
- https://www.cyberciti.biz/faq/how-to-install-kvm-on-centos-7-rhel-7-headless-server/
- https://teamsmiley.github.io/2020/02/06/KVM/
