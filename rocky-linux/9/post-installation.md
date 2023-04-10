# Post-installation steps after PostgreSQL installation on RockyLinux 9

>These steps are specific for RockyLinux.
>There are also [common post installation steps](../../configuration) that needs to be done as well


## Installation of PostgreSQL contrib package for standard extensions

A restart of postgresql.service will be necessary to load new packages

```
sudo dnf install -y postgresql15-contrib
```

After configuration of _postgresql.conf_ 
```
systemctl restart postgresql-15.service
```

## Firewall settings

You might need to enable PostgreSQL ports on firewall, you can use this script.

```
firewall-cmd --zone=public --add-service=postgresql --permanent
firewall-cmd --reload
```
