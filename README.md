## Шаблон для третьего ДЗ по ИПР

Условие дз доступно по [ссылке](https://github.com/Nikfilk2030/missing_semester_hse_cs/blob/main/2026/amcs/year_1/hw_3/README.md)
## Запуск

Для запуска необходимы Docker и Docker Compose.

```bash
docker compose up -d --build
```

Приложение доступно по адресу http://127.0.0.1:8080/.

## Настройка

Переменные окружения:

- `HOST_PORT` — порт приложения на компьютере, по умолчанию `8080`.
- `GREETING` — приветствие, по умолчанию `Hello`.

Пример настроек находится в `.env.example`. Чтобы изменить значения, можно создать файл `.env`.

Приложение использует Redis для хранения счётчика. Данные сохраняются в именованном томе `redis-data`.


4 балла:
сборка docker-образа:
docker build -t ipr-hw-3:1 .

Вывод: [+] Building 11.1s (11/11) FINISHED ...


запустил контейнер с пробросом порта и переменной окружения GREETING="Hello":

```bash
docker run -d --name ipr-container \
  -p 127.0.0.1:8080:8000 \
  -e GREETING=Hello \
  ipr-hw-3:1
```


проверил работу:
```bash
curl http://127.0.0.1:8080/
curl -X POST http://127.0.0.1:8080/increment
```
Вывод:

```bash
{"greeting": "Hello", "count": 0, "hostname": "52991442eca6", "storage": "sqlite"}%
{"greeting": "Hello", "count": 1, "hostname": "52991442eca6", "storage": "sqlite"}%
```

сохранил логи:
docker logs ipr-container > docker-logs.txt 2>&1

docker-logs.txt:

```bash
2026-11-23 20:54:12,617 listening port=8000 storage=sqlite
2026-11-23 20:56:08,076 http 172.17.0.1 "GET / HTTP/1.1" 200 -
2026-11-23 20:59:04,687 http 172.17.0.1 "POST /increment HTTP/1.1" 200 -
2026-11-23 20:59:44,220 http 172.17.0.1 "GET /readyz HTTP/1.1" 200 -
```


на 6 баллов:

сохранил count
запустил контейнер:
```bash
docker compose up -d --build
curl -fsS http://127.0.0.1:8080/
{"greeting": "Hello", "count": 0, "hostname": "27525fe71706", "storage": "sqlite"}%
curl -fsS -X POST http://127.0.0.1:8080/increment
{"greeting": "Hello", "count": 1, "hostname": "27525fe71706", "storage": "sqlite"}%
docker compose down
[+] down 2/2
 ✔ Container hw_3_template-app-1 Removed                                                                       3.1s
 ✔ Network hw_3_template_default Removed
docker compose up -d
[+] up 2/2
 ✔ Network hw_3_template_default Created                                                                       0.0s
 ✔ Container hw_3_template-app-1 Started
curl -fsS http://127.0.0.1:8080/
{"greeting": "Hello", "count": 1, "hostname": "9a64756432e4", "storage": "sqlite"}%
```
проверил, что счетчик остался прежним


на 8 баллов:
проверил работу app и redis
проверил счетчик

```bash
docker compose ps
NAME                    IMAGE               COMMAND                  SERVICE   CREATED          STATUS       PORTS
hw_3_template-app-1     hw_3_template-app   "python -u app.py"       app       35 seconds ago   Up 31 seconds (healthy)   127.0.0.1:8080->8000/tcp
hw_3_template-redis-1   redis:7.4-alpine    "docker-entrypoint.s…"   redis     35 seconds ago   Up 31 seconds (healthy)   6379/tcp
curl -fsS -X POST http://127.0.0.1:8080/increment
{"greeting": "Hello", "count": 1, "hostname": "28daeed7c621", "storage": "redis"}%
curl -fsS -X POST http://127.0.0.1:8080/increment
{"greeting": "Hello", "count": 2, "hostname": "28daeed7c621", "storage": "redis"}%
```
сохранение счетчика в redis:

```bash
docker compose down
[+] down 3/3
 ✔ Container hw_3_template-app-1   Removed                                                                     3.1s
 ✔ Container hw_3_template-redis-1 Removed                                                                     0.1s
 ✔ Network hw_3_template_default   Removed
docker compose up -d
[+] up 3/3
 ✔ Network hw_3_template_default   Created                                                                     0.0s
 ✔ Container hw_3_template-redis-1 Started                                                                     0.1s
 ✔ Container hw_3_template-app-1   Started
curl -fsS http://127.0.0.1:8080/
{"greeting": "Hello", "count": 2, "hostname": "651a5b5722c1", "storage": "redis"}%
```

```bash
docker compose stop redis
curl -i http://127.0.0.1:8080/healthz
HTTP/1.0 200 OK
Server: BaseHTTP/0.6 Python/3.13.18
Date: Mon, 23 Nov 2026 22:05:21 GMT
Content-Type: application/json; charset=utf-8
Content-Length: 19
Cache-Control: no-store

{"status": "alive"}%
oblepihor@MacBook-Air-Petr hw_3_template % curl -i http://127.0.0.1:8080/readyz 
HTTP/1.0 503 Service Unavailable
Server: BaseHTTP/0.6 Python/3.13.18
Date: Mon, 23 Nov 2026 22:05:44 GMT
Content-Type: application/json; charset=utf-8
Content-Length: 32
Cache-Control: no-store

{"error": "storage_unavailable"}%
curl -i -X POST http://127.0.0.1:8080/increment
HTTP/1.0 503 Service Unavailable
Server: BaseHTTP/0.6 Python/3.13.18
Date: Mon, 23 Nov 2026 22:06:22 GMT
Content-Type: application/json; charset=utf-8
Content-Length: 32
Cache-Control: no-store

{"error": "storage_unavailable"}%
docker compose start redis
[+] start 1/1
 ✔ Container hw_3_template-redis-1 Started
docker compose ps
NAME                    IMAGE               COMMAND                  SERVICE   CREATED         STATUS      PORTS
hw_3_template-app-1     hw_3_template-app   "python -u app.py"       app       3 minutes ago   Up 3 minutes (healthy)    127.0.0.1:8080->8000/tcp
hw_3_template-redis-1   redis:7.4-alpine    "docker-entrypoint.s…"   redis     3 minutes ago   Up 31 seconds (healthy)   6379/tcp
curl -fsS http://127.0.0.1:8080/
{"greeting": "Hello", "count": 2, "hostname": "651a5b5722c1", "storage": "redis"}%
curl -fsS -X POST http://127.0.0.1:8080/increment
{"greeting": "Hello", "count": 3, "hostname": "651a5b5722c1", "storage": "redis"}%
```

Остановил redis, проверил ответы без него.
