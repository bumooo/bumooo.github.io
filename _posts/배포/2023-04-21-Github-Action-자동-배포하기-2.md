---
title: "Github Action 자동 배포하기 - 2"
author: bumoo
date: 2023-04-21 00:16:00 +0900
categories: [배포]
tags: [AWS EC2, AWS S3, CodeDeploy]
---

## 시작하기
> 1년간 주는 프리티어를 사용한다.

## AWS S3

AWS S3란, Amazon Web Service의 객체 스토리지 서비스이다. 대용량의 데이터를 저장하고 검색할 수 있도록 되어 있다.

## S3 Bucket 생성

AWS Console에서 S3를 검색하여 `버킷 만들기`를 한다.

![S3 버킷 생성](https://user-images.githubusercontent.com/61149599/233411188-1b10c80e-1df8-41a1-8b14-32a5b57c7f29.png)

버킷 이름과 생성할 지역을 선택한다.

![S3 기본 암호화](https://user-images.githubusercontent.com/61149599/233413032-6633204c-f77d-4f68-8a59-868b3f0c7058.png)

`기본 암호화 > 암호화 키 유형`에서 `Amazon S3 관리형 키`인 경우엔 암호화를 해도 과금이 되지 않는다.

그래도 혹시 몰라서 `비활성화` 상태로 설정했다.

그리고 `버킷 만들기`를 하면 된다.

## CodeDeploy 생성

이전에는 `AWS EC2에 CodeDelpoy Agent 설치`했다면 이제 생성을 해보자.

### 1. IAM에 가서 역할을 추가

![CodeDeploy 역할 생성](https://user-images.githubusercontent.com/61149599/233414175-e8a251db-8341-4dc8-8bd9-b61500c4fada.png)
`CodeDeploy`만 선택한 후 `3단계`로 이동한 후 `역할 이름` 설정 후 생성한다.

### 2. CodeDeploy 애플리케이션 생성

`AWS Console`에서 `CodeDeploy`를 검색한 후 `CodeDeploy > 애플리케이션 > 애플리케이션 생성`을 클릭한다.

![CodeDeploy 애플리케이션 생성](https://user-images.githubusercontent.com/61149599/233415700-57a5fe30-d17e-4fa5-a3ca-c25327b6d181.png)

애플리케이션 이름과 `컴퓨팅 플랫폼 > EC2/온프레미스`로 설정 후 생성한다.

### 3. CodeDeploy 배포 그룹 생성

좌측에 `CodeDeploy > 애플리케이션 > 애플리케이션 > 배포 그룹 생성`을 클릭한다.

![CodeDeploy 배포그룹 생성 - 1](https://user-images.githubusercontent.com/61149599/233416424-3c86c755-9128-4d7f-a503-b3e8bfab59ee.png)

배포 그룹 이름과 서비스, 배포 유형을 선택한다. 

서비스 역할은 `1. IAM에 가서 역할을 추가`에서 생성한 역할을 선택한다.

![CodeDeploy 배포그룹 생성 - 2](https://user-images.githubusercontent.com/61149599/233417105-ac0ec9fe-9d35-49d6-9733-bffbc56c3898.png)

배포는 `Amazon EC2 인스턴스`를 선택하고, 이전에 EC2에 추가한 `태그 그룹`을 선택한다.

![CodeDeploy 배포그룹 생성 - 3](https://user-images.githubusercontent.com/61149599/233417632-a1acbfac-f70a-4434-a7fb-92f814371def.png)

`CodeDeploy`는 `AWS Systems Manager`를 통해 EC2 인스턴스 배포 작업을 자동화한다. EC2 인스턴스에 대한 업데이트, 패치, 환경 구성 등을 자동 수행 가능하다.

여기선 중요하지 않으니 적당히 설정 후 `로드밸런싱을 비활성화`한다.

그리고 배포 그룹 생성한다.

> CodeDeployDefault.OneAtTime : 각 인스턴스를 순차적으로 배포<br>
> CodeDeployDefault.HalfAtTime : 인스턴스를 두 그룹으로 나누어 배포<br>
> CodeDeployDefault.AllAtOnce : 배포 그룹에 해당하는 모든 인스턴스를 동시 배포<br>
{: .prompt-info }


