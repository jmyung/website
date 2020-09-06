---
title: 쿠버네티스의 윈도우 지원 소개
content_type: concept
weight: 65
---




<!-- overview -->

윈도우 애플리케이션은 많은 조직에서 실행되는 서비스 및 애플리케이션의 상당 부분을 구성한다. [윈도우 컨테이너](https://aka.ms/windowscontainers)는 프로세스와 패키지 종속성을 캡슐화하는 현대적인 방법을 제공하여, 데브옵스(DevOps) 사례를 보다 쉽게 ​​사용하고 윈도우 애플리케이션의 클라우드 네이티브 패턴을 따르도록 한다. 쿠버네티스는 사실상의 표준 컨테이너 오케스트레이터가 되었으며, 쿠버네티스 1.14 릴리스에는 쿠버네티스 클러스터의 윈도우 노드에서 윈도우 컨테이너 스케줄링을 위한 프로덕션 지원이 포함되어 있어, 광범위한 윈도우 애플리케이션 생태계가 쿠버네티스의 강력한 기능을 활용할 수 있다. 윈도우 기반 애플리케이션과 리눅스 기반 애플리케이션에 투자한 조직은 워크로드를 관리하기 위해 별도의 오케스트레이터를 찾을 필요가 없으므로, 운영 체제에 관계없이 배포 전반에 걸쳐 운영 효율성이 향상된다.



<!-- body -->

## 쿠버네티스의 윈도우 컨테이너

쿠버네티스에서 윈도우 컨테이너 오케스트레이션을 활성화하려면 기존 리눅스 클러스터에 윈도우 노드를 포함시키기만 하면 된다. 쿠버네티스의 [파드(Pods)](/ko/docs/concepts/workloads/pods/pod-overview/)에서 윈도우 컨테이너를 스케쥴링하는 것은 리눅스 기반 컨테이너를 스케쥴링하는 것 만큼 간단하고 쉽다.

윈도우 컨테이너를 실행하려면, 쿠버네티스 클러스터에 리눅스를 실행하는 컨트롤 플레인 노드와 워크로드 요구에 따라 윈도우 또는 리눅스를 실행하는 워커가 있는 여러 운영 체제가 포함되어 있어야 한다. 윈도우 서버 2019는 [쿠버네티스 노드](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/architecture/architecture.md#the-kubernetes-node)를 활성화하는 유일한 윈도우 운영 체제이다. (kubelet, [컨테이너 런타임](https://docs.microsoft.com/en-us/virtualization/windowscontainers/deploy-containers/containerd) 및 kube-proxy 포함) 윈도우 배포 채널에 대한 자세한 설명은 [Microsoft 문서](https://docs.microsoft.com/en-us/windows-server/get-started-19/servicing-channels-19)를 참조한다.

{{< note >}}
[마스터 컴포넌트](/ko/docs/concepts/overview/components/)를 포함한 쿠버네티스 컨트롤 플레인은 리눅스에서 계속 실행된다. 윈도우 전용 쿠버네티스 클러스터를 사용할 계획은 없다.
{{< /note >}}

{{< note >}}
이 문서에서 윈도우 컨테이너에 대해 이야기 할 때 프로세스 격리된 윈도우 컨테이너를 의미한다. [Hyper-V 격리](https://docs.microsoft.com/en-us/virtualization/windowscontainers/manage-containers/hyperv-container)가 있는 윈도우 컨테이너는 향후 릴리스로 계획되어 있다.
{{< /note >}}

## 지원되는 기능 및 제한

### 지원되는 기능

#### 윈도우 OS 버전 지원

쿠버네티스의 윈도우 운영 체제 지원은 다음 표를 참조한다. 단일 이기종 쿠버네티스 클러스터에는 윈도우 및 리눅스 워커 노드가 모두 있을 수 있다. 윈도우 컨테이너는 윈도우 노드에서, 리눅스 컨테이너는 리눅스 노드에서 스케쥴 되어야 한다.

| 쿠버네티스 버전 | 윈도우 서버 LTSC 릴리스 | 윈도우 서버 SAC 릴리스 |
| --- | --- | --- | --- |
| *Kubernetes v1.14* | Windows Server 2019 | Windows Server ver 1809 |
| *Kubernetes v1.15* | Windows Server 2019 | Windows Server ver 1809 |
| *Kubernetes v1.16* | Windows Server 2019 | Windows Server ver 1809 |
| *Kubernetes v1.17* | Windows Server 2019 | Windows Server ver 1809 |
| *Kubernetes v1.18* | Windows Server 2019 | Windows Server ver 1809, Windows Server ver 1903, Windows Server ver 1909 |
| *Kubernetes v1.19* | Windows Server 2019 | Windows Server ver 1909, Windows Server ver 2004 |

{{< note >}}
지원 모델을 포함한 다양한 윈도우 서버 서비스 채널에 대한 정보는 [윈도우 서버 서비스 채널](https://docs.microsoft.com/en-us/windows-server/get-started-19/servicing-channels)에서 확인할 수 있다.
{{< /note >}}
{{< note >}}
모든 윈도우 고객이 앱의 운영 체제를 자주 업데이트하는 것은 아니다. 애플리케이션 업그레이드는 클러스터에 새 노드를 업그레이드하거나 도입하는 것을 명령하고 필요하게 만든다. 쿠버네티스에서 실행되는 컨테이너의 운영 체제를 업그레이드하기로 선택한 고객을 위해 새 운영 체제 버전에 대한 지원을 추가 할 때 지침과 단계별 지침을 제공한다. 이 지침에는 클러스터 노드와 함께 사용자 애플리케이션을 업그레이드하기 위한 권장 업그레이드 절차가 포함된다. 윈도우 노드는 현재 리눅스 노드와 동일한 방식으로 쿠버네티스 [버전-스큐(skew) 정책](/docs/setup/release/version-skew-policy/) (노드 대 컨트롤 플레인 버전 관리)을 준수한다.
{{< /note >}}
{{< note >}}
윈도우 서버 호스트 운영 체제에는 [윈도우 서버](https://www.microsoft.com/en-us/cloud-platform/windows-server-pricing) 라이선스가 적용된다. 윈도우 컨테이너 이미지에는 [윈도우 컨테이너에 대한 추가 사용 조건](https://docs.microsoft.com/en-us/virtualization/windowscontainers/images-eula)이 적용된다.
{{< /note >}}
{{< note >}}
프로세스 격리가 포함된 윈도우 컨테이너에는 엄격한 호환성 규칙이 있으며, [여기서 호스트 OS 버전은 컨테이너 베이스 이미지 OS 버전과 일치해야 한다](https://docs.microsoft.com/en-us/virtualization/windowscontainers/deploy-containers/version-compatibility). 쿠버네티스에서 Hyper-V 격리가 포함된 윈도우 컨테이너를 지원하면, 제한 및 호환성 규칙이 변경될 것이다.
{{< /note >}}

#### 컴퓨트

API 및 kubectl의 관점에서, 윈도우 컨테이너는 리눅스 기반 컨테이너와 거의 같은 방식으로 작동한다. 그러나 [제한 섹션](#limitations)에 요약된 주요 기능에는 몇 가지 눈에 띄는 차이점이 있다.

윈도우에서 주요 쿠버네티스 요소는 리눅스와 동일한 방식으로 작동한다. 이 섹션에서는, 주요 워크로드 이네이블러(enabler) 일부와 이들이 윈도우에 맵핑되는 방법에 대해 설명한다.

* [파드(Pods)](/ko/docs/concepts/workloads/pods/pod-overview/)

    파드는 쿠버네티스의 기본 빌딩 블록이다 - 쿠버네티스 오브젝트 모델에서 생성하고 배포하는 가장 작고 간단한 단위. 동일한 파드에 윈도우 및 리눅스 컨테이너를 배포할 수 없다. 파드의 모든 컨테이너는 단일 노드로 예약되며 각 노드는 특정 플랫폼 및 아키텍처를 나타낸다. 다음과 같은 파드 기능, 속성 및 이벤트가 윈도우 컨테이너에서 지원된다.

  * 프로세스 분리 및 볼륨 공유 기능을 갖춘 파드 당 하나 또는 여러 개의 컨테이너
  * 파드 상태 필드
  * 준비성(readiness) 및 활성 프로브(liveness probe)
  * postStart 및 preStop 컨테이너 라이프사이클 이벤트
  * 컨피그맵(ConfigMap), 시크릿(Secrets): 환경 변수 또는 볼륨으로
  * EmptyDir
  * 명명된 파이프 호스트 마운트
  * 리소스 제한
* [컨트롤러](/ko/docs/concepts/workloads/controllers/)

    쿠버네티스 컨트롤러는 파드의 원하는 상태(desired state)를 처리한다. 윈도우 컨테이너에서 지원되는 워크로드 컨트롤러는 다음과 같다.

  * 레플리카셋(ReplicaSet)
  * 레플리케이션컨트롤러(ReplicationController)
  * 디플로이먼트(Deployment)
  * 스테이트풀셋(StatefulSet)	
  * 데몬셋(DaemonSet)
  * 잡(Job)
  * 크론잡(CronJob)
* [서비스(Service)](/docs/concepts/services-networking/service/)

    쿠버네티스 서비스는 논리적인 파드 집합과 그것을(마이크로 서비스라고도 함) 접근하는 정책을 정의하는 추상화 개념이다. 상호-운영 체제 연결을 위해 서비스를 사용할 수 있다. 윈도우에서 서비스는 다음 유형, 속성 및 기능을 활용할 수 있다.

  * 서비스 환경 변수
  * 노드포트 (NodePort)
  * 클러스터IP (ClusterIP)
  * 로드밸런서 (LoadBalancer)
  * ExternalName
  * 헤드리스 서비스 (Headless services)

파드, 컨트롤러 및 서비스는 쿠버네티스에서 윈도우 워크로드를 관리하는데 중요한 요소이다. 그러나 그 자체로는 동적 클라우드 네이티브 환경에서 윈도우 워크로드의 적절한 수명주기 관리를 수행하기에 충분하지 않다. 다음 기능에 대한 지원이 추가되었다.

* 파드와 컨테이너 메트릭
* Horizontal Pod Autoscaler 지원
* kubectl Exec
* 리소스쿼터 (Resource Quotas)
* 스케쥴러 선점 (preemption)

#### 컨테이너 런타임

##### Docker EE

{{< feature-state for_k8s_version="v1.14" state="stable" >}}

Docker EE-basic 18.09+는 쿠버네티스를 실행하는 Windows Server 2019 / 1809 노드에 권장되는 컨테이너 런타임이다. 이것은 kubelet에 포함된 dockershim 코드와 함께 작동한다.

##### CRI-ContainerD

{{< feature-state for_k8s_version="v1.19" state="beta" >}}

{{< caution >}}
커널 패치가 필요한 윈도우 네트워크 공유에 액세스하기 위해 ContainerD와 함께 GMSA를 사용할 때 [알려진 제한](/docs/tasks/configure-pod-container/configure-gmsa/#gmsa-limitations)이 있다. [Microsoft 윈도우 컨테이너 이슈 트래커](https://github.com/microsoft/Windows-Containers/issues/44)에서 업데이트를 확인한다.
{{< /caution >}}

{{< glossary_tooltip term_id="containerd" text="ContainerD" >}} 1.4.0-beta.2+는 윈도우 쿠버네티스 노드의 컨테이너 런타임으로도 사용할 수 있다.

쿠버네티스 v1.18은 윈도우에서 ContainerD 지원을 추가한다. 윈도우에서 ContainerD의 진행 상황은 [enhancements#1001](https://github.com/kubernetes/enhancements/issues/1001)에서 확인할 수 있다.

[윈도우에 ContainerD 설치](/docs/setup/production-environment/container-runtimes/#install-containerd) 방법을 확인한다.

#### 퍼시스턴트 스토리지 (Persistent Storage)

쿠버네티스 [볼륨](/ko/docs/concepts/storage/volumes/)을 사용하면 데이터 지속성(persistence) 및 파드 볼륨 공유 요구 사항이 있는 복잡한 애플리케이션을 쿠버네티스에 배포할 수 있다. 특정 스토리지 백엔드 또는 프로토콜과 관련된 퍼시스턴트 볼륨 관리에는 다음과 같은 작업이 포함된다. 볼륨 프로비저닝/디-프로비저닝/크기 조정, 쿠버네티스 노드에 볼륨 연결/분리, 데이터를 유지해야하는 파드의 개별 컨테이너에 볼륨 마운트/분리. 특정 스토리지 백엔드 또는 프로토콜에 대해 이러한 볼륨 관리 작업을 구현하는 코드는 쿠버네티스 볼륨 [플러그인](/ko/docs/concepts/storage/volumes/#볼륨-유형들)의 형태로 제공된다. 다음과 같은 광범위한 쿠버네티스 볼륨 플러그인 클래스가 윈도우에서 지원된다.

##### 인-트리(In-tree) 볼륨 플러그인
인-트리 볼륨 플러그인과 관련된 코드는 핵심 쿠버네티스 코드 베이스의 일부로 제공된다. 인-트리 볼륨 플러그인 배포는 추가 스크립트를 설치하거나 별도의 컨테이너화된 플러그인 컴포넌트를 배포할 필요가 없다. 이러한 플러그인들은 다음을 처리할 수 있다. 볼륨 프로비저닝/디-프로비저닝, 스토리지 백엔드 볼륨 크기 조정, 쿠버네티스 노드에 볼륨 연결/분리, 파드의 개별 컨테이너에 볼륨 마운트/분리. 다음의 인-트리 플러그인은 윈도우 노드를 지원한다.

* [awsElasticBlockStore](/docs/concepts/storage/volumes/#awselasticblockstore)
* [azureDisk](/docs/concepts/storage/volumes/#azuredisk)
* [azureFile](/docs/concepts/storage/volumes/#azurefile)
* [gcePersistentDisk](/docs/concepts/storage/volumes/#gcepersistentdisk)
* [vsphereVolume](/docs/concepts/storage/volumes/#vspherevolume)

##### FlexVolume 플러그인
[FlexVolume](/ko/docs/concepts/storage/volumes/#flexVolume) 플러그인과 관련된 코드는 out-of-tree 스크립트 또는 호스트에 직접 배포해야하는 바이너리로 제공된다. FlexVolume 플러그인은 쿠버네티스 노드에 볼륨 연결/분리 및 파드의 개별 컨테이너에 볼륨 마운트/분리를 처리한다. FlexVolume 플러그인과 관련된 퍼시스턴트 볼륨의 프로비저닝/디-프로비저닝은 일반적으로 FlexVolume 플러그인과는 별도의 외부 프로비저너을 통해 처리될 수 있다. 호스트에서 powershell 스크립트로 배포된 다음의 FlexVolume [플러그인](https://github.com/Microsoft/K8s-Storage-Plugins/tree/master/flexvolume/windows)은 윈도우 노드를 지원한다.

* [SMB](https://github.com/microsoft/K8s-Storage-Plugins/tree/master/flexvolume/windows/plugins/microsoft.com~smb.cmd)
* [iSCSI](https://github.com/microsoft/K8s-Storage-Plugins/tree/master/flexvolume/windows/plugins/microsoft.com~iscsi.cmd)

##### CSI 플러그인

{{< feature-state for_k8s_version="v1.16" state="alpha" >}}

{{< glossary_tooltip text="CSI" term_id="csi" >}} 플러그인과 관련된 코드는 일반적으로 컨테이너 이미지로 배포되고 데몬셋(DaemonSets) 및 스테이트풀셋(StatefulSets)와 같은 표준 쿠버네티스 구성을 사용하여 배포되는 out-of-tree 스크립트 및 바이너리로 제공된다. CSI 플러그인은 쿠버네티스에서 다양한 볼륨 관리 작업을 처리한다. 볼륨 프로비저닝/디-프로비저닝, 볼륨 크기 조정, 쿠버네티스 노드에 볼륨 연결/분리, 파드의 개별 컨테이너에 볼륨 마운트/분리, 스냅샷 및 복제를 사용하여 퍼시스턴트 데이터 백업/복원. CSI 플러그인은 일반적으로 (각 노드에서 데몬셋으로 실행되는) 노드 플러그인과 컨트롤러 플러그인으로 구성된다.

CSI 노드 플러그인 (특히 블록 디바이스 또는 공유 파일 시스템으로 노출된 퍼시스턴트 볼륨과 관련된 플러그인)은 디스크 장치 스캔, 파일 시스템 마운트 등과 같은 다양한 권한있는(privileged) 작업을 수행해야 한다. 이러한 작업은 호스트 운영 체제마다 다르다. 리눅스 워커 노드의 경우 컨테이너화된 CSI 노드 플러그인은 일반적으로 privileged 컨테이너로 배포된다. 윈도우 워커 노드의 경우 컨테이너화된 CSI 노드 플러그인에 대한 privileged 작업은 커뮤니티에서 관리되고, 각 윈도우 노드에 사전 설치되어야 하는 stand-alone 바이너리인 [csi-proxy](https://github.com/kubernetes-csi/csi-proxy)를 사용하여 지원된다. 자세한 내용은 배포하려는 CSI 플러그인의 배포 가이드를 참조한다.

#### 네트워킹

윈도우 컨테이너용 네트워킹은 [CNI 플러그인](/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/)을 통해 노출된다. 윈도우 컨테이너는 네트워킹과 관련하여 가상 머신과 유사하게 작동한다. 각 컨테이너에는 Hyper-V 가상 스위치 (vSwitch)에 연결된 가상 네트워크 어댑터(vNIC)가 있다. 호스트 네트워킹 서비스(HNS)와 호스트 컴퓨팅 서비스(HCS)는 함께 작동하여 컨테이너를 만들고 컨테이너 vNIC를 네트워크에 연결한다. HCS는 컨테이너 관리를 담당하는 반면 HNS는 다음과 같은 네트워킹 리소스 관리를 담당한다.

* 가상 네트워크 (vSwitch 생성 포함)
* 엔드포인트 / vNIC
* 네임스페이스
* 정책 (패킷 캡슐화, 로드 밸런싱 규칙, ACL, NAT 규칙 등)

다음의 서비스 사양 유형이 지원된다.

* NodePort
* ClusterIP
* LoadBalancer
* ExternalName

##### 네트워크 모드
윈도우는 L2bridge, L2tunnel, Overlay, Transparent 및 NAT의 다섯 가지 네트워킹 드라이버/모드를 지원한다. 윈도우와 리눅스 워커 노드가 있는 이기종 클러스터에서는 윈도우와 리눅스 모두에서 호환되는 네트워킹 솔루션을 선택해야 한다. 윈도우에서 다음과 같은 out-of-tree 플러그인이 지원되며 각 CNI 사용시 권장 사항이 있다.

| 네트워크 드라이버 | 설명 | 컨테이너 패킷 수정 | 네트워크 플러그인 | 네트워크 플러그인 특성 |
| -------------- | ----------- | ------------------------------ | --------------- | ------------------------------ |
| L2bridge | 컨테이너는 외부 vSwitch에 연결된다. 컨테이너는 언더레이 네트워크에 연결된다. 하지만 인그레스/이그레스시에 재작성되기 때문에 물리적 네트워크가 컨테이너 MAC을 학습할 필요가 없다. | MAC은 호스트 MAC에 다시 쓰여지고, IP는 HNS OutboundNAT 정책을 사용하여 호스트 IP에 다시 쓰여질 수 있다. | [win-bridge](https://github.com/containernetworking/plugins/tree/master/plugins/main/windows/win-bridge), [Azure-CNI](https://github.com/Azure/azure-container-networking/blob/master/docs/cni.md), Flannel 호스트 게이트웨이는 win-bridge를 사용한다. | win-bridge는 L2bridge 네트워크 모드를 사용하고, 컨테이너를 호스트의 언더레이에 연결하여 최상의 성능을 제공한다. 노드 간 연결을 위해 사용자 정의 경로(user-defined routes, UDR)가 필요하다. |
| L2Tunnel | 이것은 l2bridge의 특별한 케이스이지만 Azure에서만 사용된다. 모든 패킷은 SDN 정책이 적용되는 가상화 호스트로 전송된다. | MAC 재작성, 언더레이 네트워크 상에서 IP 보여짐 | [Azure-CNI](https://github.com/Azure/azure-container-networking/blob/master/docs/cni.md) | Azure-CNI를 사용하면 컨테이너를 Azure vNET과 통합할 수 있으며, [Azure Virtual Network에서 제공하는](https://azure.microsoft.com/en-us/services/virtual-network/) 기능 집합을 활용할 수 있다. 예를 들어 Azure 서비스에 안전하게 연결하거나 Azure NSG를 사용한다. [azure-cni 예제](https://docs.microsoft.com/en-us/azure/aks/concepts-network#azure-cni-advanced-networking)를 참고한다. |
| 오버레이 (쿠버네티스에서 윈도우 용 오버레이 네트워킹은 *알파* 단계에 있음) | 컨테이너에는 외부 vSwitch에 연결된 vNIC가 제공된다. 각 오버레이 네트워크는 사용자 지정 IP 접두사로 정의된 자체 IP 서브넷을 가져온다. 오버레이 네트워크 드라이버는 VXLAN 캡슐화를 사용한다. | 외부 헤더로 캡슐화된다. | [Win-overlay](https://github.com/containernetworking/plugins/tree/master/plugins/main/windows/win-overlay), Flannel VXLAN (win-overlay 사용) | win-overlay는 가상 컨테이너 네트워크를 호스트의 언더레이에서 격리하려는 경우 (예: 보안상의 이유로) 사용해야 한다. 데이터 센터의 IP에 제한이있는 경우, (다른 VNID 태그가 있는) 다른 오버레이 네트워크에 IP를 재사용 할 수 있다. 이 옵션을 사용하려면 Windows Server 2019에서 [KB4489899](https://support.microsoft.com/help/4489899)가 필요하다. |
| Transparent ([ovn-kubernetes](https://github.com/openvswitch/ovn-kubernetes)의 특수한 유스케이스) | 외부 vSwitch가 필요하다. 컨테이너는 논리적 네트워크 (논리적 스위치 및 라우터)를 통해 파드 내 통신을 가능하게하는 외부 vSwitch에 연결된다. | 패킷은 [GENEVE](https://datatracker.ietf.org/doc/draft-gross-geneve/) 또는 [STT](https://datatracker.ietf.org/doc/draft-davie-stt)를 통해 캡슐화되는데, 동일한 호스트에 있지 않은 파드에 도달하기 위한 터널링을 한다. <br/> 패킷은 ovn 네트워크 컨트롤러에서 제공하는 터널 메타데이터 정보를 통해 전달되거나 삭제된다. <br/> NAT는 남북 통신을 위해 수행된다. | [ovn-kubernetes](https://github.com/openvswitch/ovn-kubernetes) | [ansible을 통해 배포](https://github.com/openvswitch/ovn-kubernetes/tree/master/contrib). 분산 ACL은 쿠버네티스 정책을 통해 적용할 수 있다. IPAM 지원. kube-proxy없이 로드 밸런싱을 수행할 수 있다. NATing은 iptables/netsh를 사용하지 않고 수행된다. |
| NAT (*쿠버네티스에서 사용되지 않음*) | 컨테이너에는 내부 vSwitch에 연결된 vNIC가 제공됩니다. DNS/DHCP는 [WinNAT](https://blogs.technet.microsoft.com/virtualization/2016/05/25/windows-nat-winnat-capabilities-and-limitations/)라는 내부 컴포넌트를 사용하여 제공된다. | MAC 및 IP는 호스트 MAC/IP에 다시 작성된다. | [nat](https://github.com/Microsoft/windows-container-networking/tree/master/plugins/nat) | 완전성을 위해 여기에 포함됨 |

위에서 설명한대로 [플라넬(Flannel)](https://github.com/coreos/flannel) CNI [메타 플러그인](https://github.com/containernetworking/plugins/tree/master/plugins/meta/flannel)은 [VXLAN 네트워크 백엔드](https://github.com/coreos/flannel/blob/master/Documentation/backends.md#vxlan)(**alpha 지원**, win-overlay에 위임) 및 [host-gateway network backend](https://github.com/coreos/flannel/blob/master/Documentation/backends.md#host-gw) (안정적인 지원, win-bridge에 위임)를 통해 [윈도우](https://github.com/containernetworking/plugins/tree/master/plugins/meta/flannel#windows-support-experimental)에서도 지원된다. 이 플러그인은 자동 노드 서브넷 임대 할당과 HNS 네트워크 생성을 위해 윈도우 (Flanneld)에서 Flannel 데몬과 함께 작동하도록 참조 CNI 플러그인 (win-overlay, win-bridge) 중 하나에 대한 위임을 지원한다. 이 플러그인은 자체 구성 파일 (cni.conf)을 읽고, 이를 FlannelD 생성하는 subnet.env 파일의 환경 변수와 함께 집계한다. 이후 네트워크 연결을 위한 참조 CNI 플러그인 중 하나에 위임하고 노드 할당 서브넷을 포함하는 올바른 구성을 IPAM 플러그인 (예: 호스트-로컬)으로 보낸다.

노드, 파드, 서비스 오브젝트의 경우 TCP/UDP 트래픽에 대해 다음 네트워크 흐름이 지원된다.

* 파드 -> 파드 (IP)
* 파드 -> 파드 (Name)
* 파드 -> 서비스 (Cluster IP)
* 파드 -> 서비스 (PQDN, 단 "."이 없는 경우에만)
* 파드 -> 서비스 (FQDN)
* 파드 -> External (IP)
* 파드 -> External (DNS)
* 노드 -> 파드
* 파드 -> 노드

##### IP 주소 관리 (IPAM) {#ipam}
윈도우에서는 다음 IPAM 옵션이 지원된다.

* [호스트-로컬](https://github.com/containernetworking/plugins/tree/master/plugins/ipam/host-local)
* HNS IPAM (Inbox 플랫폼 IPAM, 이것은 IPAM이 설정되지 않은 경우 폴백(fallback)이다) 
* [Azure-vnet-ipam](https://github.com/Azure/azure-container-networking/blob/master/docs/ipam.md) (azure-cni 전용)

##### 로드 밸런싱과 서비스

윈도우에서는 다음 설정을 사용하여 서비스 및 로드 밸런싱 동작을 구성할 수 있다.

{{< table caption="윈도우 서비스 구성" >}}
| 기능 | 설명 | 지원되는 쿠버네티스 버전 | 지원되는 윈도우 OS 빌드 | 활성화하는 방법 |
| ------- | ----------- | ----------------------------- | -------------------------- | ------------- |
| 세션 어피니티 | 특정 클라이언트의 연결이 매번 동일한 팓드로 전달되도록 한다. | v1.19 이상 | [윈도우 서버 vNext Insider 프리뷰 Build 19551](https://blogs.windows.com/windowsexperience/2020/01/28/announcing-windows-server-vnext-insider-preview-build-19551/) 이상 | `service.spec.sessionAffinity`를 "ClientIP"로 설정 |
| 직접 서버 반환 | IP 주소 수정 및 LBNAT가 컨테이너 vSwitch 포트에서 직접 발생하는 로드 밸런싱 모드. 서비스 트래픽은 소스 IP가 원래 파드 IP로 설정된 상태로 도착한다. 낮은 지연 시간과 확장성을 약속한다. | v1.15 이상 | Windows Server, 버전 2004 | kube-proxy에서 다음 플래그를 설정한다. `--feature-gates="WinDSR=true" --enable-dsr=true` |
| 대상 보존(Preserve-Destination) | 서비스 트래픽의 DNAT를 스킵하여, 백엔드 파드에 도달하는 패킷에서 대상 서비스의 가상 IP를 보존한다. 이 설정은 또한 수신 패킷의 클라이언트 IP가 보존되도록 한다. | v1.15 이상 | Windows Server, 버전 1903 (이상) | 서비스 어느테이션에서 `"preserve-destination": "true"`를 설정하고 kube-proxy에서 DSR 플래그를 활성화한다. |
| IPv4/IPv6 이중 스택 네트워킹 | 클러스터 내/외부 기본 IPv4-to-IPv4 통신과 함께 IPv6-to-IPv6 통신 | v1.19 이상 | 윈도우 서버 vNext Insider Preview Build 19603 (또는 그 이상) | See [IPv4/IPv6 이중 스택](#ipv4ipv6-dual-stack) |
{{< /table >}}

#### IPv4/IPv6 이중 스택
`IPv6DualStack` [기능 게이트](https://kubernetes.io/docs/reference/command-line-tools-reference/feature-gates/)를 사용하여 `l2bridge` 네트워크에 IPv4/IPv6 이중 스택 네트워킹을 활성화할 수 있다. 자세한 내용은 [IPv4/IPv6 이중 스택 활성화](/ko/docs/concepts/services-networking/dual-stack/#ipv4-ipv6-이중-스택-활성화)을 참조한다.

{{< note >}}
윈도우에서 쿠버네티스와 함께 IPv6를 사용하려면 윈도우 서버 vNext Insider Preview Build 19603 이상이 필요하다.
{{< /note >}}

{{< note >}}
윈도우의 오버레이 (VXLAN) 네트워크는 현재 이중 스택 네트워킹을 지원하지 않는다.
{{< /note >}}

### 제한

#### 컨트롤 플레인

윈도우는 쿠버네티스 아키텍처 및 컴포넌트 매트릭스에서 워커 노드로만 지원된다. 즉, 쿠버네티스 클러스터에는 항상 리눅스 마스터 노드, 0개 이상의 리눅스 워커 노드 및 0개 이상의 윈도우 워커 노드가 포함되어야 한다.

#### 컴퓨트

##### 리소스 관리 및 프로세스 격리

 리눅스 cgroup은 리눅스에서 리소스 제어를 위한 파드 경계로 사용된다. 컨테이너는 네트워크, 프로세스 및 파일 시스템 격리를 위해 해당 경계 내에 생성된다. cgroups API는 cpu/io/memory 통계를 수집하는 데 사용할 수 있다. 반대로 윈도우는 시스템 네임스페이스 필터가 있는 컨테이너별로 잡(Job) 오브젝트를 사용하여 컨테이너의 모든 프로세스를 포함하고 호스트와의 논리적 격리를 제공한다. 네임스페이스 필터링 없이 윈도우 컨테이너를 실행할 수 있는 방법은 없다. 즉, 시스템 권한은 호스트 컨텍스트에서 삽입 될(assert) 수 없으므로 권한이 있는(privileged) 컨테이너는 윈도우에서 사용할 수 없다. 보안 계정 매니져(Security Account Manager, SAM)가 분리되어 있으므로 컨테이너는 호스트의 ID를 가정할 수 없다.

##### 운영 체제 제한

윈도우에는 호스트 OS 버전이 컨테이너 베이스 이미지 OS 버전과 일치해야 하는 엄격한 호환성 규칙이 있다. Windows Server 2019의 컨테이너 운영 체제가 있는 윈도우 컨테이너 만 지원된다. 윈도우 컨테이너 이미지 버전의 일부 이전 버전과의 호환성을 가능하게 하는 컨테이너의 Hyper-V 격리는 향후 릴리스에서 계획된다.

##### 기능 제한

* TerminationGracePeriod : 구현되지 않음
* 단일 파일 매핑 : CRI-ContainerD로 구현 예정
* 종료 메시지 : CRI-ContainerD로 구현 예정
* 권한있(Privileged) 컨테이너 : 현재 윈도우 컨테이너에서 지원되지 않음
* HugePages : 현재 윈도우 컨테이너에서 지원되지 않음
* 기존 노드 문제 감지기는 리눅스 전용이며 권한있는(privileged) 컨테이너가 필요하다. 일반적으로 권한있는 컨테이너가 지원되지 않기 때문에 윈도우에서 이 기능이 사용될 것으로 예상하지 않는다.
* 공유 네임스페이스의 모든 기능이 지원되는 것은 아니다. (자세한 내용은 API 섹션 참조).

##### 메모리 예약 및 처리

윈도우에는 리눅스처럼 out-of-memory 프로세스 킬러가 없다. 윈도우는 항상 모든 사용자 모드 메모리 할당을 가상으로 처리하며 페이지 파일은 필수이다. 결과적으로 윈도우는 리눅스와 같은 방식으로 메모리 부족 상태에 도달하지 않고 메모리 부족 (OOM) 종료 대신 페이지를 디스크로 처리한다. 메모리가 과도하게 프로비저닝되고 모든 실제 메모리가 고갈되면 페이징으로 인해 성능이 저하될 수 있다.

2단계 프로세스를 통해 적절한 범위 내에서 메모리 사용량을 유지할 수 있다. 먼저, kubelet 파라미터 `--kubelet-reserve` 그리고/또는 `--system-reserve`를 사용하여 노드(컨테이너 외부)의 메모리 사용량을 고려한다. 이렇게 하면 [노드 할당(NodeAllocatable)](/docs/tasks/administer-cluster/reserve-compute-resources/#node-allocatable)이 줄어든다. 워크로드를 배포할 때 컨테이너에 리소스 제한을 사용한다. (limits만 설정하거나 limits이 requests과 같아야 함). 또한 NodeAllocatable에서 빼고 노드가 가득 차면 스케줄러가 더 많은 파드를 추가하지 못하도록 한다. 

오버 프로비저닝을 방지하는 모범 사례는 윈도우, Docker 및 Kubernetes 프로세스를 고려하여 최소 2GB의 시스템 예약 메모리로 kubelet을 구성하는 것이다. 

플래그의 동작은 아래에 설명된대로 다르게 동작한다.

* `--kubelet-reserve`, `--system-reserve`, `--eviction-hard` 플래그는 Node Allocatable 업데이트
* `--enforce-node-allocable`을 사용한 축출(Eviction)은 구현되지 않았다.
* `--eviction-hard`와 `--eviction-soft`를 사용한 축출(Eviction)은 구현되지 않았다.
* MemoryPressure 조건은 구현되지 않았다.
* kubelet이 취한 OOM 축출 조치가 없다.
* 윈도우 노드에서 실행되는 Kubelet에는 메모리 제한이 없다. `--kubelet-reserve`와 `--system-reserve`는 호스트에서 실행되는 kubelet 또는 프로세스에 제한을 설정하지 않는다. 이는 호스트의 kubelet 또는 프로세스가 node-allocatable 및 스케줄러 외부에서 메모리 리소스 부족을 유발할 수 있음을 의미한다.

#### 스토리지

윈도우에는 컨테이너 계층을 마운트하고 NTFS를 기반으로하는 복사 파일시스템을 만드는 레이어드(layered) 파일시스템 드라이버가 있다. 컨테이너의 모든 파일 경로는 해당 컨테이너의 컨텍스트 내에서만 확인됩니다.

* 볼륨 마운트는 개별 파일이 아닌 컨테이너의 디렉토리만 대상으로 할 수 있다.
* 볼륨 마운트는 파일이나 디렉토리를 호스트 파일 시스템으로 다시 투영할 수 없다.
* 읽기 전용 파일시스템은 윈도우 레지스트리 및 SAM 데이터베이스에 항상 쓰기 액세스가 필요하기 때문에 지원되지 않는다. 그러나 읽기 전용 볼륨은 지원된다.
* 볼륨 사용자 마스크 및 권한은 사용할 수 없습니다. SAM은 호스트와 컨테이너간에 공유되지 않기 때문에 이들간에 매핑이 없다. 모든 권한은 컨테이너 컨텍스트 내에서 해결된다.

결과적으로, 다음 스토리지 기능은 윈도우 노드에서 지원되지 않습니다.

* 볼륨 하위 경로(subpath) 마운트. 전체 볼륨 만 윈도우 컨테이너에 마운트할 수 있다.
* 시크릿에 대한 하위 경로 볼륨 마운트
* 호스트 마운트 프로젝션
* DefaultMode (UID/GID 종속성에 기인함)
* 읽기 전용 루트 파일 시스템. 매핑된 볼륨은 여전히 ​​읽기 전용을 지원한다.
* 블록 디바이스 매핑
* 저장 매체로서의 메모리
* uui/guid, 사용자 별 리눅스 파일시스템 권한과 같은 파일 시스템 기능
* NFS 기반 스토리지/볼륨 지원
* 마운트된 볼륨 확장 (resizefs)

#### 네트워킹

Windows Container Networking differs in some important ways from Linux networking. The [Microsoft documentation for Windows Container Networking](https://docs.microsoft.com/en-us/virtualization/windowscontainers/container-networking/architecture) contains additional details and background.

The Windows host networking service and virtual switch implement namespacing and can create virtual NICs as needed for a pod or container. However, many configurations such as DNS, routes, and metrics are stored in the Windows registry database rather than /etc/... files as they are on Linux. The Windows registry for the container is separate from that of the host, so concepts like mapping /etc/resolv.conf from the host into a container don't have the same effect they would on Linux. These must be configured using Windows APIs run in the context of that container. Therefore CNI implementations need to call the HNS instead of relying on file mappings to pass network details into the pod or container.

The following networking functionality is not supported on Windows nodes

* Host networking mode is not available for Windows pods
* Local NodePort access from the node itself fails (works for other nodes or external clients)
* Accessing service VIPs from nodes will be available with a future release of Windows Server
* Overlay networking support in kube-proxy is an alpha release. In addition, it requires [KB4482887](https://support.microsoft.com/en-us/help/4482887/windows-10-update-kb4482887) to be installed on Windows Server 2019
* Local Traffic Policy and DSR mode
* Windows containers connected to l2bridge, l2tunnel, or overlay networks do not support communicating over the IPv6 stack. There is outstanding Windows platform work required to enable these network drivers to consume IPv6 addresses and subsequent Kubernetes work in kubelet, kube-proxy, and CNI plugins.
* Outbound communication using the ICMP protocol via the win-overlay, win-bridge, and Azure-CNI plugin. Specifically, the Windows data plane ([VFP](https://www.microsoft.com/en-us/research/project/azure-virtual-filtering-platform/)) doesn't support ICMP packet transpositions. This means:
  * ICMP packets directed to destinations within the same network (e.g. pod to pod communication via ping) work as expected and without any limitations
  * TCP/UDP packets work as expected and without any limitations
  * ICMP packets directed to pass through a remote network (e.g. pod to external internet communication via ping) cannot be transposed and thus will not be routed back to their source
  * Since TCP/UDP packets can still be transposed, one can substitute `ping <destination>` with `curl <destination>` to be able to debug connectivity to the outside world.

These features were added in Kubernetes v1.15:

* `kubectl port-forward`

##### CNI Plugins

* Windows reference network plugins win-bridge and win-overlay do not currently implement [CNI spec](https://github.com/containernetworking/cni/blob/master/SPEC.md) v0.4.0 due to missing "CHECK" implementation.
* The Flannel VXLAN CNI has the following limitations on Windows:

1. Node-pod connectivity isn't possible by design. It's only possible for local pods with Flannel v0.12.0 (or higher).
2. We are restricted to using VNI 4096 and UDP port 4789. The VNI limitation is being worked on and will be overcome in a future release (open-source flannel changes). See the official [Flannel VXLAN](https://github.com/coreos/flannel/blob/master/Documentation/backends.md#vxlan) backend docs for more details on these parameters.

##### DNS {#dns-limitations}

* ClusterFirstWithHostNet is not supported for DNS. Windows treats all names with a '.' as a FQDN and skips PQDN resolution
* On Linux, you have a DNS suffix list, which is used when trying to resolve PQDNs. On Windows, we only have 1 DNS suffix, which is the DNS suffix associated with that pod's namespace (mydns.svc.cluster.local for example). Windows can resolve FQDNs and services or names resolvable with just that suffix. For example, a pod spawned in the default namespace, will have the DNS suffix **default.svc.cluster.local**. On a Windows pod, you can resolve both **kubernetes.default.svc.cluster.local** and **kubernetes**, but not the in-betweens, like **kubernetes.default** or **kubernetes.default.svc**.
* On Windows, there are multiple DNS resolvers that can be used. As these come with slightly different behaviors, using the `Resolve-DNSName` utility for name query resolutions is recommended.

##### IPv6
Kubernetes on Windows does not support single-stack "IPv6-only" networking. However,dual-stack IPv4/IPv6 networking for pods and nodes with single-family services is supported. See [IPv4/IPv6 dual-stack networking](#ipv4ipv6-dual-stack) for more details.


##### Session affinity
Setting the maximum session sticky time for Windows services using `service.spec.sessionAffinityConfig.clientIP.timeoutSeconds` is not supported.

##### Security

Secrets are written in clear text on the node's volume (as compared to tmpfs/in-memory on linux). This means customers have to do two things

1. Use file ACLs to secure the secrets file location
2. Use volume-level encryption using [BitLocker](https://docs.microsoft.com/en-us/windows/security/information-protection/bitlocker/bitlocker-how-to-deploy-on-windows-server)

[RunAsUser ](/docs/concepts/policy/pod-security-policy/#users-and-groups)is not currently supported on Windows. The workaround is to create local accounts before packaging the container. The RunAsUsername capability may be added in a future release.

Linux specific pod security context privileges such as SELinux, AppArmor, Seccomp, Capabilities (POSIX Capabilities), and others are not supported.

In addition, as mentioned already, privileged containers are not supported on Windows.

#### API

There are no differences in how most of the Kubernetes APIs work for Windows. The subtleties around what's different come down to differences in the OS and container runtime. In certain situations, some properties on workload APIs such as Pod or Container were designed with an assumption that they are implemented on Linux, failing to run on Windows.

At a high level, these OS concepts are different:

* Identity - Linux uses userID (UID) and groupID (GID) which are represented as integer types. User and group names are not canonical - they are just an alias in `/etc/groups` or `/etc/passwd` back to UID+GID. Windows uses a larger binary security identifier (SID) which is stored in the Windows Security Access Manager (SAM) database. This database is not shared between the host and containers, or between containers.
* File permissions - Windows uses an access control list based on SIDs, rather than a bitmask of permissions and UID+GID
* File paths - convention on Windows is to use `\` instead of `/`. The Go IO libraries typically accept both and just make it work, but when you're setting a path or command line that's interpreted inside a container, `\` may be needed.
* Signals - Windows interactive apps handle termination differently, and can implement one or more of these:
  * A UI thread handles well-defined messages including WM_CLOSE
  * Console apps handle ctrl-c or ctrl-break using a Control Handler
  * Services register a Service Control Handler function that can accept SERVICE_CONTROL_STOP control codes

Exit Codes follow the same convention where 0 is success, nonzero is failure. The specific error codes may differ across Windows and Linux. However, exit codes passed from the Kubernetes components (kubelet, kube-proxy) are unchanged.

##### V1.Container

* V1.Container.ResourceRequirements.limits.cpu and V1.Container.ResourceRequirements.limits.memory - Windows doesn't use hard limits for CPU allocations. Instead, a share system is used. The existing fields based on millicores are scaled into relative shares that are followed by the Windows scheduler. [see: kuberuntime/helpers_windows.go](https://github.com/kubernetes/kubernetes/blob/master/pkg/kubelet/kuberuntime/helpers_windows.go), [see: resource controls in Microsoft docs](https://docs.microsoft.com/en-us/virtualization/windowscontainers/manage-containers/resource-controls)
  * Huge pages are not implemented in the Windows container runtime, and are not available. They require [asserting a user privilege](https://docs.microsoft.com/en-us/windows/desktop/Memory/large-page-support) that's not configurable for containers.
* V1.Container.ResourceRequirements.requests.cpu and V1.Container.ResourceRequirements.requests.memory - Requests are subtracted from node available resources, so they can be used to avoid overprovisioning a node. However, they cannot be used to guarantee resources in an overprovisioned node. They should be applied to all containers as a best practice if the operator wants to avoid overprovisioning entirely.
* V1.Container.SecurityContext.allowPrivilegeEscalation - not possible on Windows, none of the capabilities are hooked up
* V1.Container.SecurityContext.Capabilities - POSIX capabilities are not implemented on Windows
* V1.Container.SecurityContext.privileged - Windows doesn't support privileged containers
* V1.Container.SecurityContext.procMount - Windows doesn't have a /proc filesystem
* V1.Container.SecurityContext.readOnlyRootFilesystem - not possible on Windows, write access is required for registry & system processes to run inside the container
* V1.Container.SecurityContext.runAsGroup - not possible on Windows, no GID support
* V1.Container.SecurityContext.runAsNonRoot - Windows does not have a root user. The closest equivalent is ContainerAdministrator which is an identity that doesn't exist on the node.
* V1.Container.SecurityContext.runAsUser - not possible on Windows, no UID support as int.
* V1.Container.SecurityContext.seLinuxOptions - not possible on Windows, no SELinux
* V1.Container.terminationMessagePath - this has some limitations in that Windows doesn't support mapping single files. The default value is /dev/termination-log, which does work because it does not exist on Windows by default.

##### V1.Pod

* V1.Pod.hostIPC, v1.pod.hostpid - host namespace sharing is not possible on Windows
* V1.Pod.hostNetwork - There is no Windows OS support to share the host network
* V1.Pod.dnsPolicy - ClusterFirstWithHostNet - is not supported because Host Networking is not supported on Windows.
* V1.Pod.podSecurityContext - see V1.PodSecurityContext below
* V1.Pod.shareProcessNamespace - this is a beta feature, and depends on Linux namespaces which are not implemented on Windows. Windows cannot share process namespaces or the container's root filesystem. Only the network can be shared.
* V1.Pod.terminationGracePeriodSeconds - this is not fully implemented in Docker on Windows, see: [reference](https://github.com/moby/moby/issues/25982). The behavior today is that the ENTRYPOINT process is sent CTRL_SHUTDOWN_EVENT, then Windows waits 5 seconds by default, and finally shuts down all processes using the normal Windows shutdown behavior. The 5 second default is actually in the Windows registry [inside the container](https://github.com/moby/moby/issues/25982#issuecomment-426441183), so it can be overridden when the container is built.
* V1.Pod.volumeDevices - this is a beta feature, and is not implemented on Windows. Windows cannot attach raw block devices to pods.
* V1.Pod.volumes - EmptyDir, Secret, ConfigMap, HostPath - all work and have tests in TestGrid
  * V1.emptyDirVolumeSource - the Node default medium is disk on Windows. Memory is not supported, as Windows does not have a built-in RAM disk.
* V1.VolumeMount.mountPropagation - mount propagation is not supported on Windows.

##### V1.PodSecurityContext

None of the PodSecurityContext fields work on Windows. They're listed here for reference.

* V1.PodSecurityContext.SELinuxOptions - SELinux is not available on Windows
* V1.PodSecurityContext.RunAsUser - provides a UID, not available on Windows
* V1.PodSecurityContext.RunAsGroup - provides a GID, not available on Windows
* V1.PodSecurityContext.RunAsNonRoot - Windows does not have a root user. The closest equivalent is ContainerAdministrator which is an identity that doesn't exist on the node.
* V1.PodSecurityContext.SupplementalGroups - provides GID, not available on Windows
* V1.PodSecurityContext.Sysctls - these are part of the Linux sysctl interface. There's no equivalent on Windows.

## Getting Help and Troubleshooting {#troubleshooting}

Your main source of help for troubleshooting your Kubernetes cluster should start with this [section](/docs/tasks/debug-application-cluster/troubleshooting/). Some additional, Windows-specific troubleshooting help is included in this section. Logs are an important element of troubleshooting issues in Kubernetes. Make sure to include them any time you seek troubleshooting assistance from other contributors. Follow the instructions in the SIG-Windows [contributing guide on gathering logs](https://github.com/kubernetes/community/blob/master/sig-windows/CONTRIBUTING.md#gathering-logs).

1. How do I know start.ps1 completed successfully?

    You should see kubelet, kube-proxy, and (if you chose Flannel as your networking solution) flanneld host-agent processes running on your node, with running logs being displayed in separate PowerShell windows. In addition to this, your Windows node should be listed as "Ready" in your Kubernetes cluster.

1. Can I configure the Kubernetes node processes to run in the background as services?

    Kubelet and kube-proxy are already configured to run as native Windows Services, offering resiliency by re-starting the services automatically in the event of failure (for example a process crash). You have two options for configuring these node components as services.

    1. As native Windows Services

        Kubelet & kube-proxy can be run as native Windows Services using `sc.exe`.

        ```powershell
        # Create the services for kubelet and kube-proxy in two separate commands
        sc.exe create <component_name> binPath= "<path_to_binary> --service <other_args>"

        # Please note that if the arguments contain spaces, they must be escaped.
        sc.exe create kubelet binPath= "C:\kubelet.exe --service --hostname-override 'minion' <other_args>"

        # Start the services
        Start-Service kubelet
        Start-Service kube-proxy

        # Stop the service
        Stop-Service kubelet (-Force)
        Stop-Service kube-proxy (-Force)

        # Query the service status
        Get-Service kubelet
        Get-Service kube-proxy
        ```

    1. Using nssm.exe

        You can also always use alternative service managers like [nssm.exe](https://nssm.cc/) to run these processes (flanneld, kubelet & kube-proxy) in the background for you. You can use this [sample script](https://github.com/Microsoft/SDN/tree/master/Kubernetes/flannel/register-svc.ps1), leveraging nssm.exe to register kubelet, kube-proxy, and flanneld.exe to run as Windows services in the background.

        ```powershell
        register-svc.ps1 -NetworkMode <Network mode> -ManagementIP <Windows Node IP> -ClusterCIDR <Cluster subnet> -KubeDnsServiceIP <Kube-dns Service IP> -LogDir <Directory to place logs>

        # NetworkMode      = The network mode l2bridge (flannel host-gw, also the default value) or overlay (flannel vxlan) chosen as a network solution
        # ManagementIP     = The IP address assigned to the Windows node. You can use ipconfig to find this
        # ClusterCIDR      = The cluster subnet range. (Default value 10.244.0.0/16)
        # KubeDnsServiceIP = The Kubernetes DNS service IP (Default value 10.96.0.10)
        # LogDir           = The directory where kubelet and kube-proxy logs are redirected into their respective output files (Default value C:\k)
        ```

        If the above referenced script is not suitable, you can manually configure nssm.exe using the following examples.
        ```powershell
        # Register flanneld.exe
        nssm install flanneld C:\flannel\flanneld.exe
        nssm set flanneld AppParameters --kubeconfig-file=c:\k\config --iface=<ManagementIP> --ip-masq=1 --kube-subnet-mgr=1
        nssm set flanneld AppEnvironmentExtra NODE_NAME=<hostname>
        nssm set flanneld AppDirectory C:\flannel
        nssm start flanneld

        # Register kubelet.exe
        # Microsoft releases the pause infrastructure container at mcr.microsoft.com/k8s/core/pause:1.2.0
        nssm install kubelet C:\k\kubelet.exe
        nssm set kubelet AppParameters --hostname-override=<hostname> --v=6 --pod-infra-container-image=mcr.microsoft.com/k8s/core/pause:1.2.0 --resolv-conf="" --allow-privileged=true --enable-debugging-handlers --cluster-dns=<DNS-service-IP> --cluster-domain=cluster.local --kubeconfig=c:\k\config --hairpin-mode=promiscuous-bridge --image-pull-progress-deadline=20m --cgroups-per-qos=false  --log-dir=<log directory> --logtostderr=false --enforce-node-allocatable="" --network-plugin=cni --cni-bin-dir=c:\k\cni --cni-conf-dir=c:\k\cni\config
        nssm set kubelet AppDirectory C:\k
        nssm start kubelet

        # Register kube-proxy.exe (l2bridge / host-gw)
        nssm install kube-proxy C:\k\kube-proxy.exe
        nssm set kube-proxy AppDirectory c:\k
        nssm set kube-proxy AppParameters --v=4 --proxy-mode=kernelspace --hostname-override=<hostname>--kubeconfig=c:\k\config --enable-dsr=false --log-dir=<log directory> --logtostderr=false
        nssm.exe set kube-proxy AppEnvironmentExtra KUBE_NETWORK=cbr0
        nssm set kube-proxy DependOnService kubelet
        nssm start kube-proxy

        # Register kube-proxy.exe (overlay / vxlan)
        nssm install kube-proxy C:\k\kube-proxy.exe
        nssm set kube-proxy AppDirectory c:\k
        nssm set kube-proxy AppParameters --v=4 --proxy-mode=kernelspace --feature-gates="WinOverlay=true" --hostname-override=<hostname> --kubeconfig=c:\k\config --network-name=vxlan0 --source-vip=<source-vip> --enable-dsr=false --log-dir=<log directory> --logtostderr=false
        nssm set kube-proxy DependOnService kubelet
        nssm start kube-proxy
        ```


        For initial troubleshooting, you can use the following flags in [nssm.exe](https://nssm.cc/) to redirect stdout and stderr to a output file:

        ```powershell
        nssm set <Service Name> AppStdout C:\k\mysvc.log
        nssm set <Service Name> AppStderr C:\k\mysvc.log
        ```

        For additional details, see official [nssm usage](https://nssm.cc/usage) docs.

1. My Windows Pods do not have network connectivity

    If you are using virtual machines, ensure that MAC spoofing is enabled on all the VM network adapter(s).

1. My Windows Pods cannot ping external resources

    Windows Pods do not have outbound rules programmed for the ICMP protocol today. However, TCP/UDP is supported. When trying to demonstrate connectivity to resources outside of the cluster, please substitute `ping <IP>` with corresponding `curl <IP>` commands.

    If you are still facing problems, most likely your network configuration in [cni.conf](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/cni/config/cni.conf) deserves some extra attention. You can always edit this static file. The configuration update will apply to any newly created Kubernetes resources.

    One of the Kubernetes networking requirements (see [Kubernetes model](/docs/concepts/cluster-administration/networking/)) is for cluster communication to occur without NAT internally. To honor this requirement, there is an [ExceptionList](https://github.com/Microsoft/SDN/blob/master/Kubernetes/flannel/l2bridge/cni/config/cni.conf#L20) for all the communication where we do not want outbound NAT to occur. However, this also means that you need to exclude the external IP you are trying to query from the ExceptionList. Only then will the traffic originating from your Windows pods be SNAT'ed correctly to receive a response from the outside world. In this regard, your ExceptionList in `cni.conf` should look as follows:

    ```conf
    "ExceptionList": [
                    "10.244.0.0/16",  # Cluster subnet
                    "10.96.0.0/12",   # Service subnet
                    "10.127.130.0/24" # Management (host) subnet
                ]
    ```

1. My Windows node cannot access NodePort service

    Local NodePort access from the node itself fails. This is a known limitation. NodePort access works from other nodes or external clients.

1. vNICs and HNS endpoints of containers are being deleted

    This issue can be caused when the `hostname-override` parameter is not passed to [kube-proxy](/docs/reference/command-line-tools-reference/kube-proxy/). To resolve it, users need to pass the hostname to kube-proxy as follows:

    ```powershell
    C:\k\kube-proxy.exe --hostname-override=$(hostname)
    ```

1. With flannel my nodes are having issues after rejoining a cluster

    Whenever a previously deleted node is being re-joined to the cluster, flannelD tries to assign a new pod subnet to the node. Users should remove the old pod subnet configuration files in the following paths:

    ```powershell
    Remove-Item C:\k\SourceVip.json
    Remove-Item C:\k\SourceVipRequest.json
    ```

1. After launching `start.ps1`, flanneld is stuck in "Waiting for the Network to be created"

    There are numerous reports of this [issue which are being investigated](https://github.com/coreos/flannel/issues/1066); most likely it is a timing issue for when the management IP of the flannel network is set. A workaround is to simply relaunch start.ps1 or relaunch it manually as follows:

    ```powershell
    PS C:> [Environment]::SetEnvironmentVariable("NODE_NAME", "<Windows_Worker_Hostname>")
    PS C:> C:\flannel\flanneld.exe --kubeconfig-file=c:\k\config --iface=<Windows_Worker_Node_IP> --ip-masq=1 --kube-subnet-mgr=1
    ```

1. My Windows Pods cannot launch because of missing `/run/flannel/subnet.env`

    This indicates that Flannel didn't launch correctly. You can either try to restart flanneld.exe or you can copy the files over manually from `/run/flannel/subnet.env` on the Kubernetes master to` C:\run\flannel\subnet.env` on the Windows worker node and modify the `FLANNEL_SUBNET` row to a different number. For example, if node subnet 10.244.4.1/24 is desired:

    ```env
    FLANNEL_NETWORK=10.244.0.0/16
    FLANNEL_SUBNET=10.244.4.1/24
    FLANNEL_MTU=1500
    FLANNEL_IPMASQ=true
    ```

1. My Windows node cannot access my services using the service IP

    This is a known limitation of the current networking stack on Windows. Windows Pods are able to access the service IP however.

1. No network adapter is found when starting kubelet

    The Windows networking stack needs a virtual adapter for Kubernetes networking to work. If the following commands return no results (in an admin shell), virtual network creation — a necessary prerequisite for Kubelet to work — has failed:

    ```powershell
    Get-HnsNetwork | ? Name -ieq "cbr0"
    Get-NetAdapter | ? Name -Like "vEthernet (Ethernet*"
    ```

    Often it is worthwhile to modify the [InterfaceName](https://github.com/microsoft/SDN/blob/master/Kubernetes/flannel/start.ps1#L6) parameter of the start.ps1 script, in cases where the host's network adapter isn't "Ethernet". Otherwise, consult the output of the `start-kubelet.ps1` script to see if there are errors during virtual network creation.

1. My Pods are stuck at "Container Creating" or restarting over and over

    Check that your pause image is compatible with your OS version. The [instructions](https://docs.microsoft.com/en-us/virtualization/windowscontainers/kubernetes/deploying-resources) assume that both the OS and the containers are version 1803. If you have a later version of Windows, such as an Insider build, you need to adjust the images accordingly. Please refer to the Microsoft's [Docker repository](https://hub.docker.com/u/microsoft/) for images. Regardless, both the pause image Dockerfile and the sample service expect the image to be tagged as :latest.

    Starting with Kubernetes v1.14, Microsoft releases the pause infrastructure container at `mcr.microsoft.com/k8s/core/pause:1.2.0`.

1. DNS resolution is not properly working

    Check the DNS limitations for Windows in this [section](#dns-limitations).

1. `kubectl port-forward` fails with "unable to do port forwarding: wincat not found"

    This was implemented in Kubernetes 1.15, and the pause infrastructure container `mcr.microsoft.com/k8s/core/pause:1.2.0`. Be sure to use these versions or newer ones.
    If you would like to build your own pause infrastructure container, be sure to include [wincat](https://github.com/kubernetes-sigs/sig-windows-tools/tree/master/cmd/wincat)

1. My Kubernetes installation is failing because my Windows Server node is behind a proxy

    If you are behind a proxy, the following PowerShell environment variables must be defined:

    ```PowerShell
    [Environment]::SetEnvironmentVariable("HTTP_PROXY", "http://proxy.example.com:80/", [EnvironmentVariableTarget]::Machine)
    [Environment]::SetEnvironmentVariable("HTTPS_PROXY", "http://proxy.example.com:443/", [EnvironmentVariableTarget]::Machine)
    ```

1. What is a `pause` container?

    In a Kubernetes Pod, an infrastructure or "pause" container is first created to host the container endpoint. Containers that belong to the same pod, including infrastructure and worker containers, share a common network namespace and endpoint (same IP and port space). Pause containers are needed to accommodate worker containers crashing or restarting without losing any of the networking configuration.

    The "pause" (infrastructure) image is hosted on Microsoft Container Registry (MCR). You can access it using `docker pull mcr.microsoft.com/k8s/core/pause:1.2.0`. For more details, see the [DOCKERFILE](https://github.com/kubernetes-sigs/sig-windows-tools/tree/master/cmd/wincat).

### Further investigation

If these steps don't resolve your problem, you can get help running Windows containers on Windows nodes in Kubernetes through:

* StackOverflow [Windows Server Container](https://stackoverflow.com/questions/tagged/windows-server-container) topic
* Kubernetes Official Forum [discuss.kubernetes.io](https://discuss.kubernetes.io/)
* Kubernetes Slack [#SIG-Windows Channel](https://kubernetes.slack.com/messages/sig-windows)

## Reporting Issues and Feature Requests

If you have what looks like a bug, or you would like to make a feature request, please use the [GitHub issue tracking system](https://github.com/kubernetes/kubernetes/issues). You can open issues on [GitHub](https://github.com/kubernetes/kubernetes/issues/new/choose) and assign them to SIG-Windows. You should first search the list of issues in case it was reported previously and comment with your experience on the issue and add additional logs. SIG-Windows Slack is also a great avenue to get some initial support and troubleshooting ideas prior to creating a ticket.

If filing a bug, please include detailed information about how to reproduce the problem, such as:

* Kubernetes version: kubectl version
* Environment details: Cloud provider, OS distro, networking choice and configuration, and Docker version
* Detailed steps to reproduce the problem
* [Relevant logs](https://github.com/kubernetes/community/blob/master/sig-windows/CONTRIBUTING.md#gathering-logs)
* Tag the issue sig/windows by commenting on the issue with `/sig windows` to bring it to a SIG-Windows member's attention



## {{% heading "whatsnext" %}}


We have a lot of features in our roadmap. An abbreviated high level list is included below, but we encourage you to view our [roadmap project](https://github.com/orgs/kubernetes/projects/8) and help us make Windows support better by [contributing](https://github.com/kubernetes/community/blob/master/sig-windows/).

### Hyper-V isolation

Hyper-V isolation is requried to enable the following use cases for Windows containers in Kubernetes:

* Hypervisor-based isolation between pods for additional security
* Backwards compatibility allowing a node to run a newer Windows Server version without requiring containers to be rebuilt
* Specific CPU/NUMA settings for a pod
* Memory isolation and reservations

The existing Hyper-V isolation support, an experimental feature as of v1.10, will be deprecated in the future in favor of the CRI-ContainerD and RuntimeClass features mentioned above. To use the current features and create a Hyper-V isolated container, the kubelet should be started with feature gates `HyperVContainer=true` and the Pod should include the annotation `experimental.windows.kubernetes.io/isolation-type=hyperv`. In the experiemental release, this feature is limited to 1 container per Pod.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: iis
spec:
  selector:
    matchLabels:
      app: iis
  replicas: 3
  template:
    metadata:
      labels:
        app: iis
      annotations:
        experimental.windows.kubernetes.io/isolation-type: hyperv
    spec:
      containers:
      - name: iis
        image: microsoft/iis
        ports:
        - containerPort: 80
```

### Deployment with kubeadm and cluster API

Kubeadm is becoming the de facto standard for users to deploy a Kubernetes
cluster. Windows node support in kubeadm is currently a work-in-progress but a
guide is available [here](/docs/tasks/administer-cluster/kubeadm/adding-windows-nodes/).
We are also making investments in cluster API to ensure Windows nodes are
properly provisioned.

### A few other key features
* Beta support for Group Managed Service Accounts
* More CNIs
* More Storage Plugins


