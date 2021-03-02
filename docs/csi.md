# K8S CSI 
## 목표
- 3rd 파티 CSI 볼륨 드라이버와 연동 가능한 K8S API 정의 
- K8S 마스터와 노드가 CSI 볼륨 드라이버와 통신 가능한 메커니즘 정의
- K8S 마스터와 노드가 K8S 배포한 CSI 볼륨 드라이버를 발견하고, 등록하는 매커니즘 정의
- K8S 호환 가능한 CSI 볼륨 드라이버 패키징 요건을 정의
- K8S 클러스터에 CSI 볼륨 드라이버 배포 절차 정의

## 설계 개요
- CSI 볼륨 플러그인의 SetUp / TearDown 호출은 노드의 유닉스 도메인 소켓을 통해 NodePusblishVolume과 NodeUnpulishVolume CSI RPC 함수를 호출
- Provision/delete 과 attach/detach 요청은 K8S API 를 모니터하는 컴포넌트에 의해 CSI 볼륨 드라이버를 호출하며, 해당 CSI RPC 함수를 호출 

## 설계 상세
### 3rd 파티 CSI 볼륨 드라이버
K8S 는 CSI 볼륨 드라이버 패키지와 배포를 최소화한다. 
Communication Channels 사용은 외부 CSI 호환 스토리지를 활성화하는 최소 요건이다.  

### Communication Channels
#### Kubelet 과 CSI 드라이버 통신
Kubelet (마운드/언마운트 책임)이 유닉스 소켓을 통해 동일 호스트에 있는 외부 CSI 볼륨 드라이버와 통신한다. 
CSI 볼륨 드라이버는 다음 위치에 소켓을 생성해야 한다: /var/lib/kubelet/plugins/[SanitizedCSIDriverName]/csi.sock

#### Master 와 CSI 드라이버 통신
CSI 볼륨 드라이버 코드는 신뢰하지 않기 때문에, 마스터에서 실행되지 않는다.  
Kube 컨틀롤러는 유닉스 소켓으로 통신하지 않고, K8S API 를 이용하여 CSI 볼륨 드라이버와 통신한다.
K8S 제공하는 사이드 카인 "K8S to CSI" 프록시 컨테이너가 K8S API를 감시하고 있다가 "CSI 볼륨 드라이버"를 호출한다. 

### K8S 내장 CSI 볼륨 플러그인
내장 K8S CSI 볼륨 플러그인은 K8S 가 임의 CSI 호환 볼륨 드라이버와 통신을 위해 필요한 로직을 포함한다. 
볼륨 컴포넌트는 CSI 볼륨 플러그인 조작(provisioning/deleting, attaching/detaching, mounting/unmounting)을 위한 라이프사이클을 처리한다.  

## 수행 절차
## 볼륨 생성
- 어드민이 StorageClass 를 생성, 외부 배포용 CSI 드라이버와 CSI 드라이버와 파라미터 포함
- 사용자가 StorageClass 참조한 PersistentVolumeClaim 를 생성
- PV 컨트롤러가 volume.beta.kubernetes.io/storage-provisioner 어노테이션과 PVC를 표시
- 외부 배포용 CSI 드라이버가 PersistentVolumeClaim, volume.beta.kubernetes.io/storage-provisioner 을 참고하여
  동적인 볼륨 생성을 시작: CSI 드라이버 컨테이너에  StorageClass, PersistentVolumeClaim 파라이터 포함하여 CreateVolume 함수를 호출
- 볼륨 생성 후, 외부 배포용 CSI 드라이버가 PersistentVolume 개체 만들고, PersistentVolumeClaim 과 바인딩한다


## 볼륨 삭제
- 사용자가 PersistentVolumeClaim 개체를 삭제
- 외부 배포용 CSI 드라이버가 PersistentVolumeClaim 삭제를 확인 후 보유 정책을 호출:
- 보유 정책이 "삭제"인 경우, DeleteVolume 과 PersistentVolume 개체 삭제를 호출
- 보유 정책이 "유지"인 경우, PersistentVolume 개체를 삭제하지 않음

## 볼륨 연결
- K8S 마스터의 kube-controller-manager 바이너리 내에 있는 attach/detach 컨트롤러가 CSI 볼륨 플러그인 참조 Pod 가 노드에 스케쥴된 걸 확인하고, 내부 CSI 볼륨 플러그인의 attach 함수를 호출
- 내부 볼륨 플러그인은 K8S의 VolumeAttachment 개체를 생성하고 완료되길 대기
- 외부 attacher 컴포넌트가 VolumeAttachment 개체를 보고 CSI 볼륨드라이버에 ControllerPublish 함수를 호출(grpc 호출)   
- 외부 attacher 컴포넌트가 ControllerPublish 정상 처리 후, VolumeAttachment 개체 값을 업데이트 
   

# 참고자료
- https://kubernetes-csi.github.io/docs/introduction.html
- https://github.com/kubernetes/community/blob/master/contributors/design-proposals/storage/container-storage-interface.md
