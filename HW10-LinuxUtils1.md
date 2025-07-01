 # Unix утилиты №1
> Цель: освоить принципы работы текстового редактора Vim. Ознакомиться с
процессом логирования в Linux. Научиться определять уровень загрузки
системы.

## Task 1 Создание нового файла с использованием vim
6. Установите курсор на последней строке файла. Вставьте строку, содержащую
следующий текст :extention 287
> Делаем это командой `:$put ='extention 287'`
```
...
win.com
:end
extention 287
```
7. Замените слово extention на x.
> Делаем это командой `:s/extention/x `
```
...
win.com
:end
x 287
```
8. Удалите последнюю строку.
> Делаем это командой `:$d`
```
...
goto end
win.com
:end
```
9. Введите команду отмены изменений для отмены последней команды.
> Делаем это командой `u`

10. Установите курсор на первую строку (строка 5), вставьте перед ним пустую
строку и введите следующий текст:
@REM 22 apr. 1999
Оставьте пустую строку между новым разделом и следующим за ним. Нажмите
<ESC>, чтобы перейти в командный режим
> Делаем это нажатием `O`

11. Скопируйте файл testcase.c(предварительно осуществив его поиск) в
директории~/practice и вызовите vim для редактирования файла
> Запускаем файл на редактирование следующим образом `vim $(find . -name 'testcase.c')`

12. Включите отображение номеров строк. Сколько строк в данном файле?
```
...
 84 void testSortStudentsByGPA() {
 85     Student* students[3];
 86     //input
 87     students[0] = createStudent(1, "John", 3.5);
 88     students[1] = createStudent(2, "Alice", 4.0);
 89     students[2] = createStudent(3, "Bob", 3.8);
 90
 91     sortStudentsByGPA(students, 3);
 92
 93     assert(students[0]->id ==
```
Найдите слово WORD и замените его на IGNORE.  `:%s/WORD/IGNORE/g`

Найдите слово Reset и замените его на set.    `:%s/Reset/set/g`

Найдите слово input и замените его на output  `:%s/input/output/g`

Вставьте строку, заполненную вопросительными знаками <?> под строкой :state
= WORD 
> Будем делать это командой `g/state = WORD/normal o?????`
```
...
 72 // Тестовые функции
 73 void testCreateStudent() {
 74     Student* student = createStudent(1, "John Doe", 3.5);
 75     assert(student != NULL);
 76     state = WORD
 77     ?????

```
Скопируйте строки с 16 по 29 в файл printwords.c
> Запустим 2 файла в сплитрежиме `vim -O tescase.c`
> :16,19y  #это янк захват дипазона указанных строк, в пределах редактора буфер
> переходим в правую часть с помощью ctrl+w и -> и вставляем с помощью р
> 
![image](https://github.com/user-attachments/assets/d8c2ebca-5df7-4ce6-9ebc-086fa1dc6fff)

Перейдите в конец файла и удалите две последние строки.
> Сделаем это командой `#-1,$d`, G (конец файла), o (переход на новую строку), p (вставка)
```
 76 void testSortStudentsByGPA() {
 77     Student* students[3];
 78     //input
 79     students[0] = createStudent(1, "John", 3.5);
 80     students[1] = createStudent(2, "Alice", 4.0);
 81     students[2] = createStudent(3, "Bob", 3.8);
 82
 83     sortStudentsByGPA(students, 3);
 84
 85 /*Manifests
 86 void test_add() {
 87     assert(add(2, 3) == 5);
 88     assert(add(-1, 1) == 0);
 89     assert(add(0, 0) == 0);
 90     assert(add(100, 200) == 300);
 91     printf("All test cases passed for add() function.\n");
 92 }
 93 */ 
```
Запишите произведенные изменения на диск в файл testvim.c и выйдите из
редактора
> Сделем это командой :w testvim.c | q

## Task 2 Установить утилиту nginx, посмотреть ее логи и также уровень нагрузки на ОС. Просмотрите логи действий пользователей системы.
> Устновили `sudo apt install nginx`
> Проверим `journalctl -xeu nginx`
```
Jul 01 10:24:37 Lab5 systemd[1]: Starting nginx.service - A high performance web server and a reverse proxy server...
░░ Subject: A start job for unit nginx.service has begun execution
░░ Defined-By: systemd
░░ Support: http://www.ubuntu.com/support
░░
░░ A start job for unit nginx.service has begun execution.
░░
░░ The job identifier is 307219.
```
Проверим нагрузку на CPU
![image](https://github.com/user-attachments/assets/bf828115-a8a1-454a-88e2-a135a483bd64)

> Посмотрели логи действий в /var/log/nginx/error.log - пусто; В /var/log/access.log:
```
192.168.0.165 - - [01/Jul/2025:10:25:26 +0000] "GET / HTTP/1.1" 200 409 "-" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/137.0.0.0 Safari/537.36"
192.168.0.165 - - [01/Jul/2025:10:25:27 +0000] "GET /favicon.ico HTTP/1.1" 404 196 "http://192.168.1.174/" "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/137.0.0.0 Safari/537.36"
::1 - - [01/Jul/2025:10:25:55 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/8.5.0"
/var/log/nginx/access.log (END)
```


