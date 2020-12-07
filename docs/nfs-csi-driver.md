# NFS CSI Driver 

## 
## Driver 설치
```
$ curl -skSL https://raw.githubusercontent.com/kubernetes-csi/csi-driver-nfs/master/deploy/install-driver.sh | bash -s master --
$ kubectl -n kube-system get pod -o wide -l app=csi-nfs-controller
NAME                                       READY   STATUS    RESTARTS   AGE     IP             NODE
csi-nfs-controller-56bfddd689-dh5tk       4/4     Running   0          35s     10.240.0.19    k8s-agentpool-22533604-0
csi-nfs-controller-56bfddd689-8pgr4       4/4     Running   0          35s     10.240.0.35    k8s-agentpool-22533604-1
$ kubectl -n kube-system get pod -o wide -l app=csi-nfs-node
NAME                                       READY   STATUS    RESTARTS   AGE     IP             NODE
csi-nfs-node-cvgbs                        3/3     Running   0          35s     10.240.0.35    k8s-agentpool-22533604-1
csi-nfs-node-dr4s4                        3/3     Running   0          35s     10.240.0.4     k8s-agentpool-22533604-0
```

## Driver 테스트
```
$ wget https://raw.githubusercontent.com/kubernetes-csi/csi-driver-nfs/master/deploy/example/nfs-provisioner/nginx-pod.yaml

$ cat nginx-pod.yaml
  csi:
    driver: nfs.csi.k8s.io
    readOnly: false
    volumeHandle: unique-volumeid  # make sure it's a unique id in the cluster
    volumeAttributes:
      server: 127.0.0.1  # NFS Server endpoint
      share: /           # NFS share path      
      
$ kubectl create -f nginx-pod.yaml
$ kubectl exec nginx-nfs-example -- bash -c "findmnt /var/www -o TARGET,SOURCE,FSTYPE"
TARGET   SOURCE                                 FSTYPE
/var/www nfs-server.default.svc.cluster.local:/ nfs4
```

## 삭제
```
curl -skSL https://raw.githubusercontent.com/kubernetes-csi/csi-driver-nfs/master/deploy/uninstall-driver.sh | bash -s master --
```

## 참고자료
- https://github.com/kubernetes-csi/csi-driver-nfs/blob/master/docs/install-csi-driver.md
- https://github.com/kubernetes-csi/csi-driver-nfs/blob/master/deploy/example/nfs-provisioner/README.md
- https://woohhan.tistory.com/18
