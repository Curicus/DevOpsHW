# Управление конфигурацией №2
> Цель: получить практический опыт работы c Ansible-ролями
## Task 1 Роль для установки и настройки веб-сервера. Напишите роль, которая устанавливает веб-сервер (например, Apache или Nginx), настраивает его для работы с вашим приложением и добавляет необходимые конфигурационные файлы. Роль должна принимать переменные для конфигурирования сервера, такие как порт, на котором он будет слушать и путь к вашему приложению.

> В нашем случае разобьем задачу на две роли: первая будет устанавливать и настраивать работу nginx, вторая роль будет копировать из репозитория https://github.com/mattfeltonma/python-sample-web-app.git приложение, устанавливать его и запускать на nginx.

**Скажу сразу приложение было выбрано первое попавшееся, в итоге чтобы его запустить пришлось в коде некоторые вещи комментировать (намучался)**  
> Подготовим структуру 
```
user@lab:~/skurat/NginxAPP/roles$ ansible-galaxy init nginx
- Role nginx was created successfully
user@lab:~/skurat/NginxAPP/roles$ ansible-galaxy init web_app
- Role web_app was created successfully
user@lab:~/skurat/NginxAPP$ tree
.
├── ansible.cfg
├── group_vars
│   └── all.yml
├── hosts.txt
├── roles
│   ├── nginx
│   │   ├── defaults
│   │   │   └── main.yml
│   │   ├── files
│   │   ├── handlers
│   │   │   └── main.yml
│   │   ├── meta
│   │   │   └── main.yml
│   │   ├── README.md
│   │   ├── tasks
│   │   │   └── main.yml
│   │   ├── templates
│   │   │   └── nginx.conf.j2
│   │   ├── tests
│   │   │   ├── inventory
│   │   │   └── test.yml
│   │   └── vars
│   │       └── main.yml
│   └── web_app
│       ├── defaults
│       │   └── main.yml
│       ├── files
│       ├── handlers
│       │   └── main.yml
│       ├── meta
│       │   └── main.yml
│       ├── README.md
│       ├── tasks
│       │   └── main.yml
│       ├── templates
│       │   └── webapp.service.j2
│       ├── tests
│       │   ├── inventory
│       │   └── test.yml
│       └── vars
│           └── main.yml
└── site.yml
```
> Выведем сначала содержание общих файлов

```
user@lab:~/skurat/NginxAPP$ cat ansible.cfg
[defaults]
host_key_checking = False
inventory         = ./hosts.txt
become            = True
become_method     = sudo
become_user       = root
user@lab:~/skurat/NginxAPP$ cat hosts.txt
[staging_servers]
ansibleclient ansible_host=192.168.136.128 ansible_user=user ansible_ssh_private_key_file=/home/user/.ssh/id_rsa

user@lab:~/skurat/NginxAPP$ cat site.yml
---
- name: nginx+web_app
  become: yes
  hosts: all
  roles:
    - role: nginx

    - role: web_app

user@lab:~/skurat/NginxAPP$ cat group_vars/all.yml
app_port: 5000
app_dir: /var/www/sample-web-app
venv_dir: "{{ app_dir }}/venv"
app_repo: https://github.com/mattfeltonma/python-sample-web-app.git
```

> Далее по очереди выведем содержимое всех используемых узлов структуры по роли nginx
```
user@lab:~/skurat/NginxAPP$ cat roles/nginx/defaults/main.yml
---
# defaults file for nginx
web_port: 80
user@lab:~/skurat/NginxAPP$ cat roles/nginx/handlers/main.yml
---
# handlers file for nginx
- name: Reload nginx
  service:
    name: nginx
    state: reloaded
user@lab:~/skurat/NginxAPP$ cat roles/nginx/tasks/main.yml
---
# tasks file for nginx
- name: Install nginx
  apt:
    name: nginx
    state: present
    update_cache: yes

- name: Deploy nginx config
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/sites-available/web_app
  notify: Reload nginx

- name: Enable site
  file:
    src: /etc/nginx/sites-available/web_app
    dest: /etc/nginx/sites-enabled/web_app
    state: link
    force: yes

- name: Ensure nginx is running
  service:
    name: nginx
    state: started
    enabled: true
user@lab:~/skurat/NginxAPP$ cat roles/nginx/templates/nginx.conf.j2
server {
    listen {{ web_port }};
    server_name localhost;

# Тут вроде надо заворачивать на Flask для app
    location / {
        proxy_pass http://127.0.0.1:{{ app_port }};
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

> Далее по очереди выведем содержимое всех используемых узлов структуры по роли web_app
```
user@lab:~/skurat/NginxAPP$ cat roles/web_app/handlers/main.yml
---
# handlers file for web_app
- name: Enable and start webapp
  systemd:
    name: webapp.service
    enabled: yes
    state: started
user@lab:~/skurat/NginxAPP$ cat roles/web_app/tasks/main.yml
---
# tasks file for web_app
- name: Install deps
  apt:
    name: "{{ item }}"
    state: present
  loop:
    - git
    - python3-venv
    - python3-pip
    - python3

- name: Clone repo
  git:
    repo: "{{ app_repo }}"
    dest: "{{ app_dir }}"
    version: HEAD
    force: yes

- name: Create venv
  command: python3 -m venv "{{ venv_dir }}"
  args:
    creates: "{{ venv_dir }}/bin/activate"

- name: Install requarements in venv
  pip:
    name: flask
    virtualenv: "{{ venv_dir }}"
    virtualenv_command: python3 -m venv

- name: Check if requirements.txt exists
  stat:
    path: "{{ app_dir }}/requirements.txt"
  register: requirements_file

- name: Install requirements from requirements.txt if it exists
  pip:
    requirements: "{{ app_dir }}/requirements.txt"
    virtualenv: "{{ venv_dir }}"
    virtualenv_command: python3 -m venv
  when: requirements_file.stat.exists

- name: Create systemd service for web app
  template:
    src: webapp.service.j2
    dest: /etc/systemd/system/webapp.service
  notify: Restart webapp.service

- name: Enable and start webapp
  systemd:
    name: webapp.service
    enabled: yes
    state: started
user@lab:~/skurat/NginxAPP$ cat roles/web_app/templates/webapp.service.j2
[Unit]
Description=Python Web App
After=network.target

[Service]
User=www-data
WorkingDirectory={{ app_dir }}
ExecStart={{ venv_dir }}/bin/python app.py
Restart=always

[Install]
WantedBy=multi-user.target
```
> Вот такой вывод вышел, да из-за заглушек в коде не все выводится, но главное что ansible раскатал с ролями и приложение (неудочно выбранный реп) работает на nginx
<img width="1906" height="212" alt="image" src="https://github.com/user-attachments/assets/c7316bc6-a628-46a9-92b5-1ac67ff943f5" />


