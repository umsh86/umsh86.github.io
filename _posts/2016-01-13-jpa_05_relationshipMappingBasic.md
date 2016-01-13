---
layout: post
title: JPA - 05. 연관관계 매핑 기초
categories: java
---

# 05. 연관관계 매핑 기초

## 5.1 단방향 연관관계

* 다대일(N:1) 단방향 관계
    * 회원과 팀이 존재
    * 회원은 하나의 팀에만 소속
    * 회원과 팀은 다대일 관계

### 5.1.1 순수한 객체 연관관계 

### 5.1.2 테이블 연관관계

### 5.1.3 객체 관계 매핑

```java
@Entity
public class Member {
    
    @Id
    @Column(name = "MEMBER_ID")
    private String id;
    
    private String name;
    
    // 연관관계 매핑
    @ManyToOne
    @JoinColumn(name ="TEAM_ID")
    private Team team;
    
    // 연관관계 설정
    public void setTeam(Team team){
        this.team = team;
    }
    
    //Getter, Setter ...
    
}
```

```java
@Entity
public class Team{
    
    @Id
    @Column(name = "TEAM_ID")
    private String id;
    private String name;
    
    //Getter, Setter ...
}
```

* @ManyToOne : 다대일 관계라는 매핑 정보
* @JoinColumn(name="TEAM_ID") : 외래 키를 매핑할 때 사용. 생략가능

### 5.1.4 @JoinColumn

### 5.1.5 @ManyToOne


|속성          | 기능                                     |기본값                                                 |
|-------------|---------------------------------------- |-----------------------------------------------------|
|optional     | false로 설정시 연관된 엔티티가 항상 있어야 한다   |true                                                 |
|fetch        | 글로벌 페치 전략을 설정. 자세한 내용은 8장에 나옴. |@ManyToOne=FetchType.EAGER, @OneToMay=FetchType.LAZY|
|cascade      | 영속성 전이 기능을 사용. 자세한 내용은 8장에 나옴. ||
|targetEntity | 연관된 엔티티의 타입 정보를 설정(거의 사용하지 않음)||




## 5.2 연관관계 사용

### 5.2.1 저장

```java
public void testSave(){
    
    // 팀1 저장
    Team team1 = new Team("team1", "팀1");
    em.persist(team1);
    
    // 회원1 저장
    Member member1 = new Member("member1", "회원1");
    member1.setTeam(team1); // 연관관계 설정 member1 -> team1
    em.persist(member1);
    
    // 회원2 저장
    Member member2 = new Member("member2", "회원2");
    member2.setTeam(team1); // 연관관계 설정 member2 -> team1
    em.persist(member2);
}
```

### 5.2.1 조회

* 객체 그래프 탐색(객체 연관관계를 사용한 조회)

```java
Member member = em.find(Member.class, "member1");
Team team = member.getTeam();   // 객체 그래프 탐색을 이용한 조회
```

더 자세한 내용은 8장에서 다룸

* 객체지향 쿼리 JPQL 사용 (생략)


### 5.2.3 수정

```java
private static void updateRealation(EntityManager em){
    
    // 새로운 팀2
    Team team2 = new Team("team2", "팀2");
    em.persist(team2);
    
    // 회원1에 새로운 팀2 설정
    Member member1 = em.find(Member.class, "member1");
    member1.setTeam(team2);
}
```

em.update() 같은 메소드가 존재하지 않음. 참조하는 대상(Team team)만 변경하면 JPA가 자동으로 처리함.

### 5.2.4 연관관계 제거

```java
Member member1 = em.find(Member.class, "member1");
member1.setTeam(null);  // 연관관계 제거
```

### 5.2.5 연관된 엔티티 삭제

```java
// 기존에 있던 연관관계를 모두 제거
member1.setTeam(null);
member2.setTeam(null);

// 팀1 삭제
em.remove(team1); 
```

## 5.3 양방향 연관관계

팀 -> 회원으로 접근 하는 것을 추가하여 양방향으로 접근할 수 있도록 매핑하기

### 5.3.1 양방향 연관관계 매핑

```java
public class Team{

    @Id
    @Column(name = "TEAM_ID")
    private String id;
    private String name;
    
    // 추가 
    @OneToMany(mappedBy = "team")
    private List<Member> members = new ArrayList<Member>();
    
    // Getter, Setter ...
    
}
```

* 팀 -> 회원 : 일대다 관계.
    * Team Entity에 List<Member> members를 추가
    * @OneToMany 매핑 정보를 사용
    * mappedBy 속성을 사용하여 반대(N)쪽 매핑의 필드 이름을 값으로 전달(Member.team 이므로 'team'을 전달)
    

### 5.3.2 일대다 컬렉션 조회

```java
public void biDirection(){

    Team team = em.find(Team.class, "team1");
    List<Member> members = team.getMembers(); // 팀 -> 회원 으로 객체 그래프 탐색
    
}
```

## 5.4 연관관계의 주인

* 테이블은 외래 키 하나로 두 테이블의 연관관계를 관리함
* 엔티티를 양방향 연관관계로 설정하면 객체의 참조는 둘인데 외래 키는 하나여서 둘 사이에 차이가 발생한다.
(둘 중 하나를 정해서 테이블의 외래 키를 관리해야 한다) : 외래 키를 관리하는 것을 **연관관계의 주인(mappedBy)**이라 한다.

### 5.4.1 양방향 매핑의 규칙 : 연관관계의 주인

**연관관계의 주인만이 데이터베이스 연관관계와 매핑되고 외래 키를 관리(등록, 수정, 삭제)할 수 있다. 반면에 주인이 아닌 쪽은 읽기만 할 수 있다.**

* 주인은 mappedBy 속성을 사용하지 않는다.
* 주인이 아니면 mappedBy 속성을 사용해서 속성의 값(team)으로 연관관계의 주인을 지정해야 한다.

### 5.4.2 연관관계의 주인은 외래 키가 있는 곳

외래 키는 Member.team이 가지고 있으므로 연관관계의 주인은 Member.team이 된다.

```java
@ManyToOne
@JoinColumn(name ="TEAM_ID")
private Team team;
```

연관관계의 주인이 아닌 곳(Team.members)에 mappedBy="team" 속성을 사용한다.

```java
@OneToMay(mappedBy = "team") // Member.team
List<Member> members = new ArrayList<Member>();
```

### 5.5 양방향 연관관계 저장

단방향 연관관계에서 저장하는 것과 똑같다.

```java
public void testSave(){
    
    // 팀1 저장
    Team team1 = new Team("team1", "팀1");
    em.persist(team1);
    
    // 회원1 저장
    Member member1 = new Member("member1", "회원1");
    member1.setTeam(team1); // 연관관계 설정 member1 -> team1
    em.persist(member1);
    
    // 회원2 저장
    Member member2 = new Member("member2", "회원2");
    member2.setTeam(team1); // 연관관계 설정 member2 -> team1
    em.persist(member2);
}
```

양방향 연관관계에서 연관관계의 주인이 외래 키를 관리하므로, 주인이 아닌 곳에서 값을 설정하지 않아도 된다.

## 5.6 양방향 연관관계의 주의점

연관관계 주인인 곳에서 연관관계 설정을 하지 않고, 연관관계의 주인이 아닌 곳에서 연관관계 설정을 하는 경우 

```java
// 회원1 저장 : 연관관계의 주인인 곳에서 연관관계 설정을 하지 않았다.
Member member1 = new Member("member1", "회원1");
em.persist(member1);

// 주인이 아닌 곳만 연관관계 설정
Team team1 = new Team("team1", "팀1");
team1.getMembers().add(member1);
team1.getMembers().add(member2);
em.persist(team1);

```

### 5.6.1 순수한 객체까지 고려한 양방향 연관관계

**객체 관점에서 양쪽 방향에 모두 값을 입력해주는 것이 가장 안전하다.**

```java
// 팀1
Team team1 = new Team("team1", "팀1");
Member member1 = new Member("member1", "회원1");
Member member2 = new Member("member2", "회원2");

member1.setTeam(team1); // 연관관계 설정 member1 -> team1
member2.setTeam(team1); // 연관관계 설정 member2 -> team1

// team1에 해당되는 member 조회
List<Member> members = team1.getMembers();
System.out.println("members.size = " + members.size());  // 결과 : members.size = 0

```

양방향은 양쪽다 관계를 설정해야 하므로 팀 -> 회원도 설정해야 한다.

```java
team1.getMembers().add(member1); // 팀 -> 회원
```

### 5.6.2 연관관계 편의 메소드

양방향 연관관계에서 양쪽 다 신경써서 각각 호출해야되는 부분을 하나인 것처럼 사용하는 것이 안전하다.

```java
// 각각 신경써야하는 두 개의 코드
member.setTeam(team);
team.getMembers().add(member);
```

```java
// 하나인 것처럼 사용하는 Member 클래스의 setTeam() 메소드 코드 리팩토링
public void setTeam(Team team){
    this.team = team;
    team.getMembers().add(this);
}
```

연관관계 편의 메소드(양방향 관계를 한 번에 설정)를 사용한 코드

```java
Team team1 = new Team("team1", "팀1");
em.persist(team1);

Member member1 = new Member("member1", "회원1");
member1.setTeam(team1);
em.persist(member1);

```

### 5.6.3 연관관계 편의 메소드 작성시 주의사항

setTeam() 메소드에 버그가 존재함.

```java
member1.setTeam(team1);
member1.setTeam(team2);

Member findMember = team1.getMember(); // member1이 조회
```

team2로 변경할 때 team1 -> member1 관계를 제거해주는 작업이 추가로 필요하다.

```java
public void setTeam(Team team){

    // 기존 팀과 관계를 제거
    if(this.team != null){
        this.team.getMembers().remove(this);
    }
    
    this.team = team;
    team.getMembers().add(this);

}
```

## 5.7 정리

> 양방향 매핑 시 무한 루프에 빠지지 않게 조심해야함(Member.toString()에서 getTeam()을 호출하고 Team.toString()에서 getMember()를 호출하는 경우).
주로 엔티티를 JSON으로 변환하거나 Lombok 라이브러리를 사용할 때도 자주 발생함.
