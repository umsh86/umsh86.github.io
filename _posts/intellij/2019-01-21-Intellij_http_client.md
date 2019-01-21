---
layout: post
title: IntelliJ의 Http Client 사용하기
categories: intellij
---

IntelliJ에서 제공하는 Http Client가 생각보다 편리해서 간단하게 사용하는 법만 정리해놓는다.
더 자세한 내용은 https://www.jetbrains.com/help/idea/http-client-in-product-code-editor.html 를 참고하면 될 것 같다.

# 1. actuator 추가하기
IntelliJ 2018.1 버전부터 추가된 기능인데, Controller의 @RequestMapping 기반으로 http 테스트를 자동으로 실행해주는 기능이 있다.

이 기능을 사용하기 위해서는 Spring Boot의 Actuator를 추가해주어야 한다.(Http Client 사용 자체는 Actuator를 추가하지 않아도 사용할 수 있다.)

gradle 기반으로 프로젝트를 생성했기 때문에 아래와 같이 build.gradle에 추가해주었다.

```json
dependencies {
    ...
    compile('org.springframework.boot:spring-boot-starter-actuator')
}
```

그리고 Spring Boot 애플리케이션을 실행시키고 Controller에 들어가면 아래와 같이 왼쪽에 Run icon들이 생긴 것을 볼 수 있다.
![image01](https://user-images.githubusercontent.com/1261904/44409169-d57cb080-a59c-11e8-9965-eee0cad5e20b.png)

# 2. Http Client 사용하기

사용하는 방법은 간단하다. 위에 생긴 초록색 Run icon을 클릭하면, 위의 이미지와 같이 3가지를 사용할 수 있다.

* Run HTTP Request : 바로 실행이 되고, 결과는 아래와 같이 바로 볼 수 있다.
![image02](https://user-images.githubusercontent.com/1261904/44409440-4fad3500-a59d-11e8-8278-31b16551ee1e.png)

* Open in HTTP Request Editor : 아래와 같이 HTTP Request Editor를 열어서 필요한 부분들을 추가, 수정하여 사용할 수 있다.
여기서는 새로운 Account를 저장하는 부분을 테스트하기 위해서 Content-Type하고, Request Body를 추가해서 실행하였고, 결과는 바로 아래에 보는 것과 같이 정상적으로 추가되었음을 알 수 있다.
![2018-08-21 11 58 18](https://user-images.githubusercontent.com/1261904/44409864-26d96f80-a59e-11e8-8792-19538e85bd09.png)

* Open in Browser : 브라우저에서 바로 실행한다.
![image04](https://user-images.githubusercontent.com/1261904/44410582-b5022580-a59f-11e8-819d-38cacee8007a.png)


* File > New > HTTP Request 파일을 선택해서 .http 파일을 생성해서 직접 작성해 줄 수도 있다.
![2019-01-21 10 42 56](https://user-images.githubusercontent.com/1261904/51478186-138d3300-1dce-11e9-8122-714a2193d86e.png)


# 3. Http Client에 environment variables 사용하기
local, dev, st 환경에서 번갈아가면서 테스트할 때 꽤나 유용했다. `http-client.env.json` 파일을 생성하고, 아래와 같이 local, dev, st, production 등에서 사용할 정보들을 입력해준다

```json
{
  "local": {
    "apiAddress" : "http://localhost",
    "username" : "localUser01",
    "password" : "password01"
  },

  "dev": {
      "apiAddress" : "http://dev.xxxxx.com",
      "username" : "devUser01",
      "password" : "password01"
    },
    
    ...
}	
```

그리고 Request Editor에서 아래와 같이 변경해주고 실행하게 되면, 사용할 environment를 선택할 수 있다. 
![2019-01-21 11 07 15](https://user-images.githubusercontent.com/1261904/51479376-661c1e80-1dd1-11e9-8472-4c0fecbb3281.png)


그리고 응답을 받은 데이터를 `.http`의 변수에 저장하고, 다른 테스트에서 변수에 저장된 값을 사용할 수 있다. 예를 들면, 위의 `/authorizationKey` 요청을 통해서 아래와 같은 응답을 받는다고 가정해보자

```json
{
  "returnCode": "200",
  "returnData": {
    "authorizationKey": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c"    
  }
}
``` 

이 응답에 존재하는 authorizationKey를 아래와 같이 변수에 저장할 수 있다.
![2019-01-21 11 38 37](https://user-images.githubusercontent.com/1261904/51481424-2eb07080-1dd7-11e9-98e8-06a223bd3e58.png)


authorizationKey 변수를 사용해서 다른 요청을 하는 방법은 아래와 같다.
<img width="320" alt="2019-01-21 11 55 56" src="https://user-images.githubusercontent.com/1261904/51481824-27d62d80-1dd8-11e9-98ef-a82d5ee5ce74.png">



