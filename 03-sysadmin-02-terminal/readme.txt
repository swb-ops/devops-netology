"3.2. Работа в терминале, лекция 2"

1. Какого типа команда cd? Попробуйте объяснить, почему она именно такого типа; опишите ход своих мыслей, если считаете что она могла бы быть другого типа.

Втроенная команда, реализованная внутри самой оболочки.
$ type cd
cd is a shell builtin

cd часть исходного кода shell ?

Наверное cd могла бы быть внешней программой или псевдонимом.

2. Какая альтернатива без pipe команде grep <some_string> <some_file> | wc -l? man grep поможет в ответе на этот вопрос. Ознакомьтесь с документом о других подобных некорректных вариантах использования pipe.

grep --count

3. Какой процесс с PID 1 является родителем для всех процессов в вашей виртуальной машине Ubuntu 20.04?

/sbin/init

systemd имеет PID 1

4. Как будет выглядеть команда, которая перенаправит вывод stderr ls на другую сессию терминала?

$ls -la /bin/user 2> /dev/pts/0

5. Получится ли одновременно передать команде файл на stdin и вывести ее stdout в другой файл? Приведите работающий пример.

$ cat < output.txt | tee output_new.txt
1
2
3
4
5

6. Получится ли вывести находясь в графическом режиме данные из PTY в какой-либо из эмуляторов TTY? Сможете ли вы наблюдать выводимые данные?

Поскольку PTY это псевдоэмулятор терминала, ввод-вывод которого связан с
какой-то программой, к примеру ssh, ссответственно мы можем связать pty с
эмулятором TTY и наблюдать там выводимые данные.

Допустим, нам понадобилось вывести одну строку из pty в tty, это получится
выполнить?

Просто перенаправить поток на tty устройство ?
$ ls -la /home > /dev/tty1


7. Выполните команду bash 5>&1. К чему она приведет? Что будет, если вы выполните echo netology > /proc/$$/fd/5? Почему так происходит?

Создали новый файловый дескриптор с номером 5 и связали его с stdout.

vshchepkin@vshchepkin:~$ echo netology > /proc/$$/fd/5
netology

Подали на ввод команде echo netology и перенаправили поток в файловый дескриптор
5, который связан с stdout.


8. Получится ли в качестве входного потока для pipe использовать только stderr команды, не потеряв при этом отображение stdout на pty? Напоминаем: по умолчанию через pipe передается только stdout команды слева от | на stdin команды справа. Это можно сделать, поменяв стандартные потоки местами через промежуточный новый дескриптор, который вы научились создавать в предыдущем вопросе.

vshchepkin@vshchepkin:~$ ls /root/ /home/vshchepkin/devops/ 13>&1 1>&2 2>&13 | grep cannot
/home/vshchepkin/devops/:
devops-netology  sysadm-homeworks  terraform
ls: cannot open directory '/root/': Permission denied

Поменял местам stdout и stderr - создал временные дескриптор 13, перенаправил его в stdout, stdout перенаправил в stderr, stderr во временный дескриптор. Таким образом на ввод grep подали ошибку отображения директории /root, а stdout вывели в терминал.

9. Что выведет команда cat /proc/$$/environ? Как еще можно получить аналогичный по содержанию вывод?

Покажет набор переменных среды процесса.
xargs -0 < /proc/344360/environ

10. Используя man, опишите что доступно по адресам /proc/<PID>/cmdline, /proc/<PID>/exe.
exe The file is a symbolic link to the executable that spawned the process. This could be useful if you have a program that wants to re-execute itself. 
cmdlineThe file will tell you what arguments were passed at the command line to run that process.


11. Узнайте, какую наиболее старшую версию набора инструкций SSE поддерживает ваш процессор с помощью /proc/cpuinfo.
sse sse2 sse4_1 sse4_2

12 .sudo echo string > /root/new_file не даст выполнить перенаправление под обычным пользователем, так как перенаправлением занимается процесс shell'а, который запущен без sudo под вашим пользователем. Для решения данной проблемы можно использовать конструкцию echo string | sudo tee /root/new_file. Узнайте что делает команда tee и почему в отличие от sudo echo команда с sudo tee будет работать.

tee – чтение с stdin и запись в stdout и file.
Вторая команда будет работать поскольку непосредственная запись в файл выполняется от рута.

