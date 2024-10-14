1. Создать таблицу accounts(id integer, amount numeric);
```commandline
CREATE TABLE accounts (
    id INTEGER PRIMARY KEY,
    amount NUMERIC
);
```
2. Добавить несколько записей и подключившись через 2 терминала добиться ситуации взаимоблокировки (deadlock).

**Делаем две записи**

```commandline
wal=# INSERT INTO accounts (id, amount) VALUES (1, 1000), (2, 1500);
INSERT 0 2
```

Открываем сессию на первом терминале и начинаем транзакцию:
```
BEGIN;
UPDATE accounts SET amount = amount - 100 WHERE id = 1;
```
Эта строка будет заблокирована, пока транзакция не завершится.

Открываем сессию на втором терминале и начинаем транзакцию:
```
BEGIN;
UPDATE accounts SET amount = amount + 100 WHERE id = 2;
```

Эта строка также будет заблокирована в этой транзакции.

Теперь создадим deadlock:

Вернёмся на Терминал 1 и попытаемся обновить запись с id=2:
```
UPDATE accounts SET amount = amount + 100 WHERE id = 2;
```
Эта операция будет заблокирована, так как запись с id=2 уже заблокирована в транзакции на Терминале 2.

Вернёмся на Терминал 2 и попытаемся обновить запись с id=1:
```
UPDATE accounts SET amount = amount - 100 WHERE id = 1;
ERROR:  deadlock detected
DETAIL:  Process 158406 waits for ShareLock on transaction 960; blocked by process 158407.
Process 158407 waits for ShareLock on transaction 961; blocked by process 158406.
HINT:  See server log for query details.
CONTEXT:  while updating tuple (0,1) in relation "accounts"
```

3. Посмотреть логи и убедиться, что информация о дедлоке туда попала.
```commandline
root@test1:~# cat /var/log/postgresql/postgresql-16-main.log | grep -A 5 deadlock 
2024-10-14 19:01:09.212 MSK [158406] postgres@wal ERROR:  deadlock detected
2024-10-14 19:01:09.212 MSK [158406] postgres@wal DETAIL:  Process 158406 waits for ShareLock on transaction 960; blocked by process 158407.
	Process 158407 waits for ShareLock on transaction 961; blocked by process 158406.
	Process 158406: UPDATE accounts SET amount = amount - 100 WHERE id = 1;
	Process 158407: UPDATE accounts SET amount = amount + 100 WHERE id = 2;
2024-10-14 19:01:09.212 MSK [158406] postgres@wal HINT:  See server log for query details.

```
