Hardware
```
# Percona Toolkit System Summary Report ######################
        Date | 2022-08-16 11:01:17 UTC (local TZ: EDT -0400)
    Hostname | beast-node1-ubuntu
      Uptime | 124 days, 15:49,  6 users,  load average: 0.64, 0.36, 0.26
      System | Supermicro; SYS-F619P2-RTN; v0123456789 (Other)
 Service Tag | S292592X0110239A
    Platform | Linux
     Release | Ubuntu 20.04.4 LTS (focal)
      Kernel | 5.4.0-107-generic
Architecture | CPU = 64-bit, OS = 64-bit
   Threading | NPTL 2.31
     SELinux | No SELinux detected
 Virtualized | No virtualization detected
# Processor ##################################################
  Processors | physical = 2, cores = 40, virtual = 80, hyperthreading = yes
      Models | 80xIntel(R) Xeon(R) Gold 6230 CPU @ 2.10GHz
      Caches | 80x28160 KB
# Memory #####################################################
       Total | 187.6G
        Free | 4.3G
```

Storage:
`Intel® SSD DC P4610 Series`

myrocks config:
```
[mysqld]
socket=/tmp/mysql.sock
#ssl=0

log-bin=mysql-bin
#skip-log-bin
server-id=1721604
binlog_format = 'ROW'
binlog_row_image=FULL
default_authentication_plugin='mysql_native_password'

sync_binlog=10000
log-error=error.log

# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0

# Recommended in standard MySQL setup

# general
  table_open_cache = 200000
  table_open_cache_instances=64
  back_log=3500
  max_connections=4000

# files
max_prepared_stmt_count=1000000

rocksdb_max_open_files=-1
rocksdb_max_background_jobs=8
rocksdb_max_total_wal_size=4G
rocksdb_block_size=16384
rocksdb_table_cache_numshardbits=6
rocksdb_block_cache_size=30G
#rockdsb_bulk_load=1
#rocksdb_write_policy=write_unprepared

# rate limiter
rocksdb_bytes_per_sync=16777216
rocksdb_wal_bytes_per_sync=4194304

#rocksdb_rate_limiter_bytes_per_sec=104857600 #100MB/s
#
# # triggering compaction if there are many sequential deletes

rocksdb_compaction_sequential_deletes_count_sd=1
rocksdb_compaction_sequential_deletes=199999
rocksdb_compaction_sequential_deletes_window=200000

rocksdb_default_cf_options="write_buffer_size=256m;target_file_size_base=32m;max_bytes_for_level_base=512m;max_write_buffer_number=4;level0_file_num_compaction_trigger=4;level0_slowdown_writes_trigger=20;level0_stop_writes_trigger=30;max_write_buffer_number=4;block_based_table_factory={cache_index_and_filter_blocks=1;filter_policy=bloomfilter:10:false;whole_key_filtering=0};level_compaction_dynamic_level_bytes=true;optimize_filters_for_hits=true;memtable_prefix_bloom_size_ratio=0.05;prefix_extractor=capped:12;compaction_pri=kMinOverlappingRatio;compression=kLZ4Compression;bottommost_compression=kZSTD;compression_opts=-14:4:0"

rocksdb_max_subcompactions=4
rocksdb_compaction_readahead_size=16m

rocksdb_use_direct_reads=ON
rocksdb_use_direct_io_for_flush_and_compaction=ON

#default_storage_engine=RocksDB
```

InnoDB config:

```
[mysqld]
socket=/tmp/mysql.sock
#ssl=0

#log-bin=mysql-bin
#skip-log-bin

#audit_log_format=json

log-bin=mysql-bin
#skip-log-bin
server-id=1721604
binlog_format = 'ROW'
binlog_row_image=FULL
default_authentication_plugin='mysql_native_password'

sync_binlog=10000

log-error=error.log

# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0

# Recommended in standard MySQL setup

# files
max_prepared_stmt_count=1000000

#default_storage_engine=RocksDB


user=mysql

#ssl=0
performance_schema=OFF


#relay_log=relay
#sync_binlog=1000
#binlog_format = ROW
#binlog_row_image=MINIMAL


# general
table_open_cache = 200000
table_open_cache_instances=64
back_log=3500
max_connections=4000
 join_buffer_size=256K
 sort_buffer_size=256K

# files
innodb_file_per_table
innodb_log_file_size=10G
innodb_log_files_in_group=2
innodb_open_files=4000

# buffers
innodb_buffer_pool_size= 6G
innodb_buffer_pool_instances=8
innodb_page_cleaners=8
innodb_lru_scan_depth=128
innodb_log_buffer_size=64M

default_storage_engine=InnoDB

innodb_flush_log_at_trx_commit  = 1
innodb_doublewrite= 1
innodb_flush_method             = O_DIRECT
innodb_file_per_table           = 1
innodb_io_capacity=2000
innodb_io_capacity_max=4000
innodb_flush_neighbors=0


#innodb_monitor_enable=all
max_prepared_stmt_count=1000000 

bind_address = 0.0.0.0
```

start commands:
```
docker run --name ps1  -e MYSQL_ROOT_PASSWORD=secret --net=host -v /data/docker-innodb:/var/lib/mysql -v /mnt/data/vadim/servers/docker/my-inno.cnf:/etc/mysql/my.cnf -v /data/tmp:/tmp -m 160g -d percona/percona-server:8.0.28 --innodb_buffer_pool_size=120G
```

```
docker run --name ps-rocks  -e MYSQL_ROOT_PASSWORD=secret -e INIT_ROCKSDB=1 --net=host -m 160g -v /data/docker-myrocks:/var/lib/mysql -v /mnt/data/vadim/servers/docker/my-rocks.cnf:/etc/mysql/my.cnf -v /data/tmp:/tmp  -d percona/percona-server:8.0.28 --rocksdb_block_cache_size=80G
```

