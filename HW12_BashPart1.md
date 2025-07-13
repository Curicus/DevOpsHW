# Bash/Shell #1

## Task 1 Написать скрипт, выводящий на консоль и в файл все аргументы командной строки.
> Создадим файл в который будем записывать `user@Lab5:~/skurat$ touch output.txt`

> Создадим файл скрипта с кодом `user@Lab5:~/skurat$ vim output.sh`

```
  1 #!/bin/bash
  2
  3 ext_file="output.txt"
  4 echo "Commandline args:"
  5
  6 for arg in "$@"; do
  7   echo "$arg" | tee -a "$ext_file"
  8 done
```
> Сделаем его исполняемым `user@Lab5:~/skurat$ chmod +x output.sh` и запустим на исполнение 

```
user@Lab5:~/skurat$ ./output.sh 1 2 3 4
Commandline args:
1
2
3
4
user@Lab5:~/skurat$ cat output.txt
1
2
3
4
```

## Task 2 Написать скрипт, выводящий в файл (имя файла задаётся пользователем в качестве первого аргумента командной строки) имена всех файлов с заданным расширением (третий аргумент командной строки) из заданного каталога (имя каталога задаётся пользователем в качестве второго аргумента командной строки).

> Создадим файл скрипта с следующим кодом `user@Lab5:~/skurat$ vim output2.sh`
```
#!/bin/bash
if [ $# -ne 3 ]; then
  echo "Command must be like: $0 <file_name> <directory> <extension>"
  exit 1
fi

ext_file=$1
directory=$2
extension=$3

if [ ! -d "$directory" ]; then
    echo "ERROR: directory '$directory' not exists"
    exit 1
fi

find "$directory" -type f -name "*.$extension" > "$ext_file"
echo "$1 $2 $3"
```
> Посмотрим какие файлы у нас есть в рабочей дирректории
```
user@Lab5:~$ ls -la skurat/
total 20
drwxrwxr-x  2 user user 4096 Jul 13 08:24 .
drwxr-x--- 10 user user 4096 Jul 13 08:27 ..
-rwxrwxr-x  1 user user  332 Jul 13 08:23 output2.sh
-rwxrwxr-x  1 user user  135 Jul 13 07:59 output.sh
-rw-rw-r--  1 user user   10 Jul 13 07:59 output.txt
```
> Сделаем скрипт исполняемым `user@Lab5:~$ chmod +x /skurat/output2.sh` и запустим его на исполнение
`user@Lab5:~$ ./skurat/output2.sh output2.txt skurat txt`

> Проверим содержимое выходного файла
```
user@Lab5:~$ cat output2.txt
skurat/output.txt
```


