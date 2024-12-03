---
title: "Spring JDBC 자동 구성 개발 - 4"
author: bumoo
date: 2024-12-03 23:16:00 +0900
categories: ["토비의 스프링부트"]
tags: [SpringBoot]
---

> 해당 글은 토비의 스프링 부트 강의를 토대로 정리되었습니다.
{: .prompt-info }

생성한 `JdbcTemplate` 빈을 이용해서 데이터 엑세스하는 `Repository`를 생성합니다.

기능은 어떤 이름을 가진 사람이 몇 번 인사를 했는가를 카운팅해서 저장해놓는 기능입니다.

```java
public interface HelloRepository {

    Hello findHello(String name);

    void increaseCount(String name);

    default int countOf(String name) {
        Hello hello = this.findHello(name);
        return hello == null ? 0 : hello.getCount();
    }
}
```

사용할 기능들을 `HelloRepository` 인터페이스에 선언을 해놓을 수 있다. 추가로 `default`를 사용하여 구현도 해놓을 수 있다.

```java
@Repository
public class HelloRepositoryJdbc implements HelloRepository {

    private final JdbcTemplate jdbcTemplate;

    public HelloRepositoryJdbc(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    @Override
    public Hello findHello(String name) {
        try {
            return jdbcTemplate.queryForObject("select * from hello where name = '" + name + "'",
                    (rs, rowNum) -> new Hello(
                            rs.getString("name"), rs.getInt("count")
                    ));
        } catch (EmptyResultDataAccessException e) {
            return null;
        }
    }

    @Override
    public void increaseCount(String name) {
        Hello hello = findHello(name);
        if (hello == null) jdbcTemplate.update("insert into hello values (?, ?)", name, 1);
        else jdbcTemplate.update("update hello set count = ? where name = ?", hello.getCount() + 1, name);
    }
}
```

`findHello`에 `try-catch`는 해당 데이터가 없는 경우에 `EmptyResultDataAccessException`을 던지므로, `null`을 반환하도록 변경한다.

추가로 테스트 코드도 작성해봅니다.

```java
@HelloBootTest
public class HelloRepositoryTest {
    @Autowired
    HelloRepository helloRepository;

    @Autowired
    JdbcTemplate jdbcTemplate;

    @BeforeEach
    void init() {
        jdbcTemplate.execute("create table if not exists hello(name varchar(50) primary key, count int)");
    }

    @Test
    void findHelloFailed() {
        Assertions.assertThat(helloRepository.findHello("Toby")).isNull();
    }

    @Test
    void increaseCount() {
        Assertions.assertThat(helloRepository.countOf("Toby")).isEqualTo(0);

        helloRepository.increaseCount("Toby");
        Assertions.assertThat(helloRepository.countOf("Toby")).isEqualTo(1);

        helloRepository.increaseCount("Toby");
        Assertions.assertThat(helloRepository.countOf("Toby")).isEqualTo(2);
    }
}
```

정상적으로 테스트 코드가 통과하는 것을 알 수 있습니다. 

해당 파트는 `JdbcTemplate`을 이용한 데이터 조작 코드이므로 단순하게 코드만 나열했습니다.
