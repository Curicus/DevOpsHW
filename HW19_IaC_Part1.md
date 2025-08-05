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
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCMWFCJVq10Mz7QNu+KfV+Dsfgao8DKkxBoS2McZGfaxAaq2kph3bpGyX2bKEGrjDnxh5U1u98wIbso+V7iBW+xuTQ8zP3xAjdv0OrDZzqxWdX3FSh7F8M5bTxgSLMzGLk3Wviq7mFiLOJFfwtaLgZD4qKLhKs7PcT70iLm+cVhL/vgiBDpUX2xehZQZiSTF9BwkAqzqjhQk2BixqULrlmYTLrZQfvwcQfT77lPxZhtSOy2AgcmGVNaW9up8BilVAtGGkRmwYC5kw4A/CM8/5YXn/Qqhb9nSTEgST0+G3PmnQ8Jm4CezViQEZKXjGI6oGg2+OfAFafD/cUXsi6OLjvZXet9OvaVr3VdTam7uzWqOlD0620zMarWVJkkzBZJYd7Zqclp0WMAkcoQVKkkoZ9Z727gzqj2vCrz3YqzbKt7NRH7TK01RSMRWAJ5R1R7NvR9VZp93o2Yh7ejOIVBLXDTryD/4Pi+2NS4r4S8Qif+yTt+9Q9Na7bGDwslImADMMk= appuser@lab
```

