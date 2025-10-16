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

