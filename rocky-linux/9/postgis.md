# Installation of PostGIS on RockyLinux 9

> - Based on https://computingforgeeks.com/install-postgis-on-centos-rocky-almalinux/
> - Fix of powertools missing https://forums.rockylinux.org/t/how-do-i-install-powertools-on-rocky-linux-9/7427

This script assumes installation done according to [Installation of Postgres on RockyLinux9](./)

```
sudo dnf -y install https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm
sudo dnf install dnf-plugins-core

#sudo dnf config-manager --set-enabled powertools -->> in RockyLinux 9 powertools is known under crb

dnf config-manager --enable crb

# there might be a newer version of this, check with 
# dnf list postgis* --available

dnf install -y postgis33_15 
```
