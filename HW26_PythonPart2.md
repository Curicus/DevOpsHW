# Python №2
Цель: Закрепление и практика основных концепций и методов работы с
данными в Python, таких как работа со строками, списками, кортежами
и словарями, использование циклов и условных операторов.
## Task 1 
Написать скрипт, который принимает на вход строку и выводит на
экран количество букв в верхнем регистре, количество букв в нижнем
регистре, количество цифр и количество символов пунктуации.

> Активируем виртуальное окружение `user@lab:~/skurat/python$ source /home/user/venv/bin/activate`

> Выведем код скрипта
```
(venv) user@lab:~/skurat/python$ cat proj2.py
#!/usr/bin/env python3
import string
def mystring(s: str):
    upper_symbol = 0
    for x in s:
        if x.isupper():
            upper_symbol += 1
    print(f"Колличество букв в верхнем регистре равно: {upper_symbol}")

    lower_symbol = 0
    for x in s:
        if x.islower():
            lower_symbol += 1
    print(f"Колличество букв в нижнем регистре равно: {lower_symbol}")

    digit_symbol = 0
    for x in s:
        if x.isdigit():
            digit_symbol += 1
    print(f"Колличество цифр равно: {digit_symbol}")

    punct_symbol = 0
    for x in s:
        if x in string.punctuation:
            punct_symbol += 1
    print(f"Колличество символов пунктуации равно: {punct_symbol}")    

userstring = input("Введите строку: ")
mystring(userstring)
```
> Проверим работу скрипта
```
(venv) user@lab:~/skurat/python$ ./proj2.py
Введите строку: DevOps2025!!!
Колличество букв в верхнем регистре равно: 2
Колличество букв в нижнем регистре равно: 4
Колличество цифр равно: 4
Колличество символов пунктуации равно: 3
```

# Task 2 
Написать скрипт, который принимает на вход два списка и выводит на
экран элементы, которые присутствуют в обоих списках.

> Выведем код скрипта
```
#!/usr/bin/env python3
list1 = input("Введите элементы первого списка через пробел: ").split()
list2 = input("Введите элементы второго списка через пробел: ").split()

similar_elemnts = set(list1) & set(list2)

if similar_elemnts:
    print(f"Общие элементы в этих списках: ", " ".join(similar_elemnts))
else:
    print(f"Общих элементов нет.")
```
> Выведем результат исполнения, тоько сделаем файл исполняемым `chmod +x proj3.py`
```
(venv) user@lab:~/skurat/python$ ./proj3.py
Введите элементы первого списка через пробел: 1 2 3 4 5 6
Введите элементы второго списка через пробел: 3 4 5 6 7 8
Общие элементы в этих списках:  3 5 4 6                       # Интересно почему 4 и 5 поменялись местами
(venv) user@lab:~/skurat/python$ ./proj3.py
Введите элементы первого списка через пробел: 1 2 3 4 5 
Введите элементы второго списка через пробел: 6 7 8 9 0
Общих элементов нет.
```
# Task 3
Написать скрипт, который принимает на вход массив чисел и сортирует
его в порядке убывания.

> Выведем код скрипта
```
#!/usr/bin/env python3
numbers = input("Введите числа через пробел: ").split()
numbers = [int(num) for num in numbers]
numbers.sort(reverse=True)
print(f"Отсортированный массив по убыванию: {numbers}")
```

> Выведем результаты исполнения
```
(venv) user@lab:~/skurat/python$ chmod +x proj4.py
(venv) user@lab:~/skurat/python$ ./proj4.py
Введите числа через пробел: 5 4 2 3 5 77
Отсортированный массив по убыванию: [77, 5, 5, 4, 3, 2]
```

# Task 4
Написать скрипт, который принимает на вход кортеж и проверяет, все
ли его элементы являются уникальными.

> Выведем код скрипта
```
#!/usr/bin/env python3
string = input("Введите элементы кортежа через пробел: ").split()
insert_tuple = tuple(string)
if len(insert_tuple) == len(set(insert_tuple)):
    print("Все элементы уникальны")
else:
    print("Есть повторяющиеся элементы")
```

> Выведем результаты исполнения
```
(venv) user@lab:~/skurat/python$ chmod +x proj5.py
(venv) user@lab:~/skurat/python$ ./proj5.py
Введите элементы кортежа через пробел: 4 5 6 7 8 9
Все элементы уникальны
(venv) user@lab:~/skurat/python$ ./proj5.py
Введите элементы кортежа через пробел: 2 3 4 4 5 6
Есть повторяющиеся элементы
```

