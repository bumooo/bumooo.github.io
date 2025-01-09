---
title: "Spring Batch 기본 개념"
author: bumoo
date: 2025-01-09 21:04:00 +0900
categories: [Spring, Spring Batch]
tags: [Spring Batch]
---

## 배치 처리 (Batch Processing)
대량의 데이터를 일괄 처리하는 방식

주기적 또는 조건에 따라 한 번에 여러 작업 처리

## Spring Batch

### 소개
스프링 프레임워크의 일부로, 배치 처리에 필요한 다양한 기능을 제공하는 오픈소스 프레임워크

재사용 가능한 컴포넌트를 통해 복잡한 배치 작업을 간단하게 구성하고 관리 가능

### 사용 목적
대규모 데이터 처리, 스케줄링 등을 쉽게 구현할 수 있도록 지원

트랜잭션 관리, 체크포인트 설정, Retry 등 다양한 확장 가능성 제공

## Spring Batch의 주요 기능

### 다양한 데이터 소스 지원
- 파일 (Ex. FlatFileItemReader)
- 데이터베이스 (Ex. JdbcCursorItemReader)
- Kafka 지원 (Ex. KafkaItemReader)
- Redis 지원 (Ex. RedisItemReader)

### 리트라이 및 스킵 처리
- 리트라이
  - `Step` 생성시 `retryLimit`, `retry`를 통해 재시도 가능
- 스킵
  - `Step` 생성시 `skipLimit`, `skip`를 통해 스킵 가능

### 병렬 처리
데이터를 여러 스레드에서 동시 처리
- `Job`
  - `Job` 생성시 `split`를 통해 여러 `Step`을 동시 실행 가능
- `Step`
  - `Step` 생성시 `partitioner`을 통해 데이터 여러 파티션으로 나눠서 병렬 처리
  - `gridSize`를 통해 파티션 수 지정 가능

### 배치 작업 모니터링 및 관리
`JobRepository`
- Job, Step 실행 상태를 관리
- `Job`의 성공, 실패 여부, `Step`의 상태에 따라 재실행 가능

`JobExecutionListner`
- Job 시작, 종료시 특정 로직을 실행 가능

### 트랜잭션 관리
- `Chunk` 기반 트랜잭션 처리
  - Chunk 크기로 트랜잭션 자동으로 관리
  - 해당 청크의 모든 처리되면 Commit, 오류 발생시 해당 청크 Rollback
  - 대규모 작업시 메모리 부담을 줄여주고, 실패시 특정 청크만 롤백하여 작업 효율을 높힘

- 비동기 트랜잭션 작업
  - 각 스레드가 병렬 작업을 하여도 스레드별 트랜잭션 관리
