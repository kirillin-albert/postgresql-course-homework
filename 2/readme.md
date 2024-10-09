1. открыл консоль и зашел по ssh на ВМ
2. открыл вторую консоль и также зашел по ssh на ту же ВМ
3. В двух консолях выполнил su - postgres psql 
4. сделать в первой сессии новую таблицу и наполнить ее данными
```
CREATE DATABASE isolation;
\c isolation
CREATE TABLE test (i serial, amount int);
INSERT INTO test(amount) VALUES (1000);
INSERT INTO test(amount) VALUES (2000);
SELECT * FROM test;
```
5. посмотреть текущий уровень изоляции:
```
isolation=# show transaction isolation level;
 transaction_isolation 
-----------------------
 read committed
(1 row)
```
6. начать новую транзакцию в обеих сессиях с дефолтным (не меняя) уровнем
   изоляции
```
BEGIN;
```
7. в первой сессии добавить новую запись
```commandline
isolation=*# INSERT INTO test(amount) VALUES (3000);
INSERT 0 1
```
8. сделать запрос на выбор всех записей во второй сессии
```commandline
isolation=*# SELECT * FROM test;
i | amount
---+--------
1 |   1000
2 |   2000
(2 rows)
```
9. видите ли вы новую запись и если да то почему? После задания можете сверить
   правильный ответ с эталонным (будет доступен после 3 лекции)

**Новая запись отсутствует**
10. завершить транзакцию в первом окне
```commandline
isolation=*# COMMIT;
COMMIT
```
11. сделать запрос на выбор всех записей второй сессии
```commandline
isolation=*# SELECT * FROM test;
 i | amount 
---+--------
 1 |   1000
 2 |   2000
 4 |   3000
(3 rows)
```
12. видите ли вы новую запись и если да то почему?

**Вижу, потому что завершил транзакцию.**

13. завершите транзакцию во второй сессии
```commandline
isolation=*# COMMIT;
COMMIT
```
14. начать новые транзакции, но уже на уровне repeatable read в ОБЕИХ сессиях
```commandline
isolation=# BEGIN TRANSACTION ISOLATION LEVEL REPEATABLE READ;
BEGIN
```
15. в первой сессии добавить новую запись
```commandline
isolation=*# INSERT INTO test(amount) VALUES (4000);
INSERT 0 1
```
16. сделать запрос на выбор всех записей во второй сессии
```commandline
isolation=*# SELECT * FROM test;
 i | amount 
---+--------
 1 |   1000
 2 |   2000
 4 |   3000
(3 rows)
```
17. видите ли вы новую запись и если да то почему?

**Не вижу новую запись**

18. завершить транзакцию в первом окне
```commandline
isolation=*# COMMIT;
COMMIT
```
19. сделать запрос во выбор всех записей второй сессии
```commandline
isolation=*# SELECT * FROM test;
 i | amount 
---+--------
 1 |   1000
 2 |   2000
 4 |   3000
(3 rows)
```
20. видите ли вы новую запись и если да то почему?

**Не вижу новую запись, так как транзакция не завершена во втором терминале, а изоляция repeatable read так и работает**
