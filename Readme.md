preinstall

```
sudo apt-get update -y
sudo apt-get install apache2 php7.0 libaprutil1-dbd-mysql –y
Sudo apt-get install software-properties-common –y
Sudo apt-key adv --recv-keys --keyserver hkp://keyserver.ubuntu.com:80 0xF1656F24C74CD1D8 –y
add-apt-repository 'deb [arch=amd64,i386,ppc64el] http://www.ftp.saix.net/DB/mariadb/repo/10.1/ubuntu xenial main' -y
sudo apt-get update –y
sudo apt-get install mariadb-server mariadb-client –y
sudo apt-get install apache2 php7.0 libaprutil1-dbd-mysql –y
sudo systemctl start apache2
sudo systemctl start mysql
sudo systemctl enable apache2
sudo systemctl enable mysql
```

configure database

```
sudo mysql -u root 
(ENTER PASSWORD)
create database defaultsite_db;
GRANT SELECT, INSERT, UPDATE, DELETE ON defaultsite_db.* TO 'defaultsite_admin'@'localhost' IDENTIFIED BY 'password';
GRANT SELECT, INSERT, UPDATE, DELETE ON defaultsite_db.* TO 'defaultsite_admin'@'localhost.localdomain' IDENTIFIED BY 'password';
flush privileges;
use defaultsite_db;
create table mysql_auth ( username varchar(191) not null, passwd varchar(191), groups varchar(191), primary key (username) );
```

add users:
```
htpasswd -bns [user] [password]
```

config in db:
```
sudo mysql -u root 
INSERT INTO `mysql_auth` (`username`, `passwd`, `groups`) VALUES('siteuser','{SHA}tk7HEH6Wo7SKT6+3FHCgiGnJ6dA=', 'sitegroup');
exit;
```

Config for apache2:
```
sudo a2enmod dbd
sudo a2enmod authn_dbd
sudo a2enmod socache_shmcb
sudo a2enmod authn_socache
sudo mkdir /var/www/html/protecteddir
sudo chown -R www-data:www-data /var/www/html/protecteddir
sudo nano /etc/apache2/sites-available/000-default.conf
```

Attach next row to end of 000-default.conf and save 
```
DBDriver mysql 
 DBDParams "dbname=defaultsite_db user=defaultsite_admin pass=password"
 
 DBDMin 4 
 DBDKeep 8 
 DBDMax 20 
 DBDExptime 300
 
 <Directory "/var/www/html/protecteddir"> 
 # mod_authn_core and mod_auth_basic configuration 
 # for mod_authn_dbd 
 AuthType Basic 
 AuthName "My Server"
 
 # To cache credentials, put socache ahead of dbd here 
 AuthBasicProvider socache dbd
 
 # Also required for caching: tell the cache to cache dbd lookups! 
 AuthnCacheProvideFor dbd 
 AuthnCacheContext my-server
 
 # mod_authz_core configuration 
 Require valid-user
 
 # mod_authn_dbd SQL query to authenticate a user 
 AuthDBDUserPWQuery "SELECT passwd FROM mysql_auth WHERE username = %s" 
 </Directory>
```

restart apache
```
systemctl restart apache2
```

Now test url:
http://localhost/protecteddir


