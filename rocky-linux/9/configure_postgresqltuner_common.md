# Standard PostgreSQL Tuner configuration

> PostgreSQL Tuner checks few common configurations and these are steps to deal with some of them

## Create config directory and files 

Under `postgres` user

Create directory `/var/lib/pgsql/[PostgreSQL version]/config`

Example
```bash
mkdir /var/lib/pgsql/13/config
touch /var/lib/pgsql/13/config/01-postgresql.cybertec.conf
touch /var/lib/pgsql/13/config/02-postgresql.standard.conf
```

Update `/var/lib/pgsql/[PostgreSQL version]/data/postgresql.conf`
At the bottom of the file
```
include_dir = /var/lib/pgsql/[PostgreSQL version]/config
```

## Extensions pg_stat_statements is disabled in database template1

Under `postgres` user

In `postgresql.conf` file add pg_stat_statements as shared_preload_libraries
```
shared_preload_libraries = "pg_stat_statements" # add any other library
```

```bash
psql -d template1 -c "create extension pg_stat_statements;"
```


