## Ocatant 

## 설치
```
wget https://github.com/vmware-tanzu/octant/releases/download/v0.16.2/octant_0.16.2_Linux-64bit.rpm
rpm -Uvh octant_0.16.2_Linux-64bit.rpm
kubectl cluster-info

mkdir -p /home/centos/.config/octant/plugins
octant --listener-addr 10.1.6.44:8080  --disable-open-browser

curl -v 10.1.6.44:8080
```

## 참고자료
- https://github.com/vmware-tanzu/octant/releases
- https://reference.octant.dev/?path=/docs/docs-intro--page#getting-started
