---
layout: post
title: DDD와 Setter, Default Constructor에 대해서
categories: ddd
---

회사에서 진행하고 있는 프로젝트에 DDD(Domain-Driven Design)를 적용하고, 코드리뷰를 진행하면서 몇 분들이 궁금해했던 내용에 대해서 정리를 해두려고한다.

# 1. JPA를 사용하는 Domain class에서 Setter 메소드를 만들지 않아도 되는가?
결론은 만들지 않아도 된다이다. JPA는 필드와 메서드 2가지 방식으로 매핑처리를 할 수 있다.
명시적으로 Domain에 @Access(AccessType.FIELD)을 써주지 않는다면 @Id나 @EmbbededId가 어디(필드 또는 get메소드)에 위치해 있느냐에 따라서 접근 방식을 결정한다. 
그래서 명시적으로 @Access(AccessType.FIELD)를 써주거나, 필드에 @Id 애노테이션만 붙어있으면 Setter 메소드는 필요하지 않는다.

# 2. 기본생성자가 protected로 하면 오류가 발생하지 않는가?
기본생성자가 필요한 이유는 하이버네이트와 같은 JPA프로바이더는 DB에서 데이터를 읽어와 매핑된 객체를 생성할 때 기본생성자를 사용해서 객체를 생성하기 때문에 기본생성자는 꼭 추가해주어야 한다. 
하지만, 기본 생성자를 다른 코드에서 사용하면 값이 없는 온전하지 못한 객체를 만들게 되므로, 다른 코드에서 기본 생성자를 사용하지 못하도록 proteced로 해주었다.

또 하이버네이트는 클래스를 상속한 프록시 객체를 이용해서 지연로딩을 구현하는데, 이 경우에 프록시 클래스에서 상위 클래스의 기본 생성자를 호출할 수 있어야 하므로 
지연 로딩 대상이되는 @Entity, @Embeddable의 기본생성자는 private이 아닌 proteced로 지정해주어야한다.

> 출처 : DDD Start! - 도메인 주도 설계 구현과 핵심 개념 익히기 (지은이 최범균)