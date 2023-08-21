---
layout: post
title:  "벅스리움 (시흥시 곤충 박물관)"
date:   2023-08-20
excerpt: "인턴 - (주) 해나소프트"
project: true
tags: [project, 인턴]
comments: true
---

## [벅스리움 (시흥시 곤충 박물관)](https://bugsrium.siheung.go.kr)

----

### 1. 개발 환경 및 역할

* 참여 인원: 개발자 3명 / 디자이너 1명
* 기간 : 2022-06-27~2022-08-19
* 개발 환경
  * Open JDk 1.8
  * Spring 4.x
  * Mybatis 
  * Mysql 8.x
  * Apache tomcatn 9.x
  * AngularJs
  * HTML, CSS, JavaScript
  * Jenkins
  * Linux, Window
* 역할 : 풀스택 개발
  * backend : Java/Spring + Mybatis 기반 프레임워크 
  * Frontend : AngularJs
  * DB : ERD 설계 및 DB 스키마 생성 - DB 테이블 최소 20개 이상

----

### 2. 프로젝트 개요

시흥시 곤충 박물관 
* 대민
  * 회원관리 
  * 곤충 박물관 방문 프로그램 예약 관리 
  * 공지사항 및 1:1 문의 관리 
  * 기타 홈페이지 필요 정보 페이지 구성
* 관리자 
  * 예약프로그램 등록 및 예약자 조회 및 등록 
  * 공지사항 등록 및 1:1문의 답변
  * 기타 관리자 페이지 필요 기능

----

### 3. 프로젝트를 통해 얻은 점

* **Java/Spring + Mybatis + Mysql + AngularJs 기반 프레임워크 설계 방법**
* 고급 쿼리 기술 
  * 부분 집계 함수 - Rollup 함수 사용
* ERD 설계 및 테이블 스키마 생성 방법
* DB 엔진에 대한 이해
* **데이터의 정합성을 보장하기 위한 트랜잭션의 사용과 Locking**
* 스토리보드 작성 방법
* **일을 보고하는 방법**

----

### 4. 프로젝트에 기여한 점

* 예약 관리 
  * 예약관리 시스템 구축
  * 이슈 : 매달 1일 예약을 위한 트래픽이 한번에 몰림으로써 데이터의 정합성이 보장되지 않는 문제 발생(프로그램의 정원은 20명이지만 22명 예약이 되는 상황 발생)
  * 해결 : Transactional, Locking

* SNS(문자 및 카카오톡) 전송 모듈 구현
  * 이슈 : 예약 확정, 1:1문의 등록 등, DB에 관련 내용을 등록하는 로직과 SNS 전송 로직을 처리하는 쓰레드를 분리  
  * 해결 : Multi Thread 

* 그 외 간단한 CRUD 구현
  * DB 쿼리르 작성하는데 어려움이 없음.

----

### 5. 실제 페이지

<img width="1312" alt="image1" src="https://github.com/MinChangJeong/minchangjeong.github.io/assets/65451455/f7efef1d-1e09-457a-9a15-78f2eb78f45a">
<img width="1312" alt="image2" src="https://github.com/MinChangJeong/minchangjeong.github.io/assets/65451455/6a59b7d9-03ce-4517-9b69-ce3fbaa798e5">
<img width="1292" alt="image3" src="https://github.com/MinChangJeong/minchangjeong.github.io/assets/65451455/e11a41b3-1538-42f6-92ca-06468a6d1afa">
----