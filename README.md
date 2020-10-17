# docker-2

## Практика

Нам необходимо создать `docker-compose` спецификацию
для локальной разработки и "тестового" продакшена.

Из чего будут состоять наша спецификация?
1. Приложение https://hub.docker.com/r/hashicorp/http-echo которое всегда должно возвращать `Hello world!`
2. Приложение https://hub.docker.com/r/williamyeh/json-server/ будет возвращать `json` данные
3. Прокси сервер перед ними: https://hub.docker.com/_/caddy

### http-echo

Данное приложение всегда возвращает текст `Hello world!`.
Оно должно быть доступно по адресу `/echo`

### json-server

Документация: https://github.com/typicode/json-server
Данное приложение возвращает `json` данные из файла:

```json
{
  "posts": [
    { "id": 1, "title": "json-server", "author": "typicode" },
    { "id": 2, "title": "second-post", "author": "typicode" }
  ],
  "comments": [
    { "id": 1, "body": "some comment", "postId": 1 }
  ]
}
```

Данный файл мы должны поместить по пути `/data/db.json`.
Вся папка `/data` должна быть примонтирована в качестве `volume` внутри контейнера.

Данное приложение:
- `/posts` выведет список всех постов
- `/posts/1` выведет первый `post`
- `/comments` - аналогично

### Среды

У нас будет две среды:
- `docker-compose.yml` и `docker-compose.override.yml` для определения среды разработки (по-умолчанию)
- `docker-compose.prod.yml` для переопределения для прода

В среде для разработки все работает так:
- Мы опубликовываем порты `:5000` для `echo-server` и `:8000` для `json-server` на локальную машину
- Мы сможем обращаться к `localhost:5000` для просмотра данных из `echo-server` и к `localhost:8000/posts` для просмотра данных из `json-server`

В среде production все работает так:
- сервисы `echo-server` и `json-server` нигде не должны быть опубликованы на локальной машине. а должны быть доступны только в своей изолированной сети(контейнерной)
- Сервис Caddy будет использоваться только в `docker-compose.prod.yml`, который будет слушать 80 порт на локальной машине.
Нам потребуется сделать роутинг в сервисе caddy:

- Запрос `/echo` будет уходить в контейнер `echo-server`
- Любые другие запросы будут уходить в `json-server`

### Результаты

В результате у вас должно получиться как минимум три файла:
- `docker-compose.yml`
- `docker-compose.override.yml`
- `docker-compose.prod.yml`

Проверьте, что вся конфигурация валидна.

Вам могут потребоваться свои `Dockerfile`. Но можно решить без них.

Формат сдачи:

- Создайте новый репозиторий docker-2 в персональном пространстве в scm.x5.ru. При создании поставьте галку "Initialize repository with a README" (чтобы создалась ветка master с файлом README.md).
- Создайте ветку "homework". В неее загрузите полученые результаты
- В файле README.md (в ветке homework) опишите последовательность команд, которые Вы запускали во время выполнения задания и напишите как запустить 2 проекта для development и для production окружения.
- Создайте Merge Request из ветки homework в ветку master. Укажите в "Labels"для Merge Request-а метку "Docker-2". Указывать Assignee НЕ нужно.

## Запуск

В dev:
	docker-compose up

В prod:
	docker-compose -f docker-compose.yml -f docker-compose.prod.yml up

## history
```
brew install docker-compose
docker-compose up
touch data/db.json
touch Caddyfile
docker-compose up
cp docker-compose.yml docker-compose.override.yml
cp docker-compose.yml docker-compose.prod.yml
docker-compose config
docker-compose -f docker-compose.yml -f docker-compose.prod.yml config
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up
docker-compose up
```
