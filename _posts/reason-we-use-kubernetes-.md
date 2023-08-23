---
layout: post
title: "쿠버네티스를 사용하는 이유 == 오케스트레이션이 필요한 이유"
date: 2023-08-23
excerpt: "쿠버네티스 생초보 탈출 - with minikube"
tags: [kubernetes, backend]
comments: true
---

## 쿠버네티스를 사용하는 이유 == 오케스트레이션이 필요한 이유
 
컨테이너들이 무작위로 놓여 있으면 관리가 안된다. 같은 성격의 컨테이너는 모아놓기도 하고 공간이 낭비되지 않도록 하며, 때가 되면 새로운 버전의 컨테이너로 변경할 필요도 있다.  

오케스트레이션이 필요한 이유는 컨테이너(컨테이너는 어플리케이션을 패키징하고 실행하는 기술)들 끼리 잘 조화를 이루도록 관리가 필요하다. 

컨테이너를 가지고 실제 서비스를 운영하기 위해서는 고려할 요소가 많은데, 예를 들어 하나의 컨테이너가 다운되면 다른 컨테이너를 띄워야 하고 이 다운되는 시간을 최소화 하거나 아예 서버의 이중화를 통해 없애야 하는 경우도 있다. 

하루에도 수십번씩 컨테이너가 오르고 내리는 작업을 반복하기 때문에 이 작업을 자동화 할 필요가 있었다. 마이크로 서비스 시대에서 사용자의 개입없이 오케스트레이션을 하는 것이 반드시 필요했다. 

* 오케스트레이션이 필요한 이유

    * 컨테이너가 모인 클러스터에 상태를 지속적으로 관찰할 컨트롤 타워가 필요. 
        * 자동화된 스케일링
        * 자동화된 롤아웃과 롤백
        * 자동화된 복구(self-healing)

    * 빈 패킹이란(bin packing) 필요.
        * 자동화된 빈 패킹
        * 빈 패킹 : 자원 효율화를 위한 개념으로 컨테이너가 올라가는 호스트 노드의 cpu나 메모리 자원이 놀거나 낭비되지 않도록 다양한 크기의 컨테이너를 테트리스 하듯 배치하는 전략이다. 사람이 계산 하는 것은 복잡한 알고리즘으로 구현하기도 하고 계속 변하는 컨테이너 때문에 사실상 사람이 쫒아다니면서 최적화하는 것은 불가능하다. 특히 노드의 사용시간이나 사용 대수를 기반으로 비용을 측정하는 클라우드 환경에서는 빈 패킹이 중요하기에 이것을 자동화할 수 있어야 한다. 

    * 민감한 정보를 저장. 
        * 시크릿과 구성 관리 

    * 안정적인 확장과 배포. 
        * 서비스 디스커버리와 로드 밸런싱

    * 로컬 저장소나 클라우드 저장소 처럼 다양한 옵션이 있을 때 이런걸 바인딩 해줄 수 있는 시스템이 필요.
        * 스토리지 오케스트레이션

    * 인프라의 코드화 
        * 선언적 코드를 사용한 운영(IaC)

----

## 쿠버네티스 아키텍처

<img width="671" alt="스크린샷 2023-08-22 오후 8 01 00" src="https://github.com/MinChangJeong/minchangjeong.github.io/assets/65451455/ad8a0f36-ae6c-4c3e-9031-0f16cc9d6fe7">

* Control plane(Master mode) : 쿠버네티스 전체를 통제/관리
    * kube-apiserver
    * etcd
    * kube-schduler
    * kube-controller-manager(cloud-controller-manager)


* Data plane(Workder node) : 실제 사용자의 애플리케이션을 배포되어 컨테이너가 구동됨
    * kubelet
    * kube-proxy
    * container runtime

controller plane은 하나의 컨트롤 타워로써 동작하지만, 가용성을 위해 이중, 삼중으로 구성 가능하고 컨트롤 노드는 Active-Active 모드나 leader-Follower 모드로 동작한다. 

사용자는 kubectl이라는 명령어를 통해서 이 쿠버네티스 클러스터를 관리함. 이 명령어는 control plane에 전달하게 된다. 

data plane은 Control plane과 네트워크 통신을 하면서 사용자의 명령이나 control plane의 명령을 전달받아 실제 어플리케이션을 구동하게된다. 엔드 유저는 컨테이너에서 구동된 어플리케이션을 소비하게 되는 구조다. 

----

## 쿠버네티스 클러스터에 대한 개념

쿠버네티스는 어플리케이션의 가용성과 확장성을 올리기 위해 클러스터라는 개념을 사용한다. 
쿠버네티스에서 '클러스터' 라는 용어의 의미는 여러 대의 컴퓨터(호스트)가 함께 작동하여 쿠버네티스에서 관리도니느 컨테이너화된 애플리케이션 실행환경을 의미한다. 

클러스터 주요 구성 요소와 개념은 다음과 같다. 

* 마스터 노드 (Master Node): 마스터 노드는 클러스터를 제어하고 관리하는 중앙 제어 플레인이다.  주요 구성 요소로는 다음이 포함된다. 
    * API 서버 (API Server): 클러스터의 모든 작업을 관리하고 API를 노출하여 사용자 및 다른 구성 요소가 클러스터와 상호 작용할 수 있도록 한다. 
    * etcd: 분산형 키-값 저장소로 클러스터의 모든 구성 데이터와 상태 정보를 저장한다. 
    * 스케줄러 (Scheduler): 새로운 컨테이너가 어떤 노드에 배치될지 결정하여 작업을 분산합니다.
    * 컨트롤러 매니저 (Controller Manager): 클러스터의 상태를 지속적으로 모니터링하며, 필요한 상태로 유지하기 위한 컨트롤러를 관리한다. 
    * 클러스터 상태 저장소 (Cluster State Store): 클러스터의 모든 구성과 상태 정보를 저장하는 곳이다. 

* 워커 노드 (Worker Node 또는 Minion): 워커 노드는 컨테이너화된 애플리케이션을 실행하는 물리적 또는 가상의 호스트이다. 주요 구성 요소로는 다음이 포함된다.
    * 컨테이너 런타임 (Container Runtime): 컨테이너를 실행하는 데 사용되는 소프트웨어, 예를 들어 Docker와 같은 것이다.
    * Kubelet: 마스터 노드로부터 할당된 작업을 받아 해당 노드에서 컨테이너를 실행 및 모니터링한다. 
    * Kube Proxy: 네트워크 프록시로서, 컨테이너 간의 네트워킹 및 로드 밸런싱을 관리한다. 

* 포드 (Pod): 포드는 쿠버네티스에서 배포하는 가장 작은 배포 단위이다.  하나 이상의 컨테이너로 구성되며, 공유된 네트워크 및 저장소를 사용하여 함께 실행된다. 포드는 동일한 노드에서 실행되며, 서로 통신하고 데이터를 공유할 수 있다. 

* 서비스 (Service):
서비스는 포드의 네트워크 접근성을 관리하는 추상적인 개념이다. 포드의 IP 주소와 포트를 안정적으로 유지하며, 로드 밸런싱 및 서비스 디스커버리 기능을 제공한다. 

----

## 쿠버네티스 설치 

* 쿠버네티스 구축 환경

    * 로컬 환경
        * 스터디 용 또는 테스트 
        * Minikube
    * manual 구축 with 도구
        * 도구를 사용해서 원하는 곳에 클러스터를 구축
    * 관리형 쿠버네티스 서비스 (CSP 제공)
        * 퍼블릭 클라우드에서 제공하는 관리형 서비스 사용

    * minikube
        * 쿠버네티스를 로컬에서 실행할 수 있는 도구
        * 단일 노드 all in one 구성(control plane과 data plane으로 구분이 안되어 있음)으로 이중화가 보장되지 않기 때문에 상용에서 사용불가 
        * 로클 가상 머신에 쿠버네티스를 구성하기 때문에 하이퍼바이저 설치가 필요하다. 

* 쿠버네티스 in MacOS

    * install kubectl

    ```shell
    brew install kubectl
    ```

    * minikube install (아직 쿠버네티스 클러스터 설치 전)

    ```shell
    brew install minikube
    ```

* 클러스터 실행

```shell
minikube start
```

Hyper-V가 필요한데 기본적으로 내가 사용하고 있는 환경은 Docker 가 깔려 있어서 docker 드라이버가 선택되어 사용되는 중. 드라이버 선택을 하고 싶으면 명령어에 drive 옵션을 주면된다고 한다. 

```shell
~ » minikube status                   

minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```

* kubeconfig
    * 클러스터 인증 정보와 컨텍스트 
    * kubectl이 쿠버네티스와 통신할 때 필요한 접속 대상의 서버 정보, 인증 정보 등을 정의함.  
    * 기본위치: ~/.kube/config

```shell

apiVersion: v1
clusters:
- cluster:
    certificate-authority: /Users/jeongminchang/.minikube/ca.crt
    extensions:
    - extension:
        last-update: Tue, 22 Aug 2023 20:44:11 KST
        provider: minikube.sigs.k8s.io
        version: v1.31.2
      name: cluster_info
    server: https://127.0.0.1:58150
  name: minikube
contexts:
- context:
    cluster: minikube
    extensions:
    - extension:
apiVersion: v1
clusters:
clusters:
- cluster:
    certificate-authority: /Users/jeongminchang/.minikube/ca.crt
    extensions:
    - extension:
        last-update: Tue, 22 Aug 2023 20:44:11 KST
        provider: minikube.sigs.k8s.io
        version: v1.31.2
      name: cluster_info
    server: https://127.0.0.1:58150
  name: minikube
contexts:
- context:
    cluster: minikube
    extensions:
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /Users/jeongminchang/.minikube/ca.crt
    extensions:
    - extension:
        last-update: Tue, 22 Aug 2023 20:44:11 KST
        provider: minikube.sigs.k8s.io
        version: v1.31.2
      name: cluster_info
    server: https://127.0.0.1:58150
  name: minikube
clusters:
- cluster:
    certificate-authority: /Users/jeongminchang/.minikube/ca.crt
    extensions:
    - extension:
        last-update: Tue, 22 Aug 2023 20:44:11 KST
        provider: minikube.sigs.k8s.io
        version: v1.31.2
      name: cluster_info
    server: https://127.0.0.1:58150
  name: minikube
contexts:
- context:
    cluster: minikube
    extensions:
- context:
    cluster: minikube
    extensions:
    - extension:
        last-update: Tue, 22 Aug 2023 20:44:11 KST
        provider: minikube.sigs.k8s.io
        version: v1.31.2
      name: context_info
    namespace: default
    user: minikube
  name: minikube
current-context: minikube
kind: Config
preferences: {}
users:
- name: minikube
  user:
    client-certificate: /Users/jeongminchang/.minikube/profiles/minikube/client.crt
    client-key: /Users/jeongminchang/.minikube/profiles/minikube/client.key

```

여기서 중요한 정보는 **current-context: minikube**로 현재 컨테스트 정보를 나타내는데, 지금은 minikube라는 클러스터 하나만 가지고 있지만 실제 데브옵스 팀에서 운영할 때는 클러스터가 여러개인 경우가 일반적이다. 만약 컨텍스트 정보가 명시되어 있지 않으면, 많은 클러스터들이 동일한 kubectl 명령어를 쓰게 되는데 사용자가 내리는 명령어가 dev인지 prod인지 qa인지 클러스터 정보를 파악할 수 없기 때문에 매우 중요하다. 

* kubeconfig 컨텍스트 

```shell
# 컨텍스트 전환
kubectl config use-context minikube

# 현재 컨텍스트 확인
kubectl config cuurent-context

# 컨텍스트 목록 조회 
kubectl config get-contexts
```

* kubectl 명령어 확인하기 : [쿠버네티스 공식 문서](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

----

이 글은 쿠버네티스를 처음 학습 하면서 메모형식으로 정리한 글이므로 내용 정리가 안되어 있을 수 있습니다. 양해 부탁드립니다. 