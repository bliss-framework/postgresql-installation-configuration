# Installation of PostgreSQL Barman (Backup/Restore Management) extension on RockyLinux 9

> Based on various sources
> - https://www.youtube.com/watch?v=Q3OfrqTd85w

> Installation expects two machines, one for Barman and one running PostgreSQL instance

## [Barman server] Installation of Barman on a new machine

```bash

sudo dnf -y install https://download.postgresql.org/pub/repos/yum/reporpms/EL-9-x86_64/pgdg-redhat-repo-latest.noarch.rpm 
sudo dnf -y update
sudo dnf -qy module disable postgresql
sudo dnf -y install epel-release

# install client executables like pg_basebackup with server package because client package does not contain all executables, don't initialize the DB or services after the installation
sudo dnf -y install postgresql15-server

sudo dnf -y install barman

```

## [PostgreSQL server] Configuration of PostgreSQL Database

Steps done under postgres account (`su - postgres`)

```bash
createuser -s -P barman
# enter password


# create streaming user if you are planning to use it
createuser --replication -P barman_streaming

# IF YOU HAVE PostgreSQL Server version < 15 Beta RUN THIS COMMAND
psql << EOF
GRANT EXECUTE ON FUNCTION pg_start_backup(text, boolean, boolean) to barman;
GRANT EXECUTE ON FUNCTION pg_stop_backup() to barman;
GRANT EXECUTE ON FUNCTION pg_stop_backup(boolean, boolean) to barman;
GRANT EXECUTE ON FUNCTION pg_switch_wal() to barman;
GRANT EXECUTE ON FUNCTION pg_create_restore_point(text) to barman;

GRANT pg_read_all_settings TO barman;
GRANT pg_read_all_stats TO barman;
EOF

# IF YOU HAVE PostgreSQL Server version >= 15 Beta RUN THIS COMMAND
psql << EOF
GRANT EXECUTE ON FUNCTION pg_backup_start(text, boolean) to barman;
GRANT EXECUTE ON FUNCTION pg_backup_stop(boolean) to barman;
GRANT EXECUTE ON FUNCTION pg_switch_wal() to barman;
GRANT EXECUTE ON FUNCTION pg_create_restore_point(text) to barman;

GRANT pg_read_all_settings TO barman;
GRANT pg_read_all_stats TO barman;
EOF
```

> If you are planning to user WAL replication you might need to configure `max_wal_senders` in `postgresql.conf` to number bigger than 0


**!!! Update pg_hba.conf file for barman/barman_streaming so they can connect !!!**

For standard backup account, for example
```
host    postgres             barman             10.11.12.0/24             scram-sha-256
```

For streaming account, for example 
```
host    replication     barman_streaming             10.11.12.0/24             scram-sha-256
```

Reload `pg_hba.conf` with 
```bash
psql -c "select pg_reload_conf();"
```


## [Barman server] Create `~barman/.pgpass` for secure connection

```bash
cd ~barman

# create records for both users (or the one you use) in this format
# hostname:port:*:username:password

nano .pgpass

```

## [Barman server] Test connection

Run
```

# To test standard backup account connection

psql -h my-db-server.com -U barman -d postgres -c "select current_user;"

# Sample result
barman



# To test WAL streaming account connection

psql "host=my-db-server.com user=barman_streaming dbname=postgres replication=database" -c "IDENTIFY_SYSTEM;"

# Sample result
#      systemid       | timeline |  xlogpos   |  dbname
#---------------------+----------+------------+----------
# 7050253711504958258 |        1 | 7/9C9D6E28 | postgres


```

## [Barman server] Create `my-db-server.com` Barman configuration


This configuration is only for "Streaming-Only" setup, see the official documentation for all possible backup scenarios

```bash

nano /etc/barman.d/my-db-server.com.conf

## insert this text and configure properly, server name in [] is crucial, it is used by every command

[my-db-server.com]
description =  "My DB Server - Streaming-Only"
conninfo = host=my-db-server.com user=barman dbname=postgres
streaming_conninfo = host=my-db-server.com user=barman_streaming
backup_method = postgres
streaming_archiver = on
slot_name = barman
path_prefix = /usr/pgsql-15/bin  # crucial to point to correct folder, 15 libraries seems working just fine with v14 of PostgreSQL Server

```

After that is done run under `barman` user

```bash
barman check my-db-server.com
```

Almost all should be green except

``` 
WAL archive: FAILED (please make sure WAL shipping is setup)
replication slot: FAILED (replication slot 'barman' doesn't exist. Please execute 'barman receive-wal --create-slot my-db-server.com')
```

You can create replication socket with given command, this will create a replication slot on PostgreSQL Server just for barman

```
barman receive-wal --create-slot my-db-server.com
```

Run the check again
```bash
barman check my-db-server.com
```

All should be green except

```
WAL archive: FAILED (please make sure WAL shipping is setup)
```

Run this command to fix it

```
barman switch-xlog --force --archive my-db-server.com
```

Now all should be working...


Run this command to do a full backup of the server

```
barman backup my-db-server.com
```