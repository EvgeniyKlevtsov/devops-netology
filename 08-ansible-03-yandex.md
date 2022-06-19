## Домашнее задание к занятию "08.03 Использование Yandex Cloud"
### Подготовка к выполнению

1.  Подготовьте в Yandex Cloud три хоста: для  `clickhouse`, для  `vector`  и для  `lighthouse`.

Готово.

### Основная часть

1.  Допишите playbook: нужно сделать ещё один play, который устанавливает и настраивает lighthouse.

Готово. Для дополнительно установлены пакеты epel-release, nginx и git.

2.  При создании tasks рекомендую использовать модули:  `get_url`,  `template`,  `yum`,  `apt`.
3.  Tasks должны: скачать статику lighthouse, установить nginx или любой другой webserver, настроить его конфиг для открытия lighthouse, запустить webserver.

Сайт lighthouse запущен на порту 80.
```
server {
    listen 80;
    server_name localhost;

    access_log   /var/log/nginx/{{ lighthouse_access_log_name }}.log  main;

    location / {
        root {{ lighthouse_location_dir }};
        index index.html;
    }
}
```
4.  Приготовьте свой собственный inventory файл  `prod.yml`.

Файл inventory формируется в скриптами терраформ в который подставляются ip адреса ВМ.
```yml
    clickhouse:
      hosts:
        clickhouse-01:
          ansible_host: ${yandex_compute_instance.node01.network_interface.0.nat_ip_address}
          ansible_connection: ssh
          ansible_user: centos
          ansible_become_user: root
    vector:
      hosts:
        vector-01:
          ansible_host: ${yandex_compute_instance.node02.network_interface.0.nat_ip_address}
          ansible_connection: ssh
          ansible_user: centos
          ansible_become_user: root
    lighthouse:
      hosts:
        lighthouse-01:
          ansible_host: ${yandex_compute_instance.node03.network_interface.0.nat_ip_address}
          ansible_connection: ssh
          ansible_user: centos
          ansible_become_user: root
```

5.  Запустите  `ansible-lint site.yml`  и исправьте ошибки, если они есть.

Готово.

6.  Попробуйте запустить playbook на этом окружении с флагом  `--check`.

Готово.

7.  Запустите playbook на  `prod.yml`  окружении с флагом  `--diff`. Убедитесь, что изменения на системе произведены.

Готово.

8.  Повторно запустите playbook с флагом  `--diff`  и убедитесь, что playbook идемпотентен.

Готово.

9.  Подготовьте README.md файл по своему playbook. В нём должно быть описано: что делает playbook, какие у него есть параметры и теги.

Готово.

10.  Готовый playbook выложите в свой репозиторий, поставьте тег  `08-ansible-03-yandex`  на фиксирующий коммит, в ответ предоставьте ссылку на него.
