# Домашнее задание к занятию "3.1. Работа в терминале, лекция 1"

У меня есть ВМ с linux терминалом, обязательно ли мне ставить Virtualbox + Vargant ?

---
1. Ознакомиться с разделами `man bash`, почитать о настройках самого bash:
    * какой переменной можно задать длину журнала `history`, и на какой строчке manual это описывается?

HISTSIZE
Строка №562 в мане
```
HISTSIZE
        The number of commands to remember in the command history (see HISTORY below).  If the value is 0, commands are not saved in the history list.    
	Numeric values less than zero result in every command being saved on the history list (there is no limit).
        The shell sets the default value to 500 after reading any startup files.
```
    * что делает директива `ignoreboth` в bash?
```
 HISTCONTROL
              A  colon-separated  list  of  values  controlling how commands are saved on the history list.  
		If the list of values includes ignorespace, lines which begin with a space character are not saved in the history list.  A value of ignoredups causes lines  
              matching the previous history entry to not be saved.  A value of ignoreboth is shorthand for ignorespace and ignoredups.
```
Убирает из истории строки начинающиеся с пробелом и повторяющиеся.

---
2. В каких сценариях использования применимы скобки `{}` и на какой строчке `man bash` это описано?
Подстановка фигурных скобок - с помощью этого механизма можно получить из одного шаблона множетсво текстовых строк  
Хороший пример в мане:  
```
 For example, a{d,c,b}e expands into `ade ace abe'.
```

Строка №706 в мане.

---
3. Основываясь на предыдущем вопросе, как создать однократным вызовом `touch` 100000 файлов? А получилось ли создать 300000?
```
$touch {0..100000}

$touch {0..300000}
-bash: /usr/bin/touch: Argument list too long
```

---
4. В man bash поищите по `/\[\[`. Что делает конструкция `[[ -d /tmp ]]`

Проверяет является ли /tmp директорией.
$ if [[ -d "/tmp" ]]; then echo "This is a directory"; fi
This is a directory

---
5. Основываясь на знаниях о просмотре текущих (например, PATH) и установке новых переменных; командах, которые мы рассматривали, добейтесь в выводе type -a bash в виртуальной машине наличия первым пунктом в списке:

	```bash
	bash is /tmp/new_path_directory/bash
	bash is /usr/local/bin/bash
	bash is /bin/bash
	```



6. Чем отличается планирование команд с помощью `batch` и `at`?

---
