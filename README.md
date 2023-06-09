# 진짜진짜 쉬운 통합 운영환경 (Monitoring & Logging)

<img src="https://capsule-render.vercel.app/api?type=waving&color=auto&height=200&section=header&text=통합모니터링&fontSize=90" />

<p align='center'> 통합 모니터링 같이 알아봅시다! </p>
<p align='center'>
  <a href="https://github.com/sangwonchoi/observability/labels/Idea">
    <img src="https://img.shields.io/badge/IDEA%20ISSUE%20-%23F7DF1E.svg?&style=for-the-badge&&logoColor=white"/>
  </a>
  <a href="#demo">
    <img src="https://img.shields.io/badge/DEMO%20-%234FC08D.svg?&style=for-the-badge&&logoColor=white"/>
  </a>
</p>

# Navigation
1. [Session Agenda](#Session-Agenda)
2. [쿠버네티스 짧게 복습하기](#쿠버네티스-짧게-복습하기)
3. [통합모니터링이 왜 필요할까요?](#통합모니터링이-왜-필요할까요?)
5. [Github Clone](#Github-Clone)

# Session Agenda
1. 세션을 통해 쿠버네티스와 통합 모니터링에 대한 개념을 이해한다.
2. OpenSource에 대해 간략히 알아본다.
3. 통합 모니터링에 필요한 컴포넌트를 이해한다.
4. 통합 로깅에 필요한 컴포넌트를 이해한다.
5. 직접 모니터링 클러스터와 로깅 클러스터를 각자의 환경에 맞게 구축해본다.
6. Index 관리에 대해 이해한다.

# 쿠버네티스 짧게 복습하기
![Kubernetes](https://img.shields.io/badge/kubernetes-%23326ce5.svg?style=for-the-badge&logo=kubernetes&logoColor=white)
https://kubernetes.io/ko/docs/home/ \
쿠버네티스(Kubernetes)는 컨테이너화된 애플리케이션을 배포, 확장 및 관리하기 위한 오픈 소스 플랫폼입니다. 구글에서 개발된 Borg 시스템을 기반으로 2014년에 오픈소스로 공개되었고, 현재는 Cloud Native Computing Foundation (CNCF)에 의해 관리되고 있습니다.

쿠버네티스는 다음과 같은 주요 개념과 구성 요소로 구성되어 있습니다:

1. 마스터 노드 (Master Node): 쿠버네티스 클러스터의 제어 플레인을 관리하는 주요 컴포넌트입니다. 마스터 노드에는 API 서버, 스케줄러, 컨트롤 매니저 등의 컴포넌트가 포함되어 있습니다.

2. 워커 노드 (Worker Node): 애플리케이션 컨테이너가 실행되는 노드입니다. 워커 노드에는 컨테이너 런타임(예: Docker), 쿠버네티스 에이전트(kubelet)와 프록시(kube-proxy) 등의 컴포넌트가 설치되어 있습니다.

3. 파드 (Pod): 쿠버네티스에서 가장 작은 배포 단위인 컨테이너 그룹입니다. 파드는 한 개 이상의 컨테이너로 구성되며, 공유된 네트워크 및 스토리지를 사용할 수 있습니다. 파드는 수평 확장, 로드 밸런싱, 스케줄링 등의 기능을 제공합니다.

4. 서비스 (Service): 쿠버네티스 내의 파드에 대한 네트워크 서비스를 정의하는 리소스입니다. 서비스는 파드의 안정적인 네트워크 주소와 로드 밸런싱을 제공하여 클러스터 내부 및 외부에서 파드에 접근할 수 있도록 합니다.

5. 볼륨 (Volume): 컨테이너에 데이터를 영구적으로 저장하기 위한 쿠버네티스의 추상화된 스토리지입니다. 볼륨은 파드 내의 컨테이너 간에 데이터를 공유하고, 데이터의 영속성과 안정성을 보장합니다.

6. 네임스페이스 (Namespace): 쿠버네티스 클러스터를 가상으로 분할하는 방법으로, 리소스 그룹화와 권한 분리를 위해 사용됩니다. 네임스페이스는 클러스터 내에서 리소스의 범위를 지정하고 격리를 제공합니다.

쿠버네티스는 다양한 기능을 제공하여 애플리케이션의 배포, 확장, 관리를 간편하게 할 수 있습니다. 이를 통해 고가용성, 확장성, 안정성이 높은 애플리케이션을 구축하고 운영할 수 있으며, 클라우드 환경 뿐만 아니라 온프레미스 환경에서도 쉽게 사용할 수 있습니다.

# 통합모니터링이 왜 필요할까요?
![Go](https://img.shields.io/badge/go-%2300ADD8.svg?style=for-the-badge&logo=go&logoColor=white) \
![Prometheus](https://img.shields.io/badge/Prometheus-E6522C?style=for-the-badge&logo=Prometheus&logoColor=white)https://prometheus.io/docs/introduction/overview/ \
![Grafana](https://img.shields.io/badge/grafana-%23F46800.svg?style=for-the-badge&logo=grafana&logoColor=white)https://grafana.com/docs/ \
https://thanos.io/v0.26/thanos/getting-started.md/ \
https://docs.fluentd.org/ 
https://docs.fluentbit.io/manual/ \
https://opensearch.org/docs/2.6/ 
1. 시스템 전반적인 가시성 확보: 쿠버네티스는 여러 개의 컨테이너로 구성된 분산 시스템을 관리하는데 사용됩니다. 통합 모니터링은 이러한 컨테이너들과 관련된 리소스, 성능, 상태 등을 실시간으로 모니터링하여 전반적인 시스템 가시성을 확보합니다. 이는 시스템의 상태를 신속하게 파악하고, 잠재적인 문제를 사전에 예측하고 예방하는 데 도움을 줍니다.

2. 이상 상황 탐지와 대응: 통합 모니터링은 쿠버네티스 클러스터 내의 모든 컨테이너와 리소스의 동작 상태를 모니터링하므로, 이상 상황이 발생할 경우 신속하게 탐지할 수 있습니다. 예를 들어, 컨테이너의 비정상적인 CPU 또는 메모리 사용, 네트워크 연결 문제 등을 실시간으로 감지하여 이를 적절히 대응할 수 있습니다. 이를 통해 시스템의 안정성과 가용성을 높일 수 있습니다.

3. 자원 최적화와 확장성 관리: 통합 모니터링은 쿠버네티스 클러스터 내의 리소스 사용량을 추적하고 분석함으로써 자원 최적화를 도모할 수 있습니다. 리소스 사용량의 패턴을 파악하여 리소스의 효율적인 할당과 확장을 관리할 수 있습니다. 이는 비용 효율성을 높이고, 클러스터 성능을 향상시키는 데 도움이 됩니다.

4. 로그 및 이벤트 관리: 쿠버네티스 환경에서는 여러 컨테이너 및 노드에서 생성되는 로그와 이벤트 데이터를 수집하고 분석할 수 있어야 합니다. 통합 모니터링은 이러한 로그 및 이벤트 데이터를 중앙 집중화하여 저장하고, 이를 기반으로 문제 해결을 위한 분석 및 디버깅 작업을 수행할 수 있습니다.

5. 경험 기반의 운영 개선: 통합 모니터링은 운영팀에게 시스템의 운영 상태와 성능 통계, 경험을 제공합니다. 이를 통해 운영팀은 시스템의 동작을 실시간으로 파악하고, 병목 현상이나 성능 저하 등을 조기에 인지할 수 있습니다. 이는 운영 개선을 위한 의사 결정과 운영 전략 수립에 큰 도움을 줍니다.

6. 종합적으로, 쿠버네티스에서 통합 모니터링 환경을 구축하면 시스템 가시성, 이상 상황 탐지, 자원 최적화, 로그 관리, 경험 기반의 운영 개선 등 다양한 이점을 얻을 수 있습니다. 이는 쿠버네티스 환경에서 안정적이고 효율적인 운영을 위해 필수적입니다.

# Github Clone
```
git clone https://github.com/sangwonchoi/observability.git
```
