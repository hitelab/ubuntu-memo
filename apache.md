# Apache

## install apache & php5
```
apt-get update
apt-get install apache2 apache2-utils
apt-get install php5 libapache2-mod-php5 php5-mysql php5-curl php5-gd php5-mcrypt
```

## Could not reliably determine the server's fully qualified domain name
```
vi /etc/apache2/apache2.conf
```
>ServerName 127.0.0.1

## a2enmod, a2dismod, a2ensite, a2dissite
```
sudo a2enmod headers
sudo a2enmod rewrite
sudo a2enmod ldap authnz_ldap
sudo a2enmod ssl
```

## start,restart,reload,stop
```
service apache2 restart
```
