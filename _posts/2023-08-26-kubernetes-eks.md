---
layout: post
title: "AWS EKS란?"
date: 2023-08-26
excerpt: "AWS EKS를 사용해 쿠버네티스 컨트롤 플레인 영역의 설정 시간을 단축"
tags: [kubernetes, backend]
comments: true
---

AWS EKS는 쿠버네티스 컨틀롤 플레인을 구성하거나 유지할 필요없이 AWS에서 쿠버네티스를 쉽게 실행할 수 있는 관리형 서비스를 의미한다. 

## AWS AKS란?

가장 큰 차이점은 Control-Plane 영역이다. EKS에서 컨트롤 플레인 영역은 AWS에서 완전 관리형으로 제공된다. AWS가 쿠버네티스 마스터 노드(컨트롤 플레인)를 관리하고 사용자는 본인의 계정에서 워커 노드(데이터 플레인)를 관리하도록 분담된다. 

<img width="684" alt="스크린샷 2023-08-26 오후 4 56 32" src="https://github.com/MinChangJeong/minchangjeong.github.io/assets/65451455/8e8a60d7-2419-434c-9ab5-5cc7de6fa3fb">

그림에서 보는 것처럼 사용자와 AWS 영역의 VPC가 애초에 분리되어 있다. 

VPC는 가설 사설망으로 클라우드 서비스 제공업체가 제공하는 가상 네트워크 환경을 의미한다. 마치 물리적인 네트워크와 유사하지만 클라우드 환경에서 가상으로 생성되는 것이다. 

예를 들어 오피스텔에 3개의 방이 존재할 때 각 방에서 살고 있는 사람들간에는 오갈 수 없다고 가정해본다면, 각 방은 VPC를 의미한다. 

즉, 같은 방안에는 있는 사람들끼리는 소통이 가능하지만, 다른 방안에 있는 사람들간에는 소통할 수 없다는게 특징이다. 

다시 돌아와 AWS 영역의 컨트롤 플레인과 사용자 측의 워커 노드들은 다른 VPC에 존재 하기 때문에 서로 존재 유무를 알 수 없다. 

가장 중요한 비용은 **1 클러스터 x 0.10 시간당 USD x 750 월별 시간 = 75.00 USD** 즉, 약 10만원인듯하다. 


