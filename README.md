# Мониторинг сервиса скоринга фрода (Prometheus + Grafana)

## Запуск
1. `cp .env.example .env`, в `.env` указать путь к `train.csv` в `TRAIN_DATA_CSV` (абсолютный).
2. `docker compose up --build`, подождать 1-2 минуты, пока загрузятся данные и пойдёт поток.

## Адреса
| Сервис | Адрес | Логин |
|---|---|---|
| Grafana | http://localhost:3000 | admin / admin |
| Prometheus | http://localhost:9090 | — |
| Airflow | http://localhost:8081 | admin / admin |
| Kafka UI | http://localhost:8080 | — |
| Метрики скоринга | http://localhost:8000/metrics | — |
| Метрики writer | http://localhost:8001/metrics | — |

## Метрики
Скоринг: `transactions_total` (счётчик), `transaction_processing_seconds` (summary),
`fraud_score` (гистограмма), `fraud_ratio` (gauge), `model_version_updates_total` (счётчик смен версии),
**<своя метрика: название, тип, зачем нужна, на каком графике смотреть>**.
Writer: `scorings_total`, `scoring_db_write_seconds`, `scoring_fraud_score`, `fraud_detected_total`.
Система: метрики Node Exporter (`node_*`).

## Дашборды (папка grafana/dashboards, подключаются при старте)
- **Скоринг**: скорость потока, время обработки, доля фрода, распределение скоров, отметка смены модели.
- **Результаты**: скорость и время записи, число фрода, распределение записанных скоров, таблица по версиям модели из PostgreSQL.
- **Система**: CPU, память, сеть, диск.

## Алерты (Grafana → Alerting → Alert rules)
- Доля фрода выше 5%.
- Скорость потока ниже 0,1 транзакции/с.
Проверка: `docker compose stop transaction_simulator`, через 1-2 минуты второй алерт станет Firing; `docker compose start transaction_simulator` возвращает Normal.

## Как проверить
1. В Prometheus (Status → Targets) все цели UP.
2. На дашборде «Скоринг» (диапазон Last 15 minutes, обновление 5s) графики растут вместе с потоком.
3. Новая версия модели: в Airflow запустить DAG `retrain_catboost`, на дашборде «Скоринг» появляется отметка с версией.
