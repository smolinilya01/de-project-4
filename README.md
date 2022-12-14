# Проект настройки DWH.


## Требования 
Основная задача — реализация ETL-процессов, которые будут преобразовывать и перемещать данные от источников до конечных слоёв данных DWH.
Нужно также подтвердить, что написанный код работает и удовлетворяет требованиям: подготовить тест-кейсы, написать код для тестирования, провести само тестирование.
Для передачи результатов заказчику оформите всю необходимую документацию.

## Целевая витрина cdm.dm_courier_ledger
Данные за весь период, обновление на усмотрение разработчика.

1. id — идентификатор записи.
2. courier_id — ID курьера, которому перечисляем.
3. courier_name — Ф. И. О. курьера.
4. settlement_year — год отчёта.
5. settlement_month — месяц отчёта, где 1 — январь и 12 — декабрь.
6. orders_count — количество заказов за период (месяц).
7. orders_total_sum — общая стоимость заказов.
8. rate_avg — средний рейтинг курьера по оценкам пользователей.
9. order_processing_fee — сумма, удержанная компанией за обработку заказов, которая высчитывается как orders_total_sum * 0.25.
10. courier_order_sum — сумма, которую необходимо перечислить курьеру за доставленные им/ей заказы. За каждый доставленный заказ курьер должен получить некоторую сумму в зависимости от рейтинга (см. ниже).
11. courier_tips_sum — сумма, которую пользователи оставили курьеру в качестве чаевых.
12. courier_reward_sum — сумма, которую необходимо перечислить курьеру. Вычисляется как courier_order_sum + courier_tips_sum * 0.95 (5% — комиссия за обработку платежа).

## Запуск 
1. Запускам Apache Airflow
2. Создаем connection 'postgresql_de' к базе данных PostgreSQL
3. Проводим миграции БД с помощью DDL SQL из \src\db_migrations в любом порядке файлов
4. Переносим python dags из \src\dags в директорию с дагами airflow
5. Запускаем задачи с тэгами de-project-4, dwh, stg
6. Проверяем в Airflow, что задачи запустились и отработали успешно
7. Запускаем задачи с тэгами de-project-4, dwh, dds
8. Проверяем в Airflow, что задачи запустились и отработали успешно
9. Запускаем задачи с тэгами de-project-4, dwh, cdm, test
10. Проверяем в Airflow, что задачи запустились и отработали успешно и тесты прошли успешно (в таблице public_test.testing_result)

## Схема взаимодействия
API -> STG layer -> DDS layer -> CDM layer

## Замечания
API не выдает временную метку об обновлении записи (например, как в монго update_ts), получается, нужно раз за разом выгружать все данные из api, так как нет гарантии что прошлые записи не поменялись.
Данные из API deliveries загружаются с заклядванием назад на 3 дня (для того что бы отследить изменения заказов, если они были).
