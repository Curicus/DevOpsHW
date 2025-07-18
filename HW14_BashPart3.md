# Bash/Shell #3
> Цель: Научиться выполнять оптимизировать и делать отладку каждого этапа в Bash-скриптинге.
## Task 1 Повторить шаги, указанные в приведенной статье: 
https://andreyex.ru/linux/kak-otlazhivat-stsenarij-bash/

**Отладка всего сценария**
Запуск оболочки с параметром –x запустит весь сценарий в режиме отладки. В этом режиме следы команд и их аргументов отображаются на выходе до их выполнения.

Это наш пример сценария с именем «myscript.sh»:
```
#!/bin/bash
exec 5> debug_output.txt
BASH_XTRACEFD="5"
PS4='$LINENO: '
var="Привет мир!"
# печать
echo "$var"
# альтернативный способ печати
printf "%s\n" "$var"
echo "Мой дом - это: $HOME"
```

Чтобы запустить весь сценарий в режиме отладки, добавьте параметр -x перед запускающим скриптом следующим образом:
`$ bash -x ./myscript.sh`
```
+ exec
+ BASH_XTRACEFD=5
Привет мир!
Привет мир!
Мой дом - это: /home/user
```
**Отладка части скрипта**
Если мы уверены, что только часть скрипта вызывает ошибки, тогда нет необходимости отлаживать весь скрипт. Мы можем отладить часть или несколько частей скрипта следующим образом:

Поместите опцию «set –x» в начальную точку области, в которой требуется отладка, и поместите опцию «set + x» там, где вы хотите, чтобы она остановилась. 
Например, в следующем скрипте мы хотим отладить область, начиная с $var, потому что мы думаем, что она работает не так, как ожидалось.
Поэтому мы заключим его, поместив опцию set –x перед строкой $var, и закончим, поместив опцию set + x во вторую последнюю строку.
```
  1 #!/bin/bash
  2 exec 5> debug_output.txt
  3 BASH_XTRACEFD="5"
  4 PS4='$LINENO: '
  5 set -x
  6 var="Привет мир!"
  7 # печать
  8 echo "$var"
  9 # альтернативный способ печати
 10 printf "%s\n" "$var"
 11 set +x
 12 echo "Мой дом - это: $HOME"
```
> Вывод
```
user@Lab5:~/skurat$ ./myscript.sh
Привет мир!
Привет мир!
Мой дом - это: /home/user
```
**Отключение оболочки (параметр -n)**
Параметр -n (сокращение от noexec или без выполнения) включает режим проверки синтаксиса. Он указывает оболочке не выполнять команды, а просто проверять синтаксические ошибки.
Это безопасный способ выполнить отладку, поскольку он не выполняет команды, если они содержат какую-либо ошибку.
Это наш пример сценария с именем «myscript1.sh»:
```
  1 #!/bin/bash
  2 var="Привет мир"
  3 echo "$var"
  4 echo "Мой домашний каталог is=$HOME
  5 echo "Мое имя is=$USER"
```
```
user@Lab5:~/skurat$ sh -n ./myscript1.sh
./myscript1.sh: 6: Syntax error: Unterminated quoted string
```
**Отображение команд сценария (параметр -v)**

-v (сокращение от verbose) указывает оболочке работать в подробном режиме. При использовании этого режима он отображает все команды в сценарии перед их выполнением.
```
user@Lab5:~/skurat$ sh -v ./myscript1.sh
#!/bin/bash
var="Привет мир"
echo "$var"
Привет мир
echo "Мой домашний каталог is=$HOME"
Мой домашний каталог is=/home/user
echo "Мое имя is=$USER"
Мое имя is=user
```
> Вторая ссылка не работает для повторения

## Task 2 Создать скрипт для мониторинга активных SSH-сессий с оповещением о подозрительной активности. 
Требования: 

Выводит список пользователей с >2 активными сессиями Проверяет подключения с недоверенных IP (из файла blocklist.txt) Логирует подозрительные события в /var/log/auth_audit.log Формат лога:

[Дата] [Пользователь] [IP] [Кол-во сессий]

> Предварительно создал файл blocklist.txt и закинул туда несколько ip адресов

> Создадим файл ssh_mon.sh с кодом:
```
  1 #!/bin/bash
  2
  3 blocklist="blocklist.txt"
  4 logfile="/var/log/auth_audit.log"
  5 data=$(date "+%Y-%m-%d %H:%M:%S")
  6
  7 who | awk 'NF >=5 {print $1, $5}' | uniq | while read -r user ip; do
  8   session_count=$(who | awk -v u="$user" '$1 == u {count++} END {print count}')
  9
 10   if [ $session_count -gt 2 ]; then
 11      echo "[$data] $user $ip $session_count" >> "$logfile"
 12   fi
 13
 14   if grep -q "$ip" "$blocklist"; then
 15     echo "[$data] $user $ip $session_count" >> "$logfile"
 16   fi
 17 done
```
```
user@Lab5:~/skurat$ sudo ./ssh_mon.sh
user@Lab5:~/skurat$ sudo tail -n 5 /var/log/auth_audit.log
[2025-07-18 10:28:46] user (192.168.0.165) 6
[2025-07-18 10:28:46] user (192.168.0.19) 6
```

## Task 3 Написать скрипт для мониторинга даты продления доменного имени
> Создадим файл domain_expiration_check.sh с кодом
```
  1 #!/bin/bash
  2
  3 DOMAIN="agat.by"
  4 DAYS_WARN=30
  5 LOGFILE="/var/log/domain_expiration.log"
  6 DATE_NOW=$(date +%s)
  7
  8 echo "[$(date '+%Y-%m-%d %H:%M:%S')] Проверка домена $DOMAIN" >> "$LOGFILE"
  9 EXPIR_DATE=$(whois "$DOMAIN" | grep "Expir" | awk '{print $3}')
 10 EXPIR_SEC=$(date -d "$EXPIR_DATE" +%s)
 11
 12 DAYS_LEFT=$(( (EXPIR_SEC - DATE_NOW) / 86400 ))
 13
 14 if (( DAYS_LEFT < 0 )); then
 15   echo "[$(date '+%Y-%m-%d %H:%M:%S')] Домен $DOMAIN уже истек!" | tee -a "$LOGFILE"
 16 elif (( DAYS_LEFT <= DAYS_WARN )); then
 17   echo "[$(date '+%Y-%m-%d %H:%M:%S')]  ВНИМАНИЕ: до истечения домена $DOMAIN осталось $DAYS_LEFT дней." | tee -a "$LOGFILE"
 18 else
 19   echo "[$(date '+%Y-%m-%d %H:%M:%S')] Домен $DOMAIN активен, до истечения: $DAYS_LEFT дней." | tee -a "$LOGFILE"
 20 fi
```
> Проверим выполнение
```
user@Lab5:~/skurat$ sudo ./domain_expiration_check.sh
[2025-07-18 13:36:10] Домен agat.by активен, до истечения: 159 дней.
user@Lab5:~/skurat$ sudo tail -n 10 /var/log/domain_expiration.log
[2025-07-18 13:36:10] Проверка домена agat.by
[2025-07-18 13:36:10] Домен agat.by активен, до истечения: 159 дней.
```

> Остальные задания из допов для меня пока не понятно как делать :(


