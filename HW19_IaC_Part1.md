# Ansible
> Цель: получить практический опыт работы с системой управления конфигурацией Ansible.
## Task №1 Установить Ansible
 user@Lab5  ~  sudo apt update
 user@Lab5  ~   sudo apt install software-properties-common
 user@Lab5  ~  sudo apt-add-repository ppa:ansible/ansible
 user@Lab5  ~  sudo apt update
 user@Lab5  ~  sudo apt install ansible
## Task #2 Сгенерировать SSH ключи
user@lab:~$ ssh-keygen -t rsa -b 4096
user@lab:~$ ssh-copy-id user@192.168.136.128
user@lab:~$ vim hosts.txt
  1 [staging_servers]
  2 linux1 ansible_host=192.168.136.128 ansible_user=user ansible_ssh_private_key_file=/hoome/user/.ssh/id_rsa
