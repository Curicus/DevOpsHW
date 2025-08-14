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

> Создадим раздел с переменными и зашифруем их паролем
```
user@lab:~/skurat$ mkdir vars
user@lab:~/skurat/vars$ vim vault_vars.yml
api_token: "s3cr3t_t0k3n"
api_user: ecom
user@lab:~/skurat$ ansible-vault encrypt vars/vault_vars.yml
user@lab:~/skurat$ cat vars/vault_vars.yml
$ANSIBLE_VAULT;1.1;AES256
31303332393234653031313535393139663864343131393634633439666631386438653165363565
6136663934613036613834353832633637636565323839620a383164376237643733373932303864
36316239363133613765613264623635346631353738663133366536333038343836306534633535
3466646130333939350a373239636132333734316462383861376231343537336164323064663134
35643832653137626335326430616164633965366331633863613361363131383236643534333636
3663343336383063363262653732663532306536363539373432
```
> Создадим плейбук create_env.yml
```
user@lab:~/skurat$ cat playbooks/create_env.yml
- name: Create .env file
  hosts: all
  become: true
  vars_files:
    - /home/user/skurat/vars/vault_vars.yml

  tasks:
    - name: Check /opt/app dir exists
      file:
        path: /opt/app
        state: directory
        owner: root
        group: root
        mode: '0755'
    - name: Generate .env file
      copy:
        dest: /opt/app/.env
        content: |
          APP_TOKEN={{ api_token }}
          API_USER={{ api_user }}
          ENVIRONMENT=production
        owner: root
        group: root
        mode: '0600'
```
> Запустим плейбук на прокатку
```
user@lab:~/skurat$ ansible-playbook playbooks/create_env.yml --ask-vault-pass
Vault password:

PLAY [Create .env file] **********************************************************************************************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************************************************************************************************
Enter passphrase for key '/home/user/.ssh/id_rsa':
[WARNING]: Platform linux on host ansibleclient is using the discovered Python interpreter at /usr/bin/python3.10, but future installation of another Python interpreter could change the meaning of that
path. See https://docs.ansible.com/ansible-core/2.17/reference_appendices/interpreter_discovery.html for more information.
ok: [ansibleclient]

TASK [Check /opt/app dir exists] *************************************************************************************************************************************************************************
changed: [ansibleclient]

TASK [Generate .env file] ********************************************************************************************************************************************************************************
changed: [ansibleclient]

PLAY RECAP ***********************************************************************************************************************************************************************************************
ansibleclient              : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```
> Проверим на клиентской машине
```
user@ansibleclient:~/skurat$ sudo cat /opt/app/.env
APP_TOKEN=s3cr3t_t0k3n
API_USER=ecom
ENVIRONMENT=production
```
> Вроде так

## Task 4 Написать роль sys_utils_manager, которая устанавливает инструменты
мониторинга: atop, iotop
сетевые: nmap, tcpdump, mtr
отладочные: strace
Роль должна использовать теги

monitoring - для установки утилит мониторинга
network - для установки сетевых утилит
debug - для установки отладочных утилит
config_atop - для настройки утилиты atop -> изменение времени сбора метрик с 10 минут на значение, указанное в переменной atop_time

