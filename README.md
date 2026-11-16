# Резервное копирование баз данных
# Антон Сап

## Задание 1. 

**1.1. Восстановление данных за предыдущий день.**  
Подойдёт ежедневное полное резервное копирование (Full Backup). Для восстановления используется последняя копия, созданная за предыдущий день.

**1.2. Восстановление до момента неисправности.**  
Подойдёт PITR (Point-in-Time Recovery) — восстановление на выбранный момент времени. Для этого нужна полная физическая резервная копия и сохранённые журналы изменений (WAL в PostgreSQL или Binary Log в MySQL). Это позволяет восстановить данные непосредственно перед ошибкой, если необходимые журналы сохранились.

## Задание 2. 

**2.1. Пример создания резервной копии базы `company_db`:**

```bash
pg_dump -U postgres -Fc -f company_db.backup company_db
```

`pg_dump` создаёт логический бэкап в формате Custom.

**Пример восстановления в отдельную базу:**

```bash
createdb -U postgres -T template0 company_db_test
pg_restore -U postgres -d company_db_test company_db.backup
```

Сначала создаётся пустая база `company_db_test`, затем в неё восстанавливаются данные из архива.

Документация: [pg_dump](https://www.postgresql.org/docs/17/app-pgdump.html), [pg_restore](https://www.postgresql.org/docs/17/app-pgrestore.html).

## Задание 3. MySQL

**3.1. Пример инкрементального резервного копирования через Binary Log.**

Сначала создаётся полный бэкап:

```bash
mysqldump -u root -p --single-transaction --source-data=2 --flush-logs --all-databases > full_backup.sql
```

Затем можно сохранять новые события Binary Log:

```bash
mysqlbinlog --read-from-remote-server --host=localhost --user=backup_user -p --raw --stop-never binlog.000001
```

`binlog.000001` — пример имени журнала; на реальном сервере нужно указать существующий файл. Для команды нужны включённый Binary Log и соответствующие права пользователя.

При восстановлении сначала загружается полный бэкап, затем применяются сохранённые события Binary Log после точки создания бэкапа. Резервные копии необходимо хранить отдельно от основного сервера.

Документация: [MySQL — резервное копирование Binary Log](https://dev.mysql.com/doc/refman/8.4/en/mysqlbinlog-backup.html).

> Приведены примеры команд по документации. 
