---
title: "GitHub Actions 이해하기"
author: bumoo
date: 2023-02-09 00:12:00 +0900
categories: [배포]
tags: [Github Actions]
---

## GitHub Actions 이해
GitHub Actions는 빌드, 테스트 및 배포 파이프라인을 자동화할 수 있는 CI/CD 플랫폼이다.
Repository에 대한 모든 PR(Push Request)을 빌드 및 테스트하거나, 프로덕션에 Merge PR을 배포하는 WorkFlow를 만들 수 있다.

단순한 DevOps을 넘어 Repository에서 다른 이벤트가 발생했을 때 WorkFlow를 실행할 수 있도록 한다.

Github에서 WorkFlow를 실행할 Linux, Windos, macOs 가상 머신을 제공하거나, 사용자의 고유의 데이터 센터 또는 클라우드 인프라에 자체 호스팅 실행기를 호스트할 수 있다.

> CI(Continuous Integration) : 개발자를 위한 자동화 프로세스인 지속적인 통합<br>
> CD(Continuous Delivery/Deployment) : 지속적인 서비스 제공 및 지속적인 배포

## GitHub Actions 구성 요소

PR이나 이슈가 생성되는 것과 같은 'Event'가 Repository에 발생할 때 Trigger되도록 GitHub Actions WorkFlow 구성할 수 있다.

워크플로우는 순차적 또는 병렬로 실행될 수 있는 'Job'을 하나 이상 포함한다.
각 'Job'은 자체 가상 머신 'Runner' 또는 컨테이너 내에서 실행되며, 스크립트를 실행하거나 WorkFlow를 간소화할 수 있는 재재사용 가능한 'Step'을 하나 이상 포함합니다.

![GitHub Actions의 구성](https://user-images.githubusercontent.com/61149599/217564607-96b3b8a3-c612-495e-819c-ef3b57d5cbe4.png)

> Trigger(트리거) : 이벤트에 반응해 자동으로 실행되는 작업

#### WorkFlow

하나 이상의 'Job'을 실행하는 구성 가능한 자동화된 프로세스이다.
Events로 Trigger될 때 실행되거나 수동으로 또는 특정 일정에 따라 트리거될 수 있다.

WorkFlow는 Repository의 `.github/workflows` 디렉토리에 정의하고, 여러 Workflow를 가질 수 있다.
각 WorkFlow는 다른 작업을 수행할 수 있다.

### Event

Workflow를 실행을 Trigger하는 Repository의 특정 활동
예를 들어 PR 요청하거나, issue를 생성하거나, Repository에 commit을 push할 때 등이 될 수 있다.

### Job

동일한 Runner에서 실행되는 Workflow의 Step이다.
각 Step은 쉘 스크립트를 실행하거나, actions를 실행한다. Step은 순서대로 실행되며 서로 종속적이다.
같은 Runner에서 실행되어, 데이터를 공유할 수 있다.

종속성을 구성할 수 있다. 기본적으로 작업은 종속성 없이 병렬로 실행된다. 다른 작업에 대한 종속성이 있다면, 종속 작업이 완료 후 실행될 수 있다.

### Action

복잡하지만, 자주 반복되는 일을 수행하는 GitHub Actions 플랫폼용 사용자 지정 애플리케이션이다.
Workflow 파일에 작성하는 반복 코드의 양을 줄일 수 있다.

### Runner

Trigger 될 때 Workflow를 실행하는 서버이다. 각 Runner는 한 번에 하나의 Job을 실행할 수 있다.
GitHub는 Workflow를 실행할 Linux, Windows, macOS Runner를 제공한다.

## 예제 WorkFlow
Repository에 `.github/workflows'에 YAML 파일로 저장한다.

> 예시 : Repository에 Push가 될 때마다 자동으로 Trigger하는 Workflow 생성

1. Repository에 `.github/workflows` 디렉토리 생성한다.
2. `.github/workflows` 디렉토리 안에 `learn-github-actions.yml` 파일 생성하고, 아래 코드 작성한다.
    ```yaml
    name: learn-github-actions # GitHub Repository의 Action에 표시되는 Workflow의 이름
    run-name: ${{ github.actor }} is learning GitHub Actions # Workflow를 실행한 사용자 이름을 표시
    on: [push] # Workflow의 Trigger 지정
    jobs: # Workflow에서 실행되는 모든 Job을 그룹화
        check-bats-version: # 'check-bats-verions'이라는 Job을 정의
            runs-on: ubuntu-latest # Runner의 버전을 작성
            steps: # Job의 모든 Step을 그룹
                - uses: actions/checkout@v3 # 'actions/checkout@v3'를 이용하여 checkout하고 Runner에 다운로드하여 작업을 실행
                - uses: actions/setup-node@v3 # 'actions/setup-node@v3'를 이용하여 Node.js 을 설치
                    with: # action에 값을 전달
                        node-versions: '14' # 설치하는 Node.js의 버전 14 설정
                - run: npm install -g bats # 'npm'을 사용하여 'bats'를 설치
                - run: bats -v # 'bats'의 버전을 확인
    ```
3. 변경 내용을 Commit 후 Repository에 Push 한다.

> `- uses` : 누군가 만들어 놓은 명령어를 찾아 실행<br>
> `- run` : Runner에 커맨드 명령어 실행

## 참조
- [GitHub Docs](https://docs.github.com/ko/actions/learn-github-actions/understanding-github-actions)
- [카카오웹툰은 GitHub Actions를 어떻게 사용하고 있을까?](https://fe-developers.kakaoent.com/2022/220106-github-actions/)
