# AKS 설치

## 사전 준비사항

### Azure CLI

Azure 명령 줄 인터페이스 \(Azure CLI\)는 Azure 리소스를 만들고 관리하는 데 사용되는 명령 집합입니다. Azure CLI는 Azure 서비스에서 사용할 수 있으며 자동화에 중점을두고 Azure에서 빠르게 작업 할 수 있도록 설계되었습니다.

[Azure CLI 설치 가이드 문서](https://docs.microsoft.com/ko-kr/cli/azure/install-azure-cli?view=azure-cli-latest)를 통해 각 OS 별 설치하기 위한 방법에 대해 소개합니다. 여기서는 Ubuntu 18.04 환경에서 apt 패키지 명령어를 통해 Azure CLI 설치하는 방법에 대해 소개하겠습니다.

#### 1. Azure CLI 설치

Azure CLI 모든 설치 명령을 한 번에 실행하는 스크립트를 제공합니다. curl 및 pipe를 사용하여 bash에 직접 실행하거나 스크립트를 파일로 다운로드하여 실행 가능합니다. 아래와 같이 curl 명령어를 통해 한번에 Azure CLI 설치가 가능합니다.

```text
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```

#### 2. Azure Account 생성

#### Azure Login

[http://portal.azure.com](http://portal.azure.com) 에서 생성한 계정 정보를 이용해서 아래와 같이 로그인이 가능합니다.

```text
az login -u your_id
```

Azure 구독에 여러개의 계정 등록이 가능하며, 구독에 등록된 계정 정보는 아래와 같이 확인 가능합니다.

```text
az account list
```

구독에 등록된 계정 중에서 현재 활성화된 게정은 다음과 같이 확인 가능합니다

```text
az accout show
```

## AKS 생성

### Azure SP\(Service Principal\) 생성

Azure 는 서비스 주체\(Service Principal\)을 통해 RBAC\(역할 기반 엑세스 제어\) 기능을 제공하고 있습니다. Azure AD\(Active Ditectory\) 를 통해 서비스 주체를 생성하고, AKS 리소스를 제어할 수 있는 RBAC 을 설정합니다. 아래와 같이 서비스 주체가 생성되면 서비스 주체를 식별하기 위한 정보들이 표시되고, AKS 생성시, 아래 정보를 활용하게 되니 따로 저장 관리가 필요합니다.

```text
az ad sp create-for-rbac --skip-assignment

"appId": "7248f250-0000-0000-0000-dbdeb8400d85",
"displayName": "azure-cli-2017-10-15-02-20-15",
"name": "http://azure-cli-2017-10-15-02-20-15",
"password": "77851d2c-0000-0000-0000-cb3ebc97975a",
"tenant": "72f988bf-0000-0000-0000-2d7cd011db47"
```

### AKS 생성에 필요한 리소스 정보 사전 정의

* CLUSTERNAME: 클러스터 이름을 임의로 정의합니다.
* RGNAME: Azure Resource Group 이름을 정의 합니다. \(폴더와 같은 논리적인 리소스 그룹\)
* K8SVERSION: AKS에 사용할 쿠버네티스 엔진의 버전을 명시합니다.
* LOCATION: Azure Region 중에 한국 중부\(koreacentral\)을 선택합니다.
* NODETYPE: 쿠버네티스 클러스터 노드 타입을 범용 VM \(Standard\_B2ms\)로 설정합니다.

```text
CLUSTERNAME=prl-kc-k8s-istiobooks
RGNAME=prl-kc-aks-rg
K8SVERSION=1.17.13
APPID=a80952d4-2a0f-45c2-b7d1-97a9d8c43618
CLIENTSECRET=your-service-principal-password
LOCATION=koreacentral
NODETYPE=Standard_B2ms
```

### Azure Resource Group 생성

AKS가 생성될 리소스 그룹을 생성합니다. 위 사전 정의된 값에 의해 아래 명령어를 실행하면 한국 중부\(koreacentral\)에 리소스 그룹\(prl-kc-aks-rg\)을 생서하게 됩니다. 이 리소스 그룹에 AKS 리소스를 생성할 예정입니다.

```text
az group create --location $LOCATION --name $RGNAME
```

### AKS에서 사용 가능한 쿠버네티스 버전 확인

AKS 에서 사용 가능한 쿠버네티스트 버전 확인이 가능합니다. preview 버전들은 안정성이 검증된 버전은 아니기 때문에 여기서는 1.17.13 버전을 설치하여 진행하겠습니다.

```text
az aks get-versions -l $LOCATION --output table

KubernetesVersion    Upgrades
-------------------  -----------------------------------------
1.19.3(preview)      None available
1.19.0(preview)      1.19.3(preview)
1.18.10              1.19.0(preview), 1.19.3(preview)
1.18.8               1.18.10, 1.19.0(preview), 1.19.3(preview)
1.17.13              1.18.8, 1.18.10
1.17.11              1.17.13, 1.18.8, 1.18.10
1.16.15              1.17.11, 1.17.13
1.16.13              1.16.15, 1.17.11, 1.17.13
```

### AKS 생성

위에서 사전 정의한 AKS 리소스 정보를 활용하여 AKS를 생서합니다. azcli 명령어를 통해 AKS 생성시 --no-wait 옵션을 설정했기 때문에 비동기 모드로 설치가 진행이 되고, 이후 시간이 지나면 쿠버네티스 클러스터 상태 확인을 통해 정상 설치 여부를 확인할 수 있습니다. 주의할 점은 $APPID 와 $CLIENTSECRET 은 서비스 주체 생성시 화면에 표출되어 별도 저장된 값을 활용하여 생성해야 합니다.

```text
# Create AKS Cluster
az aks create -n $CLUSTERNAME -g $RGNAME -s $NODETYPE \
--kubernetes-version $K8SVERSION \
--service-principal $APPID \
--client-secret $CLIENTSECRET \
--generate-ssh-keys -l $LOCATION \
--node-count 3 \
--no-wait
```

### AKS 상태 확인

일정 시간이 지나면 AKS의 ProvisioningState 가 Scceeded 상태로 변경되어 정상적으로 실행된 것을 확인할 수 있습니다.

```text
# verify aks cluster status
az aks list -o table          

Name                   Location      ResourceGroup    KubernetesVersion    ProvisioningState    Fqdn
---------------------  ------------  ---------------  -------------------  -------------------  -------------------------------------------------------------------
prl-kc-k8s-istiobooks  koreacentral  prl-kc-aks-rg    1.17.13              Succeeded            prl-kc-k8s-prl-kc-aks-rg-203f8c-f911d974.hcp.koreacentral.azmk8s.io
```

### AKS Credential 가져오기

kubectl 명령어를 통해 AKS를 제어하기 위해서는 AKS 에 접근하기 위한 credential 정보가 포함된 kubernetes config 파일이 필요합니다. 아래 명령어를 통해 ~/.kube/config 파일에 저장하도록 합니다.

```text
az aks get-credentials -n $CLUSTERNAME -g $RGNAME
```

### AKS 클러스터 정보 확인

이제 kubectl 명령어를 통해 AKS 제어가 가능하고, 설치된 클러스터의 정보를 아래 명령어릍 통해 확인 가능합니다.

```text
kubectl cluster-info

Kubernetes master is running at https://prl-kc-k8s-prl-kc-aks-rg-203f8c-f911d974.hcp.koreacentral.azmk8s.io:443
CoreDNS is running at https://prl-kc-k8s-prl-kc-aks-rg-203f8c-f911d974.hcp.koreacentral.azmk8s.io:443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
Metrics-server is running at https://prl-kc-k8s-prl-kc-aks-rg-203f8c-f911d974.hcp.koreacentral.azmk8s.io:443/api/v1/namespaces/kube-system/services/https:metrics-server:/proxy
```

### AKS 삭제

AKS 를 생성하는 순간부터 VM 노드에 대한 비용이 발생합니다. 따라서 필요에 따라 AKS를 삭제하거나, 클러스터 노드의 VMSS를 일시 deallocation 해야 비용을 절약할 수 있습니다.

* VMSS list 확인

```text
# vmss list
az vmss list --resource-group MC_prl-kc-aks-rg_prl-kc-k8s-istiobooks_koreacentral -o table

Name                         ResourceGroup                                        Location      Zones    Capacity    Overprovision    UpgradePolicy
---------------------------  ---------------------------------------------------  ------------  -------  ----------  ---------------  ---------------
aks-nodepool1-18165888-vmss  MC_prl-kc-aks-rg_prl-kc-k8s-istiobooks_koreacentral  koreacentral           3           False            Manual
```

* VMSS deallocation

```text
# vmss deallocation
az vmss deallocate --resource-group $RGNAME --name aks-nodepool1-18165888-vmss
```

* AKS 삭제

```text
# uninstall
az aks delete --name $CLUSTERNAME --resource-group $RGNAME --no-wait
```

Ref

* [https://github.com/Azure/kubernetes-hackfest/blob/master/labs/create-aks-cluster/README.md](https://github.com/Azure/kubernetes-hackfest/blob/master/labs/create-aks-cluster/README.md)

