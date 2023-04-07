## Build of PGSodium

[PGSodium] is an encryption extensions for PostgreSQL based on libsodium

> - This guide assumes you are building libsodium on a similar machine that your production server is running and you want to just copy the finally library to your production servers
> - This guide is a continuation of PostgreSQL build process


### First build PGSodium
```
curl -s -L https://download.libsodium.org/libsodium/releases/libsodium-1.0.18.tar.gz | tar zxvf - && cd libsodium-1.0.18 && ./configure && make check && make -j 4 install

mkdir -p /home/postgres/pgsodium
cd /home/postgres/pgsodium
wget https://github.com/michelp/pgsodium/archive/refs/tags/v3.1.6.tar.gz
tar -xzf v3.1.6.tar.gz -C /home/postgres/pgsodium
rm v3.1.6.tar.gz
cd pgsodium-3.1.6
make -j 4 && make install
```

### Extract these files from your build server

> - If you are not planning to use server key generation based on _/dev/urandom_ you can skip copy of key generation script

```
# Extension's library
/usr/lib64/postgresql/pgsodium.so

# Extension's description for PostgreSQL
/usr/share/postgresql/extension/pgsodium.control

# Extension's SQL create scripts for functions
/usr/share/postgresql/extension/pgsodium--1.0.0--1.1.0.sql
/usr/share/postgresql/extension/pgsodium--1.0.0.sql
/usr/share/postgresql/extension/pgsodium--1.1.0--1.1.1.sql
/usr/share/postgresql/extension/pgsodium--1.1.1--1.2.0.sql
/usr/share/postgresql/extension/pgsodium--1.2.0--2.0.0.sql
/usr/share/postgresql/extension/pgsodium--2.0.0--2.0.1.sql
/usr/share/postgresql/extension/pgsodium--2.0.1--2.0.2.sql
/usr/share/postgresql/extension/pgsodium--2.0.2--3.0.0.sql
/usr/share/postgresql/extension/pgsodium--3.0.0--3.0.2.sql
/usr/share/postgresql/extension/pgsodium--3.0.2--3.0.3.sql
/usr/share/postgresql/extension/pgsodium--3.0.3--3.0.4.sql
/usr/share/postgresql/extension/pgsodium--3.0.4--3.0.5.sql
/usr/share/postgresql/extension/pgsodium--3.0.5--3.0.6.sql
/usr/share/postgresql/extension/pgsodium--3.0.6--3.0.7.sql
/usr/share/postgresql/extension/pgsodium--3.0.7--3.1.0.sql
/usr/share/postgresql/extension/pgsodium--3.1.0--3.1.1.sql
/usr/share/postgresql/extension/pgsodium--3.1.1--3.1.2.sql
/usr/share/postgresql/extension/pgsodium--3.1.2--3.1.3.sql
/usr/share/postgresql/extension/pgsodium--3.1.3--3.1.4.sql
/usr/share/postgresql/extension/pgsodium--3.1.4--3.1.5.sql
/usr/share/postgresql/extension/pgsodium--3.1.5--3.1.6.sql

# Extension's server key generation script
/home/postgres/pgsodium/pgsodium-3.1.6/getkey_scripts/pgsodium_getkey_urandom.sh
```

Once extracted, copy files like this

- Extension files copy to extension folder on production server
	+ If you installed PostgreSQL with _dnf_ then it should be this location _/usr/pgsql-15/share/extension/_
- Library file copy to lib folder
	+ If you installed PostgreSQL with _dnf_ then it should be this location _/usr/pgsql-15/lib_
- Rename _pgsodium_getkey_urandom.sh_ to _pgsodium_getkey_ and place it to extension folder on production server
	+ If you installed PostgreSQL with _dnf_ then it should be this location _/usr/pgsql-15/share/extension/_
	+ Change _pgsodium_getkey_ to executable with ```chmod +x ./pgsodium_getkey```


According to [Server Key Management](https://github.com/michelp/pgsodium#server-key-management) section of PGSodium documentation, you need to add pgsodium to _shared_preload_libraries_ configuration value in your PostgreSQL configuration file. This way pgsodium_getkey should be started automatically with server startup and ensure there is a server key generated in PostgreSQL data folder
