# Postgres Install

#### links
https://www.postgresql.org/docs/10/install-procedure.html  

https://git.postgresql.org/gitweb/?p=postgresql.git  

https://www.tecmint.com/install-postgresql-from-source-code-in-linux/  
https://www.postgresql.org/docs/11/server-start.html  

https://pgtune.leopard.in.ua/#/  


## simple install
### prepare postgres

#### prepare folder for install postgres (optional)
mkdir /data  
mkdir /data/postgres  
cd /data/postgres  
mkdir pg  

#### Copy source
-- git clone source  
git clone git://git.postgresql.org/git/postgresql.git  

#### change directory
cd postgresql  

#### Use Tag
--git branch new_branch tag  
git branch REL_11_1 REL_11_1  

#### install dependencies
sudo apt install gcc make flex bison libreadline-dev zlib1g-dev libxml2-dev libsystemd-dev  


#### do config
./configure --help  

for example from file  
touch conf.sh  
nano conf.sh  

```
./configure --prefix=/data/postgres/pg \
--with-systemd
```

chmod +rwx conf.sh  
./conf.sh  

or simple run  

```
./configure --prefix=/data/postgres/pg --with-systemd
```

#### make and install postgres
make  
make install  

### create database
useradd postgres  
passwd postgres  
mkdir /data/postgres/pg/data  
chown -R postgres. /data/postgres/pg/data  
echo 'export PATH=$PATH:/data/postgres/pg/bin' > /etc/profile.d/postgres.sh  

su postgres  
/data/postgres/pg/bin/initdb -D /data/postgres/pg/data/ -U postgres -W  

### config system.d
#### create service file
cd /etc/systemd/system  

touch postgres.service  
nano postgres.service  

```
[Unit]
Description=PostgreSQL database server
Documentation=man:postgres(1)

[Service]
Type=notify
User=postgres
ExecStart=/data/postgres/pg/bin/postgres -D /data/postgres/pg/data
ExecReload=/bin/kill -HUP $MAINPID
KillMode=mixed
KillSignal=SIGINT
TimeoutSec=0
Restart=always
RestartSec=3s

[Install]
WantedBy=multi-user.target
```
#### enable service
systemctl enable postgres  

#### start service
systemctl start postgres  

### view logs
journalctl --since='5 minutes ago' -u postgres  

## Create Deb Package
### install fpm
https://fpm.readthedocs.io/en/latest/installing.html  

sudo apt install ruby ruby-dev rubygems build-essential  
sudo gem install --no-ri --no-rdoc fpm  


### prepare postgres
#### Copy source
-- git clone source  
git clone git://git.postgresql.org/git/postgresql.git  

#### change directory
cd postgresql  

#### Use Tag
--git branch new_branch tag  
git branch REL_11_1 REL_11_1  

#### install dependencies
sudo apt install gcc make flex bison libreadline-dev zlib1g-dev libxml2-dev libsystemd-dev  

#### do config
-- ./configure  
./configure --with-gssapi --with-pam --with-ldap --with-openssl --with-selinux --with-libxml --with-libxslt --with-systemd CFLAGS="-O2 -fno-omit-frame-pointer" --prefix=/opt/postgresql/11.1  

if some errors you can search libs  
apt-cache search libselinux | less  
sudo apt install libselinux1-dev  

#### make postgres
make  

#### install into temp directory
export DESTDIR="/tmp/install_dir" && make install  

### create Deb package

fpm -s dir -t deb -n postgresql-mf -v 11.1 -C /tmp/install_dir -p ~/Documents/temp/postgres-mf-111.deb -m "abc@abc.abc" -d 'libc6' -d 'libpq5' -d 'libsystemd0' -d 'libxml2' -d 'zlib1g' -d 'sysstat' -d 'libedit2' --deb-no-default-config-files --description="postgres 11.1 package mf_build" --vendor="mf_team"  

