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
**Добавить ко всем файлам формата log timestamp в качестве названия.
Должно получиться filename_{timestamp}.log**
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
**Для всех файлов с расширением Py добавьте в конец названия хэш коммита.**
> Предварительно сэмитировали файл ввиде хэш-файла, его имя нам понадобится для аргумента
```
user@Lab5:~/skurat$ ls -la | grep -v '\.'
total 36
-rw-rw-r--  1 user user    0 Jul 13 13:28 ecad48e50a909450da935d84894c46d681aadd51
```
> Создадим файл скрипта с именем hash_commit.sh с кодом, сделаем его исполнительным и запустим
```
  1 #!/bin/bash
  2
  3 if [ $# -ne 1 ]; then
  4   echo "Need to enter hash id as Arg: $0 <commit_hash>"
  5   exit 1
  6 fi
  7
  8 commit_hash=$1
  9
 10 #Проверку на правильность введенного хэша опустим
 11
 12 for file in *.py; do
 13  
 14   core="${file%.py}"
 15   name_suffix="${core}_${commit_hash}.py"
 16
 17   mv "$file" "$name_suffix"
 18   echo "$name_suffix"
 19 done
```
> Убедимся, что у нас есть файлы с расширением .py
```
user@Lab5:~/skurat$ ls -la | grep .py
-rw-rw-r--  1 user user    0 Jul 13 12:57 1.py
-rw-rw-r--  1 user user    0 Jul 13 12:57 2.py
-rw-rw-r--  1 user user    0 Jul 13 12:57 3.py
```
> Запустим скрипт и посмотрим на результат
```
user@Lab5:~/skurat$ ./hash_commit.sh ecad48e50a909450da935d84894c46d681aadd51
1_ecad48e50a909450da935d84894c46d681aadd51.py
2_ecad48e50a909450da935d84894c46d681aadd51.py
3_ecad48e50a909450da935d84894c46d681aadd51.py
user@Lab5:~/skurat$ ls -la | grep .py
-rw-rw-r--  1 user user    0 Jul 13 12:57 1_ecad48e50a909450da935d84894c46d681aadd51.py
-rw-rw-r--  1 user user    0 Jul 13 12:57 2_ecad48e50a909450da935d84894c46d681aadd51.py
-rw-rw-r--  1 user user    0 Jul 13 12:57 3_ecad48e50a909450da935d84894c46d681aadd51.py
```
> It works.

