---
layout: post
title: OSX에 Docker로 Jenkins 설치하기
categories: tool
---

오랜만에 블로그에 글을 쓰게 되었다. 

원래는 일을 하거나 공부하면서 생겼던 것들을 나중에 까먹지 않기 위해서 기록하는 용도로 주기적으로 글을 쓰려고 했었다.
하지만 공개적으로 노출이 되는 것이라서 나만 알아볼 수 있게 대충 적어놨던 것들을 다시 잘 정리해서 쉽게 풀어쓰는 작업이 필요했다.
이 부분을 하는데 아무래도 시간도 걸리고 게을러서 거의 2년간 새로운 글을 쓴 적이 없다.

게다가 Dropbox와 같은 것들과 연동해서 사용할 수 있는 마크다운 앱을 구매하고 사용하면서 해당 앱에만 정리하기 시작한 것도 블로그에 글을 쓰지 않게 된 것에 상당히 크게 한 몫 했다.

그러던 도중에 원래 주기적으로 토즈에서 가지던 지인들과의 소규모 모임이 있는데, 최근에는 각자의 바쁜 사정들로 인해서 진행되지 않았었다.
그 중 우리의 모임을 주도해서 이끌어나가고 있던 Jo님과의 만남에서 다시 슬슬 진행해보는게 어떻겠냐는 이야기가 나왔고, 다음 모임에서 내가 Jenkins에 대해서 발표를 해줬으면 좋겠다고 이야기를 하셨다.
나도 흔쾌히 승낙하였고, Jenkins에 대해서 무엇을 발표할까 생각하다가 Docker에 대한 소개도 간단히 할겸, Docker로 Jenkins를 설치하고, 어떻게 사용하는지에 대해서 발표를 해야겠다는 생각을 했다.
그리고 함께 실습을 하면서 진행사항이 조금 늦거나 지나간 내용을 다시 확인하고 싶은 분들을 위해서 자료를 공유해서 각자 볼 수도 있게 해야겠다는 생각이 들었다.
그래서 자료 공유하는 방법에 대해서 생각하다가 그냥 블로그에 글을 써서 같이 봐야겠다는 생각이 들었고, 덕분에 이렇게 오랜만에 글을 쓰게 되었다.


# 1. 목표

* Docker를 통해서 Jenkins 설치
* GitHub에 있는 프로젝트와 연동해서 Push가 일어날 때, 자동으로 빌드되게 하기
* 작업이 완료되면 Slack을 통해서 알림받기

# 2. Jenkins

## 2.1. Jenkins 무엇?

https://ko.wikipedia.org/wiki/젠킨스

## 2.2. 실제로 회사에서는 어떻게 사용하고 있는가?

* 개발자들이 일일 commit한 내용을 하루에 1번(예를 들면 오전 7시) 자동으로 build, deploy를 통해서 개발(Dev)서버에 반영되고 있다.
* 프로젝트의 개발 내용이 모두 끝나면(예를 들어 1.0.0 -> 1.0.1 업데이트 기능 모두 개발 완료) QA팀의 품질 테스트를 위해서 Staging서버에 build, deploy를 한다.
* QA가 모두 종료되고, 실제 운영서버에 업데이트 하는 경우 Jenkins를 통해서 deploy를 하고 있다.

# 3. Docker

## 3.1. Docker 무엇?
Docker는 애플리케이션을 신속하게 구축, 테스트 및 배포할 수 있는 소프트웨어 플랫폼이며, 쉽게 말하면 VMware, Virtual PC와 같은 가상머신이다. 
그러나 OS를 설치하지 않고, OS자원은 호스트와 공유해서 사용하며, 서버 운영에 필요한 프로그램과 라이브러리만 격리해서 설치할 수 있는 장점이 있다. 
정리해서 말하자면 가상화머신보다 훨씬 빠르고, 호스트와 거의 동일한 속도로 사용할 수 있다!

> 도커에 대해 더 자세히 알아보고 싶다면 [가장 빨리 만나는 Docker](http://pyrasis.com/docker.html)를 추천한다.

## 3.2. Mac에서 Docker 설치하기

https://www.docker.com/get-docker 에 접속해서 DOCKER COMMUNITY EDITION (무료)을 다운로드 해서 설치한다. 

설치 완료 및 실행이 정상적으로 되면 아래와 같은 화면을 볼 수 있다.

![2018-03-21 10 45 19](https://user-images.githubusercontent.com/1261904/37872219-cbfafbfe-303d-11e8-9575-d38a1eff9097.png)

## 3.3. 정상적으로 설치되었는지 확인

터미널에서 아래와 같은 명령어를 통해서 Docker가 정상적으로 설치되었는지 확인한다.  

Docker가 설치된 버전 확인

```
$ docker version 
```

hello-world 실행. 아래와 같은 메시지가 출력된다면 정상이다.

```
$ docker run hello-world  
Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
1. The Docker client contacted the Docker daemon.
2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
  (amd64)
3. The Docker daemon created a new container from that image which runs the
  executable that produces the output you are currently reading.
4. The Docker daemon streamed that output to the Docker client, which sent it
  to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
$ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
https://cloud.docker.com/

For more examples and ideas, visit:
https://docs.docker.com/engine/userguide/
```

# 4. Docker를 통해서 Jenkins 설치하기

터미널에서 Docker 명령어를 통해서 설치도 가능하지만, Kitematic(Docker 컨테이너들을 관리하는 그래픽 인터페이스)을 통해서 설치 및 실행 해보자.
우측 상단에서 도커 아이콘을 선택하여 Kitematic을 선택한다.
![kitematic select](https://user-images.githubusercontent.com/1261904/37872297-abcddba6-303f-11e8-9e87-bb49ba1d1268.png)

Kitematic이 설치가 안되어 있다면 아래와 같은 화면이 나올 것이다.
![2018-03-21 11 07 28](https://user-images.githubusercontent.com/1261904/37872322-4844bc16-3040-11e8-9978-e8c9571174c0.png)

설치를 하고, `/Applications`에 옮긴다면 아래와 같은 화면이 뜬다.
![2018-03-21 11 14 09](https://user-images.githubusercontent.com/1261904/37872325-4bd332c2-3040-11e8-90c8-bfc3d070d0ef.png)

회원가입을 하고 로그인을 하면 아래와 같이 Docker image 저장소인 Docker Hub에서 Docker image를 검색할 수 있는 화면이 나온다.
![2018-03-21 11 15 23](https://user-images.githubusercontent.com/1261904/37872326-4bfabafe-3040-11e8-9db3-c175764a4497.png)

jenkins가 공식적으로 제공해주는 Docker 이미지 검색하고, Create 버튼을 눌러서 이미지 pull 받고 실행한다.
![2018-03-21 11 20 07](https://user-images.githubusercontent.com/1261904/37872384-9adfff16-3041-11e8-9d43-558da438bb03.png)

정상적으로 진행이 완료되면 아래와 같은 화면을 볼 수 있고, WEB PREVIEW 아래의 화면을 누르거나 http://localhost:32769 로 접속한다  
![2018-03-21 11 25 08](https://user-images.githubusercontent.com/1261904/37872328-4c46581a-3040-11e8-8d8e-552950c42a71.png)

설치를 진행하려면 아래와 같이 Jenkins의 administrator 패스워드를 입력해야 한다. 해당 패스워드는 Jenkins 컨테이너 내부의 `/var/jenkins_home/secrets/initialAdminPassword`에 존재한다.
외부에서 컨테이너 내부의 파일을 읽기 위해서 먼저 컨테이너의 ID 또는 NAME을 확인해야 한다. 확인 방법은 아래와 같이 2가지 방법이 있다.

먼저 Kitematic에서 확인하는 방법이다.
![2018-03-21 11 52 54](https://user-images.githubusercontent.com/1261904/37872471-9542345a-3043-11e8-85e6-7ada0b97b91a.png)

그리고 터미널에서 `docker ps` 명령어를 통해서 확인하는 방법이다.
![2018-03-21 11 57 44](https://user-images.githubusercontent.com/1261904/37872472-956d3efc-3043-11e8-9643-76c297648fdb.png)

실행중인 Jenkins 컨테이너의 ID 또는 NAME을 확인한 뒤에 아래의 shell 명령어를 통해서 컨테이너 내부의 패스워드 파일을 읽을 수 있다.
  
```
# 외부에서 컨테이너 내부의 명령을 실행하는 exec 명령. 컨테이너 안의 /bin/bash를 실행시킨다.
$ docker exec -i -t jenkins /bin/bash

# 컨테이너 내부의 패스워드 파일을 읽는다.
$ cat /var/jenkins_home/secrets/initialAdminPassword
c977cc7aa46e4d8fa8d2bd64043f4fde
```

그러면 설치 방법 2가지 중에서 하나를 선택할 수 있다. 
install suggested plugins은 많이 사용되는 플러그인들이 자동으로 설치되는 것이고, select plugins to install은 필요한 플러그인들만 선택해서 설치가 가능하다.
나는 install suggested plugins로 설치를 진행했다.
![2018-03-22 12 08 30](https://user-images.githubusercontent.com/1261904/37872512-67fb34a0-3044-11e8-82c6-105edbe2b70b.png)

설치 완료 후 관리자 계정 정보를 생성하면 아래와 같이 정상적으로 접속이 된다.
![2018-03-22 12 19 14](https://user-images.githubusercontent.com/1261904/37872532-bd8d87b0-3044-11e8-971c-5c8f9dd4cfb2.png)

# 5. Jenkins Job 생성 및 Github 연동하기

## 5.1. Jenkins Job 생성하기

오른쪽위에 새로운 Item을 선택한다. 
![2018-03-22 12 19 14](https://user-images.githubusercontent.com/1261904/37876225-5efc040c-3084-11e8-97fa-7ad4ab66de63.png)
 
item name을 입력하고 Freestyle Project를 선택해서 새로운 Job을 생성해준다.
![2018-03-26 9 38 27](https://user-images.githubusercontent.com/1261904/37906834-43e438e6-313e-11e8-9d55-7be91c117e4e.png) 

Jenkins로 작업할 프로젝트 하나가 필요한데, Spring Boot 2.0 기반으로 Swagger를 적용한 프로젝트를 하나 만들었다. 
https://github.com/umsh86/demo 에 접속해서 아래와 같이 Repository URL을 Copy 한다.
![2018-03-26 9 48 50](https://user-images.githubusercontent.com/1261904/37907280-94ac5708-313f-11e8-9055-26d14e024a73.png)
  
Jenkins에서 소스관리 > Git > Repository URL에 복사한 URL을 넣고, Add 버튼을 클릭한다.
![2018-03-26 9 53 12](https://user-images.githubusercontent.com/1261904/37907506-45553476-3140-11e8-8053-5d2134cc1da3.png)

자격 증명을 하는 방법에는 여러가지가 있는데, 여기서는 Github 계정과 패스워드를 사용했다.  
![2018-03-26 12 01 25](https://user-images.githubusercontent.com/1261904/37876551-fc2ae71c-3088-11e8-9f1d-1b1a2acd2a9d.png)

Credentials에 입력한 계정정보가 확인되고, 그 아래에 branch는 기본 `*/master`로 되어있다. master branch에 push가 되면 Jenkins가 자동으로 build가 될 것이다. 만약 Git Flow를 사용하고 있고 develop branch에 push 될 때 build가 되기를 원한다면 branch에 `*/develop`으로 변경해주면 된다. 
![2018-03-26 12 04 59](https://user-images.githubusercontent.com/1261904/37876582-71a5b4cc-3089-11e8-9e6e-6ddfadfa8059.png)

빌드 유발은 Github의 master branch가 push 될 때 해야되므로, `Github hook trigger for GITScm polling`을 선택한다.
만약 자동으로 특정 시간별로 빌드를 하고 싶다면, `Build periodically`을 선택해서 cron expression을 사용해서 등록해주면 된다.
![2018-03-26 12 27 06](https://user-images.githubusercontent.com/1261904/37876787-ab1ff0ca-308c-11e8-99b1-39bfd7c61b26.png)

그리고 Github의 develop 브랜치에 push 되었을 때 실행할 빌드 명령어를 입력해주기 위해서, Build > Add Build Step > Execute shell 을 선택한다.
![2018-03-26 9 58 38](https://user-images.githubusercontent.com/1261904/37907769-00f35730-3141-11e8-87f9-c0e89e4a8f06.png)

프로젝트에 별도의 그레이들 설치나 환경 설정 없이도 그레이들을 사용할 수 있도록 해주는 gradlew(윈도우는 gradlew.bat)이 있으므로,
아래와 같이 실행할 gradle 명령어를 작성하고 저장한다.
![2018-03-26 11 06 12](https://user-images.githubusercontent.com/1261904/37911206-5e932fc4-314a-11e8-869f-fbc84e00178b.png)

그러면 아래와 같이 프로젝트가 생성된 것을 볼 수 있다. 여기서 프로젝트의 이름을 선택해서
![2018-03-26 11 37 26](https://user-images.githubusercontent.com/1261904/37913183-411b8590-314f-11e8-8b01-c2519c60efe9.png)

Build를 할 수 있고, Build History에 진행중인 작업 내용을 확인할 수 있다.
![2018-03-26 11 37 57](https://user-images.githubusercontent.com/1261904/37913184-41471d18-314f-11e8-8dfe-80859a487868.png)

## 5.2. Github 연동하기

위에서 설정했던 Github 저장소의 develop branch에 push가 일어났을 때, 자동으로 build가 되도록 설정하는 방법이다.
해당 프로젝트의 오른쪽 위의 Settings > Integration & Service > Add service > Jenkins(GitHub plugin) 을 선택한다.
![2018-03-27 12 29 20](https://user-images.githubusercontent.com/1261904/37916139-141094e4-3156-11e8-99c4-4c505bcd19ef.png)

아래의 Jenkins hook url에 외부에서 접근할 수 있는 url을 입력해주면 된다.
![2018-03-27 12 23 22](https://user-images.githubusercontent.com/1261904/37945141-b33c559a-31b9-11e8-93f9-5a191169afa5.png)

현재 로컬 네트워크에서 작업중이기 때문에 외부에서 접근할 수 없으므로, `ngrok`(https://ngrok.com)을 사용해서 로컬 네트워크를 외부에서 접속할 수 있게 설정해주면 된다.

https://ngrok.com 에 접속해서 위에 Download를 선택하여, ngrok을 다운로드 받는다.
압축을 해제하고 자신의 PATH(예를 들면 /usr/local/bin)으로 옮긴 후 터미널을 통해서 아래와 같이 ngrok을 입력하면 사용법에 대해서 나온다.

```
$ ngrok                                                                                                                             ✓  592  12:16:13
NAME:
   ngrok - tunnel local ports to public URLs and inspect traffic

DESCRIPTION:
    ngrok exposes local networked services behinds NATs and firewalls to the
    public internet over a secure tunnel. Share local websites, build/test
    webhook consumers and self-host personal services.
    Detailed help for each command is available with 'ngrok help <command>'.
    Open http://localhost:4040 for ngrok's web interface to inspect traffic.

EXAMPLES:
    ngrok http 80                    # secure public URL for port 80 web server
    ngrok http -subdomain=baz 8080   # port 8080 available at baz.ngrok.io
    ngrok http foo.dev:80            # tunnel to host:port instead of localhost
    ngrok tcp 22                     # tunnel arbitrary TCP traffic to port 22
    ngrok tls -hostname=foo.com 443  # TLS traffic for foo.com to port 443
    ngrok start foo bar baz          # start tunnels from the configuration file

VERSION:
   2.2.8

AUTHOR:
  inconshreveable - <alan@ngrok.com>

COMMANDS:
   authtoken	save authtoken to configuration file
   credits	prints author and licensing information
   http		start an HTTP tunnel
   start	start tunnels by name from the configuration file
   tcp		start a TCP tunnel
   tls		start a TLS tunnel
   update	update ngrok to the latest version
   version	print the version string
   help		Shows a list of commands or help for one command
```

아래와 같이 외부에서 접근 가능하도록 포트 번호를 입력해주면 된다

```
$ ngrok http 32679
ngrok by @inconshreveable                                                                                                                   (Ctrl+C to quit)

Session Status                online
Session Expires               7 hours, 58 minutes
Version                       2.2.8
Region                        United States (us)
Web Interface                 http://127.0.0.1:4040
Forwarding                    http://c703aac0.ngrok.io -> localhost:32797
Forwarding                    https://c703aac0.ngrok.io -> localhost:32797

Connections                   ttl     opn     rt1     rt5     p50     p90
                              6       0       0.02    0.02    8.09    9.09

HTTP Requests
-------------
```

그러면 외부에서 `http://c703aac0.ngrok.io`라는 주소로 접근이 가능해진다.
이 주소를 뒤에 `/github-webhook/`을 붙여서 `http://c703aac0.ngrok.io/github-webhook/`을 위에서 이야기했던 Jenkins hook url에 입력하고 Add Service를 선택해서 완료한다.

그리고 실제로 develop branch에 push가 일어났을 때를 정상적으로 동작하는지를 테스트하기 위해서 `README.md` 파일을 생성했고, push를 하였다.
그러면 아래와 같이 Jenkins Job이 실행되는 것을 볼 수 있다.
![2018-03-27 12 34 36](https://user-images.githubusercontent.com/1261904/37945502-8b096246-31bb-11e8-92c7-0c2870c6c051.png)

위에서 Item(Job)이름을 클릭해서 들어가면 GitHub push에 의해서 작업이 시작되었다는 것을 알 수 있다.
![2018-03-27 12 35 17](https://user-images.githubusercontent.com/1261904/37945504-8b381280-31bb-11e8-80f0-867878b5b4e1.png)

# 6. Slack에 Jenkins Build 메시지 받도록 설정하기

먼저 slack에서 왼쪽 위의 Channel 옆에 + 버튼을 눌러서 새로운 채널을 생성한다.
![2018-03-27 12 44 14](https://user-images.githubusercontent.com/1261904/37945878-62dda802-31bd-11e8-8c6f-d953c2524b5a.png)

Jenkins Build 메시지를 받을 채널 이름을 입력한다.
![2018-03-27 12 44 22](https://user-images.githubusercontent.com/1261904/37945929-b72b987e-31bd-11e8-87ed-7fd8953afb7f.png)

새로운 채널이 생성되고 나면 왼쪽 아래에 `Apps` 옆에 + 를 클릭한다.
![2018-03-27 12 54 26](https://user-images.githubusercontent.com/1261904/37946030-2a27bef2-31be-11e8-8372-9af5f7836b40.png)

아래와 같이 Jenkins를 검색해서 Jenkins CI 브라우저 앱을 설치한다.
![2018-03-27 12 45 12](https://user-images.githubusercontent.com/1261904/37967915-b54f05a0-3207-11e8-8fad-e083d710414d.png)

아래에서 Install을 클릭하고
![2018-03-27 12 45 38](https://user-images.githubusercontent.com/1261904/37969088-e7c8eaa2-320a-11e8-9ba3-bd06a05a2e67.png)

앞에서 생성한 Jenkins Build Message를 받을 채널을 선택해준다.
![2018-03-27 12 46 30](https://user-images.githubusercontent.com/1261904/37969366-7c817c0e-320b-11e8-83f5-34c401c92335.png)

그러면 Jenkins에서 어떤 작업들을 추가로 해줘야하는지에 대해서 자세한 설명이 나오는데, 그대로 똑같이 진행해주면 된다.
Step을 간단하게 정리하면, Jenkins 관리 > 플러그인 관리 > 설치 가능에서 Slack Notification Plugin 검색 및 설치한다.
그리고 나서 Jenkins 관리 > 시스템설정 > Global Slack Notifier Settings를 찾아서 Base URL과 Integration Token 정보를 입력해주면 된다.

마지막으로 프로젝트의 구성 > 빌드 후 조치 추가 > Slack Notification을 선택한다.
![2018-03-27 11 34 38](https://user-images.githubusercontent.com/1261904/38006862-c6271e42-3281-11e8-8073-f7a0005c1823.png)

그리고 Slack으로 Noti를 종류를 선택해주면 되는데, Success 와 Fail에 대해서만 받기로 선택하였다.
![2018-03-27 11 35 20](https://user-images.githubusercontent.com/1261904/38006916-fd94a53e-3281-11e8-940e-0eb13689ed3f.png)

그리고 테스트를 위해서 `README.md` 파일을 수정했고, develop branch에 push를 진행하였다.
Jenkins가 다시 `./gradlew build`가 실행되는 것을 볼 수 있고, Slack의 Jenkins 채널에도 메시지가 출력되는 것을 볼 수 있다.
![2018-03-28 12 22 22](https://user-images.githubusercontent.com/1261904/38007099-b5b66134-3282-11e8-88a9-118c3bc85ffd.png)

# 정리

이렇게 해서 Docker를 가지고 Jenkins를 설치하고, GitHub 프로젝트와 연동해서 PUSH가 일어나면 자동으로 빌드가 진행되고, Slack을 통해서 작업 완료/실패에 대한 알림을 받는 부분까지 완료하였다.

이번주 주말에 진행될 모임에서 이 자료를 가지고 좋은 시간을 보내길 기대해본다.
 
 