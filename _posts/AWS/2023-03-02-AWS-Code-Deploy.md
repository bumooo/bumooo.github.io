---
title: "Github Action 자동 배포하기 - 1"
author: bumoo
date: 2023-03-02 19:28:00 +0900
categories: [프로젝트, 데이로그]
tags: [AWS EC2, CodeDeploy]
---


## Github Action에서 공식적으로 제공하는 방법
1. AWS S3 빌드파일을 압축해서 업로드 → AWS EC2 배포
2. AWS ECR에 도커 이미지 업로드 → AWS EC2 배포

둘 중에 1번으로 배포를 해볼 예정이다.

## CodeDeploy란
AWS EC2, 온프레미스 인스턴스 등 애플리케이션 배포를 자동화하는 배포 서비스이다.

1번 항목이 `CodeDeploy`를 이용한 방법이다.

## AWS EC2 추가 설정

Deploy를 설정할 AWS EC2에 설정 추가하기

1. 태그 추가하기 : `작업 > 인스턴스 설정 > 태그 관리`에서 태그의 Key, Value를 설정
![EC2 태그 추가](https://user-images.githubusercontent.com/61149599/222396991-a78f7ade-6bd1-41f7-a216-19f13ae93c73.png)


2. IAM 역할 추가
검색창에 `IAM`을 검색하여, 좌측에서 `역할`탭으로 이동 후 우측에 `역할 만들기`

신뢰할 수 있는 엔티티 유형 : `AWS 서비스`

사용 사례 : `EC2`

를 선택 후 `다음`
![IAM 역할 만들기](https://user-images.githubusercontent.com/61149599/222397595-6b3a10fb-d02d-4b4f-9c96-3cb0f5b868cf.png)

검색창에 `S3FullAccess` 검색 후 해당 권한 추가 후 `다음`

![IAM 역할 만들기 - 권한](https://user-images.githubusercontent.com/61149599/222398293-7b63d963-c632-4f5e-8cf7-f33199f216cd.png)

이름 설정 후 `역할 생성`

추가할 EC2 인스턴스 정보로 돌아와 `작업 > 보안 > IAM 역할 수정` 생성한 `IAM 역할`로 설정
![EC2 IAM 역할 설정](https://user-images.githubusercontent.com/61149599/222398911-ba396c0f-dba7-4c34-92f8-354682fe98ef.png)

## AWS EC2에 CodeDeploy Agent 설치


[CodeDeploy Agent 공식 문서](https://docs.aws.amazon.com/ko_kr/codedeploy/latest/userguide/codedeploy-agent-operations-install-ubuntu.html)를 보고 참고하면 된다.
AWS EC2에 접속 후
```bash
$ sudo apt update
$ sudo apt install ruby-full
$ sudo apt install wget
$ cd /home/ubuntu
$ wget https://aws-codedeploy-ap-northeast-2.s3.ap-northeast-2.amazonaws.com/latest/install
$ chmod +x ./install
$ sudo ./install auto > /tmp/logfile
$ sudo service codedeploy-agent status
```

> 공식 문서에 따라 진행을 했을 때<br>
> `sudo service codedeploy-agent status` → `Unit codedeploy-agent.service could not be found.`이 계속 발생<br>
> `cat /tmp/codedeploy-agent.update.log`으로 확인하니, Ruby 3.0.2 버전을 제공하지 않는다고 한다.<br>
> ![deploy 오류 로그](https://user-images.githubusercontent.com/61149599/222400865-4cf78cec-1def-4e48-a5e4-127f846bd6b5.png)
{: .prompt-danger }

## 해결 방법
[Github Issue](https://github.com/aws/aws-codedeploy-agent/issues/301#issuecomment-1129912011)에 나와있는 정보로 오류를 해결
해당 댓글을 따라해도 괜찮지만, 나는 `아시아 태평양(서울)`이므로 변경을 했다.
```bash
$ cd /tmp
$ wget https://aws-codedeploy-ap-northeast-2.s3.ap-northeast-2.amazonaws.com/releases/codedeploy-agent_1.3.2-1902_all.deb
$ mkdir codedeploy-agent_1.3.2-1902_ubuntu22
$ dpkg-deb -R codedeploy-agent_1.3.2-1902_all.deb codedeploy-agent_1.3.2-1902_ubuntu22
$ sed 's/Depends:.*/Depends:ruby3.0/' -i ./codedeploy-agent_1.3.2-1902_ubuntu22/DEBIAN/control
$ dpkg-deb -b codedeploy-agent_1.3.2-1902_ubuntu22/
$ sudo dpkg -i codedeploy-agent_1.3.2-1902_ubuntu22.deb
$ systemctl list-units --type=service | grep codedeploy
$ sudo service codedeploy-agent status
```

결과적을 아래 화면처럼 나오면 완료이다.
![codeDeploy Agent 설치 완료](https://user-images.githubusercontent.com/61149599/222402043-7103c150-5f44-4038-be84-1b7c2893664e.png)
