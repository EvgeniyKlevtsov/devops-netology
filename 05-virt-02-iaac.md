## Домашнее задание к занятию "5.2. Применение принципов IaaC в работе с виртуальными машинами"
### Задача 1

-   Опишите своими словами основные преимущества применения на практике IaaC паттернов.
-   Какой из принципов IaaC является основополагающим?

Меньше ошибок из-за человеческого фактора при развертывании инфраструктуры. Уменьшение времени развертывания за счет того что все действия по установке происходят автоматически. Автоматизация рутинных задач. 
Основополагающий принцип IAAC - это паттерн, по которому процесс создания/настройки инфраструктуры аналогичен процессу разработки программного обеспечения.

### Задача 2

-   Чем Ansible выгодно отличается от других систем управление конфигурациями?
-   Какой, на ваш взгляд, метод работы систем конфигурации более надёжный push или pull?

Ansible прост для понимания, не требует для работы агента, работает по SSH, совместим с множеством технологий, легко расширяется за счет модулей.
У каждого из методов работы есть свои плюсы и минусы. Самым оптимальным является комбинирование их в единой системе. Например, использование метода pull для разворачивания тестовых стендов и метода push для работы с продакшеном.

### Задача 3

Установить на личный компьютер:

-   VirtualBox
-   Vagrant
-   Ansible

_Приложить вывод команд установленных версий каждой из программ, оформленный в markdown._

```
VirtualBox
evgeniy@home:~$ VirtualBox --help
Oracle VM VirtualBox VM Selector v6.1.32
(C) 2005-2022 Oracle Corporation
All rights reserved.

No special options.

If you are looking for --startvm and related options, you need to use VirtualBoxVM.
```
```
Vagrant
evgeniy@home:~$ vagrant --version
Vagrant 2.2.3
```
```
Ansible
evgeniy@home:~$ ansible --version
ansible 2.7.7
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/home/evgeniy/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  executable location = /usr/bin/ansible
  python version = 3.7.3 (default, Jan 22 2021, 20:04:44) [GCC 8.3.0]
```
