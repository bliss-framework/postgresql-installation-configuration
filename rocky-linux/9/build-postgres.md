# Build PostgreSQL 15 on RockyLinux 9

```
sudo dnf group install -y "Development Tools"

sudo dnf install -y perl-FindBin readline readline-devel wget

groupadd -r postgres && useradd --no-log-init -r -m -s /bin/bash -g postgres -G wheel postgres

export PGDATA=/home/postgres/data
mkdir "$PGDATA"

cd /home/postgres/

git clone --branch REL_15_STABLE https://github.com/postgres/postgres.git --depth=1

cd postgres

./configure --prefix=/usr/ --enable-debug --enable-depend --enable-cassert --enable-profiling CFLAGS="-ggdb -Og -g3 -fno-omit-frame-pointer"

make -j 4

make install

chown postgres:postgres -R /home/postgres

cd ..
```

## Installation of PGSodium

Continuation of the installation process
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
