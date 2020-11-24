
## 환경 준비 (Docker)
- Docker 설치

## Kubesphere 설치
```
# sudo yum install -y socat conntrack ebtables ipset
# sudo ./kk create cluster --with-kubernetes v1.17.9 --with-kubesphere v3.0.0
# mkdir ~/.kube && cp kubekey/config ~/.kube
# kubectl logs -n kubesphere-system $(kubectl get pod -n kubesphere-system -l app=ks-install -o jsonpath='{.items[0].metadata.name}') -f
Console: http://10.1.6.44:30880
Account: admin
Password: P@88w0rd
```

## 플러그인 컴포넌트 설치
- 설정파일 다운로드
```
wget https://github.com/kubesphere/ks-installer/releases/download/v3.0.0/cluster-configuration.yaml
```
- AppStore 활성화
```
openpitrix:
    enabled: true    
```
- DevOps 활성화
```
devops:
    enabled: true
```
- Logging / Auditing / Eventing 활성화
```
logging:
    enabled: true
auditing:
    enabled: true 
events:
    enabled: true     
```
- Service mesh 활성화
```
servicemesh:
    enabled: true 
```
- Alerting & Notification 활성화
```
alerting:
    enabled: true 
notification:
    enabled: true
```
- Network Policy 활성화
```
networkpolicy:
    enabled: true 
```
- 적용 
```
kubectl apply -f cluster-configuration.yaml    
kubectl logs -n kubesphere-system $(kubectl get pod -n kubesphere-system -l app=ks-install -o jsonpath='{.items[0].metadata.name}') -f
kubectl get pod -n openpitrix-system
kubectl get pod -n kubesphere-devops-system
kubectl get pod -n kubesphere-logging-system
kubectl get pod -n istio-system
kubectl get pod -n kubesphere-alerting-system
```

## 참고자료
- https://kubesphere.io/docs/quick-start/all-in-one-on-linux/
- https://kubesphere.io/docs/pluggable-components/app-store/
