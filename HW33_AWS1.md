# Cloud Computing №1
> Цель: знакомство с облачной инфраструктурой AWS и ее основными
сервисами, а также в получении практических навыков работы с AWS
Console. Научится создавать и настраивать виртуальные машины, базы
данных, хранилища файлов, настраивать масштабирование

## Task 1 Зарегистрируйтесь в Amazon Web Services (AWS) и получите бесплатный доступ.
<img width="1916" height="943" alt="image" src="https://github.com/user-attachments/assets/32438b72-b56b-46a0-b1d4-d974417b69a0" />

## Task 2 Создайте виртуальную машину Amazon EC2, настроив операционную систему, тип инстанса и правила безопасности.
● Нажмите кнопку "Launch Instance".
● Выберите нужный вам тип инстанса и желаемый образ
операционной системы.
● На следующем шаге настройте параметры инстанса, включая
сетевые настройки и теги.
● Нажмите кнопку "Launch" и следуйте инструкциям на экране,
чтобы создать и запустить экземпляр.

> Создали и запустили Instance
<img width="795" height="358" alt="image" src="https://github.com/user-attachments/assets/3116c125-f83f-46ca-b5c4-8f92ce5923db" />
<img width="1625" height="189" alt="image" src="https://github.com/user-attachments/assets/382b7304-6298-4acd-8e6b-0ab9b12e926b" />
Зайдем по ssh используя пару ключей
<img width="1014" height="648" alt="image" src="https://github.com/user-attachments/assets/08822ad5-d8fd-4e26-9578-a016ab0c38df" />

## Task 3 Создайте новый бакет Amazon S3.
● В левом меню выберите "S3".
● Нажмите кнопку "Create bucket".
● Введите имя бакета и выберите регион.
● На следующем шаге настройте параметры доступа к бакету.
Нажмите кнопку "Create bucket", чтобы создать новый бакет.
Загрузите в него несколько файлов, настроив права доступа. Создайте
базу данных Amazon RDS и подключитесь к ней из виртуальной
машины, которую вы создали ранее.

> Создали bucket в основном с дефолтными настройками
<img width="1260" height="453" alt="image" src="https://github.com/user-attachments/assets/94a17786-72b8-4d3e-867b-4b471d1111af" />

> Записали файл, ну и доступ дали только owner`у
<img width="1323" height="641" alt="image" src="https://github.com/user-attachments/assets/2eca9c34-edee-451c-98de-827b9e841253" />

