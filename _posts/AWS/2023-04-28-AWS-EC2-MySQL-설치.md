---
title: "AWS EC2에 MySQL 설치 및 접속하기"
author: bumoo
date: 2023-04-28 15:37:00 +0900
categories: [AWS]
tags: [AWS EC2, MySQL]
---

> 해당 내용은 MySQL 8버전입니다.
{: .prompt-info }

## AWS EC2에 MySQL 설치

1. 우선 Linux 패키지 매니저를 업데이트
```bash
sudo apt-get update
```

2. MySQL 서버 설치
```bash
sudo apt-get install mysql-server
mysql -V
```

3. MySQL 접속
```bash
sudo mysql -u root -p
```

처음에 패스워드를 설정했으면 해당 `password`로 입력, 하지않았다면 그냥 `enter`를 눌러 접속한다.

접속이 완료되면 아래와 같다.

![MySQL 접속완료](https://user-images.githubusercontent.com/61149599/235070258-290acf7b-68fa-4aca-8fb3-a09314961b13.png)

`password`를 설정하지 않았다면, 설정해줘야 한다.

```sql
ALTER USER 'root'@'localhost' IDENTIFIED BY '{패스워드}';
FLUSH PRIVILEGES;
```

## DataBase 와 사용자 계정 추가

1. DataBase를 추가
```sql
create database {DB명};
```

2. 사용자 계정 추가
```sql
create user '{사용자 계정명}'@'%' identified by '{패스워드}';
```

3. 새 계정에 데이터베이스 권한 부여
```sql
grant all on {DB명}.*@ to '{사용자 계정명}'@'%' with grant option;
FLUSH PRIVILEGES;
```

## 생성한 사용자 계정으로 MySQL 접속
```bash
mysql -u {사용자 계정명} -p
```

그리고 패스워드를 입력하면 접속이 가능하다.

## 보안그룹 인바운드 규칙 추가

모든 IP에 허용하도록 수정을 했다. 각자 알맞게 설정하면 된다.

![MySQL 인바운드 규칙 추가](https://user-images.githubusercontent.com/61149599/235071797-6baea7e9-e2b5-4aff-a0b1-edd57e4fcfbe.png)

## MySQL 열어두기

AWS EC2에서 아래와 같이 입력한다.

```bash
sudo -i
vi /etc/mysql/mysql.conf.d/mysqld.cnf
```

그리고 열린 파일에서 
```bash
bind-address          = 127.0.0.1
mysqlx-bind-address   = 127.0.0.1
```
를 찾아 주석처리한다.

```bash
# bind-address          = 127.0.0.1
# mysqlx-bind-address   = 127.0.0.1
```

파일을 저장한 뒤 MySQL을 재실행한다.

```bash
sudo systemctl restart mysql
```

## MySQL WorkBench 접속

![Workbench 접속 정보](https://user-images.githubusercontent.com/61149599/235073009-2ad4eb46-5070-4ec1-8365-b23e101197a2.png)

`Store in Keychain ...`를 통해 패스워드 입력 후 `Test Connection`을 한다.

성공을 하면 아래와 같은 이미지가 나온다.

![MySQL 접속 완료](https://user-images.githubusercontent.com/61149599/235072826-dab9e17c-b922-45f6-8509-d1f55e55c7b2.png)