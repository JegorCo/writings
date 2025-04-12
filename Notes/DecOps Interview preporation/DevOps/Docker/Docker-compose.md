### Как работает Docker Compose
Сначала в манифесте Docker Compose вы описываете набор взаимосвязанных ресурсов — контейнеров, сетей и томов. Затем используется набор CLI-команд. Вы можете создавать, удалять, запускать, останавливать ресурсы, описанные в манифесте. Более подробно опишем команды в следующем практическом уроке, а сейчас — расскажем про структуру манифеста Docker Compose.
### Манифест Docker Compose
YAML-манифест. Называется compose.yaml. 
Пример манифеста:

`name: simple-python-app`
`services:`
  `web:`
    `image: nginx:alpine`
    `ports:`
      `- "80:80"`
    `networks:`
      `- frontend-net`
    `volumes:`
      `- type: bind`
        `source: ./nginx/app.conf`
        `target: /etc/nginx/conf.d/default.conf`
        `read_only: true`
    `depends_on:`
      `- simple_python_app`

  `simple_python_app:`
    `image: simple_python_app:${APP_VERSION:-latest}`
    `build: ./simple_python_app`
    `environment:`
      `API_DB_HOST: db`
      `API_DB_PASS: apipass`
      `API_DB_NAME: api`
      `API_DB_USER: apiuser`
    `networks:`
      `- frontend-net`
      `- backend-net`
    `depends_on:`
      `db:`
        `condition: service_healthy`
        `restart: true`

  `db:`
    `image: postgres:16.2-alpine`
    `environment:`
      `POSTGRES_PASSWORD: apipass`
      `POSTGRES_DB: api`
      `POSTGRES_USER: apiuser`
    `volumes:`
      `- dbdata:/var/lib/postgresql/data`
    `networks:`
      `- backend-net`
    `healthcheck:`
      `test: ["CMD-SHELL", "pg_isready"]`
      `interval: 10s`
      `timeout: 5s`
      `retries: 10`
      `start_period: 30s`
      `start_interval: 1s`

`networks:`
  `frontend-net:`
  `backend-net:`

`volumes:`
  `dbdata:`

Основные компоненты манифеста:
`services` 
В блоке `services` описываются сами сервисы, запускаемые из образов контейнеров и их конфигурации: образ контейнера, передаваемые параметры запуска, переменные окружения, подключённые сети и точки монтирования. В качестве сервиса может выступать как контейнер на базе образа с вашим приложением, так и контейнер с веб-сервером, СУБД, брокером сообщений и подобными вспомогательными сервисами, необходимыми для работы приложения.
