## Install and configure a basic LAMP Installation

Packages needed

* www-servers/apache
* dev-lang/php
* dev-db/mariadb

Some packages have multiple slot/version and they're can be install in the same time like dev-lang/php You can manage multiple PHP version on your server : PHP5 or PHP7 can be installed in the same time (or more PHP version)

Multiple PHP slots

    # equo search dev-lang/php
    dev-lang/php-5.6.36
    dev-lang/php-7.1.18

or

    # equo s dev-lang/php | grep Slot
              Slot:          5.6
              Slot:          7.1 

The PHP versions shown may differ from those in the repositories

## Install packages

    # equo i www-servers/apache dev-db/mariadb dev-lang/php --ask 

This will install Apache and Mariadb database and PHP to higher version available on repository. If you need a different PHP version or multiple PHP versions, different slots need to be installed.

PHP-5.6 only

    # equo i dev-lang/php:5.6 

PHP-5.6 and PHP-7

    # equo i dev-lang/php:5.6 dev-lang/php:7.1 

## Configure Database

MariaDBis a MySQL alternative used by SabayonLinux.

MariaDB intends to maintain high compatibility with MySQL, ensuring a "drop-in" replacement capability with library binary equivalency and exact matching with MySQL APIs and commands.It includes the XtraDB storage engine for replacing InnoDB,as well as a new storage engine, Aria, that intends to be both a transactional and non-transactional engine perhaps even included in future versions of MySQL

Configure user and password database

    # emerge --config dev-db/mariadb 

Default MySql configuration file

* /etc/mysql/my.conf

The following options will be passed to all MySQL clients

    [client]
    password                                        = your_password
    port                                            = 3306
    socket                                          = /var/run/mysqld/mysqld.sock

    [mysql]
    character-sets-dir=/usr/share/mysql/charsets
    default-character-set=utf8

    [mysqladmin]
    character-sets-dir=/usr/share/mysql/charsets
    default-character-set=utf8


By default all databases will be stored to **/var/lib/mysql**.

## Configure Apache2 web server

Apache2 configuration file is stored at 

* /etc/conf.d/apache2

Basically we need to edit APACHE2_OPTS option. Configure Apache and all PHP versions installed

* /etc/conf.d/apache2

      APACHE2_OPTS="-D DEFAULT_VHOST -D INFO -D SSL -D SSL_DEFAULT_VHOST -D LANGUAGE -D PHP" 

With "-D PHP" option you can manage multiple PHP version installed in your system. eselect php command can help you to do this.

    # eselect php list apache2
      [1]   php5.6 *
      [2]   php7.0

Set PHP version you want ; Example : php7.0

    # eselect php set apache2 2 

Back to php5.6 version

    # eselect php set apache2 1 

## Apache and MySQL services

Now is time to start all configured services .

    # systemctl start mariadb 

Check if service is running

    # systemctl status mariadb

    ● mariadb.service - MariaDB database server
       Loaded: loaded (/usr/lib64/systemd/system/mariadb.service; enabled; vendor preset: disabled)
       Active: active (running) since sab 2017-05-27 20:39:37 CEST; 9min ago
       Process: 7512 ExecStartPost=/bin/sh -c systemctl unset-environment _WSREP_START_POSITION (code=exited, status=0/SUCCESS)
       Process: 7371 ExecStartPre=/bin/sh -c [ ! -e /usr/bin/galera_recovery ] && VAR= ||   VAR=`/usr/bin/galera_recovery`; [ $? -eq 0 ]   && systemctl set-environment _WSREP_START_POSITION=$VAR || exit 1 (code=exited, status=0/SUCCESS)
       Process: 7368 ExecStartPre=/bin/sh -c systemctl unset-environment _WSREP_START_POSITION (code=exited, status=0/SUCCESS)
    Main PID: 7481 (mysqld)
       Status: "Taking your SQL requests now..."
       CGroup: /system.slice/mariadb.service
           └─7481 /usr/sbin/mysqld

    mag 27 20:39:36 sabayon.local systemd[1]: Starting MariaDB database server...
    mag 27 20:39:37 sabayon.local mysqld[7481]: 2017-05-27 20:39:37 140257335244800 [Note] /usr/sbin/mysqld (mysqld 10.1.23-MariaDB) starting a...7481 ...
    mag 27 20:39:37 sabayon.local systemd[1]: Started MariaDB database server.
    Hint: Some lines were ellipsized, use -l to show in full.

Start Apache Web Server

    # systemctl start apache2 

Check if service is running

    # systemctl status apache2

    ● apache2.service - The Apache HTTP Server
       Loaded: loaded (/usr/lib64/systemd/system/apache2.service; disabled; vendor preset: disabled)
       Active: active (running) since sab 2017-05-27 20:47:26 CEST; 2s ago
      Process: 8415 ExecStop=/usr/sbin/apache2 $APACHE2_OPTS -k graceful-stop (code=exited, status=1/FAILURE)
     Main PID: 8532 (apache2)
       CGroup: /system.slice/apache2.service
               ├─8532 /usr/sbin/apache2 -D DEFAULT_VHOST -D INFO -D SSL -D SSL_DEFAULT_VHOST -D SUEXEC -D LANGUAGE -DFOREGROUND
               ├─8534 /usr/sbin/apache2 -D DEFAULT_VHOST -D INFO -D SSL -D SSL_DEFAULT_VHOST -D SUEXEC -D LANGUAGE -DFOREGROUND
               ├─8535 /usr/sbin/apache2 -D DEFAULT_VHOST -D INFO -D SSL -D SSL_DEFAULT_VHOST -D SUEXEC -D LANGUAGE -DFOREGROUND
               ├─8536 /usr/sbin/apache2 -D DEFAULT_VHOST -D INFO -D SSL -D SSL_DEFAULT_VHOST -D SUEXEC -D LANGUAGE -DFOREGROUND
               ├─8537 /usr/sbin/apache2 -D DEFAULT_VHOST -D INFO -D SSL -D SSL_DEFAULT_VHOST -D SUEXEC -D LANGUAGE -DFOREGROUND
               ├─8538 /usr/sbin/apache2 -D DEFAULT_VHOST -D INFO -D SSL -D SSL_DEFAULT_VHOST -D SUEXEC -D LANGUAGE -DFOREGROUND
               └─8539 /usr/sbin/apache2 -D DEFAULT_VHOST -D INFO -D SSL -D SSL_DEFAULT_VHOST -D SUEXEC -D LANGUAGE -DFOREGROUND

    mag 27 20:47:26 sabayon.local systemd[1]: Started The Apache HTTP Server.

or simply go to [http://localhost](http://localhost) or [http://<server-URL>](http://<server-URL>)

Apache offers many external modules and addictional functions as www-apache/mod_<module> and you can find them on Sabayon Repository or Portage. Please, be sure to read Apache Docs before installing them.

VirtualHosts configuration files can be found at :

* /etc/apache2/httpd.conf
* /etc/apache2/vhosts.d/00_default_*.conf

To enable Apache and MariaDB service at boot just type

    # systemctl enable apache2 && systemctl enable mariadb.

## Manage MySQL/MariaDB : PhpMyAdmin

PhpMyAdminis a free software tool written in PHP, intended to handle the administration of MySQL over the Web. phpMyAdmin supports a wide range of operations on MySQL and MariaDB. Frequently used operations (managing databases, tables, columns, relations, indexes, users, permissions, etc) can be performed via the user interface, while you still have the ability to directly execute any SQL statement.

Install PhpMyAdmin

    # equo i  dev-db/phpmyadmin --ask

Go to [http://localhost/phpmyadmin](http://localhost/phpmyadmin) or [http://<server-URL>/phpmyadmin](http://<server-URL>/phpmyadmin) to configure your PhpMyAdmin installation.

By default PhpMyAdmin will be installed to **/var/www/localhost/htdocs** (the default apache2 working directory) as **/var/www/localhost/htdocs/phpmyadmin** 
