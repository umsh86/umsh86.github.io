---
layout: post
title: 스프링 시큐리티 적용하기 [1]
categories: spring
---

# 0. 들어가기전에
보안이 중요하다는 것에 대해서는 누구나 다 공감할 것이다. 보안은 여러 레이어에 걸쳐 존재하지만, 여기에서는 애플리케이션 레벨에 적용할 수 있는
Spring Security를 적용해보고, 관련 내용을 정리해놓으려고 한다. 블로그에 글을 남기는 목적 중에 가장 큰 부분은 사실 내가 나중에 다시 찾아보기 위함이지만(기억력이 좋지 않아서...),
이 글을 읽는 누군가에게도 도움이 되었으면 좋겠다. 그리고 틀린 부분이나 내가 잘못 이해하고 있는 부분들, 개선사항에 대해서 의견을 남겨주시는 것 또한 환영한다.

앞으로 쓰는 모든 글에 대한 환경은 Intellij, osx, java 1.8, gradle 이므로 참고하는 것이 좋다.

# 1. Spring Boot로 프로젝트 생성하기

**Spring Initializr**를 사용해서 프로젝트를 생성할 예정이다.
Spring Initializr는 Intellij 뿐만 아니라, STS 나 [https://start.spring.io/](https://start.spring.io/)에서도 사용할 수 있다.
Intelij에서는 File > New > Project 또는 File > New > Module 을 선택하고, Spring Initializr를 선택한다.

![image1](https://cloud.githubusercontent.com/assets/1261904/11833515/b8dadff0-a406-11e5-870e-74273fc9923a.png)



새로운 프로젝트의 기본 정보를 설정해준다.

![image2](https://cloud.githubusercontent.com/assets/1261904/11833563/3c55824a-a407-11e5-9989-81c1cdb02413.png)



다음으로 프로젝트 생성시 추가할 라이브러리들을 설정한다. 여기서 선택한 라이브러리는 자동으로 maven이나 gradle 스크립트에 추가된다. 
우선은 Spring Security 테스트를 위해 security를 선택해준다. 그리고 뷰템플릿으로는 Thymeleaf를 선택하고, Web을 선택해준다. database는 H2를 선택한다.
필수는 아니지만 DevTools와 [Lombok](https://projectlombok.org/)을 선택했다.

![image3](https://cloud.githubusercontent.com/assets/1261904/11833617/e5ccf5ec-a407-11e5-8b8e-09be5912da68.png)



다음으로 프로젝트가 저장되는 경로와 이름을 지정한다.

![2015-12-16 2 44 12](https://cloud.githubusercontent.com/assets/1261904/11833796/b04a01f6-a409-11e5-91a6-7aa7c52a12b7.png)



프로젝트가 생성되면, 오른쪽 Gradle 탭을 선택해서 상단에 위치한 + 를 눌러준다. 방금 생성한 SpringSecurityStudy 프로젝트의 build.gradle을 선택하고 OK을 눌러준다.

![2015-12-16 3 36 08](https://cloud.githubusercontent.com/assets/1261904/11833932/0a07c466-a40b-11e5-81b3-5a3a65e16d70.png)



gradle이 레파지토리에서 필요한 라이브러리들을 모두 가져오고 나면, 왼쪽 Project 탭에서 다음과 같이 정상적으로 프로젝트가 생성된 것을 볼 수 있다.

![2015-12-16 3 43 38](https://cloud.githubusercontent.com/assets/1261904/11834028/f758a6fe-a40b-11e5-9ab3-f94487771ae6.png)

# 2. Spring Security Tutorial

우선 spring.io 에서 제공하는 [Spring Security Guide](http://spring.io/guides/gs/securing-web/)를 따라해보자.

`src/main/resources/templates/home.html`을 생성하고 아래 내용을 입력해준다.
`home.html`의 내용은 모든 사용자가 접근할 수 있고, `here` 클릭시, `/hello`를 요청한다. 하지만 `/hello`는 인증된(로그인된) 사용자만 접근할 수 있기 때문에 인증이 안되었다면 로그인을 요청한다. 

```html
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml" xmlns:th="http://www.thymeleaf.org" xmlns:sec="http://www.thymeleaf.org/thymeleaf-extras-springsecurity3">
    <head>
        <title>Spring Security Example</title>
    </head>
    <body>
        <h1>Welcome!</h1>

        <p>Click <a th:href="@{/hello}">here</a> to see a greeting.</p>
    </body>
</html>
```

`home.html`에서 `here`을 클릭했을 때 보여줄 `src/main/resources/templates/hello.html`를 생성해준다.

```html
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml" xmlns:th="http://www.thymeleaf.org"
      xmlns:sec="http://www.thymeleaf.org/thymeleaf-extras-springsecurity3">
    <head>
        <title>Hello World!</title>
    </head>
    <body>
        <h1>Hello world!</h1>
    </body>
</html>
``` 

`src/main/java/com/eomdev/study/config/MvcConfig.java`를 새로 생성한 후 아래 내용을 입력해준다.
`addViewControllers()`를 Override해서 4개의 request를 view와 mapping해준다.

```java
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.ViewControllerRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;

@Configuration
public class MvcConfig extends WebMvcConfigurerAdapter {

    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/home").setViewName("home");
        registry.addViewController("/").setViewName("home");
        registry.addViewController("/hello").setViewName("hello");
        registry.addViewController("/login").setViewName("login");
    }

}
```

**Spring Security 설정**

`src/main/java/com/eomdev/study/config/WebSecurityConfig.java`를 생성한 후 아래 내용을 입력해준다.
이 `WebSecurityConfig.java`는 `/`, `/home` 요청에 대해서는 모두 접근이 가능하도록 하고, 나머지 요청에 대해서는 인증을 요구하도록 설정한다.

그리고 in-memory 저장소를 이용하며, username : eomdev / password : eomdevpwd, role : USER로 갖는 사용자 1명을 추가해준다.

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.config.annotation.web.servlet.configuration.EnableWebMvcSecurity;

@Configuration
@EnableWebMvcSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .authorizeRequests()
                .antMatchers("/", "/home").permitAll()  
                .anyRequest().authenticated()           
                .and()
            .formLogin()
                .loginPage("/login")                    
                .permitAll()
                .and()
            .logout()
                .permitAll();
    }

    @Autowired
    public void configureGlobal(AuthenticationManagerBuilder auth) throws Exception {
        auth
            .inMemoryAuthentication()
                .withUser("eomdev").password("eomdevpwd").roles("USER");
    }
}
```


그리고, `/login`요청이 들어왔을 때, username / password를 입력할 수 있는 login 페이지를 만들어주자.
이 `login.html`은 username, password를 입력받아 post로 `/login`을 요청한다.
`src/main/resources/templates/login.html`

```html
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml" xmlns:th="http://www.thymeleaf.org"
      xmlns:sec="http://www.thymeleaf.org/thymeleaf-extras-springsecurity3">
    <head>
        <title>Spring Security Example </title>
    </head>
    <body>
        <div th:if="${param.error}">
            Invalid username and password.
        </div>
        <div th:if="${param.logout}">
            You have been logged out.
        </div>
        <form th:action="@{/login}" method="post">
            <div><label> User Name : <input type="text" name="username"/> </label></div>
            <div><label> Password: <input type="password" name="password"/> </label></div>
            <div><input type="submit" value="Sign In"/></div>
        </form>
    </body>
</html>
```

Spring Security는 요청을 차단하고 사용자를 인증하는 필터를 제공한다. 사용자가 인증에 실패하면 '/login?error'로 리다이렉트되고 해당 오류메시지가 표시된다.

성공적으로 로그인이되면, `/login?logout` 을 요청해 성공메시지가 표시될 것이다.

여기에 로그인이 성공했을 때, 현재 로그인한 사용자의 이름을 표시하고 로그아웃을 할 수 있도록 `hello.html`을 수정하자.

```html
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml" xmlns:th="http://www.thymeleaf.org"
      xmlns:sec="http://www.thymeleaf.org/thymeleaf-extras-springsecurity3">
    <head>
        <title>Hello World!</title>
    </head>
    <body>
        <h1 th:inline="text">Hello [[${#httpServletRequest.remoteUser}]]!</h1>
        <form th:action="@{/logout}" method="post">
            <input type="submit" value="Sign Out"/>
        </form>
    </body>
</html>
```

`hello.html`에서 `/logout`을 요청해서 성공적으로 로그아웃이 된다면, `/login?logout`으로 리다이렉트된다.


# 3. 테스트

`SpringSecurityStudyApplication.java` 마우스 오른쪽버튼을 이용해서 Run 또는 Debug를 해서 아래 내용을 테스트해본다.

* `/`, `/home`, `/login` 은 인증없이도 정상적으로 접속된다.
![](https://cloud.githubusercontent.com/assets/1261904/11836778/2e08111c-a420-11e5-9748-a81b36588c6f.png)


* `/hello` 를 입력시 인증된 정보가 없으므로 `/login` 으로 리다이렉트 된다.
![](https://cloud.githubusercontent.com/assets/1261904/11836779/309f8a9a-a420-11e5-8520-4ea5a445db66.png)

* `/login`에서 올바른 username / password 입력시 정상적으로 로그인이 된다. 그리고 이전에 원래 요청했던 `/hello`로 리다이렉트 된다.
![](https://cloud.githubusercontent.com/assets/1261904/11840590/559d4e6c-a43a-11e5-8818-42335e3af52a.png)

* `/login` 에서 올바르지 않은 username / password 입력시 `/login?error`로 리다이렉트 된다.
![](https://cloud.githubusercontent.com/assets/1261904/11840592/55affb16-a43a-11e5-9945-09c0ca1b6630.png)

* 로그인 되어 있는 상태의 `/hello`에서 로그아웃을 실행시, 정상적으로 로그아웃이 되고 `/login?logout`으로 리다이렉트 된다.
![](https://cloud.githubusercontent.com/assets/1261904/11840591/55ab4620-a43a-11e5-8b06-1551aeb85e80.png)


# 정리하며

여기까지는 사실 별 내용이 없다. 프로젝트를 생성했고, spring security guide를 따라한 내용이 전부이다.
우리가 실제로 사용하기에는 부족한점이 많은데, 더 자세한 내용은 다음에 다루도록 하겠다.

앞으로 해야할 내용

* in-memory 저장소 -> H2 database로 변경
* 패스워드 암호화(hash와 salt에 대해서)
* UserDetails, UserDetailsService 구현
 
