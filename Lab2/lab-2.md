# Отчет. Лаба 2. Docker

## Задачи:

1. Написать *DockerFile* с «bad practices».
2. Исправить этот *DockerFile*.
3. Расписать почему практики плохие.
4. Описать плохие практике при работе с контейнерами.

## Пишем негодяйский DockerFile

```docker
# Плохая практика 1: Используем последний тег для базового образа
FROM python:latest
```
Хорошая ведь идея поддерживать актуальную версию всех зависимостей? Как бы не так, ведь завтра выйдет *Python 4.0*, где код уже будут писать маленькие котики с искусственным интелектом под копотом, а ваш собранный образ превтратится в тыкву.

```docker
# Плохая практика 2: Используем несколько FROM
FROM python:3.9
FROM node:16
```

Во имя доброты мы решили объединить Python и Node.js. Дружитесь! Но вот незадача при запуске контейнера у нас из него запускается другой контейнер, а из него третий. А каждый из них рассказывает анекдоты все хуже и хуже предыдущего.

В один момент мы понимаем, что Docker использует только последний FROM... 

И никакого Python у нас уже в образе нет.

```docker
# Плохая практика 8: Запускаем несколько сервисов (Python + Node.js)
CMD ["python", "/home/app/manage.py", "runserver", "0.0.0.0:8000"] && \
    ["node", "/home/app/server.js"]
```

В своих добрых порывах мы еще разок пытаемся подружить Python и Node.js. Теперь уже пытаясь запустить два сервера одновременно. 

В эту же секунду мы переносимся в детсво в чатик класса где Петя и Вася поругались и выясвняют отношения, каждую секунду вроде что то происходит, а разобраться в ситуации невозможно.

Логи перемешались, Node.js падал, питон перезапускался, а сервер уже стал фанатом тяжелого металла.

```docker
# Плохая практика 4: Используем внешние сервисы во время сборки
RUN python /home/app/manage.py migrate
```

Запускаем сборка и тут на!
Внезапно база данных начала выдавать ошибки и требовать отпуск. Серверы заявили, что устали от постоянных миграций и хотят осесть на одном месте.

Сборка наша становится нестабильной, внешние сервисы меняются а мы за ними не успеваем, становимся зависимыми от конкретного окружения.

Сборка Docker-образа должна быть *детерминированной* и *изолированной*

```docker
# Плохая практика 5: Сначала код, а потом зависимости
COPY . /app/
RUN pip install -r /app/requirements.txt
```
Теперь при каждой сборке у нас появляется лишнее время бахнуть лавандовый раф на безлактозном молоке.

А что случилось то?
Поменяли мы одну строчку в нашем, а Docker пересоберет все слои которые идут дальше.
А зачем нам пересобирать все зависимости? Думайте...

А мы исправим ошибку. Но кофе пить не перестанем!

## А теперь учтем все ошибки:

```docker
# Используем конкретную версию образа для предсказуемости
FROM python:3.9-slim

# Используем один базовый образ!
# Для Node.js создаем отдельный микросервис

# Установка зависимостей системы
RUN apt-get update && apt-get install -y \
    build-essential \
    libpq-dev \
    --no-install-recommends && rm -rf /var/lib/apt/lists/*

# Сначала копируем файл зависимостей, потом устанавливаем зависимости
COPY requirements.txt /app/requirements.txt

RUN pip install --no-cache-dir -r /app/requirements.txt

# Копируем код приложения после установки зависимостей
COPY . /app

WORKDIR /app

# Команды сборки не должны взаимодействовать с внешними сервисами.
# Миграции и запуск приложения должны выполняться на этапе запуска контейнера, а не сборки.

CMD ["python", "manage.py", "runserver", "0.0.0.0:8000"]
```

## Плохие практики использования контейнеров

### 1. Лапочки мои виртуальные машинки 

Пацаны не гоняйте! Докер контейнеры это не виртуальные машины, не нужно обновлять приложения внутри контейнеров, использовать ssh для подключения к ним (ну да чуть-чуть притянул, но зато показательно), извлекать логи напрямую или например развернуть все процессы в одном контейнере.

### 2. Держите права, только работайте,пожалуйста!

Не надо давать root контейнерам! Подумайте о маме. О безопасности. О римской империи.

Лучше выдавать минимально необходимые права, чтобы не создавать себе возможных дыр.

Спасибо за прочтение, дорогие читатели. Любите маму.