---
layout: post
title: FEST(Fixtures for Easy Software Testing) Assert
categories: java
---

Intellij IDE를 쓰기 이전(Eclipse 나 STS IDE를 사용하던 시절)에 JUnit이나 TDD에 대해 관심을 가지기 시작한 적이 있었다.
결과 확인을 위해서 Matcher, Hamcrest 라이브러리 등을 사용했었는데, 이것들을 사용하면서 매번 불편하게 느끼는 것이 있었다.
바로 static 클래스의 메소드(예를 들면 Matcher.is)를 사용하기 위해서 static import로 지정을 해야하는데
IDE가 모든 static import를 알고 있을 수는 없기 때문에 사용자가 직접 static import를 입력해줘야 한다는 점이었다.
(최신 Eclipse 나 STS를 사용해보지 않아서 현재는 또 어떻게 변경되었는지 잘 알지 못하지만..)
물론 약간의 불편함을 감수하면 Eclipse나 STS의 Preferences -> Favorties에 statc import문을 추가하여 자동으로 추가될 수 있도록 설정할 수 있고,
IntelliJ를 사용하는 요즘은 static import를 자동으로 찾아주긴 하지만 정확한 클래스의 이름을 알고 있지 않으면 여전히 조금은 불편하다.

그러던 중에 우연히 FEST(Fixtures for Easy Software Testing) Assert에 대해서 알게 되었다.
쉬운 소프트웨어 테스팅을 위한 픽스쳐들이라는 의미이고, 사용해보니 많은 수의 static import를 해야하던 불편함이 줄어들게 되었다.

아래는 Java8에 JSR-310 표준 명세로 날짜와 시간에 대해 새로 추가된 API를 사용하여 2015년 12월 31일이 목요일인지를 확인하는 단위테스트이다.
IDE는 IntelliJ를 사용했고, 우선은 기존에 하던대로 Assert와 Hamcrest를 사용하여 두 개의 static import가 생긴 것을 볼 수 있다.


```java
package com.eomdev.study;

import org.junit.Test;
import java.time.DayOfWeek;
import java.time.LocalDate;

import static org.hamcrest.Matchers.is;
import static org.junit.Assert.assertThat;

public class HamcrestTest {
    
    @Test
    public void testHamcrest() throws Exception {
        // given : 테스트 상황을 설정
        LocalDate today = LocalDate.of(2015, 12, 31);

        // when : 테스트 대상을 실행
        DayOfWeek dayOfWeek = today.getDayOfWeek();

        // then : 결과를 검증
        assertThat(dayOfWeek, is(DayOfWeek.THURSDAY));

        
    }
    
}
```

그리고 아래는 Fest Assert를 사용한 경우이다.

```java
package com.eomdev.study;

import org.junit.Test;
import java.time.DayOfWeek;
import java.time.LocalDate;

import static org.fest.assertions.api.Assertions.assertThat;


public class FestTest {

    @Test
    public void testFest() throws Exception {
        // given : 테스트 상황을 설정
        LocalDate today = LocalDate.of(2015, 12, 31);

        // when : 테스트 대상을 실행
        DayOfWeek dayOfWeek = today.getDayOfWeek();

        // then : 결과를 검증
        assertThat(dayOfWeek).isEqualTo(DayOfWeek.THURSDAY);
    }
}
```

서로 비교해보면 Fest Assert를 사용한 경우 기존과는 다르게 assertThat()를 사용하기 위한 하나의 static import만을 사용한다.
그리고 assertThat()에 결과를 검증할 파라미터만 추가해주고, 값을 비교하기 위한 Matcher와 관련된 것들은 메소드 체이닝으로 사용하면 되기 때문에
추가로 static import를 사용할 일이 없다는 장점이 있다.

관련된 github는 [https://github.com/alexruiz/fest-assert-2.x](https://github.com/alexruiz/fest-assert-2.x)이고, 
[mvnrepository](http://mvnrepository.com/artifact/org.easytesting/fest-assert-core)에도 등록되어있기 때문에 
maven이나 gradle과 같은 의존성관리 도구를 이용해서도 쉽게 추가하여 사용할 수 있다.

헛, 글을 작성하다보니 밤 12시가 지나 2016년 새해가 되었다. 

모두 새해 복 많이 받으세요 ~


