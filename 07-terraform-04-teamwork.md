## Домашнее задание к занятию "7.4. Средства командной работы над инфраструктурой."

### Задача 1. Настроить terraform cloud (необязательно, но крайне желательно).В это задании предлагается познакомиться со средством командой работы над инфраструктурой предоставляемым разработчиками терраформа.

-   Зарегистрируйтесь на  [https://app.terraform.io/](https://app.terraform.io/). (регистрация бесплатная и не требует использования платежных инструментов).
-   Создайте в своем github аккаунте (или другом хранилище репозиториев) отдельный репозиторий с конфигурационными файлами прошлых занятий (или воспользуйтесь любым простым конфигом).
-   Зарегистрируйте этот репозиторий в  [https://app.terraform.io/](https://app.terraform.io/).
-   Выполните plan и apply.

В качестве результата задания приложите снимок экрана с успешным применением конфигурации.

[![imageban](https://i2.imageban.ru/out/2022/04/05/c710a49bfccf1c6e24c1b306cff08a5e.png)](https://imageban.ru)
Статус выполнения:
[![imageban](https://i2.imageban.ru/out/2022/04/05/e9647de0b359dec1cd607455de4b32c5.png)](https://imageban.ru)

[![imageban](https://i6.imageban.ru/out/2022/04/05/551c2adeba20d6120db248137bf72654.png)](https://imageban.ru)

[![imageban](https://i2.imageban.ru/out/2022/04/05/e0c7d85bb1968c0ec4df4096bc73a9c9.png)](https://imageban.ru)


### Задача 2. Написать серверный конфиг для Atlantis'a.

Смысл задания – познакомиться с документацией о  [серверной](https://www.runatlantis.io/docs/server-side-repo-config.html)  конфигурации и конфигурации уровня  [репозитория](https://www.runatlantis.io/docs/repo-level-atlantis-yaml.html).

Создай  `server.yaml`  который скажет атлантису:

1.  Укажите, что атлантис должен работать только для репозиториев в вашем github (или любом другом) аккаунте.
2.  На стороне клиентского конфига разрешите изменять  `workflow`, то есть для каждого репозитория можно будет указать свои дополнительные команды.
3.  В  `workflow`  используемом по-умолчанию сделайте так, что бы во время планирования не происходил  `lock`  состояния.

Создай  `atlantis.yaml`  который, если поместить в корень terraform проекта, скажет атлантису:

1.  Надо запускать планирование и аплай для двух воркспейсов  `stage`  и  `prod`.
2.  Необходимо включить автопланирование при изменении любых файлов  `*.tf`.

В качестве результата приложите ссылку на файлы  `server.yaml`  и  `atlantis.yaml`.

server.yaml
```
repos:
- id: github.com/EvgeniyKlevtsov/devops-lessons

  workflow: terraflow
  apply_requirements: [approved, mergeable]

  allowed_overrides: [workflow]
  allow_custom_workflows: true

workflows:
  hinetology:
    plan: 
      steps:
        - init:
            extra_args: ["-lock=false"]
        - plan:
            extra_args: ["-lock=false"]
    apply:
      steps: [apply]
```
atlantis.yaml
```#atlantis.yaml
version: 3
projects:
- dir: terraflow
  workspace: stage
  autoplan:
    when_modified: ["../modules/**/*.tf", "*.tf*"]
    enabled: true
- dir: terraflow
  workspace: prod
  autoplan:
    when_modified: ["../modules/**/*.tf", "*.tf*"]
    enabled: true
```
### Задача 3. Знакомство с каталогом модулей.

-   В каталоге модулей найдите официальный модуль от aws для создания ec2 инстансов.
-   Изучите как устроен модуль. Задумайтесь, будете ли в своем проекте использовать этот модуль или непосредственно ресурс aws_instance без помощи модуля?
-   В рамках предпоследнего задания был создан ec2 при помощи ресурса aws_instance. Создайте аналогичный инстанс при помощи найденного модуля.

В качестве результата задания приложите ссылку на созданный блок конфигураций.

К сожалению, я не нашёл никаких модулей под YC, а AWS не дает регистрироваться в моём регионе. Да и GitHub "фигвамы" рисует 
```
It appears your account may be based in a U.S.-sanctioned region.
As a result, due to U.S. trade controls law restrictions, we are unable to provide private repository services and paid services
for your account. GitHub has preserved, however, your access to [certain free services for public repositories]
https://docs.github.com/en/github/site-policy/github-and-trade-controls#what-is-available-and-not-available). If your account has been flagged in error, and you are not located in or resident in a sanctioned region, please [file an appeal]
(https://docs.github.com/en/github/site-policy/github-and-trade-controls#how-is-github-ensuring-that-folks-not-living-in-andor-having-professional-links-to-the-sanctioned-countries-and-territories-still-have-access-or-ability-to-appeal). 
Please read about [GitHub and Trade Controls] https://docs.github.com/en/github/site-policy/github-and-trade-controls) for more information.
```
