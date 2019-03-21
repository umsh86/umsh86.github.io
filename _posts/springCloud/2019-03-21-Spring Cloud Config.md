---
layout: post
title: Spring Cloud Config
categories: SpringCloud
---

Spring Cloud Config Server에 대해서 알아봅니다.

관련 예제는 Github에 있습니다. 
* [Config-Repo](https://github.com/umsh86/blog-samples/tree/master/config-repo)
* [Config-Server](https://github.com/umsh86/blog-samples/tree/master/config-server)
* [Config-Client](https://github.com/umsh86/blog-samples/tree/master/config-client)


# 1. Spring Cloud Config Server?

일반적인 Spring Boot Application 이라면 환경설정과 관련된 부분들은 해당 프로젝트에 함께 포함되어 있는 yml이나 properties 파일들을 읽어서 사용을 했었습니다. 

하지만 이 경우에 어떤 이유에서 환경설정 정보가 변경된다면(예를 들어, 환경설정 정보에 저장되어 있는 rabbitmq의 host와 password가 변경되어  `spring.rabbitmq.host`와 `spring.rabbitmq.password` 정보를 수정해야되는 경우), 애플리케이션 전체를 다시 빌드를 해야된다는 문제점이 있습니다.

그래서 사실 dev, st, prod 환경에서는 아래와 같이 애플리케이션 배포 패키지에서 분리해서 외부화하기 위해서 환경설정 파일의 위치를 직접 지정해주면서 실행시켜서 사용하고 있었습니다.

```shell
$ java -jar application.jar --spring.config.location=<file location>
```

하지만 이 방법도 서버에 마운트돼 있는 로컬이나 공유된 파일 시스템에 의존하게 되고, 대규모 배포에서는 더 강력한 중앙 집중형 환경설정 정보 관리 솔루션이 필요합니다.

그것이 바로 `환경설정 서버`인데, Spring Cloud Project에서 환경설정 서버를 만들 수 있도록 지원하고 있는 것들이 `Config Server`, `ZooKeeper Configuration`, `Consul Configuration`이 존재하는데, 여기에서는 `Config Server` 만 알아볼 예정입니다.

Spring Config Server의 아키텍처는 아래와 같습니다.

![스크린샷 2019-03-15 오후 5 02 35](https://user-images.githubusercontent.com/1261904/54416946-2ffd7a00-4744-11e9-9403-f4902698917a.png)

`Config Server`는 환경설정 정보들을 `Git Repository`(SVN도 가능)에 저장되어 있는 환경설정 정보 파일들(yml 또는 properties)들을 읽습니다.

그리고 각각의 마이크로서비스 애플리케이션들(Config Client)은 필요한 환경설정 정보들을 Config Server에서 읽어서 가져오며, 로컬에 캐시해둡니다. 여기에서 Config Server는 환경설정 정보가 변경되면(위의 예제처럼 rabbitmq의 host와 password가 변경되면) Config Server를 바라보고 있는 모든 마이크로서비스에게 변경 사항들을 전파하고, 변경 사항을 로컬 캐시에 반영하게 됩니다.



# 2. Git Repository 구성

그러면 우선 Config Server가 읽어서 사용할 Git Repository를 생성해주어야 합니다.

`config-repo`라는 새로운 디렉토리를 생성하고 그 하위에 아래와 같은 작업을 진행해주었습니다.



우선 Config Server가 직접 사용할 환경설정 정보를 추가해줍니다. 파일명은 application.yml 입니다.

```yaml
spring:
  rabbitmq:
    host: 192.168.99.100
    port: 5672
    username: guest
    password: guest
    virtual-host: vhost01
```



그리고 마이크로서비스1, 2, 3이 dev, st, prod 에서 사용할 api-dev.yml, api-st.yml, api-prod.yml 을 각각 생성해줍니다

```yaml
# api-dev.yml
spring:
  rabbitmq:
    host: dev.eomdev.com
    port: 5672
    username: devUser
    password: devPassword
    virtual-host: vhost01
server:
  port: 9998    
custom:
  message: Hello, api-dev!

# api-st.yml
spring:
  rabbitmq:
    host: st.eomdev.com
    port: 5672
    username: stUser
    password: stPassword
    virtual-host: vhost02
server:
  port: 9999
custom:
  message: Hello, api ST!


# api-prod.yml
spring:
  rabbitmq:
    host: eomdev.com
    port: 5672
    username: user
    password: password
    virtual-host: vhost03
custom:
  message: Hello, api prod!
```



또는 아래와 같이 하나의 api.yml을 생성해서 profile 별로 분리를 시켜줄 수도 있습니다

```yaml
spring:
  profiles: dev
  rabbitmq:
    host: dev.eomdev.com
    port: 5672
    username: devUser
    password: devPassword
    virtual-host: vhost01
server:
  port: 9998    
custom:
  message: Hello, api-dev!
---
# api-st.yml
spring:
  profiles: st
  rabbitmq:
    host: st.eomdev.com
    port: 5672
    username: stUser
    password: stPassword
    virtual-host: vhost02
server:
  port: 9999
custom:
  message: Hello, api ST!
---
# api-prod.yml
spring:
  profiles: prod
  rabbitmq:
    host: eomdev.com
    port: 5672
    username: user
    password: password
    virtual-host: vhost03
custom:
  message: Hello, api prod!
```



그리고 해당 디렉토리를 초기화해주고 git remote repository로 push해 줍니다. 

```shell
$ git init
$ git add . 
$ git commit -m "first commit"
$ git remote add origin <Remote Repository 주소>
$ git push -u origin master
```



# 3. Spring Config Server 생성

 `Config Server`와 `Actuator`를 추가해서 config-server 프로젝트를 새로 만들어 줍니다

![2019-03-04 10 35 10](https://user-images.githubusercontent.com/1261904/53705652-62b49200-3e69-11e9-84bf-abbc042fd566.png)

아래와 같이 Application.java 에 `@EnableConfigServer`를 추가해줍니다

```java
@SpringBootApplication
@EnableConfigServer      // 추가
public class ConfigServer {
  public static void main(String[] args) {
    SpringApplication.run(ConfigServer.class, args);
  }
}
```



## 3.1 local 개발 환경 설정

local에서 개발시에는 자유롭게 properties 들을 추가, 수정, 삭제하면서 개발하기 위해서 `remote git repository`를 바라보는 것이 아니라, **로컬에 존재하는 환경설정 파일을 바라보게 설정**해줘야 합니다.

이렇게 하기 위해서는 `application.yml`에 읽어 올 Config file 의 위치를 `spring.cloud.config.server.native.searchLocations`의 값으로 설정해주고, local에 존재하는 해당 환경설정 파일을 읽어올 수 있도록 `native` Profile을 활성화 해야 합니다.

즉, 아래와 같이 설정해 줍니다.

```yaml
spring:
  profiles:
    active: native # file system에서 Config file을 읽어 올 수 있도록 native profile을 활성화시켜야 합니다.
---
spring:
  profiles: native
  cloud:
    config:
      server:
        native:
          search-locations: file://${user.home}/IdeaProjects/study/blog-samples/config-repo # config file의 위치. 윈도우의 경우에는 "/"가 추가로 필요합니다.(file:///${user.home}/...)
server:
  port: 8888
```



그리고 실제로 `Config Server`에게 http://localhost:8888/api/dev 를 호출해서 `api-dev.yml`의 환경설정 정보를 가져왔다는 것을 아래와 같이 확인할 수 있습니다.

```json
{
  "name": "api",
  "profiles": [
    "dev"
  ],
  "label": null,
  "version": null,
  "state": null,
  "propertySources": [
    {
      "name": "file:///Users/eomdev/IdeaProjects/study/blog-samples/config-repo/api.yml (document #0)",
      "source": {
        "spring.profiles": "dev",
        "spring.rabbitmq.host": "dev.eomdev.com",
        "spring.rabbitmq.port": 5672,
        "spring.rabbitmq.username": "devUser",
        "spring.rabbitmq.password": "devPassword",
        "spring.rabbitmq.virtual-host": "vhost01",
        "server.port": 9998,
        "custom.message": "Hello, api-dev!"
      }
    }
  ]
}
```

> Config Server에게 url을 요청해서 환경설정 정보를 가져오는 전략에 대해서는 아래 3.3 Config Server URL 설명에서 다루겠습니다.



## 3.2 Remote Git Repository 설정

dev, st, prod 에서는 `Remote Git Repository(여기에서는 Github)`를 바라보고 사용하도록 설정해주면 됩니다

`Config Server` 애플리케이션의 `application.yml`에 아래 내용을 추가해줍니다

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
---
spring:
  profiles: dev
  cloud:
    config:
      server:
        git:
          uri: https://github.com/umsh86/blog-samples.git
          search-paths: config-repo # 저는 환경설정 파일들이 /blog-samples/config-repo 하위에 있어서 search-paths를 설정해주었습니다.
#          username: # private-repo라면 접근권한이 있는 username, password를 설정해주면 됩니다
#          password: #
```



그리고 `spring.profiles.active=dev`로 변경해서 실행하거나, 아래와 같이 옵션을 사용해서 애플리케이션을 실행합니다.

```shell
$ gradle bootRun -Dspring.profiles.active=dev

$ java -jar -Dspring.profiles.active=dev ./api.jar
```



그리고 `Remote Git Repository`에서 정상적으로 환경설정 파일을 가져왔는지 `Config Server`의  http://localhost:8888/api/dev 를 호출해서 확인해줍니다.

```json
{
  "name": "api",
  "profiles": [
    "dev"
  ],
  "label": null,
  "version": "8a736c7b1e061366fb12ba38ecd6eb61ddeeeeae",
  "state": null,
  "propertySources": [
    {
      "name": "https://github.com/umsh86/blog-samples.git/config-repo/api.yml (document #0)",
      "source": {
        "spring.profiles": "dev",
        "spring.rabbitmq.host": "dev.eomdev.com",
        "spring.rabbitmq.port": 5672,
        "spring.rabbitmq.username": "devUser",
        "spring.rabbitmq.password": "devPassword",
        "spring.rabbitmq.virtual-host": "vhost01",
        "server.port": 9998,
        "custom.message": "Hello, api-dev!"
      }
    }
  ]
}
```



## 3.3 Config Server URL 설명

위에서 `Config Server`에서 url을 호출해서 환경설정 정보를 확인하기 위해  http://localhost:8888/api/dev 를 호출했는데, 이 부분은  http://localhost:8888/{application}/{profile}/{label} 형태 입니다.

- `{application}` : `Config Client`에서 설정되어 있는 `spring.application.name` 속성으로 지정된 애플리케이션의 이름을 의미합니다.  여기에서는  `spring.application.name=api`으로 설정하여, `api.yml`에 있는 정보들을 사용할 예정입니다.
- `{profile}` : `Config Client`에 설정되어 있는 `spring.profiles.active` 의 값이고, comma(,)로 구분되어 있는 여러 개의 값일 경우 마지막 값이 적용 됩니다. 기본 profile 값은 default 입니다.
- `{label}` : 필수가 아닌 옵션이며, git branch의 이름을 나타냅니다. 기본 값은 master입니다.

이 {application}, {profile}, {lable}의 값을 가지고 아래와 같은 방법으로도 Config Server에게 환경설정 정보 값들을 확인 할 수 있습니다.

```http
/{application}/{profile}[/{label}]
/{application}-{profile}.yml
/{label}/{application}-{profile}.yml
/{application}-{profile}.properties
/{label}/{application}-{profile}.properties
```





# 4. Config Client(샘플 애플리케이션) 생성

`Config Client(샘플 애플리케이션)`을 만들고, 필요한 환경설정 정보들을 `Config Server`에 요청해서 가져오는 부분입니다.

우선 Spring Initializr로 `Config Client`, `Web`, `Actuator`, `lombok`를 선택해서 프로젝트를 생성해줍니다.

![스크린샷 2019-03-20 오전 11 14 20](https://user-images.githubusercontent.com/1261904/54654415-774b8800-4b01-11e9-822c-417f4142fc17.png)



## 4.1 bootstrap.yml 설정

Spring Boot Application은 일반적으로 applicatin.yml 이나 application.properties를 사용합니다. 하지만 `Config Client`는 `bootstrap.yml` 파일을 사용하며, 여기에 설정된 내용은 `application.yml`보다 먼저 로드됩니다.

`bootstrap.yml`을 생성해주고 아래와 같이 설정해줍니다

```yaml
spring:
  profiles:
    active: dev
  application:
    name: api # /{application}/{profile}/{label}을 생각해서 애플리케이션 이름을 정해줍니다.

management:
  endpoints:
    web:
      base-path: /actuator 
      exposure:
        include: ['refresh'] # actuator의 refresh를 사용할 수 있게 해줍니다. 뒤에서 설명합니다.
---
spring:
  profiles: dev
  cloud:
    config:
      uri: http://localhost:8888 # config server의 uri를 설정해줘야합니다
---
spring:
  profiles: st
  cloud:
    config:
      uri: http://stapi.eomdev.com
---
spring:
  profiles: prod
  cloud:
    config:
      uri: http://api.eomdev.com
```



그리고 실제로 설정된 환경설정 값을 확인하기 위해서 `custom.message`만 읽어서 확인할 수 있도록 아래의 2개 class를 생성해줍니다.

```java
@Component
@ConfigurationProperties(prefix = "custom")
@RefreshScope
@Getter @Setter
public class ConfigProperties {

  private String message;

}

```

```java
@RestController
public class ConfigController {

  @Autowired
  private ConfigProperties configProperties;

  @GetMapping("/config")
  public ResponseEntity getConfig() {
    return ResponseEntity.ok(configProperties.getMessage());
  }

}
```



> @RefreshScope로 선언해주면 Actuator의 /refresh 호출을 통해서 해당 Bean의 값이 변경됩니다.



그리고 애플리케이션을 실행하면 아래와 같이 console log를 통해서 `Config Server`에서 환경설정 정보들을 가져왔다는 것을 확인할 수 있습니다.

```shell
# config server localhost:8888에서
2019-03-20 17:51:30.046  INFO 25096 --- [           main] c.c.c.ConfigServicePropertySourceLocator : Fetching config from server at : http://localhost:8888

# 아래의 environment 정보로
2019-03-20 17:51:30.376  INFO 25096 --- [           main] c.c.c.ConfigServicePropertySourceLocator : Located environment: name=api, profiles=[dev], label=null, version=null, state=null

# 실제로 환경설정 정보 파일을 가져왔는지 확인할 수 있습니다.
2019-03-20 17:51:30.376  INFO 25096 --- [           main] b.c.PropertySourceBootstrapConfiguration : Located property source: CompositePropertySource {name='configService', propertySources=[MapPropertySource {name='file:///Users/sheom/IdeaProjects/study/blog-samples/config-repo/api.yml (document #0)'}]}


```



실제로 `/config`로 요청해보면 `Hello, api-dev!` 메시지가 정상적으로 나오는 것을 확인할 수 있습니다.



# 5. 테스트

실제로 값이 제대로 변경되는지 테스트를 하기 위해서 `Config-Repo`의 `api.yml`에서 값을 변경해줍니다.

```yaml
spring:
  profiles: dev
  rabbitmq:
    host: dev.eomdev.com
    port: 5672
    username: devUser
    password: devPassword
    virtual-host: vhost01
server:
  port: 9998
custom:
  message: Hello, api-dev! Test!!!  # Test!!! 를 추가해주었습니다.
  
  ...

```



그리고 `Config-Client`애플리케이션에게 변경된 정보를 가져오라고 알려주기 위해서 `Actuator`의 `/refresh`를 `POST`로 호출해줘야 합니다.

```shell
$ curl localhost:8080/actuator/refresh -d {} -H "Content-Type: application/json"
["custom.message"] # custom.message가 바뀌었음을 알려줌

```



그리고 다시 한번 `/config`를 호출하면 `Hello, api-dev! Test!!!`로 변경된 값이 정상적으로 출력되는 것을 확인해볼 수 있습니다.



# 6. 정리

이렇게 `Config Server`를 사용하면 애플리케이션을 재시작하지 않고도 환경설정 정보를 변경할 수 있습니다. 하지만, 변경된 내용을 적용하기 위해서는 각각의 `Config Client`인스턴스에서 `/refresh`를 호출해줘야하는 불편한점이 있습니다.

이 부분은`Spring Cloud Bus`를 사용하면, `Config-Repo`에서 환경설정 정보가 변경되면  `Config-Client`에게 변경되었다고 전파해서 자동으로 변경되도록 할 수 있다고 합니다.

관련 내용은 다음에 정리해보도록 하겠습니다.