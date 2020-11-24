# Istio 주요기능

쿠버네티스를 현재 개발/운영하고 있다면 Istio 는 풍부한 여러가지 유용한 도구를 통해 업무 효율성을 향상시킬 수 있습니다. Istio 에서 제공하는 주요 기능들을 살펴보면 다음과 같습니다.

### 모니터링 \(Observability\)

Kubernetes 클러스터에 Istio를 설치하고 실행하면 클러스터 안에서 실행되는 여러 서비스들이 어떻게 동작하고 있는지 궁금합니다. Istio 에서 제공하는 쿠버네티스 클러스터 시각화, 마이크서비스 아키텍처 기반의 복잡한 Pod 단위 수준까지 트랙피이 어떻게 발생하고, 어느 구간에서 문제가 발생하는지 분산 추적하여 정확히 무슨 일이 일어나고 있는지 디버깅하기 위한 인사이트를 제공합니다. 이를 위해 Istio 는 아래와 같은 각 기능을 제공하여 Service Mesh 환경에서 분산 트레이싱이 가능한 환경을 제공합니다.

* TCP Metric
* Kiali
* Jageer
* Prometheus & Grafana

### 트래픽 관리 \(Traffic Management\)

쿠버네티스 클러스터에서 배포하여 서비스 개발/운영중인 트래픽의 흐름을 제어할 수 있습니다. 외부 네트워크 장비의 정책을 변경하지 않고, 클러스터 내에서 발생하는 모든 애플리케이션 트래픽에 대해 심플하게 제어함으로써  가능합니다. 예를 들면 테스트되지 않는 새로운 애플리케이션을 상용\(Production\) 환경에서 시작하여 서비스 할때 일반 사용자로부터는 숨기고 내부 시스템 엔지니어만 엑세스하여 신규 서비스에 대한 품질 체크를 할 수 있습니다. 즉 엔지니어가 원하는 목적에 맞게 트래픽을 제어하고, 시스템의 탄력성과 안정성을 높이기 위한 다양한 방법으로 활용 가능합니다.

### 탄력성 & 안정성 \(Service Resillency\)

Istio는 트래픽 관리 기능을 통해 시스템의 탄력성과 안정성을 높이기 위한 다양한 방법을 제공합니다. 대규모 분산 소프트웨어의 발전으로 개발의 유연성과 배포의 속도를 높이는 다양한 기술들이 제공되고 있고, 이러한 복잡한 MSA 아키텍처 환경에서의 운영자의 핵심은 "우리 상용 환경에서의 서비스를 신뢰할 수 있는가?" 입니다. 분산 시스템내 각 서비스들이 올바르게 동작하는 경우에도 가 서비스 간에 예측하기 힘든 상호작용으로 인해 장애를 발생시키고, 결국 시스템 전체에 혼란\(Chaos\)를 가져옵니다. Istio는 카오스 엔지니어링을 통해 분산 시스템의 불확실성을 구체적으로 해결하기 위한 예측하기 힘든 시스템의 약점을 밝히고 쉽게 실험하기 위한 방법론을 아래와 같이 제공합니다.

* 결함주입\(Fault Injection\)
* 써킷 브레이커\(Circuit Breaker\)
* 시간 초과\(Timeout\)
* 재시도\(Retries\)

### Canary 배포

![](../.gitbook/assets/image%20%2824%29.png)

카오스 엔지니어링을 통해 분산 시스템의 불확실성을 해소하였다고 하더라도, 신규 애플리케이션 또는 기존 애플리케이션을 버그 수정하여 새로운 버전으로 상용망 환경에 배포하는 작업은 매우 보수적으로 접근해야 합니다. 이를 위해 Istio 는 카나리 배포를 제공합니다. 카나리 배포는 위험을 빠르게 감지할 수 있는 배포 전략입니다. Istio 트래픽 제어를 통해 특정 테스트 유저 그룹 또는 내부 시스템 엔지니어에게만 배포하고, 이후 검증되면 모든 사용자에게 배포합니다. 이러한 카나리 배포의 전략은 A/B 테스트가 가능하며, 이를 통해 가정에 대한 불확실성이 아닌 실제 검증을 통한 확실성을 통해 시스템 전체로 배포를 위한 의사결정을 제공합니다.

### Security

![](../.gitbook/assets/image%20%2812%29.png)

#### Encryption \(암호화\)

Istio 는 쿠버네티스 클러스터에서 발생하는 모든 트래픽에 대해 envoy 를 통해 암호화 통신 보안을 제공합니다. 쿠버네티스에 배포되는 Pod 에 Sidecar Pattern 을 통해 envoy가 자동 배포되고, Pod에서 발생하는 모든 트래픽에 대해 envoy 가 기본적으로 TLS 암호화하여 서비스간 데이터 전송에 대한 암호화 기능을 제공합니다.

#### Authentication \(인증\)

Istio 는 서비스에 대한 인증\(Authentication\)을 제공합니다. 여기서 말하는 인증이란 서비스 간 통신시 해당 서비스 주체를 식별하고, 인증 이후 서비스에 대한 권한\(Authorization\) 제어가 가능합니다.  Istio는 서비스 간 상호 인증을 위해 mTLS\(mutual TLS\)를 이용하여 서비스 간 인증 및 트래픽 암호화 기능을 제공합니다. 또한 서비스 간 인증뿐만 사용자\(End User\)를 인증하기 위한 JWT\(Json Web Token\) 방식의 인증기술을 제공합니다.

#### Authorization \(권한\)

Istio는 인증된 서비스 또는 사용자 주체에 대해 서비스 권한\(Authorization\)을 통제하기 위한 기능을 제공합니다. 이를 위해 역할 기반 권한 인증 \(RBAC: Role Based Authorization Control\)을 지원함으로써 인증된 사용자\(End User\), 서비스 계정\(i.e: Kubernetes Service Account\) 에 적절한 역할\(Role\)을 부여하여 권한 관리가 가능합니다.
