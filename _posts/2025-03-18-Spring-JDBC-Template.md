---
title: JDBC API 간편 사용을 위한 JdbcTemplate
description: >-
  Spring의 JdbcTemplate을 활용하여 JDBC API의 반복적인 코드와 복잡한 예외 처리를 간편하게 해결하는 방법
author: JWHONG
date: 2025-03-18 15:00:00 +0900
categories: [Spring , Spring Boot , JDBC Template, Database]
tags: [Spring , Spring Boot , JDBC Template, Database]
pin: true
media_subpath: "/posts/20250318"
---

## JDBC API의 문제점과 Spring의 해결책

JDBC(Java Database Connectivity)는 데이터베이스와 연결하여 SQL을 실행하는 표준 API입니다.  
하지만 JDBC를 직접 사용할 경우 다음과 같은 문제점이 있습니다.

1. **반복적인 코드** : **Connection, PreparedStatement, ResultSet** 등의 객체를 매번 생성하고 닫아야 합니다.
2. **예외 처리의 번거로움** : **SQLException**을 매번 처리해야 합니다.
3. **트랜잭션 관리의 복잡성** : 트랜잭션을 직접 관리해야 하므로 코드가 복잡해집니다.

Spring에서는 이러한 문제를 해결하기 위해 **JdbcTemplate**을 제공합니다.

---

## JdbcTemplate을 사용한 간편한 JDBC 처리

JdbcTemplate은 JDBC API의 3개 주요 인터페이스(**Connection, PreparedStatement, ResultSet**)를 내부적으로 처리하여 개발자가 더 간결한 코드로 SQL을 실행할 수 있도록 도와줍니다.


**JdbcTemplate을 사용한 데이터 조회 예제**

```java
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.RowMapper;
import org.springframework.stereotype.Repository;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.time.ZoneId;
import java.util.Optional;
import java.util.List;

@Repository
public class MemberJdbcTemplateDao {
    private final JdbcTemplate jdbcTemplate;

    public MemberJdbcTemplateDao(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    public Optional<Member> findById(int id) {
        String sql = "SELECT * FROM member WHERE id = ?";
        return jdbcTemplate.query(sql, new MemberRowMapper(), id)
                           .stream()
                           .findFirst();
    }

    public List<Member> findAll() {
        String sql = "SELECT * FROM member";
        return jdbcTemplate.query(sql, new MemberRowMapper());
    }

    public Member save(String name, Integer age, String job, String email) {
        String sql = "INSERT INTO member (name, age, job, email, created_at) VALUES (?, ?, ?, ?, NOW())";
        jdbcTemplate.update(sql, name, age, job, email);

        Integer createdMemberId = jdbcTemplate.queryForObject("SELECT LAST_INSERT_ID()", Integer.class);
        return findById(createdMemberId).orElseThrow(() -> new RuntimeException("회원 생성 실패"));
    }
}

class MemberRowMapper implements RowMapper<Member> {
    @Override
    public Member mapRow(ResultSet rs, int rowNum) throws SQLException {
        return new Member(
                rs.getInt("id"),
                rs.getString("name"),
                rs.getInt("age"),
                rs.getString("job"),
                rs.getString("email"),
                rs.getTimestamp("created_at")
                  .toInstant()
                  .atZone(ZoneId.systemDefault())
                  .toLocalDateTime()
        );
    }
}
```

- **SQL 실행** : **jdbcTemplate.query()**를 사용해 데이터를 조회합니다.
- **결과 매핑** : **MemberRowMapper**를 사용하여 **ResultSet** 데이터를 **Member** 객체로 변환합니다.
- **간결한 코드** : **Connection, PreparedStatement, ResultSet**을 직접 관리하지 않아도 됩니다.

---

## 구조적인 반복 코드의 해결

기존의 순수 JDBC API를 사용할 경우 다음과 같은 반복적인 코드가 필요합니다.

```java
public Member findById(Integer id) throws SQLException {
        Connection connection = null;
        PreparedStatement statement = null;
        ResultSet resultSet = null;

        try {
            connection = dataSource.getConnection();
            statement = connection.prepareStatement("SELECT * FROM member WHERE id = ?");
            statement.setInt(1, id);
            resultSet = statement.executeQuery();

            if (resultSet.next()) {
                return new Member(
                        resultSet.getInt("id"),
                        resultSet.getString("name"),
                        resultSet.getInt("age"),
                        resultSet.getString("job"),
                        resultSet.getString("email"),
                        resultSet.getTimestamp("created_at")
                                .toInstant()
                                .atZone(ZoneId.systemDefault())
                                .toLocalDateTime()
                );
            }
            throw new ResponseStatusException(HttpStatus.NOT_FOUND, "유저 정보가 존재하지 않습니다 - id : " + id);
        } catch (SQLException e) {
            throw new ResponseStatusException(HttpStatus.INTERNAL_SERVER_ERROR, "자원에 대한 접근에 문제가 있습니다.");
        } finally {
            if (resultSet != null) resultSet.close();   // 1
            if (statement != null) statement.close();   // 2
            if (connection != null) connection.close(); // 3
        }
    }
```

- **Try-with-resources** 사용으로 자원 관리는 가능하지만 여전히 코드가 장황합니다.
- **SQL 실행 및 매핑 과정이 복잡합니다**.
- **예외 처리**가 필요합니다.

---

## 결론 

Spring의 **JdbcTemplate**을 사용하면 JDBC API의 복잡한 과정을 단순화 할 수 있습니다.

|**기능**|**순수 JDBC**|**JdbcTemplate**|
|:---:|:---:|:---:|
|커넥션 관리|수동 관리|자동 관리|
|예외 처리|직접 처리|Spring이 처리|
|SQL 실행|`PreparedStatement` 수동 생성|간단한 메소드 호출|
|결과 매핑|`ResultSet` 직접 매핑|`RowMapper` 활용|