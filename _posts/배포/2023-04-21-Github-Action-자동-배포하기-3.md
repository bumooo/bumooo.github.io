---
title: "Github Action 자동 배포하기 - 3"
author: bumoo
date: 2023-04-21 18:46:00 +0900
categories: [배포]
tags: [AWS EC2, AWS S3, CodeDeploy]
---

## 시작하기
> 1년간 주는 프리티어를 사용한다.

[배포하기 1](https://bumooo.github.io/posts/Github-Action-%EC%9E%90%EB%8F%99-%EB%B0%B0%ED%8F%AC%ED%95%98%EA%B8%B0-1/), 
[배포하기 2](https://bumooo.github.io/posts/Github-Action-%EC%9E%90%EB%8F%99-%EB%B0%B0%ED%8F%AC%ED%95%98%EA%B8%B0-2/)
에 이어서 작업을 진행한다.


## Github Actions에서 사용할 IAM 사용자 추가

AWS를 Github Actions 워크 플로우에서 접근하려면 권한이 필요하다.

현재까진 AWS 서비스 사용 권한만 추가했지만, 이번엔 외부에서 접근하는 것이다.

`AWS Console > IAM > 액세스 관리 > 사용자 > 사용자 추가`를 클릭한다.

![GitHub Actions 사용자 추가 - 1](https://user-images.githubusercontent.com/61149599/233607473-c6b19137-c2ca-439f-b449-cb4b3b88c201.png)

사용자의 이름을 지정 후 `다음`

![GitHub Actions 사용자 추가 - 2](https://user-images.githubusercontent.com/61149599/233607317-2b18e0fe-fe21-47d2-9e7b-15c87392232b.png)

`직접 정책 연결`을 선택 후 `AWSCodeDeployFullAccess`와 `AmazonS3FullAccess`를 추가한다.

추가로 생성했던 사용자를 클릭하면 `보안 자격 증명 > 액세스 키 > 액세스 키 만들기`를 한다.

![액세스 키 만들기 - 1](https://user-images.githubusercontent.com/61149599/233609382-31c2e620-d532-4349-9f31-c76eea070d72.png)

`AWS 외부에서 실행되는 애플리케이션`을 선택하고 생성한다.

![액세스 키 만들기 완료](https://user-images.githubusercontent.com/61149599/233609231-89642d9e-881f-488a-b1fd-2acc6918ff67.png)

`액세스 키`와 `비밀 액세스`를 어딘가에 잘 저장해놓는다.

## Github Repository의 Secrets 추가

`Github Repository > Settings > Secrets and variables > Actions > New Repository secret`에서 액세스 값을 추가한다.

![Github Repository Secrets 추가](https://user-images.githubusercontent.com/61149599/235062807-659788e2-cf03-4ace-a90e-544b5af5a643.png)

값들을 각 추가를 한다. 추가를 해놓는다고 값을 볼 수 없기 때문에 따로 적어놔야 한다.

## AppSpect 파일 작성

CodeDeploy에서 배포를 위해 참조할 
[AppSpec 파일](https://docs.aws.amazon.com/ko_kr/codedeploy/latest/userguide/application-specification-files.html)을 작성한다.

파일을 프로젝트의 루트 위치에 위치해야 한다.

```yaml
version: 0.0
os: linux

files:
  - source:  /
    destination: /home/ubuntu/app
    overwrite: yes

permissions:
  - object: /
    pattern: "**"f
    owner: ubuntu
    group: ubuntu

hooks:
  AfterInstall:
    - location: scripts/stop.sh
      timeout: 60
      runas: ubuntu
  ApplicationStart:
    - location: scripts/start.sh
      timeout: 60
      runas: ubuntu
```

AppSpec 파일에 작성하는 방법의 설명은 
[공식문서](https://docs.aws.amazon.com/ko_kr/codedeploy/latest/userguide/reference-appspec-file-structure.html)를 확인하면 된다.

### files
배포 파일에 대한 설정
- source : 인스턴스에 복사할 디렉토리 경로
- destination : 인스턴스에서 파일이 복사되는 위치
- overwrite : 복사할 위치에 파일이 있는 경우 대체할지 유무 

### permissions
file 섹션에서 복사한 파일에 대한 권한 설정
- object : 권한이 지정되는 파일 또는 디렉토리
- pattern(선택) : 매칭되는 패턴에만 권한 부여
- owner(선택) : `object`의 소유자
- group(선택) : `object`의 그룹 이름

### hooks
배포 이후에 수행할 스크립트를 지정할 수 있다.

라이프사이클이 존재하기 때문에 적절한 Hook을 찾아 실행할 스크립트를 지정하면 된다.

- location : hooks에서 실행할 스크립트 위치
- timeout(선택) : 스크립트 실행에 허용되는 최대 시간, 넘으면 배포 실패로 간주
- runas(선택) : 스크립트를 실행하는 사용자

## 배포 스크립트 작성

AppSpec 파일의 hooks에서 실행할 스크립트를 작성한다.

해당 내용은 `/scripts`에 작성한다.

### stop.sh
실행중이던 애플리케이션을 종료

```bash
#!/usr/bin/env bash

PROJECT_ROOT="/home/ubuntu/app"
JAR_FILE="$PROJECT_ROOT/spring-webapp.jar"

DEPLOY_LOG="$PROJECT_ROOT/deploy.log"

TIME_NOW=$(date +%c)

# 현재 구동 중인 애플리케이션 pid 확인
CURRENT_PID=$(pgrep -f $JAR_FILE)

# 프로세스가 켜져 있으면 종료
if [ -z $CURRENT_PID ]; then
  echo "$TIME_NOW > 현재 실행중인 애플리케이션이 없습니다" >> $DEPLOY_LOG
else
  echo "$TIME_NOW > 실행중인 $CURRENT_PID 애플리케이션 종료 " >> $DEPLOY_LOG
  kill -15 $CURRENT_PID
fi
```

### start.sh
새로운 애플리케이션을 실행

```bash
#!/usr/bin/env bash

PROJECT_ROOT="/home/ubuntu/app"
JAR_FILE="$PROJECT_ROOT/spring-webapp.jar"

APP_LOG="$PROJECT_ROOT/application.log"
ERROR_LOG="$PROJECT_ROOT/error.log"
DEPLOY_LOG="$PROJECT_ROOT/deploy.log"

TIME_NOW=$(date +%c)

# build 파일 복사
echo "$TIME_NOW > $JAR_FILE 파일 복사" >> $DEPLOY_LOG
cp $PROJECT_ROOT/build/libs/*.jar $JAR_FILE

# jar 파일 실행
echo "$TIME_NOW > $JAR_FILE 파일 실행" >> $DEPLOY_LOG
nohup java -jar $JAR_FILE > $APP_LOG 2> $ERROR_LOG &

CURRENT_PID=$(pgrep -f $JAR_FILE)
echo "$TIME_NOW > 실행된 프로세스 아이디 $CURRENT_PID 입니다." >> $DEPLOY_LOG
```

> `start.sh`에서 `cp $PROJECT_ROOT/build/libs/*.jar $JAR_FILE`를 통해 `모든 jar 파일`을 복사하므로 `build.gradle`에 내용을 추가한다. 
```gradle
jar {
  enabled = false
}
```
{: .prompt-info }

## Github Actions Workflow 작성

모든 작업이 끝났으니, `Github Actions 워크 플로우`를 작성한다.

`.github/workflows/deploy.yml`로 작성했다.

```yml
name: Deploy to Amazon EC2

on:
  push:
    branches:
      - main

# 본인이 설정한 값을 여기서 채워넣습니다.
# 리전, 버킷 이름, CodeDeploy 앱 이름, CodeDeploy 배포 그룹 이름
env:
  AWS_REGION: ap-northeast-2

permissions:
  contents: read

jobs:
  deploy:
    name: Deploy
    runs-on: ubuntu-latest
    environment: production

    steps:
      # (1) 기본 체크아웃
      - name: Checkout
        uses: actions/checkout@v3

      # (2) JDK 11 세팅
      - name: Set up JDK 11
        uses: actions/setup-java@v3
        with:
          distribution: 'temurin'
          java-version: '11'

      # (3) Gradle build (Test 제외)
      - name: Build with Gradle
        uses: gradle/gradle-build-action@0d13054264b0bb894ded474f08ebb30921341cee
        with:
          arguments: clean build -x test

      # (4) AWS 인증 (IAM 사용자 Access Key, Secret Key 활용)
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v1
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS }}
          aws-region: ${{ env.AWS_REGION }}

      # (5) 빌드 결과물을 S3 버킷에 업로드
      - name: Upload to AWS S3
        run: |
          aws deploy push \
            --application-name ${{ secrets.AWS_CODE_DEPLOY_APPLICATION_NAME }}\
            --ignore-hidden-files \
            --s3-location s3://${{ secrets.AWS_S3_BUCKET }}/$GITHUB_SHA.zip \
            --source .

      # (6) S3 버킷에 있는 파일을 대상으로 CodeDeploy 실행
      - name: Deploy to AWS EC2 from S3
        run: |
          aws deploy create-deployment \
            --application-name ${{ secrets.AWS_CODE_DEPLOY_APPLICATION_NAME }}\
            --deployment-config-name CodeDeployDefault.AllAtOnce \
            --deployment-group-name ${{ secrets.AWS_CODE_DEPLOY_DEPLOYMENT_NAME }}\
            --s3-location bucket=${{ secrets.AWS_S3_BUCKET }},key=$GITHUB_SHA.zip,bundleType=zip
```

간단히 설명해서 프로젝트 빌드 후 AWS S3 버킷으로 이동 후 CodeDeploy를 수행하는 일을 한다.

> 주의점<br>
> - PROJECT_ROOT인 `/home/ubuntu/app`가 존재해야한다.
>   - AWS EC2의 `/home/ubuntu`에서 `mkdir app` 실행해서 app 디렉토리 생성<br>
> - AWS EC2에 Java가 설치되어 있어야한다.
>   - `sudo apt-get install openjdk-{version}-jdk`으로 설치 후 `java -version` 설치 확인
{: .prompt-warning }

## 배포하기

`main` 브랜치가 배포가 되면, 아래와 같이 확인할 수 있다.

![GitHub Repo 배포 완료](https://user-images.githubusercontent.com/61149599/235063199-ada6b97a-59f0-4c7e-9ec9-247a2b3c743f.png)

![Github Actions 배포 완료](https://user-images.githubusercontent.com/61149599/235063179-66c06b4d-0fbf-4ec8-9412-d75147deb9fd.png)

배포 완료 후 AWS Console의 CodeDeploy에서 아래와 같이 확인이 가능하다.

![AWS Console CodeDeploy 완료](https://user-images.githubusercontent.com/61149599/235063381-33d81223-9a88-4d22-840c-ed9e49536bb1.png)

## 참조
- [Github Actions CD: AWS EC2 에 Spring Boot 배포하기](https://bcp0109.tistory.com/363)