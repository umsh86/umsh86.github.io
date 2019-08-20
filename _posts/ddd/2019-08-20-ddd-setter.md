---
layout: post
title: 도메인 모델에 set 메소드 사용하지 않기
categories: ddd
---

DDD Start! - 도메인 주도 설계 구현과 핵심 개념 익히기 (지은이 최범균)의 Chapter1에 해당 되는 내용을 다시 한 번 보면서 정리한 내용이다
 
Domain 모델을 생성할 때, 예전(DDD를 적용하지 않았던 시절..)에는 기계적으로 Lombok의 @Getter, @Setter를 사용했었다.
이렇게 도메인 모델에 public set method를 추가하게 되면, 의도하지 않은 곳에서 의도하지 않은 용도로 setter method를 사용하는 경우가 많이 생겼고, 리팩토링시 또는 기능 수정시 문제가되거나 불편한점들이 있었다.

set 메소드를 사용하지 않고 어떻게 처리해야되는지, 어떻게 처리하고 있는지 간단하게 기록해두려고 한다.


## 1. 객체를 생성하는 경우

아래와 같이 set 메서드로 필요한 값을 전달하는 것은 지양하고 있다.
```java
    User user = new User();
    user.setTeam(team);
```

대신에 생성자를 통해서 필요한 데이터들을 모두 받아서 처리해야 한다.
```java
User user = new User(team, username, password);
```

또는 builder 패턴을 사용해서 객체를 생성할 수도 있다.
```java
@Entity
@Table(name = "mt_user")
@EqualsAndHashCode(callSuper = false, of = "id")
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Getter
public class User {
    
    @Id
    @Column(name = "id")
    @GenericGenerator(name = "system-uuid", strategy = "uuid2")
    @GeneratedValue(generator = "system-uuid")
    private String id;
    private String username;
    private String password;
    private String phoneNumber;
    
    @OneToOne
    @JoinColumn(name = "team_id")
    private Team team;
    
    @Builder(builderClassName = "ByCreateBuilder", builderMethodName = "ByCreateBuilder")
    public User(String username, String password, Team team) {
      Assert.notNull(username, "username must not be null"); // 안저한 객체 생성을 위하여 객체 생성시 필수값들은 체크해 준다.
      Assert.notNull(password, "password must not be null");
      Assert.notNull(team, "team must not be null");
      this.username = username;
      this.password = password;
      this.team = team;
    }
    
}
```


```java
 @Test
 @DisplayName("builder 패턴으로 User객체가 정상적으로 생성되어야 한다")
  public void createUser_test() {
    final User user = User.ByCreateBuilder()
        .username("eomdev")
        .password("123123")
        .team(team)
        .build();
    assertThat(user.getUsername()).isEqualTo("eomdev");
    assertThat(user.getPassword()).isEqualTo("123123");
  }
```


## 2. 도메인 모델 내부에서만 사용할 수 있도록 private set 메서드는 허용하고 있다.

도메인 모델에 외부에서는 접근할 수 없도록 private 접근자를 사용하여 set 메서드를 생성하여 내부에서만 사용하는 것까지는 허용하고 있다.
최소한 외부에서 다른 의도로 set 메소드를 사용하는 경우는 막을 수 있으므로.

하지만, 메서드의 이름에 set을  붙이는 것보다는 의미있는 이름을 지어주는 것이 좋다고 생각하고 있다.


## 3. 불변 밸류 타입에는 set 메서드를 통해서 값을 변경하는 것보다는 새로운 밸류 객체를 생성하자

Address 또는 Money와 같은 불변 밸류 타입은 새로운 밸류 객체를 생성하는 것이 불변 타입을 사용해서 얻는 이점인 안전한 코드를 작성할 수 있다는 부분을 챙길 수 있다.
```java
public class Money {
    
    private int alue;
    
    public Money add(Money money) {
        return new Money(this.value + money.value);
    }
    
}
```

## 4. Lombok의 @Setter 사용을 강제적으로 제한할 수도 있다.

현재 새로 진행하는 프로젝트들은 가능하면 Set method를 쓰지 않기 위해서 프로젝트 내에 `lombok.config` 을 생성해서 아래와 같이 설정해서 사용하고, 진행해보고 있다.
```
lombok.setter.flagUsage=ERROR
```


> 참고
> * DDD Start! - 도메인 주도 설계 구현과 핵심 개념 익히기 (지은이 최범균)
> * Builder 기반으로 객체를 안전하게 생성하는 방법 : https://cheese10yun.github.io/spring-builder-pattern/

 