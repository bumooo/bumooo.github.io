---
title: "Spring JDBC 자동 구성 개발 - 3"
author: bumoo
date: 2024-12-03 22:39:00 +0900
categories: ["토비의 스프링부트"]
tags: [SpringBoot]
---

> 해당 글은 토비의 스프링 부트 강의를 토대로 정리되었습니다.
{: .prompt-info }

이번엔 앞에서 만든 `DataSourceConfig`에 `JdbcTemplate`, `JdbcTransactionManager`를 추가합니다.

추가할 2개의 빈은 `DataSource`클래스 빈에 의존합니다. 또한 `DataSource` 빈이 1개인 경우에만 동작하도록 합니다.

> JdbcTemplate : SQL을 이용한 Java 코드를 작성할때 번거로운 코드 구성을 간결하게 사용해주는 템플릿 클래스<br/>
> JdbcTransactionManager : JDBC를 활용하는 코드의 트랜잭션 관리의 복잡한 작업들을 스프링의 트랜잭션 추상화를 이용해 편리하게 관리할 수 있는 클래스
{: .prompt-info }

```java
@MyAutoConfiguration
@ConditionalMyOnClass("org.springframework.jdbc.core.JdbcOperations")
@EnableMyConfigurationProperties(MyDataSourceProperties.class)
@EnableTransactionManagement
public class DataSourceConfig {
    /* hikariDataSource, dataSource 유지 */

    @Bean
    @ConditionalOnMissingBean
    @ConditionalOnSingleCandidate(DataSource.class)
    JdbcTemplate jdbcTemplate(DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }

    @Bean
    @ConditionalOnMissingBean
    @ConditionalOnSingleCandidate(DataSource.class)
    JdbcTransactionManager jdbcTransactionManager(DataSource dataSource) {
        return new JdbcTransactionManager(dataSource);
    }
}
```
`@ConditionalOnSingleCandidate(DataSource.class)`는 `DataSource.class`로 된 빈이 1개인 경우에만 작동하도록 설정합니다.

`JdbcTransactionManager`는 직접 액세스해서 트랜잭션 관리하는 기능을 사용할 수 있지만, 대체로 `@Transactional`을 통해 관리하게 됩니다.

`@EnableTransactionManagement`는 `@Transactional`을 사용하기 위해선 AOP 관련된 기능을 사용할 수 있어야합니다. AOP 사용하기 위한 구성 정보를 가진 애너테이션입니다.

이전 `DataSourceTest`에 추가된 애너테이션에 `@Transactional`을 추가하여, 복합적 기능을 하는 애너테이션을 추가합니다.

```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@ExtendWith(SpringExtension.class)
@ContextConfiguration(classes = HellobootApplication.class)
@TestPropertySource("classpath:/application.properties")
@Transactional
public @interface HelloBootTest {
}
```

그럼 모든 DB 테스트에는 `@HelloBootTest`만 추가하면 됩니다.

> 테스트 코드에 `@Transactional`이 붙는 이유는 해당 애너테이션이 없으면 테스트로 인한 데이터가 DB에 반영된 상태로 유지가 되기 때문에 롤백을 위해 추가합니다.
{: .prompt-info }

그럼 테스트 코드를 추가합니다.

```java
@HelloBootTest
public class JdbcTemplateTest {

    @Autowired
    JdbcTemplate jdbcTemplate;

    @BeforeEach
    void init() {
        jdbcTemplate.execute("create table if not exists hello(name varchar(50) primary key, count int)");
    }

    @Test
    void insertAndQuery() {
        jdbcTemplate.update("insert into hello values(?, ?)", "Toby", 3);
        jdbcTemplate.update("insert into hello values(?, ?)", "Spring", 1);

        Long count = jdbcTemplate.queryForObject("select count(*) from hello", Long.class);
        Assertions.assertThat(count).isEqualTo(2);
    }
}
```

해당 테스트 코드가 통과하는 것을 확인할 수 있습니다.

> `@Transactional`을 확인하고 싶으면 `@Rollback(false)`를 추가하고, DB 데이터를 추가하는 테스트 메소드를 추가하여 실행하면 됩니다.
{: .prompt-info }

