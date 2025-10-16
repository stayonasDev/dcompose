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
