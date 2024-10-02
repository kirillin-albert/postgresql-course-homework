1. Создана виртуальная машина Debian 11
2. Установлен PostgreSQL 16 по документации https://www.postgresql.org/download/linux/debian/
3. Выполнены команды
```
su - postgres
wget https://storage.googleapis.com/thaibus/thai_small.tar.gz && tar -xf thai_small.tar.gz && psql < thai.sql
psql
\c thai
select count(*) from book.tickets;
```
4. Ответ **5185505**
