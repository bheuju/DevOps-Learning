# mysql

sudo apt install mariadb-server
sudo systemctl list-units --type=service
sudo systemctl list-units --type=service | grep mariadb

sudo mysql_secure_installation

sudo systemctl status mariadb
service file

/home/ubuntu
cat .my.cnf

```ini
[client]
user=root
password=redhat
```

```sql
show database;
use mysql;
show tables;
desc user;
select Host,User,Password from user;
create user bishal@'%' identified by 'redhat';

flush privileges;
grant select on mysql.user to bishal@'%';
grant all privileges on mysql.user to bishal@'%';
flush privileges;

create database dbname;
grant all privileges on *.* to bishal@'%';
grant all privileges on <database>.* to bishal@'%';

cat .mysql_history
```

allow remote
`cd /etc/mysql/mariadb.conf.d`
`cat mariadb.conf.d/50-server.cnf | grep bind`
change `127.0.0.1` to `0.0.0.0`
