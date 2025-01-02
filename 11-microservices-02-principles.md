
# Домашнее задание к занятию «Микросервисы: принципы»

Вы работаете в крупной компании, которая строит систему на основе микросервисной архитектуры.
Вам как DevOps-специалисту необходимо выдвинуть предложение по организации инфраструктуры для разработки и эксплуатации.

## Задача 1: API Gateway 

Предложите решение для обеспечения реализации API Gateway. Составьте сравнительную таблицу возможностей различных программных решений. На основе таблицы сделайте выбор решения.

Решение должно соответствовать следующим требованиям:
- маршрутизация запросов к нужному сервису на основе конфигурации,
- возможность проверки аутентификационной информации в запросах,
- обеспечение терминации HTTPS.

Обоснуйте свой выбор.

### Решение
 API Gateway, походящие под условия и чаще всего упоминаемые во всевозможных топах: Kong Gateway, Apache APISIX, Tyk, KrakenD, Amazon API Gateway.


| Критерий | Kong Gateway | Apache APISIX | Tyk | KrakenD | Amazon API Gateway |
|----------|--------------|---------------|-----|---------|-----------|
| Цена | Есть Open Source и платный Enterprise | Open Source (есть платный облачный API7) | Есть Open Source и платные тарифы в облаке | Есть Community Edition и Enterprise Edition | Есть free tier с ограничением по количеству запросов. |
| Размещение | SaaS (Kong Konnect) и размещение на хосте, в том числе с использованием контейнеров. | Размещение на хосте, в том числе с использованием контейнеров. Есть SaaS API7. | Размещение на хосте, в том числе с использованием контейнеров. Есть SaaS. | Размещение на хосте, в том числе с использованием контейнеров, в облаке. | Облако |
| Производительсность | Хвалятся производительностью | Хвалятся, что они производительнее Kong | Хвалятся, что они производительнее Kong | Быстрый и нетребовательный | Хвалится производительностью |
| Расширения | Kong Plugin Hub | Apache APISIX®️ Plugin Hub | Можно создавать свои плагины, можно найти готовые на GitHub | Можно создавать свои плагины | Есть расширения |
| Аутентификация | Basic, Key, OAuth 2.0, LDAP, OpenID Connect, HMAC, JWT | Key, HMAC, OpenID Connect, LDAP , Basic, Wolf RBAC | Basic, Bearer Token, OAuth 2.0, HMAC, JWT, OpenID Connect | Basic, JWT, OAuth 2.0, mTLS, Google Cloud, NTLM | OAuth 2.0, Basic, JWT  |
| Управление трафиком | Timeout, Retry, Circuit Breaker, Rate Limit | Rate Limit, Circuit Breaker, Retry, Timeout | Rate Limit, Circuit Breaker, Timeout, Retry | Rate Limit, Circuit Breaker, Timeout, Retry, Bot detector | Timeout, Retry, Circuit Breaker, Rate Limit |
| Мониторинг | Built-in, Prometheus, Datadog, StatsD | Prometheus | Built-in | OpenTelemetry, Prometheus, InfluxDB, Datadog, AWS X-Ray, Azure Monitor и др. | Amazon CloudWatch Alarms, Amazon CloudWatch Logs, Amazon EventBridge, AWS CloudTrail Log Monitoring |



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

### Решение

Рассмотрим основные брокеры сообщений, которые обычно фигурируют в списках: RabbitMQ, Kafka, ActiveMQ, Redis, Amazon SNS/SQS, Apache Pulsar, NATS. 

Критериев много, так что поэтапно отсеем выбранные решения.

* Поддержка кластеризации. <br/>
RabbitMQ, Kafka, ActiveMQ, Redis, Apache Pulsar, NATS.<br/>
Amazon SNS/SQS мало документации.
* Хранение сообщений на диске в процессе доставки.<br/>
RabbitMQ (с 3.10.0), Kafka, ActiveMQ, Redis (при fsync every write), Apache Pulsar (при соответствующей настройке), NATS (при использовании с JetStream).
* Высокая скорость работы.<br/>
Высокую скорость отмечают у NATS, Kafka. Apache Pulsar, RabbitMQ и ActiveMQ при сравнении оказываются более медленными, чем конкуренты.
* Поддержка разных форматов сообщений.<br/>
По условию подходят RabbitMQ (AMQP 0-9-1, AMQP 1.0, RabbitMQ Stream Protocol, MQTT, STOMP), ActiveMQ (AMQP, AUTO, MQTT, OpenWire, REST, RSS and Atom, Stomp, WSIF, WS Notification, XMPP), 
Здесь мы отсекаем Kafka (использует Kafka Protocol), Redis (использует RESP), Apache Pulsar (с одноимённым протоколом), NATS (NATS Protocol).
* Разделение прав доступа к различным потокам сообщений.<br/>
У RabbitMQ есть права пользователей, у ActiveMQ есть роли.
* Простота эксплуатации.<br/>
RabbitMQ или ActiveMQ примерно одинаковые по сложности системы.

Мой выбор  RabbitMQ и ActiveMQ. 


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
