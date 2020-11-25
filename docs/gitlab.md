# GitLab 환경 구성

## 준비 환경
A domain to which you or your company owns, to which you can add a DNS record.
A Kubernetes cluster.
A working installation of kubectl.
A working installation of Helm v3.

## 설치 절차

- 설치
```
curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
helm repo add gitlab https://charts.gitlab.io/
helm install gitlab gitlab/gitlab \
  --set global.hosts.domain=gitlab.default.14.49.100.100.xip.io  \
  --set certmanager-issuer.email=gsikchoi@abc.com
```
- 설치 확인
```
$ kubectl get ingress -lrelease=gitlab  
NAME               HOSTS                 ADDRESS         PORTS     AGE
gitlab-minio       minio.domain.tld      35.239.27.235   80, 443   118m
gitlab-registry    registry.domain.tld   35.239.27.235   80, 443   118m
gitlab-webservice  gitlab.domain.tld     35.239.27.235   80, 443   118m
```
- 접속 비번 확인
```
$kubectl get secret gitlab-gitlab-initial-root-password -ojsonpath='{.data.password}' | base64 --decode ; echo
```

## 참고자료
- https://docs.gitlab.com/charts/quickstart/index.html