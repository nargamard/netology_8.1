#Домашнее задание к занятию "08.02 Работа с Playbook"

PLAYBOOK `site.yml` содержит 2 PLAY:
 - 'Install Clickhouse'
 - 'Install Vector'

###PLAY "Install Clickhouse"
Таски:
- Get clickhouse distrib
```text
Таск скачивает пакеты clickhouse с помочью get_url.
С помощью with_items организован цикл, который перебирает элементы clickhouse_packages.
clickhouse_packages указаны в clickhouse/vars.yml
С помощью шаблона в url вставляется номер версии пакетов из переменной clickhouse_version в том же файле.
dest показывает, как должен называться пакет и куда его положить после получения.
Так как название одного из пакетов не укладывается в шаблон, использована конструкция rescue.
Тот пакет, который не удастся получить будет получен через get_url, указанный после rescue.
```
- Install clickhouse packages
```text
Включаем become чтобы было можно устанавливать пакеты.
Даём ansible.builtin.yum список пакетов для установки, где снова используется параметр clickhouse_version.
После этого notify показывает, что надо вызвать хендлер "Start clickhouse service", который запустит сервис.
```
- Flash handlers
```text
flush_handlers вызывает хендлер сейчас, иначе он выполнится после всех TASK, а запущенный сервис нужен до выполнения следующего TASK.
```
- Create database
```text
ansible.builtin.command запускает команду "clickhouse-client -q 'create database logs;'", которая создаёт базу данных.
```
Хендлеры:
```text
"Start clickhouse service", запускает сервис clickhouse-server (чтобы это было позволено, включаем become).
```

###PLAY "Install vector"
Таски:
- Get vector distrib
```text
Таск скачивает пакет vector с помочью get_url.
В имени пакета используется шаблон, который берет номер версии пакета из параметра vector_version.
Этот параметр задан в файле vector/vars.yml
```
- Install vector packages
```text
Включаем become чтобы было можно устанавливать пакеты.
Даём ansible.builtin.yum список пакетов для установки, где всего один пакет, в имени которого используется тот же параметр vector_version.
После этого notify показывает, что надо вызвать хендлер "Start vector service", который запустит сервис.
```
Хендлеры:
```text
"Start vector service", запускает сервис vector (чтобы это было позволено, включаем become).
```

###PLAY "Install Lighthouse"
Пре-таски:
- Install unzip
```text
Устанавливает unzip, который понадобится для распаковки архива с lighthouse
```
Таски:
- Add repo Nginx
```text
Добавляет официальный репозиторий nginx, become включен для этой операции
```
- Install Nginx
```text
Устанавливает nginx, become включен для этой операции, далее вызывается хендлер "Start nginx service".
Он запустится после всех task.
```
- get zip & unarchive
```text
ansible.builtin.unarchive распаковывает архив, полученный напрямую из репозитория
```
- Copy lighthouse sources
```text
Копирует распакованные файлы в /usr/share/nginx/html - дефолтная директория для html nginx
```
-  Cleaning files
```text
Удаляет директорию, полученную после распаковки архива
```
Хендлеры:
- Start nginx service
```text
Запускает сервис nginx (чтобы это было позволено, включаем become).
```