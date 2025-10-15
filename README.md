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
