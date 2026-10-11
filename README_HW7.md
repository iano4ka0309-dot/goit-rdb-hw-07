# HW7. Часові ряди та напівструктуровані дані в ML (NYC Taxi + JSONB)

Фінальне домашнє завдання курсу (Тема 7). Pipeline: time-series ознаки з NYC Taxi та JSONB event-log з GIN-індексами.

## Джерело даних
- NYC TLC Yellow Taxi Trip Records, одна місячна партиція `yellow_tripdata_2024-03.parquet`.
- Сторінка: https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page
- Файл завантажується автоматично в першій клітинці (`urlretrieve`) у папку запуску. Можна вказати власний шлях через змінну `TAXI_PARQUET`.
- Raw-timestamp TLC це локальний час Нью-Йорка. Для production-типу TIMESTAMPTZ використано `AT TIME ZONE 'America/New_York'`, а день (`ts_day`, тип DATE) рахується за локальним нью-йоркським днем.
- Очищення: `fare_amount` у (0; 500), `trip_distance` у (0; 100).
- Для обмеження обсягу береться **випадкова вибірка** до 500 000 рядків (`random_state=42`) із поїздок березня 2024, а не перші 500 000 рядків. Перші рядки файлу покривають лише початок місяця, і lag-30d залишився б порожнім.
- Якщо завантажити файл не вдалося, notebook автоматично переходить на stub-schema (Варіант B) і виводить повідомлення `Використовується Варіант B (stub-schema)`. Усі наступні кроки працюють на однаковій схемі `t7_hw_taxi_daily`.
- Фінальний запуск використовував: Варіант A (реальні дані) / Варіант B (stub) <- залиши той, що відповідає твоєму запуску.

## Що побудовано
**Time-series (`pickup_zone_id × ts_day`)**
- `t7_hw_taxi_daily`: щоденний агрегат (поїздки, середній тариф, дистанція).
- Lag: `trips_lag_1d`, `trips_lag_7d`, `trips_lag_30d`.
- Rolling: середній тариф за 7 днів, STDDEV тарифу за 30 днів.
- Expanding: накопичена кількість поїздок і середнє.
- View `v_t7_hw_taxi_features` з усіма ознаками, демонстрація помилки без `PARTITION BY`.

**JSONB**
- `t7_hw_events`: 50 000 синтетичних подій, 5 типів з різною схемою payload.
- Запити: `@>`, `?`, `?|`, `jsonb_path_exists`, `jsonb_path_query`.
- `EXPLAIN (ANALYZE, BUFFERS)`: containment до та після `GIN (jsonb_path_ops)`; `->> 'device'` та expression B-tree index.
- `t7_hw_checkouts_norm`: нормалізована таблиця й порівняння з JSONB.
- MongoDB-еквівалент (pseudocode) і Reflection.

## Як запустити
1. PostgreSQL 16 (pgAdmin): створити порожню базу `hw7`.
2. Встановити залежності: `pip install jupyter psycopg2-binary sqlalchemy pandas pyarrow requests`.
3. Відкрити `HW7_Hrysiuk.ipynb`, **Restart & Run All**. Пароль до Postgres запитується через `getpass` або береться зі змінної середовища `HW7_DB_URL` (формат `postgresql+psycopg2://postgres:ПАРОЛЬ@localhost:5432/hw7`). У коді паролів немає.
4. Усі об'єкти мають префікс `t7_hw_` / `idx_t7_`; notebook можна запускати повторно без ручних виправлень.
5. Після запуску зберегти notebook **разом із виводами**.
