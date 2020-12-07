# GitLab 환경 구성

## 준비 환경
- A domain to which you or your company owns, to which you can add a DNS record.
  - gitlab.example.com, registry.example.com and minio.example.com to A record of IP (14.49.100.100)
- A Kubernetes cluster.
- A working installation of kubectl.
- A working installation of Helm v3.
- 기본 리소스:  8vCPU and 30gb of RAM 
  - 더 낮은 리소스에서 실행하려면 설정 변경 필요 ([minikube 2vCPU 4G RAM](https://gitlab.com/gitlab-org/charts/gitlab/-/blob/master/examples/values-minikube-minimum.yaml))

## 설치 절차

- 설치
```
$ curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash
$ helm repo add gitlab https://charts.gitlab.io/

$ cat gitlab.yaml
USER-SUPPLIED VALUES:
certmanager-issuer:
  email: gsikchoi@abc.com
global:
  hosts:
    domain: clap.abc.com
    externalIP: 13.135.178.254
gitlab:
  gitaly:
    persistence:
      storageClass: "nfs-csi"
      size: 50Gi
postgresql:
  persistence:
    storageClass: "nfs-csi"
    size: 8Gi
minio:
  persistence:
    storageClass: "nfs-csi"
    size: 10Gi
redis:
  master:
    persistence:
      storageClass: "nfs-csi"
      size: 5Gi
      
$ helm install gitlab gitlab/gitlab -f gitlab.yaml
```
- 최소 설정
```
$ cat gitlab.yaml
USER-SUPPLIED VALUES:

global:
  hosts:
    domain: clap.abc.com
    externalIP: 13.135.178.254
prometheus:
  install: false
gitlab-runner:
  install: false
gitlab:
  webservice:
    minReplicas: 1
    maxReplicas: 1
  sidekiq:
    minReplicas: 1
    maxReplicas: 1
  gitlab-shell:
    minReplicas: 1
    maxReplicas: 1
  gitaly:
    persistence:
      storageClass: "nfs-csi"
      size: 50Gi
registry:
  hpa:
    minReplicas: 1
    maxReplicas: 1
postgresql:
  persistence:
    storageClass: "nfs-csi"
    size: 8Gi
minio:
  persistence:
    storageClass: "nfs-csi"
    size: 10Gi
redis:
  master:
    persistence:
      storageClass: "nfs-csi"
      size: 5Gi
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
$ kubectl get secret gitlab-gitlab-initial-root-password -ojsonpath='{.data.password}' | base64 --decode ; echo
```

- 설정 변경
```
$ helm get values gitlab > gitlab.yaml

$ cat gitlab.yaml #변경
USER-SUPPLIED VALUES:
certmanager-issuer:
  email: gsikchoi@crossent.com
global:
  hosts:
    domain: default.14.49.100.100.xip.io
    https: false
    registry:
      servicePort: 8888
    minio:
      servicePort: 8888

$ helm upgrade gitlab gitlab/gitlab -f gitlab.yaml
```

- 삭제
```
helm uninstall gitlab
```


## 참고자료
- https://docs.gitlab.com/13.6/charts/installation/deployment.html
- https://docs.gitlab.com/charts/
- https://docs.gitlab.com/charts/charts/globals.html
- https://speakerdeck.com/shkitayama/cd-with-rancher?slide=17
