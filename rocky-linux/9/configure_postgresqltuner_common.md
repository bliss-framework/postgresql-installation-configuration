# Standard PostgreSQL Tuner configuration

> PostgreSQL Tuner checks few common configurations and these are steps to deal with some of them


 ## Extensions pg_stat_statements is disabled in database template1

Under postgres user
```bash
psql -d template1 -c "create extension pg_stat_statements;"
```
