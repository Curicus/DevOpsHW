# Python #4
Цель: Практически применить изученные темы: ООП, классы и ООП в
Python.

## Task 1 Создайте класс "Круг", который имеет атрибуты радиус и цвет, и методы вычисления площади и длины окружности. Создайте несколько объектов этого класса и вызовите его методы для каждого объекта.

> Выведем код исполняемого файла less28proj1.py
```
(venv) user@lab:~/skurat/python/lesson28$ cat less28proj1.py
import math

class Circle:
    def __init__(self, radius, color):
        self.radius = radius
        self.color = color

    def area(self):
        return math.pi * self.radius ** 2

    def circumference(self):
        return 2 * math.pi * self.radius

    def __str__(self):
        return f"Круг(radius={self.radius}, color='{self.color}')"

circle1 = Circle(5, "красный")
circle2 = Circle(3, "синий")
circle3 = Circle(7, "зелёный")

circles = [circle1, circle2, circle3]

for c in circles:
    print(c)
    print(f"Площадь: {c.area():.2f}")
    print(f"Длина окружности: {c.circumference():.2f}")
    print("-" * 30)
```
> Проверим выполнение
```
(venv) user@lab:~/skurat/python/lesson28$ python3 less28proj1.py
Круг(radius=5, color='красный')
Площадь: 78.54
Длина окружности: 31.42
------------------------------
Круг(radius=3, color='синий')
Площадь: 28.27
Длина окружности: 18.85
------------------------------
Круг(radius=7, color='зелёный')
Площадь: 153.94
Длина окружности: 43.98
------------------------------
```

## Task 2 
**Создайте класс "Банковский счет", который имеет атрибуты номер
счета, имя владельца, баланс и методы пополнения и снятия денег со
счета. Создайте несколько объектов этого класса и вызовите его методы
для каждого объекта.**

> Выведем код исполняемого файла less28proj2.py  
```
(venv) user@lab:~/skurat/python/lesson28$ cat less28proj2.py
class BankAccount:
    def __init__(self, account_number, owner_name, balance=0.0):
        self.account_number = account_number
        self.owner_name = owner_name
        self.balance = balance
 
 #Пополнение
    def deposit(self, summa):
        if summa > 0:
            self.balance += summa
            print(f"На счёт {self.account_number} запулено {summa:.2f} руб.")
        else:
            print("Сумма пополнения должна быть положительной!")

#Снятие
    def withdraw(self, summa):
        if summa <= 0:
            
            print("Сумма снятия должна быть положительной!")
        elif summa > self.balance:
            print(f"Пробуем снять {summa}") 
            print(f"Недостаточно средств на счёте {self.account_number}.")
        else:
            self.balance -= summa
            print(f"Со счёта {self.account_number} снято {summa:.2f} руб.")

    def __str__(self):
        return f"Счёт №{self.account_number}, Владелец: {self.owner_name}, Баланс: {self.balance:.2f} руб."


account1 = BankAccount("12345", "Вася Пупкин", 1000.0)
account2 = BankAccount("67890", "Петя Петечкин", 500.0)
account3 = BankAccount("54321", "Марфа Васильевна")

print(account1)
account1.deposit(500)
account1.withdraw(200)
print(account1)
print("-" * 40)

print(account2)
account2.withdraw(600)  
account2.deposit(300)
print(account2)
print("-" * 40)

print(account3)
account3.deposit(1000)
account3.withdraw(250)
```
> Проверим выполнение

```
print(account3)(venv) user@lab:~/skurat/python/lesson28$ python3 less28proj2.py
Счёт №12345, Владелец: Вася Пупкин, Баланс: 1000.00 руб.
На счёт 12345 запулено 500.00 руб.
Со счёта 12345 снято 200.00 руб.
Счёт №12345, Владелец: Вася Пупкин, Баланс: 1300.00 руб.
----------------------------------------
Счёт №67890, Владелец: Петя Петечкин, Баланс: 500.00 руб.
Пробуем снять 600
Недостаточно средств на счёте 67890.
На счёт 67890 запулено 300.00 руб.
Счёт №67890, Владелец: Петя Петечкин, Баланс: 800.00 руб.
----------------------------------------
Счёт №54321, Владелец: Марфа Васильевна, Баланс: 0.00 руб.
На счёт 54321 запулено 1000.00 руб.
Со счёта 54321 снято 250.00 руб.
Счёт №54321, Владелец: Марфа Васильевна, Баланс: 750.00 руб.
```

## Task 3
**Создайте класс "Студент", который имеет атрибуты имя, возраст и
средний балл. Создайте методы для вычисления среднего балла и
определения статуса студента (отличник, хорошист, троечник).Создайте
несколько объектов этого класса и вызовите его методы для каждого
объекта.**

> Выведем код исполняемого файла less28proj3.py
```
class Student:
    def __init__(self, name, age, marks):
        self.name = name
        self.age = age
        self.marks = marks

    def average(self):
        if len(self.marks) == 0:
            return f"Введите оценки для {self.name}"
        return sum(self.marks) / len(self.marks)

    def status(self):
        avg = self.average()
        if avg >= 4.5:
            return "Отличник"
        elif avg >= 3.5:
            return "Хорошист"
        else:
            return "Троечник"

    def __str__(self):
        avg = self.average()
        if isinstance(avg, str):
            return f"Введите оценки для {self.name}"
        else:
            return f"Студент: {self.name}, Возраст: {self.age}, Средний балл: {self.average():.2f}, Статус: {self.status()}"

student1 = Student("Вася", 20, [2, 3, 3, 3])
student2 = Student("Марфа", 19, [4, 4, 3, 4])
student3 = Student("Петя", 21, [])
student4 = Student("Фрося", 22, [5, 4, 5, 5, 5])

students = [student1, student2, student3, student4]

for s in students:
    print(s)
```
> Проверим имполнение с проверкой на отсутствие оценок
```
(venv) user@lab:~/skurat/python/lesson28$ python3 less28proj3.py
Студент: Вася, Возраст: 20, Средний балл: 2.75, Статус: Троечник
Студент: Марфа, Возраст: 19, Средний балл: 3.75, Статус: Хорошист
Введите оценки для Петя
Студент: Фрося, Возраст: 22, Средний балл: 4.80, Статус: Отличник
```

## Task 4
**Создайте класс "Книга", который имеет атрибуты название, автор и год
издания. Создайте методы для получения и изменения этих
атрибутов.Создайте несколько объектов этого класса и вызовите его
методы для каждого объекта.**

> Выведем код исполняемого файла less28proj4.py
```
(venv) user@lab:~/skurat/python/lesson28$ cat less28proj4.py
class Book:
    def __init__(self, title, author, year):
        self.title = title
        self.author = author
        self.year = year

    def get_title(self):
        return self.title

    def get_author(self):
        return self.author

    def get_year(self):
        return self.year

    def set_title(self, new_title):
        self.title = new_title

    def set_author(self, new_author):
        self.author = new_author

    def set_year(self, new_year):
        self.year = new_year

    def __str__(self):
        return f"'{self.title}' — {self.author}, {self.year} г."

book1 = Book("Идиот", "Ф.М. Достоевский", 1868)
book2 = Book("Граф Монте-Кристо", "А. Дюма", 1846)
book3 = Book("Мастер и Маргарита", "М.А. Булгаков", 1967)

# Выводим книги
print(book1)
print(book2)
print(book3)

print("\n--- Исправленное ---")
book1.set_title("Идиот (новое издание)")
book1.set_year(2020)
book2.set_title("Граф Монте-Кристо (новое издание)")
book2.set_year(2019)
print(book1)
```
> Проверим имполнение
```
(venv) user@lab:~/skurat/python/lesson28$ python3 less28proj4.py
'Идиот' — Ф.М. Достоевский, 1868 г.
'Граф Монте-Кристо' — А. Дюма, 1846 г.
'Мастер и Маргарита' — М.А. Булгаков, 1967 г.

--- Изменяем данные ---
'Идиот (новое издание)' — Ф.М. Достоевский, 2020 г.
'Граф Монте-Кристо (новое издание)' — А. Дюма, 2019 г.
```


## Task 5
**Создайте класс "Автомобиль", который имеет атрибуты марка, модель,
цвет и год выпуска. Создайте методы для получения и изменения этих
атрибутов. Создайте несколько объектов этого класса и вызовите его
методы для каждого объекта**

> Выведем код исполняемого файла less28proj5.py
```
(venv) user@lab:~/skurat/python/lesson28$ cat less28proj5.py
class Car:
    def __init__(self, brand, model, color, year):
        self.brand = brand
        self.model = model
        self.color = color
        self.year = year

    def get_brand(self):
        return self.brand

    def get_model(self):
        return self.model

    def get_color(self):
        return self.color

    def get_year(self):
        return self.year

    def set_brand(self, new_brand):
        self.brand = new_brand

    def set_model(self, new_model):
        self.model = new_model

    def set_color(self, new_color):
        self.color = new_color

    def set_year(self, new_year):
        self.year = new_year

    def __str__(self):
        return f"{self.year} {self.brand} {self.model}, цвет: {self.color}"

car1 = Car("Toyota", "Camry", "черный", 2016)
car2 = Car("Mitsubishi", "ASX", "белый", 2014)
car3 = Car("Audi", "A4", "синий", 2005)

print(car1)
print(car2)
print(car3)

print("\n--- Иcправленное ---")
car1.set_color("мокрый асфальт")
car1.set_year(2026)
car2.set_color("хаки")
car2.set_year(2027)
print(car1)
print(car2)
```
> Проверим имполнение
```
(venv) user@lab:~/skurat/python/lesson28$ python3 less28proj5.py
2016 Toyota Camry, цвет: черный
2014 Mitsubishi ASX, цвет: белый
2005 Audi A4, цвет: синий

--- Иcправленное ---
2026 Toyota Camry, цвет: мокрый асфальт
2027 Mitsubishi ASX, цвет: хаки
```
