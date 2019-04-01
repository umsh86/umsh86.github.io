---
layout: post
title: Spring Cloud Bus
categories: SpringCloud
---

Spring Cloud Bus에 대해서 알아봅니다.

관련 예제는 Github에 있습니다. 
* [Config-Repo](https://github.com/umsh86/blog-samples/tree/master/config-repo)
* [Config-Server](https://github.com/umsh86/blog-samples/tree/master/config-server)
* [Config-Client](https://github.com/umsh86/blog-samples/tree/master/config-client)


이 전 [Spring Cloud Config](<http://blog.eomdev.com/springcloud/2019/03/21/Spring-Cloud-Config.html>)에서도 이야기 했듯이, `Spring Cloud Config Server`를 사용하면 `Config Client(MSA)` 애플리케이션을 수정하거나 재시작 하지 않아도 변경된 환경설정 정보들을 적용할 수 있습니다.
하지만, 변경된 환경설정 정보를 다시 적용시키기 위해서는 각각의 `Config Client` 인스턴스들에게 각각 Actuator의 `/refresh`를 호출을 해줘야한다는 단점이 있었습니다.


# 1. Spring Cloud Bus?

위에서 이야기한 각각의 `Config Client` 인스턴스마다 `/actuator/refresh`를 호출해줘야하는 단점은 `Spring Cloud Bus`를 통해서 하나의 `메시지 브로커(여기서는 RabbitMQ)`에 모든 인스턴스들을 연결해서 해결할 수가 있습니다.



![스크린샷 2019-03-29 오전 10 32 15](https://user-images.githubusercontent.com/1261904/55203234-18c38f80-520e-11e9-8a26-80d4eba89558.png)

개념은 위의 그림과 같고, 간단하게 설명하자면 아래와 같습니다.

1. 개발자가 변경할 프로퍼티를 수정하고, 변경된 `Config-Repo`를 Git Repository에 push 합니다.
2. 만약 Git Repository로 Github와 같은 저장소를 사용하고 있다면, Git Repository에 변경이 일어났을 때 webhook 으로 특정 url을 호출할 수 있습니다. 이 때 `Config-Server`의 `/actuator/bus-refresh`를 POST로 호출하도록 해주면 되는데, 여기서는 webhook을 사용하지 않고 직접 `Config-Server`에 url을 호출해주도록 하겠습니다.
3. `/actuator/bus-refresh`를 통해서 변경되었음을 통보받은 `Config-Server`는 변경된 환경설정 정보를 다시 갱신합니다.
4. `Spring Cloud Bus`가 변경된 정보가 있다고 RabbitMQ에게 메시지를 발행합니다.
5. 메시지를 구독받은 `Config Client`들은 환경설정 정보가 변경되었음을 알게 되고, 각각 환경설정 정보를 업데이트 합니다.





# 2. Config Server 수정

`Config-Server` 프로젝트에 `Spring Cloud Bus`를 사용하기 위해서 `spring-cloud-starter-bus-amqp`  디펜던시를 추가해주어야 합니다.

```groovy
dependencies {
    ...
    implementation 'org.springframework.cloud:spring-cloud-starter-bus-amqp' // 추가
}
```



그리고 `application.yml`에 `RabbitMQ` 정보와 `actuator`의 `/bus-refresh`를 사용가능하도록 노출시켜 주기 위해서 `management.endpoints.web.exposure.include`에 등록해줍니다.

```yaml
spring:
  profiles:
    active: native
server:
  port: 8888
---
spring:
  profiles: native
  cloud:
    config:
      server:
        native:
          search-locations: file://${user.home}/IdeaProjects/study/blog-samples/config-repo
  rabbitmq:                   # rabbitmq와 관련된 부분 추가
    host: 192.168.99.100
    username: guest
    password: guest
management:
  endpoints:
    web:
      base-path: /actuator
      exposure:
        include: ['bus-refresh'] # actuator의 bus-refresh endpoint 노출시켜줍니다
---
spring:
  profiles: dev
  cloud:
    config:
      server:
        git:
          uri: https://github.com/umsh86/blog-samples.git
          search-paths: config-repo
```





# 3. Config Client 수정

`Config-Server`와 마찬가지로 `Spring Cloud Bus`를 사용하기 위해서 디펜던시를 아래와 같이 추가해줍니다.

```groovy
dependencies {
    ...
    implementation 'org.springframework.cloud:spring-cloud-starter-bus-amqp' // 추가
}
```



그리고 이번에는 `Config-Repo`의 `api.yml`에 `RabbitMQ 정보`를 수정해주고, 테스트를 위해서 `server.port`를 삭제해줍니다(Config Client를 로컬에 2개 띄울 예정이며, `-Dserver.port`옵션으로 port 중복을 회피하기 위해서 입니다).

```yaml
spring:
  profiles: dev
  rabbitmq:       # RabbitMQ 정보 수정
    host: 192.168.99.100   
    port: 5672
    username: guest
    password: guest
# server:
#   port: 9998
custom:
  message: Hello, api-dev!
---
...
```





# 4. 테스트

위의 내용을 모두 수정했으면, 우선 `Config-Server`를 실행시켜 줍니다. 애플리케이션이 정상적으로 실행되고 나면 RabbitMQ managerment > Exchanges 에 `springCloudBus`가 추가된 것을 볼 수 있습니다. 

![스크린샷 2019-03-27 오후 3 09 30](https://user-images.githubusercontent.com/1261904/55054359-e6e1ea00-50a2-11e9-84fa-e4a35dd15a9b.png)



그리고 이번에는 `Config-Client`프로젝트를 빌드하고, 생성된 jar 파일을 각각 9999와 9998 포트로 실행합니다.

```shell
$ java -jar -Dserver.port=9999 config-client-0.0.1-SNAPSHOT.jar
$ java -jar -Dserver.port=9998 config-client-0.0.1-SNAPSHOT.jar
```



정상적으로 실행이 되면, 각각의 `Config-Client` 인스턴스에서 `custom.message`가 정상적으로 나오는지 `http://localhost:9999/config`와 `http://localhost:9999/config` 을 호출해서 확인해봅니다.

`Hello, api-dev!`라는 메시지가 정상적으로 나오는 것을 확인할 수 있습니다.

그리고 local `Config-Repo`의 `api.yml`에서 `custom.message`의 내용을 변경해줍니다.

```yaml
spring:
  profiles: dev
  rabbitmq:
    host: 192.168.99.100
    username: guest
    password: guest
custom:
  message: Hello, api-dev! Cloud Bus Test!! # Cloud Bus Test!! 메시지를 추가했습니다.
```



그리고 `Config-Server`의 `/actuator/bus-refresh`를 `POST`로 호출해줍니다. 저는 InteliiJ의 [Http Client](<http://blog.eomdev.com/intellij/2019/01/21/Intellij_http_client.html>)를 사용해서 호출해주었습니다.

```http
POST http://localhost:9998/actuator/bus-refresh

HTTP/1.1 204 
Date: Fri, 29 Mar 2019 05:56:22 GMT

<Response body is empty>

Response code: 204; Time: 970ms; Content length: 0 bytes
```



아래는 `Config Server`의 console 로그 입니다.

```shell
2019-03-29 14:56:22.028  INFO 11736 --- [nio-8888-exec-6] trationDelegate$BeanPostProcessorChecker : Bean 'org.springframework.cloud.autoconfigure.ConfigurationPropertiesRebinderAutoConfiguration' of type [org.springframework.cloud.autoconfigure.ConfigurationPropertiesRebinderAutoConfiguration$$EnhancerBySpringCGLIB$$38bc02b3] is not eligible for getting processed by all BeanPostProcessors (for example: not eligible for auto-proxying)
2019-03-29 14:56:22.154  INFO 11736 --- [nio-8888-exec-6] o.s.boot.SpringApplication               : The following profiles are active: native
2019-03-29 14:56:22.174  INFO 11736 --- [nio-8888-exec-6] o.s.boot.SpringApplication               : Started application in 0.491 seconds (JVM running for 1888.348)
2019-03-29 14:56:22.471  INFO 11736 --- [nio-8888-exec-7] o.s.c.c.s.e.NativeEnvironmentRepository  : Adding property source: file:///Users/sheom/IdeaProjects/study/blog-samples/config-repo/api.yml (document #0)
2019-03-29 14:56:22.529  INFO 11736 --- [nio-8888-exec-6] o.s.cloud.bus.event.RefreshListener      : Received remote refresh request. Keys refreshed []
```



아래는 `Config Client`의 로그입니다.

```shell
2019-03-29 14:56:22.486  INFO 11833 --- [7e5Xfw4e8qNZg-1] o.s.boot.SpringApplication               : The following profiles are active: dev
2019-03-29 14:56:22.524  INFO 11833 --- [7e5Xfw4e8qNZg-1] o.s.boot.SpringApplication               : Started application in 0.611 seconds (JVM running for 806.562)
2019-03-29 14:56:22.828  INFO 11833 --- [7e5Xfw4e8qNZg-1] o.s.cloud.bus.event.RefreshListener      : Received remote refresh request. Keys refreshed [custom.message]
2019-03-29 14:56:22.924  INFO 11833 --- [7e5Xfw4e8qNZg-1] o.s.a.r.c.CachingConnectionFactory       : Attempting to connect to: [192.168.99.100:5672]
2019-03-29 14:56:22.948  INFO 11833 --- [7e5Xfw4e8qNZg-1] o.s.a.r.c.CachingConnectionFactory       : Created new connection: rabbitConnectionFactory.publisher#35b2d39c:0/SimpleConnection@41f917fb [delegate=amqp://guest@192.168.99.100:5672/, localPort= 64087]

```



그리고 각각의 `Config-Client` 인스턴스에서 `/config`를 호출해 보면 둘 다 `Hello, api-dev! Cloud Bus Test!!`메시지가 나오는 것을 확인할 수 있습니다.

이렇게 해서 `Config Server`의 `/actuator/bus-refresh`만을 호출해서 구독하고 있는 모든 `Config Client` 인스턴스들에게 한 번에 변경된 환경설정 정보를 변경할 수 있다는 것을 확인할 수 있었습니다.

실제로는 `Github`의 `Webhook`을 사용해서 자동으로 호출하게 할 수 있으므로 직접 `/actuator/bus-refresh`를 호출해주는 작업도 필요하지 않습니다.

다음에는 Config Server의 고가용성에 대한 이야기를 정리해보겠습니다.



# 5. Config Server 고가용성 적용

Config Server의 고가용성 확보를 위한 아키텍처는 아래와 같습니다. 

그리고 `config Server`를 구성할 때 고가용성에 대해서 고려해야하는 부분은 총 3가지로 `Config Server`, `RabbitMQ`, `Config-Repo(Git Repository)`입니다.

![스크린샷 2019-04-01 오후 2 17 10](https://user-images.githubusercontent.com/1261904/55304951-0f922700-5489-11e9-98aa-db2e4572dc4f.png)




## 5.1 Config Server

`Config Server`에 장애가 발생하면 환경설정 정보를 읽어서 사용하는 `Config Client`서비스들이 정상적으로 실행될 수 없으므로, 고가용성이 필요합니다.

하지만, `Config Server`에 장애가 발생하더라도 `Config Client`가 이미 정상적으로 작동되고 있는 상태라면 로컬에 캐시해둔 환경설정 정보를 기준으로 동작하기 때문에 크게 문제가 되지 않습니다(물론 새로 애플리케이션이 실행된다면 문제…).

다만, `Config Client`의 `bootstrap.yml`에서 `spring.cloud.config.uri`는 하나만 설정할 수 있으므로, 다수의 `Config Server`에 대한 정보는 로드 밸런서와 같은 곳에 저장돼야 합니다.

그래서 위의 아키텍처에서는 `Config Client`들이 `ELB`를 통해 `Config Server`를 호출하는 형태입니다.




## 5.2 RabbitMQ

`RabbitMQ`의 경우에도 고가용성이 적용되야하지만, `RabbitMQ`에 장애가 발생하더라도, 각각의 마이크로서비스 인스턴스의 `/actuator/refresh`를 직접 호출해서 변경된 환경설정 정보를 가져올 수 있으므로 크게 문제는 되지 않습니다.




## 5.3 Git Repository

`Git Repository`의 경우에도 로컬의 파일 기반의 Repository를 사용하는 것보다는 고가용성이 확보된 Git 서비스를 기반으로 운영돼야 합니다.




# 6. 정리

이렇게 해서 지난 번에 `Spring Cloud Server`, `Spring Cloud Client`에 대해서 다뤘고, 이번에는 `Spring Cloud Bus`와 `고가용성`에 대해서 정리를 해보았습니다.


> 참고 : 스프링 5.0 마이크로서비스 2/e (에이콘출판사)