# Installation of PostgreSQL Barman (Backup/Restore Management) extension on RockyLinux 9

> Based on various sources
> - https://www.youtube.com/watch?v=Q3OfrqTd85w

> Installation expects two machines, one for Barman and one running PostgreSQL instance

## Installation of Barman on a new machine

```

sudo dnf -y install https://download.postgresql.org/pub/repos/yum/reporpms/EL-9-x86_64/pgdg-redhat-repo-latest.noarch.rpm 
sudo dnf -y update
sudo dnf -qy module disable postgresql
sudo dnf -y install epel-release

# install client executables like pg_basebackup with server package because client package does not contain all executables, don't initialize the DB or services after the installation
sudo dnf -y install postgresql15-server

sudo dnf -y install barman

```




## Securing _postgres_ account

> Update password in SQL script below

```
sudo passwd postgres
su - postgres
psql -c "ALTER USER postgres WITH PASSWORD 'your-password';"
```

## (OPTIONAL) Install Postgrestuner.pl
> - https://github.com/jfcoz/postgresqltuner
```
sudo dnf install epel-release -y
sudo dnf install postgresqltuner -y
```

> BEFORE RUNNING THIS TOOL
> Create ~/.pgpass for postgres account so no passwords are visible in logs/console history
> https://www.postgresql.org/docs/current/libpq-pgpass.html
> OR ran under postgres user

_~/.pgpass_
```
localhost:5342:template1:postgres:your-password
```

## OS Configuration

> - Based on https://www.enterprisedb.com/blog/improving-postgresql-performance-without-making-changes-postgresql
> - https://www.centlinux.com/2021/08/disable-transparent-huge-pages-centos-rhel-8.html
> - https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/virtualization_tuning_and_optimization_guide/sect-virtualization_tuning_optimization_guide-memory-huge_pages-1gb-runtime
> - Test what HugePage memory sizes are supported

If first line returns _pse_ 2MB page size is supported
If second line returns pdpe1gb 1GB page size is supported - BETTER VARIANT for bigger servers
```
cat /proc/cpuinfo | grep -o ".\{0,0\}pse.\{0,0\}" | head -n 1
cat /proc/cpuinfo | grep -o ".\{0,0\}pdpe1gb.\{0,0\}" | head -n 1
```

> - Disable Transparent Huge Pages, PostgreSQL doesn't like them
> - Update grub configuration

_/etc/default/grub_
```
# add following configurations as last parameters in GRUB_CMDLINE_LINUX
# this line disables transparent hugepages and allocated 4 1GB blocks and 2*1024MB memory blocks

transparent_hugepage=never default_hugepagesz=1G hugepagesz=1G hugepages=4 hugepagesz=2M hugepages=1024
```

After that run this command to update grup settings and reboot
```
grub2-mkconfig -o /boot/grub2/grub.cfg
reboot
```

To check if the settings are valid you could run this command, but it only shows 4 1GB block allocated, if you use them
```
grep Huge /proc/meminfo
```

You can also check _/sys/devices/system/node/node0/hugepages_ where you'll find, for example, _nr_hugepages_ for total number of  1GB huge pages in _/hugepages-1048576kB_ folder and number of free 1GB huge pages in _free_hugepages_ file

![image](https://user-images.githubusercontent.com/6738956/229386364-40d8daeb-b80b-4c8d-b78f-31a841e7569e.png)

> System configuration
> 


This change in _/etc/sysctl.conf_ will disable memory overcommitment 
```
# PostgreSQL required settings
vm.overcommit_memory=2
```

## 
