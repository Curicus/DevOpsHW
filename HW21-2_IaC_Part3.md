# Типы архитектур, введение в High Availability и Fault Tolerance
## Task1 Написать роль logrotate_config, которая генерирует конфиг для демона logrotate на основе переменных:
logrotate_paths  
rotate_count  

**Роль должна перед применением конфига проверять его с помощью logrotate -dv /etc/logrotate.d/your_config_file и перезапускать демон через handler**
> Создадим структуру, hosts.txt и ansibles.cfg находятся выше
```
├── playbooks
│   ├── logrotate.yml
│   └── site.yml
├── roles
│   └── logrotate_config
│       ├── defaults
│       │   └── main.yml
│       ├── handlers
│       │   └── main.yml
│       ├── tasks
│       │   └── main.yml
│       └── templates
│           └── logrotate.conf.j2
```
> Содержимое структурных файлов:
```
user@lab:~/skurat$ cat playbooks/logrotate.yml
- name: logrotate
  become: yes
  hosts: all
  roles:
    - role: /home/user/skurat/roles/logrotate_config

user@lab:~/skurat$ cat roles/logrotate_config/defaults/main.yml
logrotate_path: /var/log/nginx.log
rotate_count: 5
config_name: nginx

user@lab:~/skurat$ cat roles/logrotate_config/handlers/main.yml
- name: Test logrotate config
  command: logrotate -dv /etc/logrotate.d/{{ config_name }}
  register: logrotate_test
  failed_when: logrotate_test.rc != 0
  notify:
    - Restart logrotate daemon

- name: Restart logrotate daemon
  service:
    name: logrotate
    state: restarted

user@lab:~/skurat$ cat roles/logrotate_config/tasks/main.yml
- name: Check logrotate.d dir exists
  file:
    path: /etc/logrotate.d
    state: directory
    mode: '0755'

- name: Create config
  template:
    src: logrotate.conf.j2
    dest: /etc/logrotate.d/{{ config_name }}
    owner: root
    group: root
    mode: '0644'
  notify:
    - Test logrotate config

user@lab:~/skurat$ cat roles/logrotate_config/templates/logrotate.conf.j2
{{ logrotate_path }} {
    daily
    rotate {{ rotate_count }}
}
```
> Запустим прокатку
```
user@lab:~/skurat$ ansible-playbook -i /home/user/skurat/hosts.txt playbooks/logrotate.yml

PLAY [logrotate] *****************************************************************************************************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************************************************************************************************
Enter passphrase for key '/home/user/.ssh/id_rsa':
[WARNING]: Platform linux on host ansibleclient is using the discovered Python interpreter at /usr/bin/python3.10, but future installation of another Python interpreter could change the meaning of that
path. See https://docs.ansible.com/ansible-core/2.17/reference_appendices/interpreter_discovery.html for more information.
ok: [ansibleclient]

TASK [/home/user/skurat/roles/logrotate_config : Check logrotate.d dir exists] ***************************************************************************************************************************
ok: [ansibleclient]

TASK [/home/user/skurat/roles/logrotate_config : Create config] ******************************************************************************************************************************************
ok: [ansibleclient]

PLAY RECAP ***********************************************************************************************************************************************************************************************
ansibleclient              : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

> Проверим появился ли конфиг на клиенте
```
user@ansibleclient:~/skurat$ cat /etc/logrotate.d/nginx
/var/log/nginx.log {
    daily
    rotate 5
}
```
> Вроде как все норм


## Task 2 Зашифровать с помощью ansible vault переменные
api_token: "s3cr3t_t0k3n"  
api_user: ecom  
**Написать плейбук, который на основе этих переменных генерирует файл /opt/app/.env с правами 0600 и содержимым вида**  
APP_TOKEN={{ api_token }}  
API_USER={{ api_user }}  
ENVIRONMENT=production  
