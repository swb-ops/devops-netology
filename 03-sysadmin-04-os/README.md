### 3.4. Операционные системы, лекция 2

---
1.	На лекции мы познакомились с node_exporter. В демонстрации его исполняемый файл запускался в background. Этого достаточно для демо, но не для настоящей production-системы,  
где процессы должны находиться под внешним управлением. Используя знания из лекции по systemd, создайте самостоятельно простой unit-файл для node_exporter:  
o	поместите его в автозагрузку, o	предусмотрите возможность добавления опций к запускаемому процессу через внешний файл (посмотрите, например, на systemctl cat cron),
o	удостоверьтесь, что с помощью systemctl процесс корректно стартует, завершается, а после перезагрузки автоматически поднимается.
```
#wget https://github.com/prometheus/node_exporter/releases/download/v1.1.2/node_exporter-1.1.2.linux-amd64.tar.gz

#tar xvfz node_exporter-*.*-amd64.tar.gz

#cd node_exporter-1.1.2.linux-amd64

# ls
LICENSE  node_exporter  NOTICE

#cp node_exporter /usr/local/bin

# useradd --system --shell /bin/false node_exporter

# > /etc/system/system/node_exporter
[Unit]
Description=Node Exporter

[Service]
User=node_exporter
Group=node_exporter
EnvironmentFile=-/etc/sysconfig/node_exporter
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target

# systemctl daemon-reload
# systemctl start node_exporter
# systemctl status node_exporter  
● node_exporter.service - Node Exporter
     Loaded: loaded (/etc/systemd/system/node_exporter.service; disabled; vendor preset: enabled)
     Active: active (running) since Fri 2021-07-02 16:43:09 UTC; 8s ago
   Main PID: 534027 (node_exporter)
      Tasks: 4 (limit: 4617)
     Memory: 2.3M
     CGroup: /system.slice/node_exporter.service
             └─534027 /usr/local/bin/node_exporter
# systemctl enable node_exporter
```

#### Доработка - передача доп аргумента.
add to service:
```
[Service]
EnvironmentFile=/etc/.progconfta
ExecStart=/usr/local/bin/node_exporter $ARG1

# cat /etc/.progconfta
ARG1=--version

systemctl status node_exporter $ARG1
● node_exporter.service - Node Exporter
     Loaded: loaded (/etc/systemd/system/node_exporter.service; enabled; vendor preset: enabled)
     Active: inactive (dead) since Mon 2021-07-05 09:03:21 UTC; 53s ago
    Process: 32048 ExecStart=/usr/local/bin/node_exporter $ARG1 (code=exited, status=0/SUCCESS)
   Main PID: 32048 (code=exited, status=0/SUCCESS)

Jul 05 09:03:21 vshchepkin systemd[1]: Started Node Exporter.
Jul 05 09:03:21 vshchepkin node_exporter[32048]: node_exporter, version 1.1.2 (branch: HEAD, revision: b597c1244d7bef49e6f3359c87a56dd7707f6719)
Jul 05 09:03:21 vshchepkin node_exporter[32048]:   build user:       root@f07de8ca602a
Jul 05 09:03:21 vshchepkin node_exporter[32048]:   build date:       20210305-09:29:10
Jul 05 09:03:21 vshchepkin node_exporter[32048]:   go version:       go1.15.8
Jul 05 09:03:21 vshchepkin node_exporter[32048]:   platform:         linux/amd64
Jul 05 09:03:21 vshchepkin systemd[1]: node_exporter.service: Succeeded.
```

----
2.	Ознакомьтесь с опциями node_exporter и выводом /metrics по-умолчанию. Приведите несколько опций, которые вы бы выбрали для базового мониторинга хоста по CPU,  
памяти, диску и сети.

cpu, cpufreq, diskstats, filesystem, loadavg, meminfo, netdev, netstat

---
3.	Установите в свою виртуальную машину Netdata. Воспользуйтесь готовыми пакетами для установки (sudo apt install -y netdata). После успешной установки:
o	в конфигурационном файле /etc/netdata/netdata.conf в секции [web] замените значение с localhost на bind to = 0.0.0.0,
o	добавьте в Vagrantfile проброс порта Netdata на свой локальный компьютер и сделайте vagrant reload:
config.vm.network "forwarded_port", guest: 19999, host: 19999
После успешной перезагрузки в браузере на своем ПК (не в виртуальной машине) вы должны суметь зайти на localhost:19999. Ознакомьтесь с метриками, которые по умолчанию
собираются Netdata и с комментариями, которые даны к этим метрикам.

![alt text](/net_data.png "Net_data Shchepkin")

---
4.	Можно ли по выводу dmesg понять, осознает ли ОС, что загружена не на настоящем оборудовании, а на системе виртуализации?

Осознает:
```
#dmesg -T | more
[Fri Jun 18 17:49:08 2021] DMI: VMware, Inc. VMware Virtual Platform/440BX Desktop Reference Platform, BIOS 6.00 04/05/2016
[Fri Jun 18 17:49:08 2021] vmware: hypercall mode: 0x00
[Fri Jun 18 17:49:08 2021] Hypervisor detected: VMware
[Fri Jun 18 17:49:08 2021] vmware: TSC freq read from hypervisor : 2261.255 MHz
[Fri Jun 18 17:49:08 2021] vmware: Host bus clock speed read from hypervisor : 66000000 Hz
[Fri Jun 18 17:49:08 2021] vmware: using sched offset of 12213924848 ns
```

---
5.	Как настроен sysctl fs.nr_open на системе по-умолчанию? Узнайте, что означает этот параметр. Какой другой существующий лимит не позволит достичь
такого числа (ulimit --help)?

Ограничение на количество открытых дескрипторов в системе для пользователя.
```
# sysctl -a | grep open
fs.nr_open = 1048576

# ulimit -a
open files                      (-n) 1024
```

---
6.	Запустите любой долгоживущий процесс (не ls, который отработает мгновенно, а, например, sleep 1h) в отдельном неймспейсе процессов; покажите, что ваш процесс
работает под PID 1 через nsenter. Для простоты работайте в данном задании под root (sudo -i). Под обычным пользователем требуются дополнительные опции (--map-root-user) и т.д.
```
root@vshchepkin:/# unshare -f --pid --mount-proc /bin/bash
root@vshchepkin:/# ps aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.1  0.0   7368  3996 pts/3    S    13:33   0:00 /bin/bash
root           8  0.0  0.0   8892  3372 pts/3    R+   13:33   0:00 ps aux

root@vshchepkin:/# sleep 1h &
[1] 11
root@vshchepkin:/# ps aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  0.0   7368  3996 pts/3    S    13:33   0:00 /bin/bash
root          11  0.0  0.0   5476   596 pts/3    S    13:34   0:00 sleep 1h
root          12  0.0  0.0   8892  3292 pts/3    R+   13:34   0:00 ps aux
```

---
7.	Найдите информацию о том, что такое :(){ :|:& };:. Запустите эту команду в своей виртуальной машине Vagrant с Ubuntu 20.04
(это важно, поведение в других ОС не проверялось). Некоторое время все будет "плохо", после чего (минуты) – ОС должна стабилизироваться. Вызов dmesg расскажет,
какой механизм помог автоматической стабилизации. Как настроен этот механизм по-умолчанию, и как изменить число процессов, которое можно создать в сессии?


Определяет функцию с именем : , которая порождает саму себя и создает фон
```
vshchepkin:~$ :(){ :|:& };:
-bash: fork: Resource temporarily unavailable
-bash: fork: Resource temporarily unavailable
…
```
После 30 минут ожидания результата не увидел. Ubuntu 20.04 VmWare.

