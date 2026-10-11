# HW7. Часові ряди та напівструктуровані дані в ML (NYC Taxi + JSONB)

Фінальне домашнє завдання курсу (Тема 7): time-series ознаки з NYC Taxi та JSONB event-log з GIN-індексами.

## Джерело даних
- NYC TLC Yellow Taxi Trip Records, партиція `yellow_tripdata_2024-03.parquet`: https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page
- Файл завантажується автоматично першою клітинкою (`urlretrieve`) у папку запуску; власний шлях можна задати змінною `TAXI_PARQUET`.
- Raw-timestamp TLC це локальний час Нью-Йорка: для TIMESTAMPTZ використано `AT TIME ZONE 'America/New_York'`, а `ts_day` (DATE) рахується за локальним нью-йоркським днем.
- Очищення: `fare_amount` у (0; 500), `trip_distance` у (0; 100).
- Береться випадкова вибірка до 500 000 рядків (`random_state=42`) з поїздок березня 2024, а не перші 500 000 рядків: перші рядки файлу покривають лише початок місяця, і lag-30d залишився б порожнім.
- Якщо завантажити файл не вдалося, notebook автоматично переходить на stub-schema (Варіант B) і виводить повідомлення про це.

## Що побудовано
- `t7_hw_taxi_daily`: щоденний агрегат за зоною; lag-1d/7d/30d, rolling 7d середній тариф, rolling 30d STDDEV, expanding сума/середнє; view `v_t7_hw_taxi_features`.
- `t7_hw_events`: 50 000 JSONB-подій, 5 типів; запити `@>`, `?`, `?|`, JSONPath.
- EXPLAIN (ANALYZE, BUFFERS): containment до та після GIN (`jsonb_path_ops`); `->> 'device'` до та після expression B-tree index.
- `t7_hw_checkouts_norm`: нормалізована таблиця і порівняння з JSONB; Reflection.

## Як запустити
1. PostgreSQL 16 (pgAdmin): створити порожню базу `hw7`.
2. `pip install jupyter psycopg2-binary sqlalchemy pandas pyarrow requests`
3. Відкрити `HW7_Hrysiuk.ipynb` і виконати Restart & Run All. Пароль Postgres запитується через `getpass` або береться зі змінної `HW7_DB_URL`. У коді паролів немає.
4. Усі об'єкти мають префікс `t7_hw_` / `idx_t7_`, notebook можна запускати повторно.
