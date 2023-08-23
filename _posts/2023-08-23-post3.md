---
layout: post
title: "쿠버네티스 아키텍처 개념 정리 - Control Plane"
date: 2023-08-23
excerpt: "쿠버네티스 생초보 탈출 - with minikube"
tags: [kubernetes, backend]
comments: true
---

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


## etcd

쿠버네티스 Control plane에서 필요한 모든 데이터를 키-값 형태로 저장하는 데이터베이스를 의미한다. 쿠버네티스 클러스터의 모든 정보가 etcd에 담긴다. 

etcd가 다운되면 모든 컴포넌트가 미아가 되기 때문에 가용성이 매우 중요하다. etcd는 가용성을 위해 삼중 이상으로 클러스터링 구성을 하고 이 클러스터를 구성하기 위해서 분산 HA 알고리즘을 사용한다. 

<img width="636" alt="스크린샷 2023-08-23 오전 11 33 31" src="https://github.com/MinChangJeong/minchangjeong.github.io/assets/65451455/3c1ca936-9e4d-4629-9163-55ae35e38989">

클러스터에는 단일 Leader가 존재하며 나머지는 Follower로 등록 된다. 팔러워가 리더의 헬스체크를 했을 때 어떤 이유로든 그 헬스체크가 실패하면 팔러워중 한 대를 새로운 리더로 선출해서 서비스가 중단에 대비한다. 

etcd는 매우 중요한 저장소 역할을 하기 때문에 control 노드를 직접 운영하는 경우에는 etcd를 필요에 따라 백업하거나 복구를 해야하고 다양한 정보를 조회할 필요가 있을 수 있다고 한다. 그럴 때 사용하는 etcd 전용 cli가 있는데 ETCDctl이라고 한다. 

----

## kube-apiserver

사용자 kubectl 명령어를 사용해서 실제로 접근하는 주체이다. 쿠버네티스 프론트엔드로써 클러스터로 온 요청의 유혀서을 검증하고 다른 컴포넌트 간 통신을 중재한다. 

----

## kube-scheduler

클러스터 안에서 자원 할당이 가능한 노드 중 알맞은 노드를 선택하는 역할을 한다. (노드는 클러스터내에서 작업을 수행하는 개별적인 컴퓨터 시스템이나 서버를 가리킨다.)

그리고 실제로 새로운 pod를 실행해주는 역할은 kube-apiserver랑 worker node에 kubelet이 담당한다. 처음 파드가 실행될 때 최소 할당되어야 하는 cpu나 메모리같은 설정들을 할 수가 있는데 그런 조건에 맞춰 가장 알맞은 노드에 파드를 실행시켜주는 역할을 하는게 kube-scheduler이다. 

* pod 스케줄리의 필요성 - 예시
    * 머신러닝 워크로드를 돌리는 특정 pod는 GPU가 탑재된 node에서만 돌아야 한다. 
    * consumer들은 네트워크 intensive하므로 전용 node group을 쓰고 싶다. 
    * 팀별로 node를 나눠서 사용하고 싶다.

* pod 스케줄링 분류 
    * 사용자가 특정 노드에 pod를 배치하고 싶을 때 - Selector, Affinity
    * 관리자가 특정 노드에는 pod가 배치되는 것을 막고 싶을 때 - Taints, Tolerations

사용자와 관리자를 나눠서 쓴이유는 실제 데브옵스 조직에서는 클러스터의 전반적으로 관리하는 관리자와 
만들어진 클러스터를 소비하는 사용자가 나뉠수 있다. 이때 사용자는 신규 서버를 개발하고 서비스하는 개발자가 될 수도 있다. 그래서 사용자는 클러스터의 전반적인 내용을 다 알지 못하기 때문에 원하는 곳에 배치하고자 할 것이고 관리자는 그럼에도 불구하고 파드가 배치되는 안되는 노드가 있을 수 있는데 예를들어 서비스 목적이 아닌 관리 목적 노드가 그럴 수 있다 때문에 관리자는 특정 노드가 스케줄링 되는 것을 막아야 할 필요가 있다. 

* Taints and Toleration
    * 어떤 파드가 어떤 노드에 스케줄링 될 수 있는지를 제한한다. 
    * 예를 들어 쿠버네티스의 control node에는 파드가 스케줄링 되지 않도록 taint 되어 있다. 

<img width="644" alt="스크린샷 2023-08-23 오후 12 56 27" src="https://github.com/MinChangJeong/minchangjeong.github.io/assets/65451455/370ae42d-ee06-460e-969f-43bb086344c2">

위의 그림을 참고하자면, 오른쪽 파드는 아래의 노드의 오염됨을 견딜 수 있기에 스케줄링 될 수 있지만 오른쪽은 아래 노드의 오염을 견딜 수 없기 때문에 스케줄링 될 수 없다. 

Taints는 노드에 설정되는 라벨과 비슷한 속성으로 해당 노드에 어떤 제약을 설정하는 역할을 한다. 예를 들어 특정 노드에 특정 파드를 배치하지 않기 위해 'NoSchedule' Taints를 설정할 수 있다. 

Toleration은 파드가 특정 노드의 Taint를 견딜 수 있는 정도를 지정하는 개념이다. 자신이 허용하는 Taint와 맞지 않는 노드에 스케줄 되지 않는다. 

예를 들어 노드에 'gpu=true:NoSchedule'이라는 Taint가 설정되어 있다면 이 노드에는 GPU를 필요로 하는 파드가 스케줄 되지 않는다. 하지만 파드 스팩에 'gpu=true:NoSchedule' Taint를 허용하는 Toleration을 설정했다면 해당 노드에 GPU가 있는 경우에만 그 파드가 스케줄된다. 

* Lables and Selector(Affinity)
    * Node Selector
    * NodeAffinity

Node Selector는 파드를 특정 노드에 스케줄링 하기 위해 사용되는 방법중 하나이다. 예를들어 특정 노드에 gpu=true라는 라벨이 있고 파드의 nodeSelector에 gpu=true를 설정하면 해당 노드에만 gpu를 필요로 하는 파드가 스케줄링된다. 

<img width="637" alt="스크린샷 2023-08-23 오후 1 05 13" src="https://github.com/MinChangJeong/minchangjeong.github.io/assets/65451455/8c71e473-26c8-46d3-bba6-5ab5f7275a5c">

Node Affinity는 파드를 특정 노드에 스케줄링하거나 특정 노드에서 스케줄링하지 않도록 제어하기 위한 더 강력한 메커니즘이다. 

<img width="651" alt="스크린샷 2023-08-23 오후 1 06 00" src="https://github.com/MinChangJeong/minchangjeong.github.io/assets/65451455/2c2550e4-6617-4ef9-a4df-7d94bf1ca471">

Node Affinity는 두가지 유형으로 나뉜다. 

RequiredDuringSchedulingIgnoredDuringExecution(스케줄링 시 필요, 실행 중 무시): 이 옵션을 사용하면 파드가 해당 조건을 만족하는 노드에만 스케줄링된다. 실행 중에는 조건이 변해도 파드가 영향을 받지 않습니다.

PreferredDuringSchedulingIgnoredDuringExecution(스케줄링 시 선호, 실행 중 무시): 이 옵션을 사용하면 파드는 해당 조건을 가진 노드를 선호하지만, 다른 노드에 스케줄링될 수 있습니다. 실행 중에도 조건이 변해도 파드가 영향을 받지 않는다.

* Taint & Toleration 과 selector & affinify가 그러면 차이가 없는게 아닌가? 

<img width="651" alt="스크린샷 2023-08-23 오후 1 10 15" src="https://github.com/MinChangJeong/minchangjeong.github.io/assets/65451455/c9234dc3-dda4-49bd-907b-90612d90bb9f">

위의 그림의 예시에서 만약 Taint & Toleration만 있다고 가정해보자. Taint & Toleration는 특정 노드에 설정된 Taint에 대한 Toleration이 설정된 파드만 스케줄되도록 막는 역할을 한다. 그런데 회색 노드에는 아무런 taint가 설정되지 않았기 때문에 blue나 green 파드가 스케줄링되는 것을 막을 수 없다. 

이번에는 만약 selector & affinify만 있다고 가정해보자. 이 경우에는 회색 컨테이너가 blue나 green 노드에 스케줄링되는 상황을 막을 수 없을 것이다. 

따라서 노드나 파드에대한 요구사항이 엄격한 경우라면 Taint & Toleration 과 selector & affinify를 함께 사용하는게 각각의 파드가 특정 노드에 스케줄링 되는 것이 가능할 것이다. 

minikube는 단일 노드라 실습할 수 없다...

----

## controller-manager

쿠버네티스 클러스터 내에서 여러가지 컨트롤러들을 관리하고 운영하기 위한 프로세스이다. 여기서 컨트롤러들은 클러스터 내 리소스의 상태를 관리하고 원하는 상태로 유지하기 위해 동작한다. 

controller-manager는 이러한 컨트롤러들의 동작을 감독하며, 클러스터의 원활한 운영과 상태 유지를 돕는 중요한 구성 요소 중 하나이다. 

* controller-manager는 여러가지 내장 컨트롤러와 커스텀 컨트롤러를 관리한다. 
    * Replication Controller : 파드의 복제본 수를 유지하고 실패한 파드를 재시작 하는 역할을 한다. 이를 통해 파드의 가용성을 유지하고 원하는 파드 복제본 수를 유지할 수 있다. 

    * Deployment Controller : Replication Controller와 비슷하지만 더 많은 기능을 제공하며 롤링 업데이트와 롤백, 버전 관리 등을 지원하여 어플리케이션을 배포관리한다. 

    * StatefulSet Controller : StatefulSet은 각 파드에 고유한 식별자를 부여하여 지속적인 상태를 관리하는 컨트롤러이다. 주로 DB와 같이 상태를 유지해야하는 어플리케이션에 사용된다. 

    * DaemonSet Controller : 모든 노드에 특정 파드를 하나씩 배포하는 역할을 한다. 모니터링, 로그 수집 등 노드 별로 실행되어야 하는 파드에 사용된다. 

    * Job Controller : 한 번만 실행되는 일꾼 파드를 관리한다. 배치 작업과 같이 지정된 작업을 완료하는 파드를 관리한다. 

    * CronJob Controller : 스케줄에 따라 주기적으로 실행되는 작업을 관리한다. Cron 작업과 유사한 방식으로 일정 시간마다 작업을 수행하는 파드를 관리한다. 