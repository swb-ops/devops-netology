### 3.8. Компьютерные сети, лекция 3

1.	Подключитесь к публичному маршрутизатору в интернет. Найдите маршрут к вашему публичному IP
telnet route-views.routeviews.org
Username: rviews
show ip route x.x.x.x/32
show bgp x.x.x.x/32

route-views>show ip route 217.25.228.11
Routing entry for 217.25.224.0/20, supernet
  Known via "bgp 6447", distance 20, metric 0
  Tag 6939, type external
  Last update from 64.71.137.241 1w2d ago
  Routing Descriptor Blocks:
  * 64.71.137.241, from 64.71.137.241, 1w2d ago
      Route metric is 0, traffic share count is 1
      AS Hops 2
      Route tag 6939
      MPLS label: none
      
route-views>show bgp 217.25.228.11
BGP routing table entry for 217.25.224.0/20, version 189208585
Paths: (25 available, best #23, table default)
  Not advertised to any peer
  Refresh Epoch 1
  3267 20485 6856
    194.85.40.15 from 194.85.40.15 (185.141.126.1)
      Origin IGP, metric 0, localpref 100, valid, external
      path 7FE028E055F0 RPKI State valid
      rx pathid: 0, tx pathid: 0

---
2.	Создайте dummy0 интерфейс в Ubuntu. Добавьте несколько статических маршрутов. Проверьте таблицу маршрутизации.
```
# modprobe -v dummy numdummies=2
# ip addr add 192.168.100.150/24 dev dummy0
# ip add | grep dummy
11: dummy0: <BROADCAST,NOARP> mtu 1500 qdisc noop state DOWN group default qlen 1000
    inet 192.168.100.150/24 scope global dummy0

# ip rou add 192.168.200.0/24 via 10.63.204.1
# ip rou
default via 10.63.204.1 dev ens192 proto static 
10.63.204.0/25 dev ens192 proto kernel scope link src 10.63.204.35 
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1 linkdown 
192.168.49.0/24 dev br-c946aa256583 proto kernel scope link src 192.168.49.1 
192.168.200.0/24 via 10.63.204.1 dev ens192 
```

---
3.	Проверьте открытые TCP порты в Ubuntu, какие протоколы и приложения используют эти порты? Приведите несколько примеров.
```
# netstat -tapn
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      1054/nginx: master  
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      134180/systemd-reso 
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      1052/sshd: /usr/sbi 
tcp        0      0 127.0.0.1:49153         0.0.0.0:*               LISTEN      954840/docker-proxy 
tcp        0      0 127.0.0.1:49154         0.0.0.0:*               LISTEN      954876/docker-proxy 
tcp        0      0 127.0.0.1:49155         0.0.0.0:*               LISTEN      954889/docker-proxy 
tcp        0      0 127.0.0.1:49156         0.0.0.0:*               LISTEN      954902/docker-proxy 
tcp        0      0 127.0.0.1:49157         0.0.0.0:*               LISTEN      954915/docker-proxy 
tcp        0      0 127.0.0.1:8200          0.0.0.0:*               LISTEN      490981/vault        
tcp        0      0 127.0.0.1:43785         0.0.0.0:*               LISTEN      941202/containerd   
tcp        0      0 127.0.0.1:8201          0.0.0.0:*               LISTEN      490981/vault        
tcp        0    232 10.63.204.35:22         10.5.25.184:50498       ESTABLISHED 821593/sshd: vshche 
tcp6       0      0 :::80                   :::*                    LISTEN      1054/nginx: master  
tcp6       0      0 :::22                   :::*                    LISTEN      1052/sshd: /usr/sbi 
```
Nginx, docker, vault используют TCP порты.

---
4.	Проверьте используемые UDP сокеты в Ubuntu, какие протоколы и приложения используют эти порты?
```
# netstat -aupn
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
udp        0      0 0.0.0.0:37897           0.0.0.0:*                           910/avahi-daemon: r 
udp        0      0 0.0.0.0:5353            0.0.0.0:*                           910/avahi-daemon: r 
udp        0      0 127.0.0.53:53           0.0.0.0:*                           134180/systemd-reso 
udp6       0      0 :::54178                :::*                                910/avahi-daemon: r 
udp6       0      0 :::5353                 :::*                                910/avahi-daemon: r
```
systemd-resolved использует 53 udp порт.

---
5.	Используя diagrams.net, создайте L3 диаграмму вашей домашней сети или любой другой сети, с которой вы работали.
Я рисую в Visio, вот схемка моего маленького ЦОДа  

![alt text](/pictures/scheme_visio.png "Visio") 


---
6*. Установите Nginx, настройте в режиме балансировщика TCP или UDP.

Принимаю сислоги от оборудования и отправляю их на два сервера
```
# cat nginx.conf 
user www-data;
worker_processes 4;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

worker_rlimit_nofile 200000;

events {
        worker_connections 65535;
        multi_accept on;
        use epoll;
}

stream {

### SYSLOG ###
  server {
    listen 192.168.226.23:514 udp;
    proxy_pass logstash_syslog;
#transparent parameter allows outgoing connections to original IP address and port
    proxy_bind $remote_addr transparent;
    proxy_responses 0;
    proxy_timeout 1s;
  }

  upstream logstash_syslog {
#selects server based on source IP address
    hash $remote_addr;
    server 192.168.226.14:514 max_fails=1 fail_timeout=10s;
    server 192.168.226.22:514 max_fails=1 fail_timeout=10s;
  }

```
