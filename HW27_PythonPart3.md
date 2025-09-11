# Python №3
> Цель: Практически применить изученные темы: функции, работу с файлами и директориями, а также шаблонизатор Jinja
## Task 1 Напишите функцию multiply_numbers(), которая принимает два аргумента и возвращает их произведение. Затем вызовите эту функцию и выведите результат на экран.

> Выведем код
```
def multiply_numbers(x, y):
  return(x * y)

a = input("Введите первый множитель: ")
b = input("Введите второй множитель: ")
res = multiply_numbers(a, b)
print(f"Произведение x * y = {res}")
```
> Проверим
```
(venv) user@lab:~/skurat/python/lesson27$ python3 less27proj1.py
Введите первый множитель: 5 
Введите второй множитель: 6
Произведение x * y = 30
```
## Task 2 Создайте файл test.txt и запишите в него строку "Это тестовый файл для домашнего задания по программированию". Затем откройте этот файл в режиме чтения, прочитайте его содержимое и выведите на экран.
> Выведем код
```
with open("text.txt", "w", encoding = "utf-8") as f:
    f.write("Это тестовый файл для домашнего задания по программированию")

with open("text.txt", "r", encoding = "utf-8")as f:
    text = f.read()

print("Читайем текст из файла text.txt:")
print(text)
```
> Проверим
```
(venv) user@lab:~/skurat/python/lesson27$ python3 less27proj2.py
Читайем текст из файла text.txt:
Это тестовый файл для домашнего задания по программированию
```

## Task 3 Создайте пустую директорию mydir в текущей рабочей директории. Затем перейдите в эту директорию и создайте в ней три пустых файла: file1.txt, file2.txt и file3.txt. Наконец, выведите список файлов в директории на экран.
> Выведем код
```
(venv) user@lab:~/skurat/python/lesson27$ cat less27proj3.py
import os

os.mkdir("mydir")
os.chdir("mydir")

open("file1.txt", "w").close()
open("file2.txt", "w").close()
open("file3.txt", "w").close()

files = os.listdir()
print("Содержимое директории mydir:", files)
```
> Проверим
```
(venv) user@lab:~/skurat/python/lesson27$ python3 less27proj3.py
Содержимое директории mydir: ['file3.txt', 'file1.txt', 'file2.txt']
```

## Task 4 
**Создайте шаблон template.html, который будет содержать HTML-код 
для отображения списка пользователей. Шаблон должен использовать
цикл for для перебора элементов списка, и выводить имя и email
каждого пользователя. Затем создайте список пользователей в виде
списка словарей, передайте его в шаблон и отобразите результат на
экране.** 

> Выведем код исполняемого файла less27proj4.py
```
(env) user@lab:~/skurat/python/lesson27$ cat less27proj4.py
from jinja2 import Environment, FileSystemLoader
import os

users = [
    {"name": "Vasia", "email": "vasia@tech.me"},
    {"name": "Petia", "email": "petia@tech.me"},
    {"name": "Marfa", "email": "marfa@tech.me"}
]

template_dir = os.path.join(os.path.dirname(__file__), "templates")

env = Environment(loader=FileSystemLoader(template_dir))
template = env.get_template("template.html")

rendered_html = template.render(users=users)

output_file = os.path.join(os.path.dirname(__file__), "result.html")
with open(output_file, "w", encoding="utf-8") as f:
    f.write(rendered_html)

print(f"HTML файл с пользователями создан: {output_file}")
```
> Выведем код html файла
```
(env) user@lab:~/skurat/python/lesson27$ cat templates/template.html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Список пользователей</title>
</head>
<body>
    <h1>Пользователи</h1>
    <ul>
        {% for user in users %}
            <li>{{ user.name }} — {{ user.email }}</li>
        {% endfor %}
    </ul>
</body>
</html>
```
> Проверим выполнение
```
(env) user@lab:~/skurat/python/lesson27$ python3 less27proj4.py
HTML файл с пользователями создан: /home/user/skurat/python/lesson27/result.html
(env) user@lab:~/skurat/python/lesson27$ cat result.html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>Список пользователей</title>
</head>
<body>
    <h1>Пользователи</h1>
    <ul>

            <li>Vasia — vasia@tech.me</li>

            <li>Petia — petia@tech.me</li>

            <li>Marfa — marfa@tech.me</li>

    </ul>
</body>
```

