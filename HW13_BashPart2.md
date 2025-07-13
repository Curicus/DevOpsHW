# Bash/Shell #2

> Цель: получить практический опыт работы со строками и регулярными выражениями в Bash. Изучить логические блоки и функции

## Task 1 Подсчет кол-ва слов в файле

**С данным условием:**

- Слово — это последовательность из букв (русских или английских), внутри которой могут быть дефисы.
- На вход даётся текст, посчитайте, сколько в нём слов.
- PS. Задача решается в одну строчку.

> Создадим файл с именем input.txt и поместим туда исходный текст
```
  1 Он --- серо-буро-малиновая редиска!!
  2 >>>:->
  3 А не кот.
  4 www.kot.ru
```
> Создадим файл word_count.sh, сделаем его сразу исполняемым и напишем в нем код с помощью регулярных выражений
```
  1 #!/bin/bash
  2
  3 word_count=$( grep -oE '[а-яА-Яa-zA-Z]+(-[а-яА-Яa-zA-Z]+)*' input.txt | wc -l)
  4 echo "Words count in file input.txt= $word_count"
```
> Посмотрим результат работы скрипта
```
user@Lab5:~/skurat$ ./word_count.sh
Words count in file input.txt= 9
```
> На выходе и должно было получиться 9 слов, исходя из постановки задачи

## Task 2  Добавление приставок к файлам определенных типов
Добавить ко всем файлам формата log timestamp в качестве названия.
Должно получиться filename_{timestamp}.log
> Создадим файл скрипта под именем suffix.sh со следующим кодом и сделаем его исполняемым
```
  1 #!/bin/bash
  2
  3 timestamp=$(date +%Y-%m-%d_%H:%M)
  4
  5 for file in *.log; do
  6   core="${file%.log}"
  7
  8   name_suffix="${core}_${timestamp}.log"
  9
 10   mv "$file" "$name_suffix"
 11
 12   echo "$name_suffix"
 13 done
```
> Предварительно создадим  2 log файла и запустим скрипт на исполнение
```
user@Lab5:~/skurat$ ls -la | grep log
-rw-rw-r--  1 user user    0 Jul 13 13:32 file1.log
-rw-rw-r--  1 user user    0 Jul 13 13:32 file2.log
user@Lab5:~/skurat$ ./suffix.sh
file1_2025-07-13_13:33.log
file2_2025-07-13_13:33.log
user@Lab5:~/skurat$ ls -la | grep log
-rw-rw-r--  1 user user    0 Jul 13 13:32 file1_2025-07-13_13:33.log
-rw-rw-r--  1 user user    0 Jul 13 13:32 file2_2025-07-13_13:33.log
``` 



