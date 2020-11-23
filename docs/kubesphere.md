
## 환경 준비 (Docker)
```
systemctl disable firewalld
systemctl stop firewalld 
systemctl status firewalld 
swapoff -a 
echo "vm.swappiness=0" >> /etc/sysctl.conf 
sed -i 's$/dev/mapper/centos-swap$#/dev/mapper/centos-swap$g' /etc/fstab
free -m 
# getenforce
Disabled

yum install -y yum-utils 
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo 
yum -y install docker-ce-18.09.9-3.el7 docker-ce-cli-18.09.9 
systemctl enable docker && systemctl start docker 
systemctl status docker 

### Set docker images accelerator 
cat > /etc/docker/daemon.json <<EOF                                                                                                
> {                                                                                                                                                             
>   "registry-mirrors": ["https://gqk8w9va.mirror.aliyuncs.com"]                                                                                                
> }                                                                                                                                                             
> EOF     

## Restart the docker service for the configuration to take effect 
systemctl restart docker

docker info|grep "Registry Mirrors" -A 1                                                                                           
Registry Mirrors:                                                                                                                                               
 https://gqk8w9va.mirror.aliyuncs.com/       
 
## Check DNS Setttings                                                                                                             
#cat /etc/resolv.conf                                                                                                               
nameserver 1.2.4.8    

## Set up kubernetes repo register. 
cat >/etc/yum.repos.d/kubernetes.repo <<EOF                                                                                        
> [kubernetes]                                                                                                                                                  
> name=Kubernetes                                                                                                                                               
> baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64                                                                                  
> enabled=1                                                                                                                                                     
> gpgcheck=0                                                                                                                                                    
> repo_gpgcheck=0                                                                                                                                               
> gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg                                                                                               
>         http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg                                                                                      
> EOF   

yum makecache fast  
```

## 설치
```
sudo yum install -y socat conntrack ebtables ipset
sudo ./kk create cluster --with-kubernetes v1.17.9 --with-kubesphere v3.0.0
kubectl logs -n kubesphere-system $(kubectl get pod -n kubesphere-system -l app=ks-install -o jsonpath='{.items[0].metadata.name}') -f
```

## 참고자료
- https://kubesphere.io/docs/quick-start/all-in-one-on-linux/
