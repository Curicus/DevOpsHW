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
## Дополнительное задание

### Создать:
- группу ftpusers `sudo groupadd ftpusers`
- группу admins `sudo groupadd admins`
- группу auditors `sudo groupadd auditors`
- пользователя logger с домашней директорией /opt/logger и без возможности входа в систему `sudo useradd -d /opt/logger -s /usr/sbin/nologin logger`

### Разрешить с помощью ACL:
- пользователю www-data чтение всех файлов `sudo setfacl -m u:www-data:r /var/www/server-app`
- группе ftpusers запись в uploads `sudo setfacl -m g:ftpusers:wX /var/www/server-app/uploads/`
- группе admins полный доступ `sudo setfacl -R -m  g:admins:rwX ./`
- пользователю logger полный доступ только к файлу app.log `sudo setfacl -m u:logger:rwX app.log`
- группе auditors чтение файла app.log  `sudo setfacl -m g:auditors:r app.log`

### Настроить систему так, чтобы новые файлы создавались с правами rw-r-----, а директории - rwxr-x---
Это надо задать Umask=027, для файлов 666-027=640(cd /et), а для дирректорий 777-027=750(rwxr-x---)
`sudo visudo`
вставили туда Defaults umask=027

```
user@Lab5:/var/www/server-app$ sudo touch file
user@Lab5:/var/www/server-app$ ls -l | grep file
-rw-r-----  1 root root    0 Jul  1 13:18 file
```
### Разрешить пользователю logger вход в систему; переключиться на него с помощью su, сохранив текущее окружение и создать файл /tmp/logger-envs и вывести в него переменные окружения. Выйти и переключиться еще раз с -; сравнить вывод env и содержимое файла /tmp/logger-envs

Разршим пользователю logger вход в систему `sudo usermod -s /bin/bash logger`
Переключимся на него `sudo su logger` а также создадим файл и запишем туда переменное окружение `env > /tmp/logger-envs' Выйдем - Exit
Переключимся еще раз с - и сравним вывод env с содержанием файла /tmp/logger-envs
```
user@Lab5:/var/www/server-app$ sudo su - logger
logger@Lab5:/var/www/server-app$ env
SHELL=/bin/bash
PWD=/var/www/server-app
LOGNAME=logger
HOME=/opt/logger
LANG=en_US.UTF-8
TERM=xterm
USER=logger
SHLVL=1
XDG_DATA_DIRS=/usr/local/share:/usr/share:/var/lib/snapd/desktop
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
MAIL=/var/mail/logger
_=/usr/bin/env
```
```
logger@Lab5:/var/www/server-app$ cat /tmp/logger-envs
SHELL=/bin/bash
SUDO_GID=1000
SUDO_COMMAND=/usr/bin/su logger
SUDO_USER=user
PWD=/var/www/server-app
LOGNAME=logger
HOME=/opt/logger
LANG=en_US.UTF-8
LS_COLORS=rs=0:di=01;34:ln=01;36:mh=00:pi=40;33:so=01;35:do=01;35:bd=40;33;01:cd=40;33;01:or=40;31;01:mi=00:su=37;41:sg=30;43:ca=00:tw=30;42:ow=34;42:st=37;44:ex=01;32:*.tar=01;31:*.tgz=01;31:*.arc=01;31:*.arj=01;31:*.taz=01;31:*.lha=01;31:*.lz4=01;31:*.lzh=01;31:*.lzma=01;31:*.tlz=01;31:*.txz=01;31:*.tzo=01;31:*.t7z=01;31:*.zip=01;31:*.z=01;31:*.dz=01;31:*.gz=01;31:*.lrz=01;31:*.lz=01;31:*.lzo=01;31:*.xz=01;31:*.zst=01;31:*.tzst=01;31:*.bz2=01;31:*.bz=01;31:*.tbz=01;31:*.tbz2=01;31:*.tz=01;31:*.deb=01;31:*.rpm=01;31:*.jar=01;31:*.war=01;31:*.ear=01;31:*.sar=01;31:*.rar=01;31:*.alz=01;31:*.ace=01;31:*.zoo=01;31:*.cpio=01;31:*.7z=01;31:*.rz=01;31:*.cab=01;31:*.wim=01;31:*.swm=01;31:*.dwm=01;31:*.esd=01;31:*.avif=01;35:*.jpg=01;35:*.jpeg=01;35:*.mjpg=01;35:*.mjpeg=01;35:*.gif=01;35:*.bmp=01;35:*.pbm=01;35:*.pgm=01;35:*.ppm=01;35:*.tga=01;35:*.xbm=01;35:*.xpm=01;35:*.tif=01;35:*.tiff=01;35:*.png=01;35:*.svg=01;35:*.svgz=01;35:*.mng=01;35:*.pcx=01;35:*.mov=01;35:*.mpg=01;35:*.mpeg=01;35:*.m2v=01;35:*.mkv=01;35:*.webm=01;35:*.webp=01;35:*.ogm=01;35:*.mp4=01;35:*.m4v=01;35:*.mp4v=01;35:*.vob=01;35:*.qt=01;35:*.nuv=01;35:*.wmv=01;35:*.asf=01;35:*.rm=01;35:*.rmvb=01;35:*.flc=01;35:*.avi=01;35:*.fli=01;35:*.flv=01;35:*.gl=01;35:*.dl=01;35:*.xcf=01;35:*.xwd=01;35:*.yuv=01;35:*.cgm=01;35:*.emf=01;35:*.ogv=01;35:*.ogx=01;35:*.aac=00;36:*.au=00;36:*.flac=00;36:*.m4a=00;36:*.mid=00;36:*.midi=00;36:*.mka=00;36:*.mp3=00;36:*.mpc=00;36:*.ogg=00;36:*.ra=00;36:*.wav=00;36:*.oga=00;36:*.opus=00;36:*.spx=00;36:*.xspf=00;36:*~=00;90:*#=00;90:*.bak=00;90:*.crdownload=00;90:*.dpkg-dist=00;90:*.dpkg-new=00;90:*.dpkg-old=00;90:*.dpkg-tmp=00;90:*.old=00;90:*.orig=00;90:*.part=00;90:*.rej=00;90:*.rpmnew=00;90:*.rpmorig=00;90:*.rpmsave=00;90:*.swp=00;90:*.tmp=00;90:*.ucf-dist=00;90:*.ucf-new=00;90:*.ucf-old=00;90:
TERM=xterm
USER=logger
DISPLAY=localhost:11.0
SHLVL=1
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
SUDO_UID=1000
MAIL=/var/mail/logger
_=/usr/bin/env
```
### Без переключения пользователя создать от имени logger директорию /tmp/logger-dir
sudo -u logger mkdir /tmp/logger-dir

