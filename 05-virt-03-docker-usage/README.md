## 5.3. Контейнеризация на примере Docker
---
### Задача 1
Посмотрите на сценарий ниже и ответьте на вопрос: "Подходит ли в этом сценарии использование докера? Или лучше подойдет виртуальная машина, физическая машина?
Или возможны разные варианты?" Детально опишите и обоснуйте свой выбор.

Сценарий:

- Высоконагруженное монолитное java веб-приложение - физический сервер, необходимости в микросервисах нет;
- Go-микросервис для генерации отчетов - docker подходит;
- Nodejs веб-приложение - docker подходит;
- Мобильное приложение c версиями для Android и iOS - docker ?!;
- База данных postgresql используемая, как кэш - физическая машина или виртуалка;
- Шина данных на базе Apache Kafka - kafka мощный брокер сообщений, возможно лучше размещать на виртуалке, хотя в сети пишут, что норм размещают в контейнерах;
- Очередь для Logstash на базе Redis - сам logstash, думаю можно без проблем размещать в контейнере, redis в виртуалке, ближе в RAM;
- Elastic stack для реализации логирования продуктивного веб-приложения - три ноды elasticsearch, два logstash и две ноды kibana -
logstash и kibana, думаю без проблем можно размещать в контейнерах, с нодами elastic не уверен, они используют в большом объеме RAM, возможно лучше в запустить их в виртуалках;
- Мониторинг-стек на базе prometheus и grafana - docker;
- Mongodb, как основное хранилище данных для java-приложения - на лекциях говорили, что лучше размещать БД на физических серверах, поэтому может зависеть от нагрузки,
возможно подойдет виртуалка;
- Jenkins-сервер - docker.

---
### Задача 2

Сценарий выполения задачи:

- создайте свой репозиторий на докерхаб;
- выберете любой образ, который содержит апачи веб-сервер;
- создайте свой форк образа;
- реализуйте функциональность: запуск веб-сервера в фоне с индекс-страницей, содержащей HTML-код ниже:
```
<html>
<head>
Hey, Netology
</head>
<body>
<h1>I’m kinda DevOps now</h1>
</body>
</html>
```
Опубликуйте созданный форк в своем репозитории и предоставьте ответ в виде ссылки на докерхаб-репо.

```
root@vshchepkin:/home/vshchepkin# docker pull httpd
root@vshchepkin:/home/vshchepkin# docker run --name apache -d httpd
root@vshchepkin:/home/vshchepkin# docker ps
CONTAINER ID   IMAGE     COMMAND              CREATED         STATUS         PORTS     NAMES
1d3ac5e08de0   httpd     "httpd-foreground"   4 seconds ago   Up 2 seconds   80/tcp    apache

root@vshchepkin:/home/vshchepkin/docker# > site.html
root@vshchepkin:/home/vshchepkin/docker# cat site.html 
<html>
<head>
Hey, Netology
</head>
<body>
<h1>I’m kinda DevOps now</h1>
</body>
</html>

root@vshchepkin:/home/vshchepkin/docker# > Dockerfile
root@vshchepkin:/home/vshchepkin/docker# cat Dockerfile
FROM httpd

COPY site.html /usr/local/apache2/htdocs
MAINTAINER swbops <vladimir.shchepkin@ya.ru>

root@vshchepkin:/home/vshchepkin/docker# docker build -t swbops/httpd_fork .
root@vshchepkin:/home/vshchepkin/docker# docker push swbops/httpd_fork         
Using default tag: latest
The push refers to repository [docker.io/swbops/httpd_fork]
2094028afc46: Pushed 
c1d6519b2482: Mounted from library/httpd 
ef38b43e89e8: Mounted from library/httpd 
7182dd4dd207: Mounted from library/httpd 
d24d666b2229: Mounted from library/httpd 
f68ef921efae: Mounted from library/httpd 
latest: digest: sha256:ec37af06a56bb6fbe82b80766a55d631ab49ddf8df48ad9922e48158a4ad6ea9 size: 1573

root@vshchepkin:/home/vshchepkin/docker# docker image ls 
REPOSITORY                    TAG       IMAGE ID       CREATED         SIZE
swbops/httpd_fork             latest    a05964f35e56   7 minutes ago   138MB
httpd                         latest    c8ca530172a8   12 days ago     138MB
gcr.io/k8s-minikube/kicbase   v0.0.25   8768eddc4356   8 weeks ago     1.1GB

root@vshchepkin:/home/vshchepkin/docker# docker run --name apache_fork -p 8080:80 -d swbops/httpd_fork
6f01352c8d7b19f9ed06229569e4b67842e02eb684e3373a8098578adc401d7f
root@vshchepkin:/home/vshchepkin/docker# docker ps
CONTAINER ID   IMAGE               COMMAND              CREATED             STATUS             PORTS                                   NAMES
6f01352c8d7b   swbops/httpd_fork   "httpd-foreground"   9 seconds ago       Up 3 seconds       0.0.0.0:8080->80/tcp, :::8080->80/tcp   apache_fork
1d3ac5e08de0   httpd               "httpd-foreground"   About an hour ago   Up About an hour   80/tcp                                  apache
root@vshchepkin:/home/vshchepkin/docker# docker exec -it apache_fork /bin/bash
root@6f01352c8d7b:/usr/local/apache2# ls /usr/local/apache2/htdocs
index.html  site.html
root@6f01352c8d7b:/usr/local/apache2# cat site.html
<html>
<head>
Hey, Netology
</head>
<body>
<h1>I’m kinda DevOps now</h1>
</body>
</html>

```
![alt text](/pictures/site_05_03.png "httpd fork")


---
### Задача 3

- Запустите первый контейнер из образа centos c любым тэгом в фоновом режиме, подключив папку info из текущей рабочей директории на хостовой машине в /share/info контейнера;
- Запустите второй контейнер из образа debian:latest в фоновом режиме, подключив папку info из текущей рабочей директории на хостовой машине в /info контейнера;
- Подключитесь к первому контейнеру с помощью exec и создайте текстовый файл любого содержания в /share/info ;
- Добавьте еще один файл в папку info на хостовой машине;
- Подключитесь во второй контейнер и отобразите листинг и содержание файлов в /info контейнера.

```
root@vshchepkin:/home/vshchepkin/docker# docker run -t -d --name my_centos -v /home/vshchepkin/docker/info/:/share/info centos
root@vshchepkin:/home/vshchepkin/docker# docker ps -a
CONTAINER ID   IMAGE               COMMAND              CREATED          STATUS          PORTS                                   NAMES
32b1a8ae8a31   centos              "/bin/bash"          2 seconds ago    Up 1 second                                             my_centos
6f01352c8d7b   swbops/httpd_fork   "httpd-foreground"   35 minutes ago   Up 35 minutes   0.0.0.0:8080->80/tcp, :::8080->80/tcp   apache_fork
1d3ac5e08de0   httpd               "httpd-foreground"   2 hours ago      Up 2 hours      80/tcp                                  apache
root@vshchepkin:/home/vshchepkin/docker# docker exec -it my_centos /bin/bash
[root@32b1a8ae8a31 /]# ls /share/     
info
[root@32b1a8ae8a31 /]# echo 'hello guys'>/share/info/file1.txt
[root@32b1a8ae8a31 /]# cat /share/info/file1.txt
hello guys

root@vshchepkin:/home/vshchepkin/docker# cat info/file1.txt 
hello guys

root@vshchepkin:/home/vshchepkin/docker# docker run -t -d --name my_debian -v /home/vshchepkin/docker/info/:/info debian            
Unable to find image 'debian:latest' locally
latest: Pulling from library/debian
4c25b3090c26: Pull complete 
Digest: sha256:38988bd08d1a5534ae90bea146e199e2b7a8fca334e9a7afe5297a7c919e96ea
Status: Downloaded newer image for debian:latest
9465220b604fd19b9b3e530d45c1cbee7a581036313c90c55c75d557a1513175
root@vshchepkin:/home/vshchepkin/docker# docker ps -a
CONTAINER ID   IMAGE               COMMAND              CREATED          STATUS          PORTS                                   NAMES
9465220b604f   debian              "bash"               32 seconds ago   Up 13 seconds                                           my_debian
32b1a8ae8a31   centos              "/bin/bash"          7 minutes ago    Up 7 minutes                                            my_centos
6f01352c8d7b   swbops/httpd_fork   "httpd-foreground"   42 minutes ago   Up 42 minutes   0.0.0.0:8080->80/tcp, :::8080->80/tcp   apache_fork
1d3ac5e08de0   httpd               "httpd-foreground"   2 hours ago      Up 2 hours      80/tcp                                  apache

root@vshchepkin:/home/vshchepkin/docker/info# echo 'by guys'>file2.txt
root@vshchepkin:/home/vshchepkin/docker/info# ls
file1.txt  file2.txt

root@9465220b604f:/# cat /info/file1.txt /info/file2.txt
hello guys
by guys

```
