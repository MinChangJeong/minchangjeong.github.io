---
layout: post
title: "쿠버네티스 오브젝트"
date: 2023-08-26
excerpt: "쿠버네티스의 모든 오브젝트에 대한 개념 정리"
tags: [kubernetes, backend]
comments: true
---

이 글에서는 크게 3가지에 대해 정리한다. 

1. 쿠버네티스 오브젝트의 yaml 구조를 이해한다. 
2. kubectl로 오브젝트를 다룬다. 
3. 서비스를 이루는 오브젝트간의 관계를 이해한다. 

----

## 쿠버네트스 오브젝트란?

쿠버네티스에서 오브젝트는 '하나의 의도를 담은 레코드'를 의미한다. 

예를 들어 어떤 컨테이너화된 어플리케이션이 몇개로 다중화되어서 컨테이너를 띄울지, CPU 몇개를 할당할지, 배포는 어떤 전략으로 진행할지와 같은 수많은 구성요소들을 사용자가 정의한 '레코드'의 형태를 띄는 것이다. 

사용자가 쿠버네티스의 워커 노드를 어떤식으로 운영할지 결졍하기 위해, 오브젝트를 어떤 의도를 가지고 생성할지 쿠버네티스에 전달한다고 볼 수 있다. 

* 오브젝트 생성

오브젝트에 대한 기본적인 정보와 의도한 상태를 기술한 오브젝트 스팩을 제시할 수 있다. 오브젝트 생성을 위한 쿠버네티스 API 요청은 JSON 형식 정보를 포함한다. 대부분의 경우 정보를 yaml 파일로 kubectl에 제공한다. 

```shell
# kubectl을 사용해 쿠버네티스에 오브젝트 생성 요청을 전달한다. 
$ kubectl apply -f deployment.yaml
```

예시) service 오브젝트 생성

```yml
apiVersion : v1
kind: Service
metadata: 
    name: mytest-service
    ports: 
        - targetPort: 80
          port: 80
          NodePort: 30008
```

* yaml의 기본 구조
    * apiVersion (string)
        * 어떤 오브젝트를 만드냐에 따라 정해진 값이 다르다. 
        * 예를들어 Service 오브젝트는 v1, Deployment 오브젝트는 apps/v1을 기술한다. 
    * kind (string)
        * 만들고자 하는 오브젝트의 종류
    * metadata (dictionary)
        * 오브젝트에 부여할 이름이나 기타 메타 데이터를 기술
    * spec (dictionary)
        * 만들고자 하는 오브젝트에 따라 '의도한 상태'를 기술
    * status (dictionary)
        * 사람이 아니라 시스템에 의해 기술되는 부분으로 오브젝트의 '실제 상태'를 기술한다. 
        * 쿠버네티스 컨트롤 플레인은 **오브젝트의 실제 상태를 의도한 상태에 일치시키는 방향으로 동작한다.** 
        * 예를들어 사용자가 파드의 개수를 2개에서 4개로 yaml 파일을 변경한다면 순간적으로 사용자가 정의한 spec부분과 쿠버네티스가 관리하는 status 정보가 어긋날 수 있다. 그럼 쿠버네티스는 다시 spec부분에 기술된 내용으로 status를 변경한다. 


----

## node(Worker node)

노드는 오브젝트라고 보기에는 예외적인 리소스이다. 그 이유는 쿠버네티스 클러스터를 만들때 같이 만들어지는 자원으로 사용자가 yaml로 따로 생성할 필요가 없기 때문이다. 

데이터 플레인의 워커 노드를 의미하고 워크 로드가 돌아가는 컨테이너를 배치하는 물리 혹은 가상머신이고 컨트롤 플레인에 의해 관리된다. 

minikube 같은 경우는 노드가 하나 만있었지만 실제로는 여러 노드로 구성하고, 각 노드는 kubelet, container-runtime, kube-proxy가 포함되어 있다. 


## Namespace

하나의 클러스터에서 여러개의 '가상' 클러스터를 지원한다. 파드 같은 쿠버네티스 리소스는 네임스페이스로 그룹화가 되는데, 그룹화의 목적은 클러스터를 논리적으로 나누고 액세스를 제한해 리소스를 제어하고 확인하며 관리하는 방법을 제공하기 위함이다. 

논리적으로 구분이 되지만 격리가 되는 것은 아니다 예를 들어 하나의 네임스페이스에 정의된 파드를 다른 네임스페이스에서 접근할 수 없는 것은 아니다. (격리를 위해서는 network policy와 같은 다른 오브젝트를 추가로 사용 해야한다.)

* Namespace 예시
    * 기본 namespace
        * default - pod, deployment가 생성될 때 기본저긍로 생성되는 namespace
        * kube-system - DNS, kube-proxy나 kubernetes dashboard 처럼 시스템 제어 리소스가 사용하는 namespace
        * kube-public - 전체 클러스터 리소스에 대한 가시서을 제공하는 경우 사용

    * custom namespace
        * prometheus, argo, istio 등등의 시스템 관련 솔루션은 독자 namespace를 할당한다. 
        * microservice 별로 namespace를 할당하여 논리적으로 분리한다. 

* namespace & resource quota yaml 구조

1. namespace
```yml
    apiVersion: v1
    kind: Namespace
    metadata: 
        name: dev
```

2. namespace quota
resource quota는 namespace에 할당할 수 있는 리소스를 제한하는 역할 


1. namespace
```yml
    apiVersion: v1
    kind: ResourceQuota
    metadata: 
        name: compute-quota
        namespace: dev
    spec: 
        hard: 
            pods : "10"
            requests.cpu: "4"
            requests.memory: "5Gi"
            limits.cpu: "10"
            limits.memory: "10Gi"
```

## pod

* 최소단위 쿠버네티스 객체 
* docker 컨테이너와는 조금 다르게, pod는 하나 이상의 컨테이너를 포함 가능하다. 
* 일반적으로 하나의 파다는 하나의 서비스를 담고 있는 경우가 많다. 
* 같은 ip 주소를 가지고 localhost 통신(port로 구분)
* 하지만 pod는 일반적으로 하나의 컨테이너를 구동함. (컨테이너간의 의존도가 높아진다는 문제가 있음.)
* 그럼 왜 하나의 pod에 여러 컨테이너를 띄울 수 있도록 지원했을까?
    * pod의 디자인 패턴
        * sidecar pattern: 메인 컨테이너에서 로그를 수집하고 서브 컨테이너에 로그를 처리하는 main-sub 컨테이너 관계
        * adapter pattern: 프로메테우스같이 특정 데이터의 형식을 준수해야하는 경우 컨테이너1에서 데이터를 처리하고 컨테이너2에서 매트릭 수집을 위한 데이터 형식 변환을 담당해 프로메테우스에서 컨테이너2로 접근하도록 하는 예시
        * ambassador pattern: 컨테이너1에서 메인로직을 수행하고 컨테이너2에서 외부로의 통신을 지원하는 패턴
    * 하나의 컨테이너에서 위에서 언급한 모든 패턴을 구현할 수 있지만 컨테이너의 목적 자체가 하나의 컨테이너는 하나의 목적만을 갖는다는 것을 달성하기 위해 다음과 같은 pod 디자인 패턴이 필요하다.
    
```yml
    // 1. 포트 할당
    apiVersion: v1
    kind: Pod
    metadata:
        name: myapp-prod
        namespace: app
        lablels: 
            app: myapp
            type: front-end
    spec: 
        containers: 
            - name: nginx-container
              image: nginx
              ports: 
              - containerports: 8080

    // 2. 멀티 컨테이너 
    apiVersion: v1
    kind: Pod
    metadata: 
        name: myapp2-prod
        namespace: app
        labels: 
            app: myapp
            type: front-end
    spec: 
        containers: 
            - name: nginx-container
              image: nginx
            - name: redis-container
              image: redis:3.2
```

## replicaset

* pod만 가지고 서비스를 운영할 수 있지만, 서비스의 가용성이나, 탄력성을 위해 replicaset 같은 오브젝트를 구성한다. 
* replicaset은 똑같은 파드를 여러개 생성하고 관리하기 위한 리소스로, 특정 수의 동일한 파드가 같이 구동되도록 하는 오브젝트이다. 

```yml
    // 1. 포트 할당
    apiVersion: v1
    kind: ReplicaSet
    metadata:
        name: myapp-replicaset
        lablels: 
            app: myapp
            type: front-end
    spec: 
        template: 
            metadata: // pod의 메타데이터 이하를 그대로 가져옴
                name: myapp-prod
                labels: 
                    app: myapp
                    type: front-end
            spec: 
                containers: 
                    - name: nginx-container
                      image: nginx
        replicas: 3
        selector: // 어떤 파드가 해당 replicaset에 할당 되는지를 나타냄
            matchLabels: 
                type: front-end
```