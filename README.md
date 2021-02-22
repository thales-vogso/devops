# 搭建devops


搭建devops，有gitlab、jenkins、nginx


## 源


修改configure，设置国mirror，因为国外的镜像太慢了


- Docker中国区官方镜像


https://registry.docker-cn.com


- 网易


http://hub-mirror.c.163.com

 
- 中国科技大学


https://docker.mirrors.ustc.edu.cn


```conf
{
  "registry-mirrors": [
    "https://registry.docker-cn.com",
    "http://hub-mirror.c.163.com",
    "https://docker.mirrors.ustc.edu.cn"
  ]
}
```


windows会提示自动重启，直接确认就行


如果linux需要用命令重启


```bash
# 守护进程重启
sudo systemctl daemon-reload
# 重启docker服务
sudo systemctl restart docker
# 关闭docker
sudo systemctl stop docker
```

## 单次操作


有些公司专门有国外源，或者某个源想指向自己服务器可以单独设置


语法为


```bash
docker pull registry.docker-cn.com/myname/myrepo:mytag
```


比如我们安装centos


```bash
docker pull registry.docker-cn.com/library/centos:8
```


之后我们可能要安装本地源


```bash
docker pull .images/centos-custom
```


# gitlab


## 直接启动


- linux


```bash
sudo docker run --detach \
  --hostname git.vogsoyuclk.com \
  --publish 443:443 --publish 80:80 --publish 22:22 \
  --name gitlab \
  --restart always \
  --volume ./gitlab/config:/etc/gitlab \
  --volume ./gitlab/logs:/var/log/gitlab \
  --volume ./gitlab/data:/var/opt/gitlab \
  gitlab/gitlab-ce:latest
```


- windows


```bash
docker run --detach `
  --hostname git.vogso.com `
  --publish 443:443 --publish 80:80 --publish 22:22 `
  --name gitlab `
  --restart always `
  --volume ./gitlab/config:/etc/gitlab `
  --volume ./gitlab/logs:/var/log/gitlab `
  --volume ./gitlab/data:/var/opt/gitlab `
  gitlab/gitlab-ce:latest
```


要等自动配置完毕，自动配置需要一些时间


然后我们进入这个创建好的容器


查看容器


```bash
docker ps
```


找到容器的id


```
CONTAINER ID   IMAGE                     COMMAND             CREATED          STATUS                    PORTS                                                          NAMES
f416db31c1a3   gitlab/gitlab-ce:latest   "/assets/wrapper"   10 minutes ago   Up 10 minutes (healthy)   0.0.0.0:22->22/tcp, 0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp   gitlab
```

进入容器


```
docker exec -it f416db31c1a3 /bin/bash 
```


重新加载一下配置，顺便重启


```
gitlab-ctl reconfigure
gitlab-ctl restart
```


## yml


使用yml就会非常轻松


```yml
version: '3.5'
services:
  gitlab:
    image: gitlab/gitlab-ce:latest
    container_name: dev-gitlab
    restart: always
    hostname: 'git.vogso.com'
    environment:
      TZ: 'Asia/Shanghai'
    ports:
      - 80:80
      - 443:443
      - 22:22
    volumes:
      - ./gitlab/data:/var/opt/gitlab
      - ./gitlab/logs:/var/log/gitlab
      - ./gitlab/config:/etc/gitlab
```


直接执行就可以了


```bash
docker-compose up -d
```