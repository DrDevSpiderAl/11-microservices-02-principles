
# Домашнее задание к занятию «Микросервисы: принципы»

Вы работаете в крупной компании, которая строит систему на основе микросервисной архитектуры.
Вам как DevOps-специалисту необходимо выдвинуть предложение по организации инфраструктуры для разработки и эксплуатации.

## Задача 1: API Gateway 

Предложите решение для обеспечения реализации API Gateway. Составьте сравнительную таблицу возможностей различных программных решений. На основе таблицы сделайте выбор решения.

### Сравнительная таблица возможностей API Gateway

| Критерий / Решение | Apache APISIX | Traefik | NGINX | Yandex API Gateway | Amazon API Gateway |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Тип развертывания** | Self-Hosted / On-Premise / K8s | Self-Hosted / On-Premise / K8s | Self-Hosted / On-Premise / K8s | Cloud Managed (Yandex Cloud) | Cloud Managed (AWS) |
| **Архитектура** | OpenResty (NGINX + LuaJIT) + etcd | Go (Go-native) | C (Event-driven, async) | Serverless / Cloud Native | Serverless / Cloud Native |
| **Маршрутизация** | Динамическая (etcd, API, без перезагрузки) | Автообнаруживаемая (K8s Ingress, Docker labels) | Статическая (`nginx.conf`) или API (Plus/Lua) | Декларативная (спецификация OpenAPI 3.0) | Декларативная (OpenAPI, AWS Console, IaC) |
| **Аутентификация** | Встроенные плагины (JWT, Key Auth, OIDC, OAuth2) | Middleware (Basic, ForwardAuth, JWT) | Базовая (Free) / JWT и OAuth2 (Plus / Lua) | Yandex IAM, JWT, Custom Authorizers (YCF) | AWS IAM, Cognito, Custom Authorizers (Lambda) |
| **Терминация HTTPS** | Нативная (SNI, динамические сертификаты) | Нативная (авто-выпуск Let's Encrypt / ACME) | Нативная (SSL/TLS termination, SNI) | Автоматическая (Certificate Manager) | Автоматическая (AWS Certificate Manager) |
| **Производительность** | Экстремально высокая, минимальный overhead | Высокая (минимальная задержка на Go) | Максимально высокая (эталонная скорость) | Зависит от инфраструктуры YC / FaaS | Зависит от типа API (REST vs HTTP API) |
| **Масштабируемость** | Горизонтальная (ноды APISIX) | Горизонтальная (pods / nodes) | Горизонтальная (Stateful / Stateless) | Автоматическая (Serverless) | Автоматическая (Serverless) |
| **Модель оплаты** | Free (Open Source) + инфраструктура | Free (Open Source) / EE по запросу | Free (Community) / Лицензия (Plus) | Pay-as-you-go (за миллионы запросов) | Pay-as-you-go (за запросы + трафик) |

Выбор будет зависеть от политики компании и текущих решений. 

On-Premise / Hybrid / Kubernetes: NGINX — из-за проверенной временем надежности, высокой скорости обработки трафика, низкого потребления ресурсов и наличия готовой экспертизы у команды, что критично для поддержания собственного контура.

Yandex Cloud: Yandex API Gateway — из-за нативной интеграции с облачными сервисами (IAM, Certificate Manager, Cloud Functions), использования стандарта OpenAPI 3.0, автоматического масштабирования и отсутствия затрат на администрирование шлюзовой инфраструктуры.

Обоснуйте свой выбор.

## Задача 2: Брокер сообщений

Составьте таблицу возможностей различных брокеров сообщений. На основе таблицы сделайте обоснованный выбор решения.

Решение должно соответствовать следующим требованиям:
- поддержка кластеризации для обеспечения надёжности,
- хранение сообщений на диске в процессе доставки,
- высокая скорость работы,
- поддержка различных форматов сообщений,
- разделение прав доступа к различным потокам сообщений,
- простота эксплуатации.

Обоснуйте свой выбор.

## Задача 3: API Gateway * (необязательная)

### Есть три сервиса:

**minio**
- хранит загруженные файлы в бакете images,
- S3 протокол,

**uploader**
- принимает файл, если картинка сжимает и загружает его в minio,
- POST /v1/upload,

**security**
- регистрация пользователя POST /v1/user,
- получение информации о пользователе GET /v1/user,
- логин пользователя POST /v1/token,
- проверка токена GET /v1/token/validation.

### Необходимо воспользоваться любым балансировщиком и сделать API Gateway:

**POST /v1/register**
1. Анонимный доступ.
2. Запрос направляется в сервис security POST /v1/user.

**POST /v1/token**
1. Анонимный доступ.
2. Запрос направляется в сервис security POST /v1/token.

**GET /v1/user**
1. Проверка токена. Токен ожидается в заголовке Authorization. Токен проверяется через вызов сервиса security GET /v1/token/validation/.
2. Запрос направляется в сервис security GET /v1/user.

**POST /v1/upload**
1. Проверка токена. Токен ожидается в заголовке Authorization. Токен проверяется через вызов сервиса security GET /v1/token/validation/.
2. Запрос направляется в сервис uploader POST /v1/upload.

**GET /v1/user/{image}**
1. Проверка токена. Токен ожидается в заголовке Authorization. Токен проверяется через вызов сервиса security GET /v1/token/validation/.
2. Запрос направляется в сервис minio GET /images/{image}.

### Ожидаемый результат

Результатом выполнения задачи должен быть docker compose файл, запустив который можно локально выполнить следующие команды с успешным результатом.
Предполагается, что для реализации API Gateway будет написан конфиг для NGinx или другого балансировщика нагрузки, который будет запущен как сервис через docker-compose и будет обеспечивать балансировку и проверку аутентификации входящих запросов.
Авторизация
curl -X POST -H 'Content-Type: application/json' -d '{"login":"bob", "password":"qwe123"}' http://localhost/token

**Загрузка файла**

curl -X POST -H 'Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJib2IifQ.hiMVLmssoTsy1MqbmIoviDeFPvo-nCd92d4UFiN2O2I' -H 'Content-Type: octet/stream' --data-binary @yourfilename.jpg http://localhost/upload

**Получение файла**
curl -X GET http://localhost/images/4e6df220-295e-4231-82bc-45e4b1484430.jpg

---

#### [Дополнительные материалы: как запускать, как тестировать, как проверить](https://github.com/netology-code/devkub-homeworks/tree/main/11-microservices-02-principles)

---

### Как оформить ДЗ?

Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.

---