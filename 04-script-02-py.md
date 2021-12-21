## Домашнее задание к занятию "4.2. Использование Python для решения типовых DevOps задач"

### Обязательная задача 1

Есть скрипт:
```python
#!/usr/bin/env python3
a = 1
b = '2'
c = a + b
```

#### Вопросы:
| Вопрос  | Ответ |
| ------------- | ------------- |
| Какое значение будет присвоено переменной `c`?  | Python попытается сложить две переменные разных типов `int` (a) и `str` (b) и выдаст ошибку: `TypeError: unsupported operand type(s) for +: 'int' and 'str'`.  |
| Как получить для переменной `c` значение 12?  | привести `a` к строке: c = str(a)+b |
| Как получить для переменной `c` значение 3?  | привести b к целому числу: c = a+int(b)  |

### Обязательная задача 2
Мы устроились на работу в компанию, где раньше уже был DevOps Engineer. Он написал скрипт, позволяющий узнать, какие файлы модифицированы в репозитории, относительно локальных изменений. Этим скриптом недовольно начальство, потому что в его выводе есть не все изменённые файлы, а также непонятен полный путь к директории, где они находятся. Как можно доработать скрипт ниже, чтобы он исполнял требования вашего руководителя?

```python
#!/usr/bin/env python3

import os

bash_command = ["cd ~/netology/sysadm-homeworks", "git status"]
result_os = os.popen(' && '.join(bash_command)).read()
is_change = False
for result in result_os.split('\n'):
    if result.find('modified') != -1:
        prepare_result = result.replace('\tmodified:   ', '')
        print(prepare_result)
        break
```

#### Ваш скрипт:
```python
#!/usr/bin/env python3

import  os

bash_command = ["cd ~/devops-netology", "git status"]
result_os = os.popen(' && '.join(bash_command)).read()
full_path = os.popen('pwd').read()
print('Рабочая директория: ' + full_path)

print('Изменённые файлы:')
for  result  in  result_os.split('\n'):
	if  result.find('изменено') != -1:
prepare_result = result.replace('\tизменено: ', '')
print(prepare_result)

print('Новые файлы:')
for  result  in  result_os.split('\n'):
	if  result.find('новый файл') != -1:
prepare_result = result.replace('\tновый файл: ', '')
print(prepare_result)

print('Неотслеживаемые файлы:')
for  result  in  result_os.split('\n'):
	if  result.find('Неотслеживаемые файлы:') != -1:
pos = result_os.split('\n').index("Неотслеживаемые файлы:")
prepare_result= result_os.split('\n')[pos:]

del  prepare_result[0]
del  prepare_result[0]
prepare_result.pop()
prepare_result.pop()
for  i  in  prepare_result:
	print(i)
```

#### Вывод скрипта при запуске при тестировании:
```bash
evgeniy@evgeniy-ms7788:~/devops-netology$ python3 test-py.py 
Рабочая директория: /home/evgeniy/devops-netology

Изменённые файлы:
	test-py.py
Новые файлы:
	test-py.py
Неотслеживаемые файлы:
	03-sysadmin-02-terminal.md
	terraform/
	"\320\237\321\203\321\201\321\202\320\276\320\271 \320\264\320\276\320\272\321\203\320\274\320\265\320\275\321\202"
evgeniy@evgeniy-ms7788:~/devops-netology$
```

### Обязательная задача 3
1. Доработать скрипт выше так, чтобы он мог проверять не только локальный репозиторий в текущей директории, а также умел воспринимать путь к репозиторию, который мы передаём как входной параметр. Мы точно знаем, что начальство коварное и будет проверять работу этого скрипта в директориях, которые не являются локальными репозиториями.

#### Ваш скрипт:
```python
#!/usr/bin/env python3
import  os

dir = 'pwd ' + os.getcwd()
bash_command = [dir, "git status"]
result_os = os.popen(' && '.join(bash_command)).read()
print('Основной каталог: ' + os.getcwd() )
print('Рабочая директория: ' + os.getcwd())
print('Изменённые файлы:')
	for  result  in  result_os.split('\n'):
	if  result.find('изменено') != -1:
	prepare_result = result.replace('\tизменено: ', '')
	print(prepare_result)
print('Новые файлы:')
	for  result  in  result_os.split('\n'):
	if  result.find('новый файл') != -1:
	prepare_result = result.replace('\tновый файл: ', '')
	print(prepare_result)
print('Неотслеживаемые файлы:')
	for  result  in  result_os.split('\n'):
	if  result.find('Неотслеживаемые файлы:') != -1:
		pos = result_os.split('\n').index("Неотслеживаемые файлы:")
	prepare_result= result_os.split('\n')[pos:]
del  prepare_result[0]
del  prepare_result[0]
prepare_result.pop()
prepare_result.pop()
for  i  in  prepare_result:
print(i)
```

#### Вывод скрипта при запуске при тестировании:
```bash
evgeniy@evgeniy-ms7788:~/devops-netology$ python3 test-py2.py 
Основной каталог: /home/evgeniy/devops-netology
Рабочая директория: /home/evgeniy/devops-netology
Изменённые файлы:
   test-py.py
Новые файлы:
 test-py.py
Неотслеживаемые файлы:
	03-sysadmin-02-terminal.md
	terraform/
	test-py2.py
	"\320\237\321\203\321\201\321\202\320\276\320\271 \320\264\320\276\320\272\321\203\320\274\320\265\320\275\321\202"
evgeniy@evgeniy-ms7788:~/devops-netology$ 
```

### Обязательная задача 4
1. Наша команда разрабатывает несколько веб-сервисов, доступных по http. Мы точно знаем, что на их стенде нет никакой балансировки, кластеризации, за DNS прячется конкретный IP сервера, где установлен сервис. Проблема в том, что отдел, занимающийся нашей инфраструктурой очень часто меняет нам сервера, поэтому IP меняются примерно раз в неделю, при этом сервисы сохраняют за собой DNS имена. Это бы совсем никого не беспокоило, если бы несколько раз сервера не уезжали в такой сегмент сети нашей компании, который недоступен для разработчиков. Мы хотим написать скрипт, который опрашивает веб-сервисы, получает их IP, выводит информацию в стандартный вывод в виде: <URL сервиса> - <его IP>. Также, должна быть реализована возможность проверки текущего IP сервиса c его IP из предыдущей проверки. Если проверка будет провалена - оповестить об этом в стандартный вывод сообщением: [ERROR] <URL сервиса> IP mismatch: <старый IP> <Новый IP>. Будем считать, что наша разработка реализовала сервисы: `drive.google.com`, `mail.google.com`, `google.com`.

#### Ваш скрипт:
```python
##!/usr/bin/env python3
import  socket
import  time
dns = [("drive.google.com","0"), ("mail.google.com", "0"), ("google.com", "0")]
a = 1
while (True):
	for  i  in  dns:
		pos = dns.index(i)
		ip_add = socket.gethostbyname(i[0])
		if  str(i[1]) == str(ip_add) or  a == 1:
			print(f'{i[0]}: {i[1]}')
		else:
			print(f'[ERROR] {i[0]} IP mismatch: {i[1]}  {ip_add}')
		dns[pos] = (i[0], ip_add)
	print('\n')
	a=a+1
	time.sleep(1)
```

#### Вывод скрипта при запуске при тестировании:
```bash
drive.google.com: 209.85.233.113
[ERROR] mail.google.com IP mismatch: 74.125.205.83 74.125.205.18
[ERROR] google.com IP mismatch: 74.125.131.138 74.125.131.102

drive.google.com: 209.85.233.113
[ERROR] mail.google.com IP mismatch: 74.125.205.18 74.125.205.19
[ERROR] google.com IP mismatch: 74.125.131.102 74.125.131.113

[ERROR] drive.google.com IP mismatch: 209.85.233.113 209.85.233.138
[ERROR] mail.google.com IP mismatch: 74.125.205.19 74.125.205.17
google.com: 74.125.131.113

[ERROR] drive.google.com IP mismatch: 209.85.233.138 209.85.233.101
[ERROR] mail.google.com IP mismatch: 74.125.205.17 74.125.205.83
[ERROR] google.com IP mismatch: 74.125.131.113 74.125.131.138

[ERROR] drive.google.com IP mismatch: 209.85.233.101 209.85.233.100
[ERROR] mail.google.com IP mismatch: 74.125.205.83 74.125.205.17
google.com: 74.125.131.138

drive.google.com: 209.85.233.100
mail.google.com: 74.125.205.17
google.com: 74.125.131.138
```
