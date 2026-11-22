# Отчет о проверках

## Проверки на 4 балла

Собрала образ и создала контейнер с помощью команд:
```bash 
docker build -t counter-app:1 .
docker run -d --name counter -p 127.0.0.1:8080:8000 -e GREETING='Hello from a container!' counter-app:1
```

Два раза отправила POST-запрос с помощью команды 
```bash
curl -fsS -X POST http://127.0.0.1:8080/increment
```

Cчетчик изменился на 2. 

Затем получила логи
```
2026-11-22 10:48:04,910 listening port=8000 storage=sqlite
2026-11-22 10:48:45,063 http 172.17.0.1 "GET / HTTP/1.1" 200 -
2026-11-22 10:51:01,877 http 172.17.0.1 "POST /increment HTTP/1.1" 200 -
2026-11-22 10:51:41,074 http 172.17.0.1 "POST /increment HTTP/1.1" 200 -
```
с помощью команды 
```bash
docker logs counter
```

## Проверки на 6 баллов

Запустила программу с помощью Docker Compose, изменила счетчик 4 раза (выполнила вторую команду 4 раза): 
```bash
docker compose up -d --build
curl -fsS -X POST http://127.0.0.1:8080/increment
```

Затем удалила контейнер и создала его заново с помощью команд
```bash
docker compose down
docker compose up -d
```
После еще одного GET-запроса счетчик остался равным 4, то есть остался прежним при изменении контейнера. 

## Проверки на 8 баллов

В `compose.yaml` добавила `redis`, а для `app` указала `STORAGE=redis` и `REDIS_URL=redis://redis:6379/0`. Для `redis` подключила `volume notes_redis`, включила сохранение данных на диск и настроила `healthcheck` обоих сервисов.
После запуска выполнила четыре POST-запроса, счётчик стал равен 4.

После удаления и повторного создания обоих контейнеров GET-запрос вернул count = 4. Данные сохранились.
```bash
docker compose down
docker compose up -d
curl -fsS http://127.0.0.1:8080/
```
Затем приостановила работу `redis`, проверила запросы к приложению. `/healthz` вернул 200, а `/readyz` и `/increment` - 503, потому что `redis` не работал.
```bash
docker compose stop redis
curl -i http://127.0.0.1:8080/healthz
curl -i http://127.0.0.1:8080/readyz
curl -i -X POST http://127.0.0.1:8080/increment
```

Затем снова запустила `redis`:
```bash
docker compose start redis
docker compose ps
curl -fsS http://127.0.0.1:8080/
curl -fsS -X POST http://127.0.0.1:8080/increment
```

После запуска оба сервиса имели статус `healthy`. Счётчик остался равным 4, а после POST-запроса увеличился до 5. Приложение вручную не перезапускала.