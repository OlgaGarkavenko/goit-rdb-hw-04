# goit-rdb-hw-04 - Olist у PostgreSQL: typed-схема, DML, сегментація, JOIN-и

Домашнє завдання 4: typed-схема з PK/FK/CHECK, DML-фічі PostgreSQL (`RETURNING`, `ON CONFLICT`, `UPDATE ... FROM`, `DELETE ... RETURNING`), RFM-сегментація покупців, JOIN-запити (INNER / LEFT / FULL OUTER / SELF / anti-join), агрегати з `HAVING` і `STRING_AGG`, set-operations, window function і Reflection.

## Датасет

| | |
|---|---|
| Назва | **Brazilian E-Commerce Public Dataset by Olist** |
| Офіційне джерело | Kaggle: <https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce> |
| Використані файли | `olist_customers_dataset.csv`, `olist_orders_dataset.csv`, `olist_order_items_dataset.csv`, `olist_products_dataset.csv`, `olist_sellers_dataset.csv`, `olist_order_reviews_dataset.csv` |
| Ліцензія | CC BY-NC-SA 4.0 |

| Таблиця | Рядків |
|---|---|
| customers | 99 441 |
| orders | 99 441 |
| order_items | 112 650 |
| products | 32 951 |
| sellers | 3 095 |
| order_reviews | 100 000 |

## Звідки беруться дані (Варіант B)

CSV зафіксовано в репозиторії в `data/olist/` (без змін, усі вибірки повні). Датасет на Kaggle оновлювався, і різні версії відрізняються, зокрема кількістю відгуків. Фіксований знімок гарантує, що інтерпретації в notebook збігаються з output при кожному запуску.

Notebook шукає CSV у такому порядку:
1. `data/olist/` поруч із notebook;
2. якщо їх немає - клонує репозиторій з `REPO_URL` у `/content/goit-rdb-hw-04` і бере `data/olist/` звідти;
3. якщо й це не вдалося - завантажує датасет з Kaggle через `kagglehub` (Варіант A).

## Запуск у Google Colab

1. У клітинці **1.2** замініть `<ВАШ_ЛОГІН>` у `REPO_URL` на свій логін GitHub.
2. *Runtime -> Restart session and run all*.

Notebook сам встановлює залежності (`pgserver`, `psycopg2-binary`, `sqlalchemy`, `pandas`, `kagglehub`), піднімає PostgreSQL через `pgserver` у `/tmp/hw4_pg`, клонує репозиторій з даними й виконує всі кроки (≈ 1-2 хв).

> **Примітка щодо драйвера.** SQLAlchemy 2.1+ для `postgresql://` за замовчуванням використовує `psycopg` (v3). Тому notebook підставляє `postgresql+psycopg2://` в URI від `pgserver`, щоб працювати з `psycopg2-binary`.

## Структура

```
goit-rdb-hw-04/
├── hw4_olist.ipynb        # notebook з output усіх клітинок
├── data/olist/            # 6 CSV Olist
└── README.md
```

## Таблиці, що створює notebook

| Таблиця | Призначення |
|---|---|
| `olist_*_raw` | проміжний шар, віддзеркалення CSV |
| `olist_customers`, `olist_orders`, `olist_order_items`, `olist_products`, `olist_sellers`, `olist_order_reviews` | typed-схема з PK, FK, CHECK, індексами |
| `seller_alerts` | alert-и продавців (`INSERT ... RETURNING`, `DELETE ... RETURNING`) |
| `seller_score` | рейтинг і tier продавця (idempotent upsert `ON CONFLICT`) |
| `olist_customer_dim` | дедуплікований покупець (`customer_unique_id` як PK) |
| `customer_segments` | RFM-сегментація покупців (identity PK, FK, UNIQUE, CHECK, DEFAULT) |
