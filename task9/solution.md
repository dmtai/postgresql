развернуть виртуальную машину любым удобным способом<br>
поставить на неё PostgreSQL 15 любым способом<br>
```
Создал vm c постгрес
```
настроить кластер PostgreSQL 15 на максимальную производительность не обращая внимание на возможные проблемы с 
надежностью в случае аварийной перезагрузки виртуальной машины<br>
```
Сгенерировал настройки через https://pgconfigurator.cybertec.at/ под виртуалку

# Connectivity
max_connections = 100
superuser_reserved_connections = 3

# Memory Settings
shared_buffers = '2048 MB'
work_mem = '32 MB'
maintenance_work_mem = '320 MB'
huge_pages = off
effective_cache_size = '6 GB'
effective_io_concurrency = 100 # concurrent IO only really activated if OS supports posix_fadvise function
random_page_cost = 1.25 # speed of random disk access relative to sequential access (1.0)

# Monitoring
shared_preload_libraries = 'pg_stat_statements' # per statement resource usage stats
track_io_timing=on # measure exact block IO times
track_functions=pl # track execution times of pl-language procedures if any

# Replication
wal_level = replica # consider using at least 'replica'
max_wal_senders = 0
synchronous_commit = off

# Checkpointing:
checkpoint_timeout = '15 min'
checkpoint_completion_target = 0.9
max_wal_size = '1024 MB'
min_wal_size = '512 MB'


# WAL writing
wal_compression = on
wal_buffers = -1 # auto-tuned by Postgres till maximum of segment size (16MB by default)
wal_writer_delay = 200ms
wal_writer_flush_after = 1MB


# Background writer
bgwriter_delay = 200ms
bgwriter_lru_maxpages = 100
bgwriter_lru_multiplier = 2.0
bgwriter_flush_after = 0

# Parallel queries:
max_worker_processes = 8
max_parallel_workers_per_gather = 4
max_parallel_maintenance_workers = 4
max_parallel_workers = 8
parallel_leader_participation = on

# Advanced features
enable_partitionwise_join = on
enable_partitionwise_aggregate = on
jit = on


# General notes:
# Note that not all settings are automatically tuned.
# Consider contacting experts at
# https://www.cybertec-postgresql.com
# for more professional expertise.
```

нагрузить кластер через утилиту через утилиту pgbench (https://postgrespro.ru/docs/postgrespro/14/pgbench)

```
starting vacuum...end.
progress: 10.0 s, 16856.2 tps, lat 2.918 ms stddev 1.941
progress: 20.0 s, 20637.2 tps, lat 2.386 ms stddev 1.796
progress: 30.0 s, 21932.4 tps, lat 2.240 ms stddev 1.389
progress: 40.0 s, 22313.0 tps, lat 2.199 ms stddev 1.273
progress: 50.0 s, 21462.7 tps, lat 2.291 ms stddev 1.378
progress: 60.0 s, 19344.0 tps, lat 2.556 ms stddev 1.795
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 50
query mode: simple
number of clients: 50
number of threads: 2
duration: 60 s
number of transactions actually processed: 1225505
latency average = 2.409 ms
latency stddev = 1.614 ms
tps = 20409.616258 (including connections establishing)
tps = 20410.702876 (excluding connections establishing)
```

написать какого значения tps удалось достичь, показать какие параметры в какие значения устанавливали и почему

```
До изменения настроек результаты pgbench

starting vacuum...end.
progress: 10.0 s, 13495.4 tps, lat 3.643 ms stddev 2.199
progress: 20.0 s, 10062.2 tps, lat 4.931 ms stddev 3.952
progress: 30.0 s, 8975.4 tps, lat 5.531 ms stddev 4.415
progress: 40.0 s, 11972.7 tps, lat 4.142 ms stddev 3.712
progress: 50.0 s, 12509.3 tps, lat 3.964 ms stddev 2.734
progress: 60.0 s, 11173.6 tps, lat 4.442 ms stddev 3.465
transaction type: <builtin: TPC-B (sort of)>
scaling factor: 50
query mode: simple
number of clients: 50
number of threads: 2
duration: 60 s
number of transactions actually processed: 681937
latency average = 4.360 ms
latency stddev = 3.463 ms
tps = 11356.056283 (including connections establishing)
tps = 11356.663575 (excluding connections establishing)
```
```
tps увеличился на 9к, во многом благодаря synchronous_commit = off

shared_buffers = '2048 MB' -  RAM/4, задает объем памяти, который будет использовать сервер бд для буферов в разделяемой
памяти. Чем больше shared_buffers, тем больше страниц будут закешированы.
work_mem = '32 MB' - задаёт максимальный объём памяти, который будет использоваться во внутренних операциях при обработке запросов,
чем выше это значение, тем быстрее будут проходить операции сортировки.
maintenance_work_mem = '320 MB' - задаёт максимальный объём
памяти для операций обслуживания БД, в частности VACUUM, CREATE INDEX и ALTER.
synchronous_commit = off - транзакция завершается только когда данные фактически сброшены на диск.
Возрастает риск потери изменений в случае сбоя
effective_io_concurrency = 100 - число параллельных операций ввода/вывода(по кол-ву дисков, для SSD - несколько сотен)
random_page_cost = 1.25 - отношение рандомного чтения к последовательному. Рекомендуется для SSD 1.1 - 1.25
```

