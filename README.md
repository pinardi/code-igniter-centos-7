# code-igniter-centos-7
Step: 1. Set Host Name :

# hostname mysite.domain.com
# vi /etc/sysconfig/network

HOSTNAME=mysite.domain.com

-- Save & Quit (:wq)

Step: 2. Bind Host File :

# vi /etc/hosts

10.100.97.38    mysite.domain.com        mysite

-- Save & Quit (:wq)

Step: 3. Stop Firewall & Disable Selinux :

# service iptables stop
# chkconfig iptables off

# vi /etc/sysconfig/selinux

SELINUX=disabled

-- Save & Quit (:wq)

Step: 4. Install NTP Server For Time Synchronization :

# yum -y install ntp ntpdate
# service ntpd restart
# chkconfig ntpd on
# ntpdate pool.ntp.org

Step: 5. Reboot the System :

# init 6

Step: 6. Install EPEL & Remi Repository :

# yum -y install epel-release
# rpm -Uvh http://rpms.famillecollet.com/enterprise/remi-release-6.rpm

Step: 7. Install Apache Server :

# yum -y install --enablerepo=remi,epel httpd httpd-devel mod_ssl

Step: 8. Remove MySQL 5.1 & Install MySQL 5.6 :

# yum -y remove mysql mysql-*

# rpm -Uvh http://repo.mysql.com/mysql-community-release-el6-5.noarch.rpm
# yum -y install mysql mysql-server

Step: 9. Start MySQL Service & Set Root Password :

# service mysqld restart
# chkconfig mysqld on

# mysql_secure_installation

Step: 10. Install PHP 5.6 :

# rpm -Uvh https://mirror.webtatic.com/yum/el6/latest.rpm
# yum -y install php56w php56w-common php56w-cli php56w-devel php56w-gd \
    php56w-mysql php56w-mcrypt php56w-mbstring php56w-imap php56w-snmp \
    php56w-xml php56w-xmlrpc php56w-ldap php56w-pdo php56w-json php56w-dom \
    wget unzip curl git openssl
  
Step: 11. Create Database for CodeIgniter :

# mysql -u root -p
Enter Password: redhat

MySQL> create database codedb;
MySQL> grant all privileges on codedb.* to code@'localhost' identified by 'password';
MySQL> grant all privileges on codedb.* to code@'%' identified by 'password';
MySQL> flush privileges;
MySQL> exit

Step: 12. Install Composer :

Note: Composer is required for installing CodeIgniter Dependencies.

# curl -sS https://getcomposer.org/installer | php
# mv composer.phar /usr/local/bin/composer
# chmod +x /usr/local/bin/composer

Step: 13. Download & Install CodeIgniter Code from Git :

# cd /var/www/html
# git clone https://github.com/bcit-ci/CodeIgniter.git mysite
# cd mysite
# composer install
# chown -Rf apache:apache /var/www/html/mysite
# chmod -Rf 775 /var/www/html/mysite

Step: 14. Set Base URL & Database Connection :

# vi /var/www/html/mysite/application/config/config.php

$config['base_url'] = 'http://mysite.domain.com';

-- Save & Quit (:wq)

# vi /var/www/html/mysite/application/config/database.php

$db['default'] = array(
        'dsn'   => '',
        'hostname' => 'localhost',
        'username' => 'code',
        'password' => 'password',
        'database' => 'codedb',
        'dbdriver' => 'mysqli',
        
-- Save & Quit (:wq)

Step: 15. Create Apache Virtual Host :

# vi /etc/httpd/conf/httpd.conf

-- Add these Lines at Line no 313 :

<Directory /var/www/html/mysite>
     Options  -Indexes +Multiviews +FollowSymLinks
        DirectoryIndex index.php index.html
     AllowOverride All
     Allow from all
</Directory>

-- Uncomment Line 996 :

NameVirtualHost *:80

-- Add this Line at the End of the File :

RewriteEngine on

-- Save & Quit (:wq)

# vi /etc/httpd/conf.d/10.100.97.38.conf

<VirtualHost *:80>

  # Admin email, Server Name (domain name) and any aliases
  ServerAdmin techsupport@domain.com
  ServerName 10.100.97.38


  # Index file and Document Root (where the public files are located)
  DirectoryIndex index.php
  DocumentRoot /var/www/html


  # Custom log file locations
  LogLevel warn
  ErrorLog  /logs/10.100.97.38-error_log
  SetEnvIf Request_URI "\.(jpg|xml|png|gif|ico|js|css|swf|js?.|css?.)$" DontLog
  CustomLog /logs/10.100.97.38-access_log combined Env=!DontLog

</VirtualHost>

-- Save & Quit (:wq)

# vi /etc/httpd/conf.d/mysite.domain.com.conf

<VirtualHost *:80>

  # Admin email, Server Name (domain name) and any aliases
  ServerAdmin techsupport@domain.com
  ServerName mysite.domain.com
  ServerAlias www.mysite.domain.com


  # Index file and Document Root (where the public files are located)
  DirectoryIndex index.php
  DocumentRoot /var/www/html/mysite


  # Custom log file locations
  LogLevel warn
  ErrorLog  /logs/mysite.domain.com-error_log
  SetEnvIf Request_URI "\.(jpg|xml|png|gif|ico|js|css|swf|js?.|css?.)$" DontLog
  CustomLog /logs/mysite.domain.com-access_log combined Env=!DontLog

</VirtualHost>

-- Save & Quit (:wq)

# mkdir /logs

Step: 16. Start Apache Server :

# service httpd restart
# chkconfig httpd on

http://mysite.domain.com
