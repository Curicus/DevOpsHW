# Ansible
> Цель: получить практический опыт работы с системой управления конфигурацией Ansible.
## Task №1 Установить Ansible
```
 user@Lab5  ~  sudo apt update
 user@Lab5  ~   sudo apt install software-properties-common
 user@Lab5  ~  sudo apt-add-repository ppa:ansible/ansible
 user@Lab5  ~  sudo apt update
 user@Lab5  ~  sudo apt install ansible
```
## Task №2 Сгенерировать SSH ключи
```
user@lab:~$ ssh-keygen -t rsa -b 4096
user@lab:~$ ssh-copy-id user@192.168.136.128
user@lab:~$ vim hosts.txt
  1 [staging_servers]
  2 ansibleclient ansible_host=192.168.136.128 ansible_user=user ansible_ssh_private_key_file=/hoome/user/.ssh/id_rsa
```
## Task  №3 Написать плейбук, который создает системного пользователя с именем appuser с домашней директорией (/home/appuser) и оболочкой /bin/bash
> Сделаем на сервере с ansible настройки файлов ansible.cfg и плейбука create_appuser.yml
```
user@lab:~$ vim ansible.cfg
  1 [defaults]
  2 host_key_checking = False
  3 inventory         = ./hosts.txt
  4 become            = True
  5 become_method     = sudo
  6 become_user       = root
```
```
user@lab:~$ vim create_appuser.yml
  1 - name: create_appuser
  2   hosts: all
  3   become: yes
  4
  5   tasks:
  6     - name: Add_appuser
  7       user:
  8         name: appuser
  9         system: yes
 10         shell: /bin/bash
 11         home: /home/appuser
 12         create_home: yes
```
> На ansibleclient разрешим выполнение sudo без пароля для пользователя user, для этого в sudo visudo добавим в конце фразу `user ALL=(ALL) NOPASSWD:ALL`
> Запускаем выполнение плейбука
```
user@lab:~$ ansible-playbook create_appuser.yml

PLAY [create_appuser] *********************************************************************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************************************************************
Enter passphrase for key '/home/user/.ssh/id_rsa':
[WARNING]: Platform linux on host lab is using the discovered Python interpreter at /usr/bin/python3.10, but future installation of another Python interpreter could change the meaning of that path.
See https://docs.ansible.com/ansible-core/2.17/reference_appendices/interpreter_discovery.html for more information.
ok: [lab]

TASK [Add_appuser] ************************************************************************************************************************************************************************************
changed: [lab]

PLAY RECAP ********************************************************************************************************************************************************************************************
lab                        : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
> На ansibleclient можем наблюдать что появился пользователь appuser  с заданными настройками
```
user@ansibleclient:~$ sudo cat /etc/passwd | grep appuser
appuser:x:998:998::/home/appuser:/bin/bash
```

## Task 4 Написать плейбук, который копирует публичный SSH-ключ с управляющей машины (files/appuser.pub) в файл authorized_keys пользователя appuser на удаленных узлах (/home/appuser/.ssh/authorized_keys). Убедиться, что у директории .ssh правильные права (700), а у файла authorized_keys - права (600)
> Сгенерируем пару ключей для пользователя appuser на управляющей машине и положим публичный ключ в /home/user/skurat/task4/files/appuser.pub
appuser@lab:~$ ssh-keygen
user@lab:~/skurat/task4$ sudo cp /home/appuser/files/id_rsa.pub /home/user/skurat/task4/files/appuser.pub
> Напишем плейбук
```
user@lab:~/skurat/task4$ vim copy_ssh_key.yml
  1 - name: Установка публичного ключа для appuser
  2   hosts: all
  3   become: true
  4
  5   tasks:
  6     - name: Ensure .ssh directory exists
  7       file:
  8         path: /home/appuser/.ssh
  9         state: directory
 10         owner: appuser
 11         group: appuser
 12         mode: '0700'
 13
 14     - name: Copy public key to authorized_keys
 15       copy:
 16         src: /home/user/skurat/task4/files/appuser.pub
 17         dest: /home/appuser/.ssh/authorized_keys
 18         owner: appuser
 19         group: appuser
 20         mode: '0600'
```
> Проведем прокатку
```
user@lab:~/skurat/task4$ ansible-playbook -i hosts.txt copy_ssh_key.yml

PLAY [Установка публичного ключа для appuser] *********************************************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************************************************************
Enter passphrase for key '/home/user/.ssh/id_rsa':
[WARNING]: Platform linux on host ansibleclient is using the discovered Python interpreter at /usr/bin/python3.10, but future installation of another Python interpreter could change the meaning of
that path. See https://docs.ansible.com/ansible-core/2.17/reference_appendices/interpreter_discovery.html for more information.
ok: [ansibleclient]

TASK [Ensure .ssh directory exists] *******************************************************************************************************************************************************************
changed: [ansibleclient]

TASK [Copy public key to authorized_keys] *************************************************************************************************************************************************************
changed: [ansibleclient]

PLAY RECAP ********************************************************************************************************************************************************************************************
ansibleclient              : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
> Проверим появился ли ключ
```
appuser@ansibleclient:~$ cat .ssh/authorized_keys
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCMWFCJVq10Mz7QNu+KfV+Dsfgao8DKkxBoS2McZGfaxAaq2kph3bpGyX2bKEGrjDnxh5U1u98wIbso+V7iBW+xuTQ8zP3xAjdv0OrDZzqxWdX3FSh7F8M5bTxgSLMzGLk3Wviq7mFiLOJFfwtaLgZD4qKLhKs7PcT70iLm+cVhL/vgiBDpUX2xehZQZiSTF9BwkAqzqjhQk2BixqULrlmYTLrZQfvwcQfT77lPxZhtSOy2AgcmGVNaW9up8BilVAtGGkRmwYC5kw4A/CM8/5YXn/Qqhb9nSTEgST0+G3PmnQ8Jm4CezViQEZKXjGI6oGg2+OfAFafD/cUXsi6OLjvZXet9OvaVr3VdTam7uzWqOlD0620zMarWVJkkzBZJYd7Zqclp0WMAkcoQVKkkoZ9Z727gzqj2vCrz3YqzbKt7NRH7TK01RSMRWAJ5R1R7NvR9VZp93o2Yh7ejOIVBLXDTryD/4Pi+2NS4r4S8Qif+yTt+9Q9Na7bGDwslImADMMk= appuser@lab
```
## Task 5 Написать плейбук, который копирует шаблон файла mod с управляющей машины (templates/motd.j2) на удаленные узлы в /etc/motd. Использовать переменную {{ inventory_hostname }} в шаблоне, чтобы в файле motd отображалось имя хоста. Шаблон (templates/motd.j2)
```
Welcome to server {{ inventory_hostname }}!
Provided by Ansible.
```
> Подготовим шаблон
```
user@lab:~/skurat/task5$ cat templates/motd.j2
Welcome to {{ inventory_hostname }}!
Managed by Ansible.
```
> Подготовим плейбук
```
- name: Copy MOTD template to remote hosts
  hosts: all
  become: yes

  tasks:
    - name: Copy /etc/motd file from template
      template:
        src: templates/motd.j2
        dest: /etc/motd
        owner: root
        group: root
```
> Проверим на удаленном хосте
```
appuser@ansibleclient:~$ cat /etc/motd
Welcome to ansibleclient!
Managed by Ansible.
```
## Task 6 Написать плейбук, который проверит, установлен ли git на удаленных узлах и склонирует репозиторий https://gitlab.com/devops201206/it-mtb-blog в директорию /var/www/simple-blog
> Сделаем плейбук check_copy_git.yml
```
user@lab:~/skurat/task6$ cat check_copy_git.yml
- name: Check and Copy git repo
  hosts: all

    tasks:
      - name: Check on exists git module
        package:
          name: git
          state: present

      - name: Copy git repo
        git:
          repo: https://gitlab.com/devops201206/it-mtb-blog.git
          dest: /var/www/simple-blog
```
> Проверим есть ли  у нас git на клиентской машине
```
user@ansibleclient:/home/appuser$ whereis git
git: /usr/share/man/man1/git.1.gz
``` 
> Запустим прокатку
```
user@lab:~/skurat/task6$ ansible-playbook -i hosts.txt check_copy_git.yml

PLAY [Check and Copy git repo] ************************************************************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************************************************************
Enter passphrase for key '/home/user/.ssh/id_rsa':
[WARNING]: Platform linux on host ansibleclient is using the discovered Python interpreter at /usr/bin/python3.10, but future installation of another Python interpreter could change the meaning of
that path. See https://docs.ansible.com/ansible-core/2.17/reference_appendices/interpreter_discovery.html for more information.
ok: [ansibleclient]

TASK [Check on exists git module] *********************************************************************************************************************************************************************
changed: [ansibleclient]

TASK [Copy git repo] **********************************************************************************************************************************************************************************
changed: [ansibleclient]

PLAY RECAP ********************************************************************************************************************************************************************************************
ansibleclient              : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0\
```

> Проверим теперь на клиентской машине
```
user@ansibleclient:/home/appuser$ whereis git
git: /usr/bin/git /usr/share/man/man1/git.1.gz
user@ansibleclient:/home/appuser$ sudo ls -la /var/www/simple-blog/
total 128
drwxr-xr-x 3 root root  4096 Aug  6 08:26 .
drwxr-xr-x 5 root root  4096 Aug  6 08:26 ..
-rw-r--r-- 1 root root 97403 Aug  6 08:26 avatar.jpg
-rw-r--r-- 1 root root   988 Aug  6 08:26 bike.html
drwxr-xr-x 8 root root  4096 Aug  6 08:26 .git
-rw-r--r-- 1 root root   744 Aug  6 08:26 index.html
-rw-r--r-- 1 root root   853 Aug  6 08:26 it.html
-rw-r--r-- 1 root root    42 Aug  6 08:26 README.md
-rw-r--r-- 1 root root   969 Aug  6 08:26 style.css
```

## Task 7 Написать плейбук для сбора системной информации об управляемых узлах  

размер оперативной памяти
размер диска, смонтированного в /
версия ОС

> Создадим плейбук system_info.yml




> Запустим прокатку
```
user@lab:~/skurat/task7$ ansible-playbook -i hosts.txt system_info.yml
PLAY [Gathering Host System Info] *********************************************************************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************************************************************************************
Enter passphrase for key '/home/user/.ssh/id_rsa':
[WARNING]: Platform linux on host ansibleclient is using the discovered Python interpreter at /usr/bin/python3.10, but future installation of another Python interpreter could change the meaning of
that path. See https://docs.ansible.com/ansible-core/2.17/reference_appendices/interpreter_discovery.html for more information.
ok: [ansibleclient]

TASK [Total RAM] **************************************************************************************************************************************************************************************
ok: [ansibleclient] => {
    "msg": "RAM: 5545 Mb"
}

TASK [Disk Size mounted /] ****************************************************************************************************************************************************************************
ok: [ansibleclient] => {
    "msg": "Root Disk Size: 9979 Mb"
}

TASK [OS version] *************************************************************************************************************************************************************************************
ok: [ansibleclient] => {
    "msg": " Os: Ubuntu 22.04"
}

PLAY RECAP ********************************************************************************************************************************************************************************************
ansibleclient              : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

