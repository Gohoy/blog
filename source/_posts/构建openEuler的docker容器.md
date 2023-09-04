---
title: openEuler docker container构建
category: docker
tag: 
 - docker
 - 教程
 - contianer
 - openEuler
---

# openeuler container 构建

## 运行官方的docker镜像

```shell
 docker run --name euler1 --network host -it  openeuler/openeuler:22.03 /bin/bash 
```

不指定 --network 的时候会连不上网，原因暂且未知，但是k8s起一个pod可以正常联网

## 安装包

```shell
yum install -y openssh-server openssh-clients lsof passwd gdb make ncurses-devel flex bison openssl openssl-devel elfutils-libelf-devel binutils binutils-devel
```

## 配置ssh

```shell
 ssh-keygen -A
 /usr/sbin/sshd
 lsof -i:22
 passwd //修改root密码为123456
```

## 将kernel源码移入container内

```shell
docker cp ./kernel euler:/home/
```

## 在内部复制一份内核文件。make clean 后进行make 确保make编译环境没有问题。

## 内部头文件环境也完整

![](http://gohoy.top/i/2023/08/11/s99838-1.png)

## 导出镜像包

```shell
docker export euler1 -o /home/gohoy/docker-img/euler-container.tar
```

## 导入镜像包，查看是否成功保存快照

```shell
cat /home/gohoy/docker-img/euler-container.tar | docker import - openeuler/container:v1
```

```shell
docker run --name euler_container --it openeuler/container:v1 /bin/bash
```

成功进入镜像中，且环境完整。

## 最终镜像大小4.56gb 其中kernel源代码3.6gb

![](http://gohoy.top/i/2023/08/11/s994bn-1.png)

## 脚本以及配置文件

```pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: container
  labels:
    stl: container
spec:
  containers:
    - name: container
      command:
        - "/bin/bash"
        - "-ce"
        - "tail -f /dev/null"
      lifecycle:
        postStart:
          exec:
            command: ["/usr/sbin/sshd"] #启动container运行sshd
      resources:
        limits:
          memory: 1G
          cpu: 1
      image: openeuler/container:v2
```

```shell
  for (( i=1; i <= $1 ; i++ ))
  do
    container=($(kubectl get pods | sed -n '2,$p'| gawk '{print $1}'))
     if echo "${container[@]}" | grep -wq "container-$i"
     then
       continue
     else
       ((ssh=30000+$i))
       sed -e "4s/service/service-$i/; 10s/30000/$ssh/; 13s/container/container-$i/" /home/gohoy/docker-img/service.yaml > /home/gohoy/docker-img/tmpservice.yaml #service
       kubectl create -f  /home/gohoy/docker-img/tmpservice.yaml
       sed -e "4s/container/container-$i/; 6s/container/container-$i/; 9s/container/container-$i/"  /home/gohoy/docker-img/pod.yaml >  /home/gohoy/docker-img/tmppod.yaml # 
       kubectl apply -f  /home/gohoy/docker-img/tmppod.yaml

     fi
  done
```
