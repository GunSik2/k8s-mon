
## 환경 준비 (Docker)
- Docker 설치

## Kubesphere 설치
```
# sudo yum install -y socat conntrack ebtables ipset
# sudo ./kk create cluster --with-kubernetes v1.17.9 --with-kubesphere v3.0.0
# mkdir ~/.kube && cp kubekey/config ~/.kube
# kubectl logs -n kubesphere-system $(kubectl get pod -n kubesphere-system -l app=ks-install -o jsonpath='{.items[0].metadata.name}') -f

#####################################################
###              Welcome to KubeSphere!           ###
#####################################################

Console: http://10.1.6.44:30880
Account: admin
Password: P@88w0rd

NOTES：
  1. After logging into the console, please check the
     monitoring status of service components in
     the "Cluster Management". If any service is not
     ready, please wait patiently until all components 
     are ready.
  2. Please modify the default password after login.

#####################################################
https://kubesphere.io             2020-11-23 04:25:27
#####################################################
```

## 참고자료
- https://kubesphere.io/docs/quick-start/all-in-one-on-linux/
