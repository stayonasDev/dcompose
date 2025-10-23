# 도커 네트워크 1
```bash
$ docker build -t myblog:1.1.0 docker_file/httpd/
$ docker run -dit --name myblog-1 -p 8051:80 myblog:1.1.0
$ docker run -dit --name myblog-2 -p 8052:80 myblog:1.1.0

$ docker build -t nginx_lb:1.1.0 docker_file/nginx/ 
$ docker run --name nginx_lb-1 -d -p 9051:80 --link myblog-1 --link myblog-2 nginx_lb:1.1.0

# http://localhost:9091 접근 시 ALB 정상 작동
$ sudo docker logs -f myblog-1
$ sudo docker logs -f myblog-2
```

# 도커 네트워크 2
```bash
# 기존 ps 삭제 후
$ docker network create blog-net
$ docker run -dit --name myblog-1 -p 8051:80 a2blog:251014.1
$ docker run -dit --name myblog-2 -p 8052:80 a2blog:251014.1
$ docker network connect blog-net myblog-1
$ docker network connect blog-net myblog-2
$ docker run --name nginx_lb-1 -d -p 9052:80 nginx_lb:251014.2
# 1-1 방법
$  docker network connect blog-net nginx_lb-1
$ docker start nginx_lb-1
# 1-2 방법
$ docker run --name nginx_lb-1 -d -p 9052:80 --network blog-net nginx_lb:251014.2
```

# Rolling Deploy (Update)
```bash
$ vim docker_file/httpd/Dockerfile
# RUN 부분에서 clone 할 git 변경
$ docker build -t myblog:2.0.0 docker_file/httpd/
$ docker stop myblog-1; docker rm myblog-1
$ docker run -dit --name myblog-1 -p 8051:80 myblog:2.0.0
```
- myblog-2
<img width="1920" height="1020" alt="image" src="https://github.com/user-attachments/assets/9d568dbe-82d1-4c92-b8b1-fc3bb7d52e1a" />

- myblog-1 
<img width="1920" height="1020" alt="image" src="https://github.com/user-attachments/assets/21051233-433d-4051-98a6-84f0c5a671a4" />

``` bash
# 배포 확인 후 myblog-2도 변경
$ docker stop myblog-2;docker rm myblog-2
$ docker run -dit --name myblog-2 -p 8052:80 myblog:2.0.0
```


# 블루 그린 배포
```bash
# 초기 정보는 삭제하고 함
$ docker stop myblog-1;docker rm myblog-1
$ docker stop myblog-2;docker rm myblog-2
$ docker stop nginx_lb-1;docker rm nginx_lb-1

#sky-net으로 네트워크 구성
$ docker network create sky-net
$ docker run -dit --name myblog-1 -p 8151:80 --network sky-net myblog:1.1.0
$ docker run -dit --name myblog-2 -p 8152:80 --network sky-net myblog:1.1.0
$ docker run --name nginx_lb-1 -d -p 9051:80 --network sky-net nginx_lb:1.1.

#새로 배포되는 서비스들
$ docker run -dit --name myblog-1-1 --network sky-net myblog:2.0.0
$ docker run -dit --name myblog-1-2 --network sky-net myblog:2.0.0

# nginx 컨테이너의 LB 설정 바꿈
upstream blog_servs {
        #server myblog-1:80;
        #server myblog-2:80;
        server myblog-1-2:80;
        server myblog-2-2:80;
}
# nginx reload로 설정 적용
$ root@96882ed6d08c:/etc/nginx/conf.d# nginx -s reload
  # 2025/10/16 07:40:21 [notice] 299#299: signal process started

# 구버전 삭제
$ docker stop myblog-1 myblog-2
$ docker rm myblog-1 myblog-2
```

# 카나리아 배포
- 여러 개의 서버에서 로드 밸런서 비율을 배포한 서버에 적게 가져가고 점차 높이며 CPU 부하 등 보고 안정적이라면 다시 로드 밸런스를 균등하게 가져가는 방식
- 카나리아 배포가 생긴 이유는 블루/그린 배포에서는 배포된 2 개의 서버를 바로 바꾸기 때문에 문제가 생긴다면 시스템이 전체 장애인데
  카나리아는 배포된 서버를 적은 트래픽으로 테스트하면서 장애가 있는지 확인하고 로드 벨런서 비율을 높이기 때문에 안정적이기 때문에 생긴 것 같다.

---------------------------------------------------
# MSA 시작
```bash
# 컴포즈 생성 조회회
$ $ sudo docker compose -f compose/manual_lb/compose.yml  up -d --build --force-recreate
$ sudo docker compose -f compose/manual_lb/compose.yml ps
```

---
# 문제 해결
- DB 의존성이 있어 모든 의존성을 삭제해도 해결이 되지 않았음 
```bash
2025-10-16T15:59:08.233Z  INFO 1 --- [applcation] [           main] lgcns.lg06.ApplicationApplication        : Starting ApplicationApplication v0.4.0 using Java 17-ea with PID 1 (/app.jar started by spring in /)
2025-10-16T15:59:08.237Z  INFO 1 --- [applcation] [           main] lgcns.lg06.ApplicationApplication        : No active profile set, falling back to 1 default profile: "default"
2025-10-16T15:59:09.342Z  INFO 1 --- [applcation] [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port 8080 (http)
2025-10-16T15:59:09.355Z  INFO 1 --- [applcation] [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2025-10-16T15:59:09.356Z  INFO 1 --- [applcation] [           main] o.apache.catalina.core.StandardEngine    : Starting Servlet engine: [Apache Tomcat/10.1.46]
2025-10-16T15:59:09.386Z  INFO 1 --- [applcation] [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2025-10-16T15:59:09.387Z  INFO 1 --- [applcation] [           main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 1092 ms
2025-10-16T15:59:09.813Z  INFO 1 --- [applcation] [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port 8080 (http) with context path '/'
2025-10-16T15:59:09.842Z  INFO 1 --- [applcation] [           main] lgcns.lg06.ApplicationApplication        : Started ApplicationApplication in 2.169 seconds (process running for 2.82)

$ curl http://localhost:8080/hello
curl: (56) Recv failure: Connection reset by peer
$ curl -X GET http://localhost:8080/hello
curl: (56) Recv failure: Connection reset by peer

# 문제는 도커의 포트 번호가 80이였는데 8080으로 하니 문제가 해결되었다.
# 해당 에러가 발생한 이유를 생각하면 도커 내부의 포트가 인바운드는 8080으로 들어오고 도커의 포트는 80이다.
# 즉 아웃바운드의 포트 번호는 80으로 스프링에게 80 Port로 도착하기 때문에 문제가 생기는 것으로 보인다.  
$ docker run -d --name spring -p 8080:8080 stayonasdev/spring-boot-docker:0.3.1
$ curl -X GET http://localhost:8080/hello         {"timesptamp":1760630425170,"message":"Hello, World!","koreatime":"2025-10-17T01:00:25.175832641+09:00[Asia/Seoul]"}%
```
---
```bash
# 도커 허브에 이미지 푸시 후 compose의 이미지 버전 변경하고 생성
$ docker compose -f compose/spring_lb/compose.yml up -d --build --force-recreate
$  curl http://localhost:9889/hello
{"koreatime":"2025-10-17T09:50:41.743404501+09:00[Asia/Seoul]","message":"Hello, World!","timesptamp":1760662241683}%

```

# ngrinder
<img width="1648" height="232" alt="image" src="https://github.com/user-attachments/assets/d4f7b9a0-e493-43df-81b9-72545fd65edc" />
<img width="1467" height="190" alt="image" src="https://github.com/user-attachments/assets/09737436-2da9-4e5c-93d3-8cb2ac743d1f" />
<img width="1497" height="797" alt="image" src="https://github.com/user-attachments/assets/6601dd6f-fd76-4ba4-9a36-6a7464920184" />   

- Nginx와 Spring Cloud Gateway 비교
- Nginx 변경 후 Nginx만 빌드
```bash
$ docker compose up -d --build nginx-proxy

# Test
$ curl http://localhost:9889/api/nginx/users/hello2
{"message":"CPU-heavy calculation completed.","koreatime":"2025-10-24T01:05:52.240536722+09:00[Asia/Seoul]","calculation_duration_ms":0,"result_snippet":"12201368259911100687012387854230469262535743428031...","input_number":500}%       
$ curl http://localhost:9889/api/nginx/users/hello
{"message":"Hello, Jenkins!","timesptamp":1761235573823,"koreatime":"2025-10-24T01:06:13.823768416+09:00[Asia/Seoul]"}%     
```

- 로그 확인
``` bash
$ docker logs -f awsgoo-sc-user-svc-
```
