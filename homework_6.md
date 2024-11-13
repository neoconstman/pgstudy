# Домашнее задание №6

 Этапы выполнения:

- Создан кластер PostgreSQL из двух экземпляров PostgreSQL на одной машине:  
`pg_createcluster -u postgres 15 master --start` (порт 5432)  
`pg_createcluster -u postgres 15 replica` (порт 5433)
Реплика пока не запускается.

- Очищена директория под данные реплики:
`rm -rf /var/lib/postgresql/15/replica/*`   
Конфигурационные файлы мастера и реплики настраивать нет необходимости, для тестовых целей подойдут значения по - умолчанию.  

- Создана резервная копия (под пользователем "postgres"):  
`pg_basebackup -p 5432 -R -h 127.0.0.1 -U postgres -D ./replica`  

- Перезапущена реплика. На этом создание кластера со схемой "мастер/асинхронная реплика" завершено.
`pg_ctlcluster restart 15 replica`  

- В кластер загружена малая БД с тайскими перевозками.

- Произведены замеры производительности с помощью pg_bench:
`pgbench -h 127.0.0.1 -p 5432 -U postgres -T 5 -S thai`  
> tps = 12462.439845 (without initial connection time)  


- Режим репликации заменен на синхронный. Для этого в конфигурационном файле устанавлены параметры:
`synchronous_standby_names = '*'` 
`synchronous_commit = remote_apply` 
(выбор пал на remote_apply, так как этот режим обеспечивает самую высокую гарантию доставки данных, однако так же сильнее всего увеличивает время записи).  
И перезапущен кластер `pg_ctlcluster restart 15 main`  

- Произведены повторные замеры производительности с помощью pg_bench:
`pgbench -b simple-update -c 10 -T 10 -U postgres thai`  
> tps = 1942.572096 (without initial connection time)  

- Настроена каскадная репликация. Для этого в конфигурационном файле устанавлены параметры:
`hot_standby = on`  
`max_wal_sender = 10`  
Перезапущен кластер: `pg_ctlcluster restart 15 main`   
Создан дополнительный кластер, выполняющий роль резервного хранилища:
`pg_createcluster -u postgres 15 backup`
Выполнен pg_basebackup с реплики `pg_basebackup -h 127.0.0.1 -p 5433 -D backup`  
