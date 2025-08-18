# Docker и контейнеризация №1
> Цель: получить практический опыт работы c Docker-командами

## Task 1 Запуск контейнера с использованием docker run
> Установим docker c официального сайта https://docker.com
```
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
> Проверим работоспособность
```
user@lab:~$ sudo docker run hello-world

Hello from Docker!
This message shows that your installation appears to be working correctly.
```
> Добавим пользователя user в группу docker и перелогинимся `user@lab:~$ sudo usermod -aG docker user`

> Запустим контейнер на основе образа grafana из docker hub
```
user@lab:~$ docker run -d --name=grafana -p 3000:3000 grafana/grafana
Unable to find image 'grafana/grafana:latest' locally
latest: Pulling from grafana/grafana
9824c27679d3: Pull complete
bcef9442db8d: Pull complete
76daf7c048bf: Pull complete
e4a43201ce3c: Pull complete
ef5cc45a8938: Pull complete
893f347dbe35: Pull complete
33d45bf7456c: Pull complete
c8d6f57cd341: Pull complete
56fcc7be3715: Pull complete
f3cdee3e03e4: Pull complete
Digest: sha256:495b5838573a0f87f26f39779675428a187b64cfc2363ebcf2c90415fd574126
Status: Downloaded newer image for grafana/grafana:latest
a8c26a79e62b65ab8ffc9fb2cd7c399120b0469afab575747d180190a5c09414
user@lab:~$ docker ps
CONTAINER ID   IMAGE             COMMAND     CREATED          STATUS          PORTS                                         NAMES
a8c26a79e62b   grafana/grafana   "/run.sh"   28 seconds ago   Up 28 seconds   0.0.0.0:3000->3000/tcp, [::]:3000->3000/tcp   grafana
```
<img width="1601" height="983" alt="image" src="https://github.com/user-attachments/assets/cdfed80b-6721-4e12-b559-c1b42111f72d" />

> Остановим и удалим данный контейнер
```
user@lab:~$ docker stop grafana
grafana
user@lab:~$ docker rm grafana
grafana
user@lab:~$ docker ps -a
CONTAINER ID   IMAGE         COMMAND    CREATED          STATUS                      PORTS     NAMES
d9fd6d0d1857   hello-world   "/hello"   7 minutes ago    Exited (0) 7 minutes ago              eloquent_cannon
aa401174f5ea   hello-world   "/hello"   20 minutes ago   Exited (0) 20 minutes ago             inspiring_allen
f2887b390129   hello-world   "/hello"   21 minutes ago   Exited (0) 21 minutes ago             competent_mccarthy
```

## Task 2 Оптимизация использования дискового пространства

**Цель этого задания - ознакомиться с командами docker inspect, docker images
и docker system prune, и использовать их для оптимизации использования
дискового пространства на вашем хосте.**

> Запустим для начала несколько контейнеров из образа докер хаба: grafana, postgres, nginx
```
user@lab:~$ docker run -d --name=grafana -p 3000:3000 grafana/grafana
user@lab:~$ docker run --name testpostgres -e POSTGRES_PASSWORD=mypass -d postgres:16.10
user@lab:~$ docker run --name testnginx -v /some/content:/usr/share/nginx/html:ro -d -p 8080:80 nginx
```
> Выведем список образов с размером
```
user@lab:~$ docker image ls
REPOSITORY        TAG       IMAGE ID       CREATED       SIZE
grafana/grafana   latest    845d83e1cf13   7 hours ago   730MB
postgres          16.10     4b802c5161dd   3 days ago    452MB
nginx             latest    ad5708199ec7   4 days ago    192MB
```
> Используя команду docker inspect, выведите информацию о размере  
каждого образа и его слоях. Найдите образы, которые занимают много  
места на диске, и определите, какие слои образа занимают больше  
всего места.

Используя команду docker inspect мы открываем JSON файл в котором можем найти поля типа "Size": 192385800 или список хешей слоев :
```
  "Layers": [
                "sha256:eb5f13bce9936c760b9fa73aeb1b608787daa36106cc888104132e353ed37252",
                "sha256:dab69e9f41e95b695c830dd40de509cac70228535889014e632834462f5683fa",
                "sha256:39bc11fab5202a44b1cd5ce2dffd1db01a5e4fbf6dbf3382be700b2b5b337fe8",
                "sha256:2988603ca26438649eac2757fb9dd9ba08319f432d4971420cfa2134a4dcafa7",
                "sha256:a0e5983a25a50e0523bf004411cbbbfc08b6735096460ec2e3e0e919633632c1",
                "sha256:129b375526fcf7be9015a58cd3abf6f623b6f82244e575de11905b9bad61e8f7",
                "sha256:45c2d10807fb1f09c464da6f87fc08f863031b0cff7689378e2fa8f9ac6956bd"
            ]
```
вот только размер слоев тут не увидеть.

Информацию о слоях образа можно узнать с помощью команды docker history, например возмем образ grafana (формат вывода подсмотрел):
```
user@lab:~$ docker history --format "{{.CreatedBy}}: {{.Size}}" grafana/grafana | sort -h
ADD alpine-minirootfs-3.22.1-x86_64.tar.gz /…: 8.31MB
ARG GF_GID=0: 0B
ARG GF_UID=472: 0B
ARG GLIBC_VERSION=2.40: 0B
ARG RUN_SH=./packaging/docker/run.sh: 0B
CMD ["/bin/sh"]: 0B
COPY ./packaging/docker/run.sh /run.sh # bui…: 3.58kB
COPY /tmp/grafana/bin/grafana* /tmp/grafana/…: 396MB
COPY /tmp/grafana/conf ./conf # buildkit: 243kB
COPY /tmp/grafana/LICENSE ./ # buildkit: 34.5kB
COPY /tmp/grafana/public ./public # buildkit: 291MB
ENTRYPOINT ["/run.sh"]: 0B
ENV PATH=/usr/share/grafana/bin:/usr/local/s…: 0B
EXPOSE map[3000/tcp:{}]: 0B
LABEL maintainer=Grafana Labs <hello@grafana…: 0B
LABEL org.opencontainers.image.source=https:…: 0B
RUN |2 GF_UID=472 GF_GID=0 /bin/sh -c if gre…: 8.16MB
RUN |3 GF_UID=472 GF_GID=0 GLIBC_VERSION=2.4…: 26.5MB
RUN |3 GF_UID=472 GF_GID=0 GLIBC_VERSION=2.4…: 94.4kB
USER 472: 0B
WORKDIR /usr/share/grafana: 0B
user@lab:~$ docker history grafana/grafana | wc -l
22
```
> Почему то, если пересчитать строчки то  будет 21 слой, а автоматический подсчет выводит 22 слоя =/  
**Больше всего объем занимают слои COPY , наверное слои данных, наполнение содержимым.**

● Если вы обнаружили образы, которые больше не нужны, удалите их,
используя команду docker rmi.  
● Используйте команду docker system prune для удаления ненужных
образов, контейнеров, томов и сетей, которые больше не используются
вашими приложениями.  
● После выполнения этих действий проверьте использование дискового
пространства на вашем хосте и убедитесь, что вы освободили
достаточно места.

> Зафиксируем загруженность дисковой системы при наличии 3 рабочих контейнеров.
```
user@lab:~$ df -h
Filesystem                         Size  Used Avail Use% Mounted on
tmpfs                              791M  1.8M  789M   1% /run
/dev/mapper/ubuntu--vg-ubuntu--lv  9.8G  6.3G  3.0G  68% /
tmpfs                              3.9G     0  3.9G   0% /dev/shm
tmpfs                              5.0M     0  5.0M   0% /run/lock
/dev/sda2                          1.8G  130M  1.5G   8% /boot
tmpfs                              791M  4.0K  791M   1% /run/user/1000
```
> Удалим образ grafana/grafana с помощью docker rmi
```
user@lab:~$ docker stop grafana
user@lab:~$ docker rm 36bc362ae527
36bc362ae527
user@lab:~$ docker rmi grafana/grafana
Untagged: grafana/grafana:latest
Untagged: grafana/grafana@sha256:495b5838573a0f87f26f39779675428a187b64cfc2363ebcf2c90415fd574126
Deleted: sha256:845d83e1cf13d8b78b3061c95de03424b5d275ab45ecfdcff9dfd4cbfcddf1a8
Deleted: sha256:e028d30c0b78ba695159f293b779def687fa8e38844640e71f29515938cd6ea6
Deleted: sha256:8fbe9fcfa0ad4a4593499832737b03e614db95686d0ec9bab1a85a7bb8c5d93c
Deleted: sha256:33a4995c7f3a10c6eaf1125bc2e0d086a2e08d3bb8989ad120d019108eb5a534
Deleted: sha256:62d7dda6aaba8a57db221001112c12bf80a209e942dfcbd3957a3159401b519a
Deleted: sha256:7f784b1496912cf879e3cf7add384efb4235d91ec3f2ddbd21a034f552f00762
Deleted: sha256:4e47f2ad08484cd37d7238077f85897004acd4919e9bd59cd8e465a2e9b33b37
Deleted: sha256:c25c48e53b7b7c0ef177c821074f1d7ae6617c667ab7ca0daf80c93b72c44e5d
Deleted: sha256:fd407c8c8d22a0f7f6cfcda7bad88e76ae067f2bc3d30b8cfd1317ffa84819aa
Deleted: sha256:d2c2fa6d4c563a73ccfdce2bb8ff76c863badaff281f4a0201965054788d09c7
Deleted: sha256:418dccb7d85a63a6aa574439840f7a6fa6fd2321b3e2394568a317735e867d35
user@lab:~$ docker images
REPOSITORY   TAG       IMAGE ID       CREATED      SIZE
postgres     16.10     4b802c5161dd   3 days ago   452MB
nginx        latest    ad5708199ec7   4 days ago   192MB
```
> Проверим работу команды docker system prune, для этого остановим контейнер testnginx и ввидем команду
```
user@lab:~$ docker stop testnginx
testnginx
user@lab:~$ docker system prune
WARNING! This will remove:
  - all stopped containers
  - all networks not used by at least one container
  - all dangling images
  - unused build cache

Are you sure you want to continue? [y/N] y
Deleted Containers:
c2e10156384bee8b00d358e45e81224f57fc556f2c89046d465b17d38606d9d5

Total reclaimed space: 1.093kB
user@lab:~$ docker ps -a
CONTAINER ID   IMAGE            COMMAND                  CREATED       STATUS       PORTS      NAMES
11f52a76d8d8   postgres:16.10   "docker-entrypoint.s…"   3 hours ago   Up 3 hours   5432/tcp   testpostgres
```
> Проверим сйечас загруженность дисковой системы
```
user@lab:~$ docker ps -a
CONTAINER ID   IMAGE            COMMAND                  CREATED       STATUS       PORTS      NAMES
11f52a76d8d8   postgres:16.10   "docker-entrypoint.s…"   3 hours ago   Up 3 hours   5432/tcp   testpostgres
user@lab:~$ df -h
Filesystem                         Size  Used Avail Use% Mounted on
tmpfs                              791M  1.7M  789M   1% /run
/dev/mapper/ubuntu--vg-ubuntu--lv  9.8G  5.6G  3.8G  60% /
tmpfs                              3.9G     0  3.9G   0% /dev/shm
tmpfs                              5.0M     0  5.0M   0% /run/lock
/dev/sda2                          1.8G  130M  1.5G   8% /boot
tmpfs                              791M  4.0K  791M   1% /run/user/1000
```

Было /dev/mapper/ubuntu--vg-ubuntu--lv  9.8G  **6.3G  3.0G  68%** /        
Стало /dev/mapper/ubuntu--vg-ubuntu--lv  9.8G  **5.6G  3.8G  60%** /
