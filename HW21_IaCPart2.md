# Управление конфигурацией #2
> Цель: получить практический опыт работы c Ansible-ролями
## Task 1 Роль для установки и настройки веб-сервера. Напишите роль, которая устанавливает веб-сервер (например, Apache или Nginx), настраивает его для работы с вашим приложением и добавляет необходимые конфигурационные файлы. Роль должна принимать переменные для конфигурирования сервера, такие как порт, на котором он будет слушать и путь к вашему приложению.

> Подготовим структуру 
```
user@lab:~/skurat/NginxAPP/roles$ ansible-galaxy init nginx
- Role nginx was created successfully
user@lab:~/skurat/NginxAPP/roles$ ansible-galaxy init web_app
- Role web_app was created successfully
user@lab:~/skurat/NginxAPP$ tree
.
├── ansible.cfg
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
│       ├── tests
│       │   ├── inventory
│       │   └── test.yml
│       └── vars
│           └── main.yml
└── site.yml
```


