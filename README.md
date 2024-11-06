# Лабораторная №2

## Цель работы

Научиться пользоваться Ansible Galaxy и писать роли

## Ход работы:
1.	Инициировать структуру роли “Docker” через Ansible-Galaxy
2.	Вынести из предыдущего плейбука задачи установки Docker в отдельную роль
3.	Параметризовать роль через переменные
4.	Создать в gitlab группу для репозиториев с ролями, запушить роль “Docker” в репозиторий
5.	Создать файл `requirements.yml` для установки роли Docker из репозитория
6.	Запустить приложение на нодах группы `[app]` используя ansible-playbook с ролью “Docker”

Из той же директории `/ansible` выполним команду для создания структуры каталогов для роли Decker:
```
ansible-galaxy init Docker
```
<img width="977" alt="Снимок экрана 2024-11-04 в 17 58 56" src="https://github.com/user-attachments/assets/31e5fb39-6c4f-4033-9b5f-f09b8cdb1d3b">

Команда создала новую папку Docker, теперь зайдем внутрь: 
```
cd Docker
```

Входим в аккаунт GitLab, создаем репозиторий с названием `ansible-docker-role`.
в файле `/ansible/Docker/tasks/main.yml` пишем:
```
- name: Update apt repository and Upgrade packages
  apt:
    update_cache: yes
    upgrade: dist

- name: Install dependencies for Docker
  apt: 
    name: "{{ packages }}"
  vars:
    packages:
    - apt-transport-https
    - ca-certificates
    - curl
    - gnupg-agent
    - software-properties-common

- name: Add Docker’s official GPG key
  apt_key:
    url: https://download.docker.com/linux/ubuntu/gpg
    state: present

- name: Set up the stable repository for Docker
  apt_repository:
    repo: deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable
    state: present

- name: Install Docker Engine
  apt: 
    name: docker-ce
    state: latest
```

Далее инициализируем git в папке Docker при помощи следующих команд:
```
git init
git add .
git commit -m "Initial commit of Docker role"
```
<img width="1142" alt="Снимок экрана 2024-11-04 в 17 59 56" src="https://github.com/user-attachments/assets/b9508bf0-c5dd-4058-8063-83012c4c232b">

Пушим код папки Docker при помощи следующими командами:
```
git remote add origin https://gitlab.com/devops9420745/ansible-docker-role
git push -u origin master --force
```
<img width="1249" alt="Снимок экрана 2024-11-04 в 18 01 45" src="https://github.com/user-attachments/assets/7b865155-4d40-4ef1-aeba-e839b8c9fcf9">

В директории `/ansible` редактируем файл `docker-install.yml`. Удаляем из него все задачи и добавляем ссылку на нашу роль:
```
- hosts: app
  become: yes
  roles:
    - Docker
```
В той же директории `/ansible` создаем файл `requirements.yml` для подключения к гиту:
```
- src: https://gitlab.com/devops9420745/ansible-docker-role
  scm: git
  version: master
  name: Docker
```
Возвращаемся обратно в директорию `/ansible` и запускаем PlayBook:
```
ansible-playbook -i inventory.yml docker-install.yml
```

<img width="671" alt="Снимок экрана 2024-11-04 в 18 06 45" src="https://github.com/user-attachments/assets/0fe0c558-b93d-447b-bfba-34da3f9001a5">

Результаты выполнения ansible-плейбука показывают, что было успешно выполнена задача установки Docker на узлах группы `[app]`


 
