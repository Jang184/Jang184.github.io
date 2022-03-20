---
layout: post
title: AWS에서 제공하는 서비스에는 무엇이 있을까?
subtitle: AWS 제공 서비스 소개
author: "Juri"
header-style: text
catalog: true
tags:
    - Cloud
    - aws
    - kocw
---

<i>이 포스팅은 부산디지털대학교의 [클라우드 인프라 구축 및 활용](www.kocw.net) 강의를 보고 작성했습니다.</i>

## AWS 제공 서비스

AWS가 제공하는 서비스는 170개 정도로 전체 서비스를 파악하기 쉽지않다. 대표적인 서비스를 알아보도록 한다.

![](/img/in-post/cloud-infra-4.png)

<div style="text-align:center">최근 사용한 AWS 서비스</div>

## 데이터 분석 서비스

**Athena**<br>
서버가 필요없는 서비스로 표준 SQL을 사용해서 S3의 데이터를 간편하게 분석할 수 있는 대화식 쿼리.

**EMR(Elsatic Map Reduce)**<br>
다양한 오픈 소스 도구를 사용하여 방대한 양의 데이터를 처리하기 위한 클라우드 빅 데이터 플랫폼.

## 분석 (Analytics)

**Elasticsearch Service**<br>
데이터 웨어하우스, 운영 데이터베이스와 데이터 레이크에 있는 petabyte 규모의 정형 데이터 및 비정형 데이터를 SQL를 통해 쿼리

**Kinesis**<br>
실시간으로 비디오 및 데이터 스트림을 수집, 처리, 분석

**QuickSight**<br>
대시보드 및 시각화, 빠른 비즈니스 분석 서비스

## VPS (Virtual Private Service)

**Amazon Elastic Compute Cloud (EC2)**<br>
클라우드에서 안전하고 크기 조정이 가능한 가성 서버 및 컴퓨팅 파워 제공

**Amazon Lightsail**<br>
애플리케이션이나 웹 사이트를 구축하는데 필요한 모든 것을 제공하며 사용이 간편한 클라우드 플랫폼

## 컨테이너 서비스

**Amazon Elastic Container Service (ECS)**<br>
컨테이너를 관리

**Amazon Elastic Container Registry (ECR)**<br>
컨테이너 이미지를 손쉽게 저장, 관리, 배포하는 서비스

## 서버리스

**AWS Lambda**<br>
서버에 대한 걱정없이 코드를 실행하며 사용한 컴퓨팅 시간에 대해서만 비용을 지불하는 서비스

**AWS Fargate**<br>
컨테이너를 위한 인스턴스를 관리할 필요없이 컨테이너 자체만을 배포할 수 있도록 하는 기술

## 객체 스토리지

**Amazon Simple Storage Service (S3)**<br>
어디서나 원하는 양의 데이터를 저장하고 검색할 수 있도록 구축된 객체 스토리지

## 블록 스토리지

**Amazon Elastic Block Store**<br>
대규모 처리량과 트랜잭션 집약적인 워크로드 모두를 지원하기 위해 제공한다. EC2에서 사용하도록 설계된 고성능 블록 스토리지

## 백업

**AWS Backup**<br>
AWS 서비스 전체에 걸쳐 중앙에서 백업 관리 및 자동화

## 데이터 전송 및 엣지 컴퓨팅

**AWS Storage Gateway**<br>
온프레미스 애플리케이션과 클라우드 스토리지를 연결해주는 서비스, 하이브리드 클라우드 스토리지

## 데이터베이스 서비스

**AWzon Aurora**<br>
MySQL, PostgreSQL 호환 관계형 데이터베이스

**Amazon RDS**<br>
클릭 몇 번으로 클라우드에서 관계형 데이터베이스 설정, 운영, 조정

**Amazon Redshift**<br>
Petabyte 단위의 데이터를 관리/사용하기 위한 강력하고 빠른 속도를 제공하는 warehouse 형테의 서비스

**Amazon DynamoDB**<br>
NoSQL 데이터베이스 서비스

**Amazon DocumentDB**<br>
탁월한 속도 및 확장성과 고가용성을 제공하는 MongoDB 호환 데이터베이스 서비스

**Amazon ElastiCache**<br>

-   캐시 또는 데이터 스토어로 사용할 수 있는 Memcahched 호환 인 메모리 키-값 스토어 서비스
-   클라우드용으로 구축된 Redis와 호환 가능한 인 메모리 데이터 스토어 서비스

## 네트워크 서비스

**AWS VPC**<br>
사용자가 정의한 가상 네트워크로 AWS 리소스를 시작할 수 있음

**Elastic Load Balancing**<br>
단일 가용 영역 또는 여러 가용 영역에서 다양한 애플리케이션 부하를 분산 처리

**Amazon Route53**<br>
최종 사용자를 인터넷 애플리케이션으로 라우팅하는 안정적이고 비용 효율적인 방법

**Amazon CloudFront**<br>
빠르고 안전하며 프로그래밍이 가능한 콘텐츠 전송 네트워크 (CDN)

**Amazon API Gateway**<br>
API생성, 게시, 유지관리, 모니터링 및 보호

## 마치며

AWS에서 아주 다양한 서비스를 제공하고 있음을 알 수 있다. 그 중에서 정말 필요한 서비스만 골라서 사용할 수 있으려면 어떤 서비스가 어떤 기능을 하는지 제대로 알고 있어야 할 것같다. 먼저, 많은 사람들이 사용하는 조합으로 AWS 서비스를 연습하고 조사한 뒤 하나씩 개발 서비스에 적용해보는 시간이 필요할 듯 하다.
