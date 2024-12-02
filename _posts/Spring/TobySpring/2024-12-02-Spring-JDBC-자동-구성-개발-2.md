---
title: "Spring JDBC 자동 구성 개발 - 2"
author: bumoo
date: 2024-12-02 22:06:00 +0900
categories: ["토비의 스프링부트"]
tags: [SpringBoot]
---

> 해당 글은 토비의 스프링 부트 강의를 토대로 정리되었습니다.
{: .prompt-info }

`DataSourceConfig`는 특정 조건에 따라 빈으로 등록이 되도록 해야합니다. 

`@ConditionalMyOnClass`에 클래스를 넣기 위해서 `JdbcOerations`를 넣어야합니다.

`build.gradle`에 `implementation 'org.springframework:spring-jdbc'`를 추가합니다.

`DataSource`에는 DB 연결과 관련된 정보가 여러가지 있습니다. ex) DriverClass, UserName, Password 등

해당 정보들을 프로퍼티값을 가져와 사용하도록 추가하겠습니다.

```java
@MyAutoConfiguration
@ConditionalMyOnClass("org.springframework.jdbc.core.JdbcOperations")
@EnableMyConfigurationProperties(MyDataSourceProperties.class)
public class DataSourceConfig {

    @Bean
    DataSource dataSource(MyDataSourceProperties properties) throws ClassNotFoundException {
        SimpleDriverDataSource dataSource = new SimpleDriverDataSource();

        dataSource.setDriverClass((Class<? extends Driver>) Class.forName(properties.getDriverClassName()));
        dataSource.setUrl(properties.getUrl());
        dataSource.setUsername(properties.getUsername());
        dataSource.setPassword(properties.getPassword());

        return dataSource;
    }
}

@MyConfigurationProperties(prefix = "data")
public class MyDataSourceProperties {

    private String driverClassName;
    private String url;
    private String username;
    private String password;

    // getter, setter
}
```

이전의 외부 설정 방식과 모두 동일합니다.

`application.properties`에 작성된 정보를 바탕으로 `MyDataSourceProperties`를 생성해 주입을 합니다.

`DataSourceConfig`는 특정 클래스가 있어야 빈으로 등록이 되며, 외부의 프로퍼티 값을 사용하므로 `MyDataSourceProperties`를 주입하도록 했습니다.

`MyDataSourceProperties`는 `prefix`가 `data`인 프로퍼티값을 가져와 해당 클래스를 생성하고, 해당 프로퍼티값을 `dataSource`에 주입하게 됩니다.

그럼 추가로 `application.properties`에 프로퍼티값을 추가합니다.

```text
data.driver-class-name=org.h2.Driver
data.url=jdbc:h2:mem:
data.username=sa
data.password=
```

h2 DB 사용하므로 `build.gradle`에 `implementation 'com.h2database:h2:2.1.214'`를 추가합니다.

정상적인 연결이 되었는지 확인을 위한 테스트 코드를 추가합니다.

```java
@ExtendWith(SpringExtension.class)
@ContextConfiguration(classes = HellobootApplication.class)
@TestPropertySource("classpath:/application.properties")
public class DataSourceTest {

    @Autowired
    DataSource dataSource;

    @Test
    void connect() throws SQLException {
        Connection connection = dataSource.getConnection();
        connection.close();
    }
}
```

`@ExtendWith(SpringExtension.class)`는 Spring Context를 이용하는 Spring Container 테스트가 가능하다.

`@ContextConfiguration`를 통해 로딩할 빈들의 정보를 넣을 수 있는데, 단순하게 넣으면 모든 빈들을 다 불러오도록 할 수 있다.

`@TestPropertySource("classpath:/application.properties")`를 통해 어떤 프로퍼티를 사용할지 지정할 수 있다.

정상적으로 테스트가 통과하는 것을 알 수 있다.

추가로 `DataSource`로 `Hikari`를 추가합니다.

`Hikari` 사용하므로 `build.gradle`에 `implementation 'hikari-cp:hikari-cp:3.0.1'`를 추가합니다.

> 해당 라이브러리가 import 실패하는 경우엔 `implementation 'com.zaxxer:HikariCP'`로 변경합니다.
{: .prompt-info }

`DataSourceConfig`에 해당 `@Bean`을 추가로 등록합니다.

```java
@Bean
DataSource hikariDataSource(MyDataSourceProperties properties) {
    HikariDataSource dataSource = new HikariDataSource();

    dataSource.setDriverClassName(properties.getDriverClassName());
    dataSource.setJdbcUrl(properties.getUrl());
    dataSource.setUsername(properties.getUsername());
    dataSource.setPassword(properties.getPassword());

    return dataSource;
}
```

이렇게 되면 2개의 빈이 모두 생깁니다. 

조건을 걸어서 `Hikari 관련 클래스`가 있는 경우엔 `hikariDataSource`가 사용되고, 없으면 `dataSource` 빈이 사용되도록 수정합니다.

```java
@MyAutoConfiguration
@ConditionalMyOnClass("org.springframework.jdbc.core.JdbcOperations")
@EnableMyConfigurationProperties(MyDataSourceProperties.class)
public class DataSourceConfig {

    @Bean
    @ConditionalMyOnClass("com.zaxxer.hikari.HikariDataSource")
    @ConditionalOnMissingBean
    DataSource hikariDataSource(MyDataSourceProperties properties) {
        /* ... */
    }

    @Bean
    @ConditionalOnMissingBean
    DataSource dataSource(MyDataSourceProperties properties) throws ClassNotFoundException {
        /* ... */
    }
}
```

`@ConditionalMyOnClass`를 사용해 히카리 클래스를 점검하고 있는 경우 `hikariDataSource` 빈을 생성하도록 했다.

`@ConditionalOnMissingBean`으로 `DataSource`의 빈이 있는 경우 생성되지 않도록 했다. 

빈 메소드는 정의된 순서대로 생성하므로 `hikariDataSource`를 먼저 읽히도록 순서를 변경했다.

> 빈 메소드는 정의된 순서대로 생성하므로 dataSource, hikariDataSource 메소드 순서인 경우 정상작동 하지 않는다.
> <br/> 순서를 변경해야 정상작동한다.
{: .prompt-tip }


