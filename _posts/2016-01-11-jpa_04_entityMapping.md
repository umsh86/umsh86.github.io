---
layout: post
title: JPA - 04. 엔티티 매핑
categories: java
---

김영한님의 [자바 ORM 표준 JPA 프로그래밍](http://www.yes24.co.kr/24/goods/19040233)를 다시 한 번 처음부터 읽어보면서,
나중에 찾아보기 위한 목적으로 간단하게 정리하려고 한다. 
 
[자바 ORM 표준 JPA 프로그래밍](http://www.yes24.co.kr/24/goods/19040233)은 꼭 한 번 정독해보기를 권한다.

(내가 너무 책에 있는 내용을 그대로 쓰는게 문제가 되지 않을까.. 하는 걱정도 되기는 한다. 만약 문제가 된다는 연락을 받으면 언제든지 관련 내용은 삭제될 수 있다. 저작권이나 라이센스는 중요할 뿐만 아니라, 좋은 책을 써준 분께 대한 최소한의 예의라고 생각하기 때문이다.)

# 04. 엔티티 매핑

## 4.1 @Entity

* 기본 생성자는 필수(파라미터가 없는 public 또는 protected 생성자) : 자바는 생성자가 하나도 없으면 기본 생성자를 자동으로 만듬
* 저장할 필드에 final 은 불가능

## 4.2 @Table

## 4.3 다양한 매핑 사용

```java
// enum타입
@Enumerated(EnumType.String)
private RoleType roletype;

public enum RoleType{
    ADMIN, USER
}
```

Java8의 날짜 API를 사용하는 방법은 [JPA와 LocalDate, LocalDateTime 사용하기](http://blog.eomdev.com/java/2016/01/04/jpa_with_java8.html)를 확인한다.

```java
// 날짜타입
@Temporal(TemporalType.TIMESTAMP)
private Date createDate;

//Java8
@Column(name="date_time", nullable = false)
private LocalDateTime dateTime;
```





```java
// 필드길이의 제한이 없는 CLOB
@Lob
private String description;
```

## 4.4 데이터베이스 스키마 자동 생성

hibernate.hbm2ddl.auto 속성

|옵션       | 설명           |
|----------|---------------|
|create     | DROP + CREATE|
|create-drop| DROP+CREATE+DROP|
|~~update~~     | ~~테이블과 엔티티 매핑정보를 비교해서 변경 사항만 수정~~|
|~~validate~~   | ~~테이블과 엔티티 매핑정보를 비교해서 차이가 있으면 경고를 남기고 애플리케이션을 실행하지 않는다.(DDL을 수정하지 않는다.)~~|
|none       | 자동생성기능을 사용하지 않으려면 hibernate.hbm2ddl.auto 속성값 자체를 삭제하거나, 유효하지 않은 값(none)을 주면 된다.|

>
* 개발 환경에 따른 추천 전략
    * 개발초기 : create 또는 ~~update~~
    * CI서버 또는 초기화 상태로 자동화된 테스트를 진행하는 개발자 환경 : create 또는 create-drop
    * ~~테스트서버 : update 또는 validate~~
    * 스테이징과 운영서버 : ~~validate~~ 또는 none


*&nbsp;*
>
JPA 2.1 부터는 자동생성기능을 표준으로 지원한다. 하지만 hibernate.hbm2ddl.auto 속성이 지원하는 update, validate옵션을 지원하지 않는다.

> * 지원 옵션 : none, create, drop-and-create, drop

*&nbsp;*


>
* 단어와 단어 구분시
    * 자바 : 카멜 표기법
    * DB : 언더스코어(`_`) 표기법
    
> jpaProperties.put("hibernate.ejb.naming_strategy", "org.hibernate.cfg.ImprovedNamingStrategy");
> 옵션을 사용하여, 이름 매핑 전략을 변경한다.


```java
@Configuration
@Slf4j
public class DataConfig {

    @Autowired
    private DatabaseProperties databaseProperties;

    @Bean
    public DataSource dataSoure(){
        DriverManagerDataSource dataSource = new DriverManagerDataSource();
        dataSource.setDriverClassName(databaseProperties.getDriver());
        dataSource.setUrl(databaseProperties.getUrl());
        dataSource.setUsername(databaseProperties.getUsername());
        dataSource.setPassword(databaseProperties.getPassword());
        return dataSource;
    }

    @Bean
    public JpaTransactionManager transactionManager(){
        return new JpaTransactionManager();
    }

    @Bean
    public LocalContainerEntityManagerFactoryBean entityManagerFactory(){

        LocalContainerEntityManagerFactoryBean factoryBean = new LocalContainerEntityManagerFactoryBean();
        factoryBean.setDataSource(dataSoure());
        factoryBean.setPackagesToScan("com.eomdev.jobplan"); // @Entity 탐색위치
        factoryBean.setJpaVendorAdapter(new HibernateJpaVendorAdapter());

        // 상세설정
        Properties jpaProperties = new Properties();
        jpaProperties.put(AvailableSettings.SHOW_SQL, true);    // SQL 보기
        jpaProperties.put(AvailableSettings.FORMAT_SQL, true);  // SQL 정렬해서 보기
        jpaProperties.put(AvailableSettings.USE_SQL_COMMENTS, true);    // SQL 코멘트 보기
        jpaProperties.put(AvailableSettings.HBM2DDL_AUTO, databaseProperties.getHbm2ddlAuto());    // DDL 자동생성
        jpaProperties.put(AvailableSettings.DIALECT, databaseProperties.getDialect());    // 방언 설정
        jpaProperties.put(AvailableSettings.USE_NEW_ID_GENERATOR_MAPPINGS, true);   // 새 버젼의 ID생성 옵션
        jpaProperties.put("hibernate.ejb.naming_strategy", "org.hibernate.cfg.ImprovedNamingStrategy");
        factoryBean.setJpaProperties(jpaProperties);

        return factoryBean;

    }
```

## 4.5 DDL 생성 기능

* not null, length 사이즈 10 제약조건 추가

```java
@Column(name="NAME", nullable=false, length=10)
private String username;
```

* 유니크 제약조건

```java
@Entity(name="Member")  
@Table(name="MEMBER", uniqueConstraints = { @UniqueConstraint( //추가
name = "NAME_AGE_UNIQUE",
columnNames {"NAME", "AGE"} )})

public class Member {
 @Id  
 @Column(name = "id")
 private String id;

 @Column(name = "name")
 private String username;

 private Integer age;
 
 ...
 
```

```sql
ALTER TABLE MEMBER ADD CONSTRAINT NAME_AGE_UNIQUE UNQIUE (NAME, AGE)
```

## 4.6 기본 키 매핑

```java
@Entity
public class Member{
    
    @Id
    @Column(name = "ID")
    private String id;
    
}
```

* @Id만 사용하는 경우 : 기본 키를 애플리케이션에서 직접 할당
* @Id에 @GeneratedValue를 추가하고 원하는 키 생성 전략을 선택 : 자동 생성 전략

* 기본키 생성 전략
    * 직접할당 : 기본 키를 애플리케이션에서 직접 할당(@Id만 사용하면 된다)
    * 자동 생성 : 대리 키 방식 사용(@Id에 @GeneratedValue를 추가하고 원하는 키 생성 전략을 선택하면 된다)
        * IDENTITY : 기본 키 생성을 데이터베이스에 위임
        * SEQUENCE : 데이터베이스 시퀀스를 사용해서 기본 키를 할당
        * TABLE : 키 생성 테이블을 사용

>
기존의 하이버네이트 시스템을 유지보수하는것이 아니라면, 더 효과적이고 JPA 규격에 맞는 새로운 키 생성 전략을 사용하기 위해서 아래 옵션을 true로 설정해주자.

jpaProperties.put(AvailableSettings.USE_NEW_ID_GENERATOR_MAPPINGS, true);   


### 4.6.1 기본 키 직접 할당 전략

엔티티로 저장하기 전에 애플리케이션에서 기본 키를 직접 할당하는 방법

```java
Board board = new Board();
board.setId("id1");
em.persist(board);
```

### 4.6.2 IDENTITY 전략

Mysql, PostgreSQL, SQL Server, DB2에서 사용.
AUTO_INCREMENT를 사용하는 전략(ID INT NOT NULL AUTO_INCREMENT PRIMARY KEY).

```java
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;
```

>
IDENTITY 식별자 생성 전략은 엔티티를 데이터베이스에 저장해야 식별자를 구할 수 있으므로 em.persist()를 호출하는 즉시 INSERT SQL이 데이터베이스에 전달된다.
그러므로 IDENTITY 생성 전략을 사용하는 INSERT SQL은 트랜잭션을 지원하는 쓰기 지연이 동작하지 않는다.


### 4.6.3 SEQUENCE 전략

시퀀스를 지원하는 Oracle, PostgreSQL, DB2, H2 데이터베이스에서 사용할 수 있다.

```sql
CREATE SEQUENCE BOARD_SEQ START WITH 1 INCREMENT BY 1;
CREATE SEQUENCE [sequenceName] START WITH [initialValue] INCREMENT BY [allocationSize];
```

```java
@Entity
@SequenceGenerator(
    name = "BOARD SEQ GENERATOR", 
    sequenceName = "BOARD_SEQ", //매핑할데이터베이스 시퀀스 이름
    initialValue = 1, allocationSize = 1)
public class Board (
    @Id  
    @GeneratedValue(strategy = GenerationType.SEQUENCE,
                generator = "BOARD_SEQ_GENERATOR")
    private Long id;
    ...
```

>
SequenceGenerator.allocationSize의 기본 값은 50이다.


### 4.6.4 TABLE 전략

키 생성 전용 테이블을 하나 만들고 이름과 값으로 사용할 컬럼을 만들어 데이터베이스의 시퀀스를 흉내내는 전략

DDL

```sql
CREATE TABLE MY_SEQUENCE{
 SEQUENCE_NAME VARCHAR(255) NOT NULL,
 NEXT_VAL BIGINT,
 PRIMARY KEY (SEQUENCE_NAME)
}
```

TABLE 전략 매핑 코드

```java
@Entity
@TableGenerator(
    name = "BOARD_SEQ_GENERATOR",
    table = "MY_SEQUENCES",
    pkColumnValue = "BOARD_SEQ",
    allocationSize = 1
)
public class Board{
    @Id
    @GeneratedValue(strategy = GenerationType.TABLE,
                    generator = "BOARD_SEQ_GENERATOR")
    private Long id;
    ...
}
```

### 4.6.5 AUTO 전략

GenerationType.AUTO는 선택한 데이터베이스의 방언에 따라 IDENTITY, SEQUENCE, TABLE 전략 중 하나를 자동으로 선택한다.

@GeneratedValue(stratey = 의 기본값은 AUTO이다.

### 4.6.6 기본 키 매핑 정리

테이블의 기본 키 전략 (**대리키 사용을 권장한다**)

 * 자연키
    * 비즈니스에 의미가 있는 키
    * 예 : 주민등록번호, 이메일, 전화번호
 * 대리키(대체키)
    * 비즈니스와 관련 없는 임의로 만들어진 키
    * 예 : 오라클의 시퀀스, auto_increment, 키 생성 테이블
    

## 4.7 필드와 컬럼 매핑 : 레퍼런스

### 4.7.1 @Column

*@Column 생략*

```java
int data1; // @Column 생략, 자바 기본 타입 -> 기본 타입에는 null을 입력할 수 없으므로 not null이다. JPA가 DDL 생성기능을 사용할 때 자동으로 not null 제약조건을 추가한다.
data1 interger not null // 생성된 DDL

Integer data2; // @Column 생략, 객체 타입 -> null 가능
data2 integer // 생성된 DDL

@Column
int data3; // @Column 사용시 nullable=true 가 기본 값이므로, 자바 기본 타입에 @Column을 사용하는 경우 nullable=false로 지정하는 것이 안전하다. 
data3 integer; // 생성되는 DDL
```

### 4.7.2 @Enumberated

EnumType.ORDINAL : enum 순서를 데이터베이스에 저장(0부터)
EnumType.STRING : enum 이름을 데이터베이스에 저장(권장한다)

```java
public enum RoleType{
    ADMIN, USER
}
```

### 4.7.3 @Temporal

Java8의 날짜 API를 사용하는 방법은 [JPA와 LocalDate, LocalDateTime 사용하기](http://blog.eomdev.com/java/2016/01/04/jpa_with_java8.html)를 확인한다.

날짜 타입(java.util.Date, java.util.Calendar)를 매핑할 때 사용

```java
@Temporal(TemporalType.DATE)
private Date date; // 날짜 2015-01-01

@Temporal(TemporalType.TIME)
private Date time; // 시간 11:11:11

@Temporal(TemporalType.TIMESTAMP)
private Date timestamp; // 날짜와 시간 2015-01-01 11:11:11

// 생성되는 DDL
date date,
time time,
timestamp timestamp,
```

@Temporal 생략시 자바의 Date와 가장 유사한 timestamp로 정의됨

* Mysql : datetime
* Oracle, H2, PostgreSQL : timestamp

### 4.7.4 @Lob

데이터베이스 CLOB(매핑하는 필드 타입이 문자인 경우 : String, char[], java.sql.CLOB), BLOB(나머지 : byte[], java.sql.BLOB) 타입과 매핑 됨

### 4.7.5 @Transient

필드를 매핑하지 않을 때 사용한다.(DB에 저장하지 않고 조회도 하지 않는다)

```java
@Transient
private String temp;
```

### 4.7.6 @Access

JPA가 필드에 접근하는 방식

* 필드 접근 : AccessType.FILED. private 이어도 직접 접근 할 수 있다.
* 프로퍼티 접근 : AccessType.PROPERTY. getter를 이용해서 접근한다.

@Access를 지정하지 않으면 @Id의 위치를 기준으로 접근 방식이 설정된다.

```java
@Entity
@Access(AccessType.FILED)
public class Member{
    @Id
    private Long id;
}
```

필드, 프로퍼티 접근을 함께 사용

```java
@Entity
public class Member{
    @Id //기본은 필드 접근 방식
    private Long id;
    
    @Transient  // DB에 저장도 조회도 하지 않는다
    private String firstName;
    
    @Transient
    private String lastName;
    
    @Access(AccessType.PROPERTY) // FULLNAME 컬럼에 firstName+lastName의 결과가 저장됨
    public String getFullName(){
        return firstName + lastName;
    }
}
```
























