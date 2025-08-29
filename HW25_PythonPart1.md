# Python №1
Цель: Познакомиться с основами языка программирования Python и его
типами данных, а также научиться создавать простые программы на Python. В
ходе выполнения заданий вы установите Python на свой компьютер, изучите
историю языка Python, типы данных Python, а также научитесь работать со
строками, числами и переменными в Python.

## Tasks
1) Напишите программу, которая выводит на экран фразу "Привет, мир!".
   ```
   print("Hello, world!")
   ```
2) Напишите программу, которая выводит на экран результат вычисления 2+2.
   ```
   arg1 = 2
   print(f"Rusult of sum 2+2= {arg1+arg1}")
   ```
3) Напишите программу, которая запрашивает у пользователя его имя и выводит на экран сообщение "Привет, [имя]!".
   ```
   name = input("Insert your name: ")
   print(f"Hello, {name}")
   ```
4) Напишите программу, которая выводит на экран числа от 1 до 10
   ```
   n = 10
   i = 1
   while i <= n:
   print(f"i = {i}")
   i += 1
   ```
5) Напишите программу, которая запрашивает у пользователя его возраст и выводит на экран сообщение "Ваш возраст [возраст] лет".
   ```
   age = input("Insert your age: ")
   print(f"Your age is: {age}")
   ```
6) Напишите программу, которая запрашивает у пользователя два числа и выводит на экран их произведение.
   ```
   num1 = int(input("Insert number1: "))
   num2 = int(input("Insert number2: "))
   print(f"Multiply of {num1} * {num2} = {num1*num2}")
   ```
7) Напишите программу, которая запрашивает у пользователя слово и выводит на экран его первую букву.
   ```
   word = input("Insert some word: ")
   if word and word.isalpha():
     print(f"First letter: {word[0]}")
   elif not word:
     print("You didn't insert word")
   else:
     print("You must insert only letter")
   ```
8) Напишите программу, которая запрашивает у пользователя целое число и выводит на экран его квадрат.
   ```
   num = input("Insert integer number: ")
   if num and num.isdigit():
     num = int(num)
     print(f"Number pow 2 = {num ** 2}")
   elif not num:
     print("You din't insert number")
   else:
     print("You must insert only numbers")
   ```
9) Напишите программу, которая выводит на экран таблицу умножения на число 5
   ```
   n = 5
   print(f"Table of multiply on {n}")
   for i in range(1,11):
     print(f"{n} x {i} = {n*i}")
   ```
10) Напишите программу, которая запрашивает у пользователя два числа и выводит на экран их среднее арифметическое.
    ```
    num1 = input("Insert number1: ")
    num2 = input("Insert number2: ")
    if (num1 and num1.isdigit()) and (num2 and num2.isdigit()):
      num1 = float(num1)
      num2 = float(num2)
      average = (num1 + num2) / 2
      print(f"Average of number1 and number2 = {average}")
    elif not num1 or not num2:
      print("You didn't insert numbers")
    else:
      print("You must insert only numbers")
    ```
