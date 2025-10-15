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
