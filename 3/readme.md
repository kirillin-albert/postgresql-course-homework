1. Создать таблицу с текстовым полем и заполнить случайными или сгенерированными данным в размере 1 млн строк
```commandline
BEGIN;
CREATE TABLE test(i TEXT);
INSERT INTO test (i)                                                
SELECT substr(md5(random()::text), 1, 5)
FROM generate_series(1, 1000000);
COMMIT;
```
2. Посмотреть размер файла с таблицей
```commandline
wal=# SELECT pg_size_pretty(pg_total_relation_size('test')) AS table_size;
 table_size 
------------
 35 MB
(1 row)
```
3. 5 раз обновить все строчки и добавить к каждой строчке любой символ
```commandline
DO $$
BEGIN
   FOR counter IN 1..5 LOOP
      UPDATE test SET i = i || 'W';
   END LOOP;
END $$;
```
4. Посмотреть количество мертвых строчек в таблице и когда последний раз приходил
   автовакуум
```commandline
wal=# SELECT relname, n_dead_tup 
FROM pg_stat_user_tables 
WHERE relname = 'test';
 relname | n_dead_tup 
---------+------------
 test    |    5000000
(1 row)

wal=# SELECT last_autovacuum 
FROM pg_stat_user_tables 
WHERE relname = 'test';
        last_autovacuum        
-------------------------------
 2024-10-14 18:40:10.951409+03
(1 row)
```
5. Подождать некоторое время, проверяя, пришел ли автовакуум

**Пришел =)**
```commandline
wal=# SELECT last_autovacuum 
FROM pg_stat_user_tables 
WHERE relname = 'test';
        last_autovacuum        
-------------------------------
 2024-10-14 18:42:15.550582+03
(1 row)

wal=# SELECT relname, n_dead_tup 
FROM pg_stat_user_tables 
WHERE relname = 'test';
 relname | n_dead_tup 
---------+------------
 test    |          0
(1 row)
wal=# SELECT pg_size_pretty(pg_total_relation_size('test')) AS table_size;
 table_size 
------------
 230 MB
(1 row)
```

6. 5 раз обновить все строчки и добавить к каждой строчке любой символ
```commandline
DO $$
BEGIN
   FOR counter IN 1..5 LOOP
      UPDATE test SET i = i || 'A';
   END LOOP;
END $$;
```
7. Посмотреть размер файла с таблицей
```commandline
wal=# SELECT pg_size_pretty(pg_total_relation_size('test')) AS table_size;
 table_size 
------------
 253 MB
(1 row)
```
8. Отключить Автовакуум на конкретной таблице
```commandline
ALTER TABLE test SET (autovacuum_enabled = false);
```
9. 10 раз обновить все строчки и добавить к каждой строчке любой символ
```commandline
DO $$
BEGIN
   FOR counter IN 1..10 LOOP
      UPDATE test SET i = i || 'L';
   END LOOP;
END $$;
```
10. Посмотреть размер файла с таблицей
```commandline
wal=# SELECT pg_size_pretty(pg_total_relation_size('test')) AS table_size;
 table_size 
------------
 767 MB
(1 row)
wal=# SELECT relname, n_dead_tup                                          
FROM pg_stat_user_tables 
WHERE relname = 'test';
 relname | n_dead_tup 
---------+------------
 test    |   14999890
(1 row)

```
11. Объясните полученный результат
- Мертвые строки: В PostgreSQL обновление строки создает новую версию этой строки, а старые версии становятся "мертвыми строками". Они занимают место, пока автовакуум не освободит их.
- Рост размера таблицы: После нескольких обновлений таблицы её размер увеличивается из-за накопления мертвых строк.
- Когда автовакуум выключен: Если автовакуум отключен, старые версии строк не удаляются, и размер таблицы может значительно увеличиваться после нескольких обновлений.
12. Не забудьте включить автовакуум)

**Автовакуум пришел, однако физической очистки с диска не происходит пока не сделать команду VACUUM FULL test;**

```commandline
ALTER TABLE test SET (autovacuum_enabled = true);
wal=# SELECT last_autovacuum 
FROM pg_stat_user_tables 
WHERE relname = 'test';
        last_autovacuum        
-------------------------------
 2024-10-14 18:42:15.550582+03
(1 row)

wal=# SELECT last_autovacuum 
FROM pg_stat_user_tables 
WHERE relname = 'test';
        last_autovacuum        
-------------------------------
 2024-10-14 18:45:39.030536+03
(1 row)

wal=# SELECT relname, n_dead_tup 
FROM pg_stat_user_tables 
WHERE relname = 'test';
 relname | n_dead_tup 
---------+------------
 test    |          0
(1 row)
wal=# SELECT pg_size_pretty(pg_total_relation_size('test')) AS table_size;
 table_size 
------------
 767 MB
(1 row)

wal=# VACUUM FULL test;
VACUUM
wal=# SELECT pg_size_pretty(pg_total_relation_size('test')) AS table_size;
 table_size 
------------
 57 MB
(1 row)

```

