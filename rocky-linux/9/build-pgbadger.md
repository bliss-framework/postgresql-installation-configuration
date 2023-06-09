# Build/Installation of PGBadger on RockyLinux 9

> - Based on https://github.com/darold/pgbadger/#INSTALLATION

PGBadger is a PostgreSQL log analyzer, check out the documentation for [important configuration changes](https://github.com/darold/pgbadger/#postgresql-configuration)


This script assumes installation done according to [Installation of Postgres on RockyLinux9](./)

```
# There might be newer version check PGBadger releases
wget https://github.com/darold/pgbadger/archive/refs/tags/v12.1.tar.gz

tar xzf v12.1.tar.gz
cd pgbadger-12.1/
dnf install -y perl-ExtUtils-MakeMaker
dnf --enablerepo=crb install perl-Pod-Markdown

perl Makefile.PL

make && sudo make install
```
