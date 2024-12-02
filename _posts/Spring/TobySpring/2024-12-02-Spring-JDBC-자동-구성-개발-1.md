---
title: "Spring JDBC 자동 구성 개발 - 1"
author: bumoo
date: 2024-12-02 21:50:00 +0900
categories: ["토비의 스프링부트"]
tags: [SpringBoot]
---

> 해당 글은 토비의 스프링 부트 강의를 토대로 정리되었습니다.
{: .prompt-info }

지금부터는 Spring Boot의 자동 구성을 Spring의 JDBC 기술을 사용하는 인프라스트럭쳐 빈을 만들어봅니다.

Spring의 JDBC 템플릿을 가져다가 JDBC를 이용해 데이터베이스 프로그래밍까지 작업을 하려고 합니다.

최종적으로 필요한 설계는 아래의 이미지와 같습니다.

![최종 설계안](https://github.com/user-attachments/assets/21a2c4ef-10c9-4c0d-adf4-8a1312c8c7be)


