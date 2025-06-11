# Lab3 — Операционные системы #2

## Task 1: Настройка сети в Ubuntu Server 24.04

Будем настраивать **статическую адресацию**, так удобнее: не нужно искать DHCP и прокидывать его в тестовую среду vSphere.

### Проверка версии ОС:

```bash
user@lab4:~$ lsb_release -a
Distributor ID: Ubuntu
Description:    Ubuntu 24.04.1 LTS
Release:        24.04
Codename:       noble
```

### Проверка сетевых интерфейсов и содержимого `/etc/netplan`:

```bash
user@lab4:~$ ls -l /etc/netplan
-rw------- 1 root root 354 Jun 9 06:47 50-cloud-init.yaml

user@lab4:~$ sudo cat /etc/netplan/50-cloud-init.yaml
network:
    ethernets: {}
    version: 2
```

```bash
user@lab4:~$ ip -o a
1: lo inet 127.0.0.1/8 scope host lo \ valid_lft forever preferred_lft forever
```

> Пока сетевого интерфейса с IP-адресом нет

### Настройка статической адресации

```bash
user@lab4:~$ sudo vim /etc/netplan/50-cloud-init.yaml
```

```yaml
network:
  ethernets:
    ens160:
      addresses:
        - 192.168.1.174/23
      nameserver:
        addresses:
          - 80.94.225.5
      routes:
        - to: 0.0.0.0/0
          via: 192.168.0.19
  version: 2
```

Применяем конфигурацию:

```bash
user@lab4:~$ sudo netplan try
Configuration accepted.
```

### Проверка:

```bash
user@lab4:~$ ip -o a
2: ens160 inet 192.168.1.174/23 brd 192.168.1.255 scope global ens160 \ valid_lft forever preferred_lft forever
```

### Проверка связи:

```bash
user@lab4:~$ ping 192.168.0.19
PING 192.168.0.19 (192.168.0.19) 56(84) bytes of data.
64 bytes from 192.168.0.19: icmp_seq=1 ttl=64 time=1.16 ms
...
```

```bash
user@lab4:~$ ping ya.ru
PING ya.ru (5.255.255.242) 56(84) bytes of data.
```

> Имя резолвится, но ICMP трафик не идёт — особенности настроек МСЭ.  
> Настройки `resolved.conf` опустим — `nameserver` уже указан в `netplan`.

---

## Task 2: Установка и настройка SSH

### Обновление репозиториев:

```bash
sudo apt update
```

### Установка SSH и OpenSSH:

```bash
sudo apt install ssh
sudo apt install openssh-server
```

> Всё уже установлено, как видно из вывода.

### Добавляем SSH в автозагрузку:

```bash
sudo systemctl enable ssh
```

### Проверка статуса SSH:

```bash
sudo systemctl status ssh
```

---

## Task 3: Настройка OpenSSH

### Открываем конфигурационный файл:

```bash
sudo vim /etc/ssh/sshd_config
```

### Меняем порт на `666`:

```conf
Port 666
#AddressFamily any
#ListenAddress 0.0.0.0
#ListenAddress ::
```

### Применение изменений:

```bash
sudo systemctl daemon-reload
sudo systemctl restart ssh.socket
sudo systemctl status sshd
```

> После этого **MobaXterm подключилась на порт 666**
