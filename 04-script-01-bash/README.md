### 4.1. Командная оболочка Bash: Практические навыки

---
1.	Есть скрипт:
```
a=1
b=2
c=a+b
d=$a+$b
e=$(($a+$b))
```
Какие значения переменным c,d,e будут присвоены? Почему?

c=a+b d=1+2 e=3

---
2.	На нашем локальном сервере упал сервис и мы написали скрипт, который постоянно проверяет его доступность, записывая дату проверок до тех пор, пока сервис не станет доступным.
В скрипте допущена ошибка, из-за которой выполнение не может завершиться, при этом место на Жёстком Диске постоянно уменьшается. Что необходимо сделать, чтобы его исправить:
```
while ((1==1)
do
curl https://localhost:4757
if (($? != 0))
then
date >> curl.log
fi
done
```

```
while ((1==1))
do
curl https://localhost:4757
if (($? != 0))
then
date >> curl.log
else break
fi
done
```
Скобка в первой строке + добавить завершение цикла в случае, если curl будет успешным.

---
3.	Необходимо написать скрипт, который проверяет доступность трёх IP: 192.168.0.1, 173.194.222.113, 87.250.250.242 по 80 порту и записывает результат в файл log.
Проверять доступность необходимо пять раз для каждого узла.

```
vshchepkin@vshchepkin:~/scripts$ cat check_hosts.sh 
#!/usr/bin/env bash

h1=192.168.0.1
h2=173.194.222.113
h3=87.250.250.242

for i in {1..5}; do
        for m in $h1 $h2 $h3; do
                date >> check.log
                curl --connect-timeout 3 $m:80 > /dev/null
                echo $m status=$? >> check.log
        done
done
vshchepkin@vshchepkin:~/scripts$ cat check.log 
Mon 26 Jul 2021 07:54:19 PM UTC
192.168.0.1 status=28
Mon 26 Jul 2021 07:54:22 PM UTC
173.194.222.113 status=0
Mon 26 Jul 2021 07:54:23 PM UTC
87.250.250.242 status=0
Mon 26 Jul 2021 07:54:23 PM UTC
192.168.0.1 status=28
Mon 26 Jul 2021 07:54:26 PM UTC
173.194.222.113 status=0
Mon 26 Jul 2021 07:54:26 PM UTC
87.250.250.242 status=0
Mon 26 Jul 2021 07:54:26 PM UTC
192.168.0.1 status=28
Mon 26 Jul 2021 07:54:29 PM UTC
173.194.222.113 status=0
Mon 26 Jul 2021 07:54:29 PM UTC
87.250.250.242 status=0
Mon 26 Jul 2021 07:54:29 PM UTC
192.168.0.1 status=28
Mon 26 Jul 2021 07:54:32 PM UTC
173.194.222.113 status=0
Mon 26 Jul 2021 07:54:32 PM UTC
87.250.250.242 status=0
Mon 26 Jul 2021 07:54:32 PM UTC
192.168.0.1 status=28
Mon 26 Jul 2021 07:54:35 PM UTC
173.194.222.113 status=0
Mon 26 Jul 2021 07:54:35 PM UTC
87.250.250.242 status=0
```

---
4.	Необходимо дописать скрипт из предыдущего задания так, чтобы он выполнялся до тех пор, пока один из узлов не окажется недоступным. Если любой из узлов недоступен - 
IP этого узла пишется в файл error, скрипт прерывается
```
vshchepkin@vshchepkin:~/scripts$ cat ./check2_hosts.sh
#!/usr/bin/env bash

h3=192.168.0.1
h1=173.194.222.113
h2=87.250.250.242

while true; do
        for m in $h1 $h2 $h3; do
                curl --connect-timeout 3 $m:80 > /dev/null
        done
                        if (($? != 0)); then
                                echo $m is fail >> check2.log
                                date >> check2.log
                                break
                        fi
done
```
