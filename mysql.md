# Mysql

## Install mysql server
```
apt-get install mysql-server
```

## Setting
```
vi /etc/mysql/my.cnf
```
>[client]
>default-character-set=utf8

>[mysqld]
>default-storage-engine=MYISAM
>character-set-server=utf8
>collation-server=utf8_general_ci

>bind-address            = 0.0.0.0

## Create administrator
mysql
```
grant all on *.* to 'admin'@'%' identified by 'password' with grant option;
```
