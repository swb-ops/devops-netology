"3.3. Операционные системы, лекция 1"

1. Какой системный вызов делает команда cd? В прошлом ДЗ мы выяснили, что cd не является самостоятельной программой, это shell builtin, поэтому запустить strace непосредственно на cd не получится. Тем не менее, вы можете запустить strace на /bin/bash -c 'cd /tmp'. В этом случае вы увидите полный список системных вызовов, которые делает сам bash при старте. Вам нужно найти тот единственный, который относится именно к cd.

$ strace /bin/bash -c 'cd /tmp'
chdir("/tmp")

2. Попробуйте использовать команду file на объекты разных типов на файловой системе.
Например:
vagrant@netology1:~$ file /dev/tty
/dev/tty: character special (5/0)
vagrant@netology1:~$ file /dev/sda
/dev/sda: block special (8/0)
vagrant@netology1:~$ file /bin/bash
/bin/bash: ELF 64-bit LSB shared object, x86-64
Используя strace выясните, где находится база данных file на основании которой она делает свои догадки.


stat("/home/vshchepkin/.magic.mgc", 0x7ffc3eab9600) = -1 ENOENT (No such file or directory)
stat("/home/vshchepkin/.magic", 0x7ffc3eab9600) = -1 ENOENT (No such file or directory)
stat("/etc/magic", {st_mode=S_IFREG|0644, st_size=111, ...}) = 0

3. Предположим, приложение пишет лог в текстовый файл. Этот файл оказался удален (deleted в lsof), однако возможности сигналом сказать приложению переоткрыть файлы или просто перезапустить приложение – нет. Так как приложение продолжает писать в удаленный файл, место на диске постепенно заканчивается. Основываясь на знаниях о перенаправлении потоков предложите способ обнуления открытого удаленного файла (чтобы освободить место на файловой системе).

# ll /var/log/nginx/
total 12
drwxr-xr-x  2 root     adm    4096 Jul  1 07:16 ./
drwxrwxr-x 11 root     syslog 4096 Jul  1 07:16 ../
-rw-r-----  1 www-data adm      86 Jul  1 07:17 access.log
-rw-r-----  1 www-data adm       0 Jul  1 07:16 error.log

# rm /var/log/nginx/access.log

# lsof | grep deleted | grep nginx
nginx     482692                          www-data    4w      REG              253,0        86     395892 /var/log/nginx/access.log (deleted)

#ll /proc/482692/fd | grep "/var/log/nginx/access.*"
l-wx------ 1 www-data www-data 64 Jul  1 07:17 4 -> /var/log/nginx/access.log (deleted)

# > /proc/482692/fd/4

4. Занимают ли зомби-процессы какие-то ресурсы в ОС (CPU, RAM, IO)?

Зомби-процессы не занимают ресурсов.

5. В iovisor BCC есть утилита opensnoop:
root@vagrant:~# dpkg -L bpfcc-tools | grep sbin/opensnoop
/usr/sbin/opensnoop-bpfcc
На какие файлы вы увидели вызовы группы open за первую секунду работы утилиты? Воспользуйтесь пакетом bpfcc-tools для Ubuntu 20.04. Дополнительные сведения по установке.

$ sudo opensnoop-bpfcc
PID    COMM               FD ERR PATH
713    irqbalance          6   0 /proc/interrupts
713    irqbalance          6   0 /proc/stat
713    irqbalance          6   0 /proc/irq/14/smp_affinity
713    irqbalance          6   0 /proc/irq/14/smp_affinity
632    vmtoolsd            7   0 /etc/mtab
632    vmtoolsd            9   0 /proc/devices
632    vmtoolsd            7   0 /sys/class/block/dm-0/slaves
632    vmtoolsd            7   0 /sys/class/block/dm-0/slaves/sda3/../device/../../../class
632    vmtoolsd            7   0 /sys/class/block/dm-0/slaves/sda3/../device/../../../label
632    vmtoolsd            7   0 /sys/class/block/sda2/../device/../../../class
632    vmtoolsd            7   0 /sys/class/block/sda2/../device/../../../label
632    vmtoolsd            7   0 /run/systemd/resolve/resolv.conf
632    vmtoolsd            7   0 /proc/net/route


6. Какой системный вызов использует uname -a? Приведите цитату из man по этому системному вызову, где описывается альтернативное местоположение в /proc, где можно узнать версию ядра и релиз ОС.

execve
executes the program referred to by pathname.  This causes the program that is currently being run by the calling process to be replaced with a new program, with newly initialized stack, heap, and (initialized and uninitialized) data segments.

$cat /proc/version
Linux version 5.4.0-74-generic (buildd@lgw01-amd64-038) (gcc version 9.3.0 (Ubuntu 9.3.0-17ubuntu1~20.04)) #83-Ubuntu SMP Sat May 8 02:35:39 UTC 2021

7. Чем отличается последовательность команд через ; и через && в bash? Например:
root@netology1:~# test -d /tmp/some_dir; echo Hi
Hi
root@netology1:~# test -d /tmp/some_dir && echo Hi
root@netology1:~#

В случае с && второе выражение выполниться если код возврата первого = 0. В случае с ; это просто разделитель.
Есть ли смысл использовать в bash &&, если применить set -e?
Из мана по set:
-e  Exit immediately if a command exits with a non-zero status.
Соответственно действия равнозначны.

8. Из каких опций состоит режим bash set -euxo pipefail и почему его хорошо было бы использовать в сценариях?
-e  Exit immediately if a command exits with a non-zero status.
-u  Treat unset variables as an error when substituting.
-x  Print commands and their arguments as they are executed.
-o pipefail     the return value of a pipeline is the status of the last command to exit with a non-zero status, or zero if no command exited with a non-zero status

Set с данными параметрами после шебанга помогает для отладки сценария.

9. Используя -o stat для ps, определите, какой наиболее часто встречающийся статус у процессов в системе. В man ps ознакомьтесь (/PROCESS STATE CODES) что значат дополнительные к основной заглавной буквы статуса процессов. Его можно не учитывать при расчете (считать S, Ss или Ssl равнозначными).

Наиболее часто встречающийся статус S - interruptible sleep (waiting for an event to complete)
347 строка в мане.


