## Домашнее задание к занятию "4.3. Языки разметки JSON и YAML"


### Обязательная задача 1
Мы выгрузили JSON, который получили через API запрос к нашему сервису:
```
    { "info" : "Sample JSON output from our service\t",
        "elements" :[
            { "name" : "first",
            "type" : "server",
            "ip" : 7175 
            }
            { "name" : "second",
            "type" : "proxy",
            "ip : 71.78.22.43
            }
        ]
    }
```
  Нужно найти и исправить все ошибки, которые допускает наш сервис
Исправленный синтаксис:
```json
{ "info" : "Sample JSON output from our service\\t", #пропущено экранирование спецсимвола \t
    "elements" :[
        { 
          "name" : "first",
          "type" : "server",
          "ip" : 7175
        },
        {
          "name" : "second",
          "type" : "proxy",
          "ip" : "71.78.22.43" #пропущены закрывающие кавычки и экранирование IP адреса
        }
    ]
}
```

### Обязательная задача 2
В прошлый рабочий день мы создавали скрипт, позволяющий опрашивать веб-сервисы и получать их IP. К уже реализованному функционалу нам нужно добавить возможность записи JSON и YAML файлов, описывающих наши сервисы. Формат записи JSON по одному сервису: `{ "имя сервиса" : "его IP"}`. Формат записи YAML по одному сервису: `- имя сервиса: его IP`. Если в момент исполнения скрипта меняется IP у сервиса - он должен так же поменяться в yml и json файле.

#### Ваш скрипт:
```python
#!/usr/bin/env python3
import  socket
from time import sleep
import json
import  yaml

list = {"drive.google.com":"0", "mail.google.com":"0", "google.com":"0"}
iteration = 1
while (1 == 1):
	for  name, address  in  list.items():
		ip = socket.gethostbyname(name)
		if  str(address) == str(ip) or  iteration == 1:
			print(f'{name}: {ip}')
		else:
			print(f'[ERROR] {name} IP mismatch: {list[name]}  {ip}')
		list[name] = ip
	print('\n')
#для работы с json-начало
	with  open('task_2_obj.json', 'w') as  json_file:
		json.dump(list, json_file)
#для работы с yml-начало
	with  open('task_2_obj.yml', 'w') as  yaml_file:
		yaml.dump(list, yaml_file, default_flow_style=False)
	iteration=iteration+1
	sleep(2)
```

#### Вывод скрипта при запуске при тестировании:
```bash
drive.google.com: 209.85.233.138
mail.google.com: 142.251.1.18
google.com: 209.85.233.101

[ERROR] drive.google.com IP mismatch: 209.85.233.138 173.194.222.102
mail.google.com: 142.251.1.18
[ERROR] google.com IP mismatch: 209.85.233.101 209.85.233.138

[ERROR] drive.google.com IP mismatch: 173.194.222.102 209.85.233.100
[ERROR] mail.google.com IP mismatch: 142.251.1.18 64.233.165.83
[ERROR] google.com IP mismatch: 209.85.233.138 209.85.233.139

[ERROR] drive.google.com IP mismatch: 209.85.233.100 209.85.233.138
[ERROR] mail.google.com IP mismatch: 64.233.165.83 64.233.165.19
[ERROR] google.com IP mismatch: 209.85.233.139 74.125.131.100

[ERROR] drive.google.com IP mismatch: 209.85.233.138 209.85.233.100
[ERROR] mail.google.com IP mismatch: 64.233.165.19 64.233.165.18
[ERROR] google.com IP mismatch: 74.125.131.100 209.85.233.138

[ERROR] drive.google.com IP mismatch: 209.85.233.100 209.85.233.102
[ERROR] mail.google.com IP mismatch: 64.233.165.18 64.233.165.83
google.com: 209.85.233.138
```

#### json-файл(ы), который(е) записал ваш скрипт:
```json
{"drive.google.com": "209.85.233.139", "mail.google.com": "64.233.165.18", "google.com": "209.85.233.113"}
```

#### yml-файл(ы), который(е) записал ваш скрипт:
```yaml
drive.google.com: 209.85.233.139
google.com: 209.85.233.113
mail.google.com: 64.233.165.18
```
