#Домашнее задание к занятию "08.04 Работа с Roles"

PLAYBOOK `site.yml` содержит 3 playbook:
 - 'Install Clickhouse' (вызывает роль clickhouse на хосте clickhouse)
 - 'Install Vector' (вызывает роль vector-role на хосте vector)
 - 'Install Lighthouse' (вызывает роль clickhouse-role на хосте lighthouse)

###Роль "clickhouse"
Получена с помощью команды 'ansible-galaxy install -r -requirements.yml' 
и вот такого файла requirements.yml:
```text
---
  - src: git@github.com:AlexeySetevoi/ansible-clickhouse.git
    scm: git
    version: "1.11.0"
    name: clickhouse
```

###Роль "vector-role"
В defaults/main.yml указана версия:
```text
---
# defaults file for vector-role
vector_version: "0.21.2"
```

Таски вынесены в tasks/main.yml
```text
---
# tasks file for vector-role
- name: Get vector distrib
  ansible.builtin.get_url:
    url: "https://packages.timber.io/vector/{{ vector_version }}/vector-{{ vector_version }}-1.x86_64.rpm"
    dest: "./vector-{{ vector_version }}-1.x86_64.rpm"
- name: Install vector packages
  become: true
  ansible.builtin.yum:
    name:
      - vector-{{ vector_version }}-1.x86_64.rpm
  notify: Start vector service
```

Хен

###Роль "clickhouse-role"



Таски:
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