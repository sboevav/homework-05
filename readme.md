# Создание сервиса мониторинга лога

1. Создаем файл кофигурации watchlog, в котором зададим переменным название файла для мониторинга и слово,которое будет искать сервис  
		[root@localhost vagrant]# > /etc/sysconfig/watchlog  
		[root@localhost vagrant]# vi /etc/sysconfig/watchlog  
		[root@localhost vagrant]# cat /etc/sysconfig/watchlog  
	```
	# File and word that we will be monit
	WORD="ALERT"
	LOG=/var/log/watchlog.log
	```

2. Создаем тестовый файл логов /var/log/watchlog.log и наполняем его данными, включае отлавливаемое слово ‘ALERT’  
		[root@localhost vagrant]# > /var/log/watchlog.log  
		[root@localhost vagrant]# vi /var/log/watchlog.log  
		[root@localhost vagrant]# cat /var/log/watchlog.log  
	```
	log1
	log2
	log3
	Alert
	log4
	...
	```

3. Создаем файл скрипта /opt/watchlog.sh, который будет искать заданное слово  
		[root@localhost vagrant]# > /opt/watchlog.sh  
		[root@localhost vagrant]# vi /opt/watchlog.sh  
		[root@localhost vagrant]# cat /opt/watchlog.sh  
	```
	#!/bin/bash

	WORD=$1
	LOG=$2
	DATE=`date`

	if grep -i $WORD $LOG &> /dev/null
	then
	  logger "$DATE: I found the word $WORD in the log $LOG, Master!"
	fi
	exit 0
	```

4. Делаем скрипт исполняемым  
		[root@localhost vagrant]# chmod +x /opt/watchlog.sh  

5. Создадим юнит /etc/systemd/system/watchlog.service для сервиса watchlog  
		[root@localhost vagrant]# > /etc/systemd/system/watchlog.service  
		[root@localhost vagrant]# vi /etc/systemd/system/watchlog.service  
		[root@localhost vagrant]# cat /etc/systemd/system/watchlog.service  
	```
	[Unit]
	Description=My watchlog service

	[Service]
	Type=oneshot
	EnvironmentFile=/etc/sysconfig/watchlog
	ExecStart=/opt/watchlog.sh $WORD $LOG
	```
6. Создадим юнит /etc/systemd/system/watchlog.timer для таймера watchlog  
		[root@localhost vagrant]# > /etc/systemd/system/watchlog.timer  
		[root@localhost vagrant]# vi /etc/systemd/system/watchlog.timer  
		[root@localhost vagrant]# cat /etc/systemd/system/watchlog.timer  

	```
	[Unit]
	Description=Runs watchlog script every 5 seconds

	[Timer]
	AccuracySec=1us
	# Run after booting 5 seconds
	OnBootSec=5
	# Run every 5 seconds
	OnUnitActiveSec=5
	Unit=watchlog.service

	[Install]
	WantedBy=multi-user.target
	```
7. Далее делаем релоад systemd  
		[root@localhost vagrant]# systemctl daemon-reload  

8. Добавляем созданный сервис watchlog.service в автозагрузку  
		[root@localhost vagrant]# systemctl enable watchlog.service  

9. Запустим сервис watchlog  
		[root@localhost vagrant]# systemctl start watchlog  

10. Убедимся в периодическом запуске сервиса и поиске слова в логе  
		[root@localhost vagrant]# tail -f -n 10 /var/log/messages  
	```
	Dec 16 10:13:36 localhost systemd: Started My watchlog service.
	Dec 16 10:13:41 localhost systemd: Starting My watchlog service...
	Dec 16 10:13:41 localhost root: Mon Dec 16 10:13:41 UTC 2019: I found the word ALERT in the log /var/log/watchlog.log, Master!
	Dec 16 10:13:41 localhost systemd: Started My watchlog service.
	Dec 16 10:13:46 localhost systemd: Starting My watchlog service...
	Dec 16 10:13:46 localhost root: Mon Dec 16 10:13:46 UTC 2019: I found the word ALERT in the log /var/log/watchlog.log, Master!
	Dec 16 10:13:46 localhost systemd: Started My watchlog service.
	Dec 16 10:13:51 localhost systemd: Starting My watchlog service...
	Dec 16 10:13:51 localhost root: Mon Dec 16 10:13:51 UTC 2019: I found the word ALERT in the log /var/log/watchlog.log, Master!
	Dec 16 10:13:51 localhost systemd: Started My watchlog service.
	Dec 16 10:13:56 localhost systemd: Starting My watchlog service...
	Dec 16 10:13:56 localhost root: Mon Dec 16 10:13:56 UTC 2019: I found the word ALERT in the log /var/log/watchlog.log, Master!
	Dec 16 10:13:56 localhost systemd: Started My watchlog service.
	Dec 16 10:14:01 localhost systemd: Starting My watchlog service...
	Dec 16 10:14:01 localhost root: Mon Dec 16 10:14:01 UTC 2019: I found the word ALERT in the log /var/log/watchlog.log, Master!
	Dec 16 10:14:01 localhost systemd: Started My watchlog service.
	```

# Изменение init-скрипта Spawn-fcgi на unit-файл

1. Устанавливаем spawn-fcgi и необходимые для него пакеты  
		[root@localhost vagrant]# yum install epel-release -y && yum install spawn-fcgi php php-cli mod_fcgid httpd -y  
	```
	...
	Installed:
	  epel-release.noarch 0:7-11                                                   

	Complete!
	...
	Installed:
	  httpd.x86_64 0:2.4.6-90.el7.centos     mod_fcgid.x86_64 0:2.3.9-6.el7        
	  php.x86_64 0:5.4.16-46.1.el7_7         php-cli.x86_64 0:5.4.16-46.1.el7_7    
	  spawn-fcgi.x86_64 0:1.6.3-5.el7       

	Dependency Installed:
	  apr.x86_64 0:1.4.8-5.el7                                                     
	  apr-util.x86_64 0:1.5.2-6.el7                                                
	  httpd-tools.x86_64 0:2.4.6-90.el7.centos                                     
	  libzip.x86_64 0:0.10.1-8.el7                                                 
	  mailcap.noarch 0:2.1.41-2.el7                                                
	  php-common.x86_64 0:5.4.16-46.1.el7_7                                        

	Complete!
	```
2. Исправим переменные в конфигурационном файле /etc/sysconfig/spawn-fcgi  
		[root@localhost vagrant]# vi /etc/sysconfig/spawn-fcgi  
		[root@localhost vagrant]# cat /etc/sysconfig/spawn-fcgi  
	```
	# You must set some working options before the "spawn-fcgi" service will work.
	# If SOCKET points to a file, then this file is cleaned up by the init script.
	#
	# See spawn-fcgi(1) for all possible options.
	#
	# Example :
	SOCKET=/var/run/php-fcgi.sock
	OPTIONS="-u apache -g apache -s $SOCKET -S -M 0600 -C 32 -F 1 -- /usr/bin/php-cgi"
	#OPTIONS="-u apache -g apache -s $SOCKET -S -M 0600 -C 32 -F 1 -P /var/run/spawn-fcgi.pid -- /usr/bin/php-cgi"
	```
3. Создадим unt-файл /etc/systemd/system/spawn-fcgi.service  
		[root@localhost bin]# > /etc/systemd/system/spawn-fcgi.service  
		[root@localhost bin]# vi /etc/systemd/system/spawn-fcgi.service  
		[root@localhost bin]# cat /etc/systemd/system/spawn-fcgi.service  
	```
	[Unit]
	Description=Spawn-fcgi startup service by Otus
	After=network.target

	[Service]
	Type=simple
	PIDFile=/var/run/spawn-fcgi.pid
	EnvironmentFile=/etc/sysconfig/spawn-fcgi
	ExecStart=/usr/bin/spawn-fcgi -n $OPTIONS
	KillMode=process

	[Install]
	WantedBy=multi-user.target
	```
4. Запускаем и убеждаемся что все успешно работает   
		[root@localhost bin]# systemctl start spawn-fcgi  
		[root@localhost bin]# systemctl status spawn-fcgi  
	```
	● spawn-fcgi.service - Spawn-fcgi startup service by Otus
	   Loaded: loaded (/etc/systemd/system/spawn-fcgi.service; disabled; vendor preset: disabled)
	   Active: active (running) since Пн 2019-12-16 10:47:38 UTC; 3s ago
	 Main PID: 4729 (php-cgi)
	   CGroup: /system.slice/spawn-fcgi.service
		   ├─4729 /usr/bin/php-cgi
		   ├─4730 /usr/bin/php-cgi
		   ├─4731 /usr/bin/php-cgi
		   ├─4732 /usr/bin/php-cgi
		   ├─4733 /usr/bin/php-cgi
		   ├─4734 /usr/bin/php-cgi
		   ├─4735 /usr/bin/php-cgi
		   ├─4736 /usr/bin/php-cgi
		   ├─4737 /usr/bin/php-cgi
		   ├─4738 /usr/bin/php-cgi
		   ├─4739 /usr/bin/php-cgi
		   ├─4740 /usr/bin/php-cgi
		   ├─4741 /usr/bin/php-cgi
		   ├─4742 /usr/bin/php-cgi
		   ├─4743 /usr/bin/php-cgi
		   ├─4744 /usr/bin/php-cgi
		   ├─4745 /usr/bin/php-cgi
		   ├─4746 /usr/bin/php-cgi
		   ├─4747 /usr/bin/php-cgi
		   ├─4748 /usr/bin/php-cgi
		   ├─4749 /usr/bin/php-cgi
		   ├─4750 /usr/bin/php-cgi
		   ├─4751 /usr/bin/php-cgi
		   ├─4752 /usr/bin/php-cgi
		   ├─4753 /usr/bin/php-cgi
		   ├─4754 /usr/bin/php-cgi
		   ├─4755 /usr/bin/php-cgi
		   ├─4756 /usr/bin/php-cgi
		   ├─4757 /usr/bin/php-cgi
		   ├─4758 /usr/bin/php-cgi
		   ├─4759 /usr/bin/php-cgi
		   ├─4760 /usr/bin/php-cgi
		   └─4761 /usr/bin/php-cgi

	дек 16 10:47:38 localhost.localdomain systemd[1]: Started Spawn-fcgi star...
	Hint: Some lines were ellipsized, use -l to show in full.
	```

# Реализуем возможность запуска нескольких инстансов сервера с разными конфигами 
http://automation-remarks.com/setting-vagrant/
https://github.com/kyourselfer/OTUS_LinuxAdmin201804/tree/master/lesson6_SystemD/SCRIPTS/apache_multipleConf


1. Копируем юнит-файл /usr/lib/systemd/system/httpd.service в каталог /etc/systemd/system/ c изменением имени на httpd@.service  
		[root@localhost vagrant]# cp /usr/lib/systemd/system/httpd.service /etc/systemd/system/httpd@.service  

2. Изменяем в юнит-файле значение параметра EnvironmentFile с /etc/sysconfig/httpd на /etc/sysconfig/httpd-%I  
		[root@localhost vagrant]# vi /etc/systemd/system/httpd@.service  
		[root@localhost vagrant]# cat /etc/systemd/system/httpd@.service  
	```
	[Unit]
	Description=The Apache HTTP Server
	After=network.target remote-fs.target nss-lookup.target
	Documentation=man:httpd(8)
	Documentation=man:apachectl(8)

	[Service]
	Type=notify
	EnvironmentFile=/etc/sysconfig/httpd-%I
	ExecStart=/usr/sbin/httpd $OPTIONS -DFOREGROUND
	ExecReload=/usr/sbin/httpd $OPTIONS -k graceful
	ExecStop=/bin/kill -WINCH ${MAINPID}
	# We want systemd to give httpd some time to finish gracefully, but still want
	# it to kill httpd after TimeoutStopSec if something went wrong during the
	# graceful stop. Normally, Systemd sends SIGTERM signal right after the
	# ExecStop, which would kill httpd. We are sending useless SIGCONT here to give
	# httpd time to finish.
	KillSignal=SIGCONT
	PrivateTmp=true

	[Install]
	WantedBy=multi-user.target
	```
3. Создаем первый файл окружения /etc/sysconfig/httpd-first, в котором указываем первый файл конфигурации сервера  
		[root@localhost vagrant]# > /etc/sysconfig/httpd-first  
		[root@localhost vagrant]# vi /etc/sysconfig/httpd-first  
		[root@localhost vagrant]# cat /etc/sysconfig/httpd-first  
	```
	# /etc/sysconfig/httpd-first
	OPTIONS=-f conf/first.conf
	```
4. Создаем второй файл окружения /etc/sysconfig/httpd-second, в котором указываем второй файл конфигурации сервера  
		[root@localhost vagrant]# > /etc/sysconfig/httpd-second  
		[root@localhost vagrant]# vi /etc/sysconfig/httpd-second  
		[root@localhost vagrant]# cat /etc/sysconfig/httpd-second  
	```
	# /etc/sysconfig/httpd-second
	OPTIONS=-f conf/second.conf
	```
5. Создадим две новых копии текущего конфигурационного файла httpd.conf - first.conf и second.conf и проверим их наличие  
		[root@localhost vagrant]# cp /etc/httpd/conf/httpd.conf /etc/httpd/conf/first.conf  
		[root@localhost vagrant]# cp /etc/httpd/conf/httpd.conf /etc/httpd/conf/second.conf  
		[root@localhost vagrant]# ls /etc/httpd/conf/  
	```
	first.conf  httpd.conf  magic  second.conf
	```

6. Исправим второй конфигурационный файл. Поменяем значения PidFile и Listen  
		[root@localhost vagrant]# vi /etc/httpd/conf/second.conf  
		[root@localhost vagrant]# cat /etc/httpd/conf/second.conf  
	```
	...
	#
	# ServerRoot: The top of the directory tree under which the server's
	# configuration, error, and log files are kept.
	#
	# Do not add a slash at the end of the directory path.  If you point
	# ServerRoot at a non-local disk, be sure to specify a local disk on the
	# Mutex directive, if file-based mutexes are used.  If you wish to share the
	# same ServerRoot for multiple httpd daemons, you will need to change at
	# least PidFile.
	#
	ServerRoot "/etc/httpd"
	PidFile /var/run/httpd-second.pid

	#
	# Listen: Allows you to bind Apache to specific IP addresses and/or
	# ports, instead of the default. See also the <VirtualHost>
	# directive.
	#
	# Change this to Listen on specific IP addresses as shown below to 
	# prevent Apache from glomming onto all bound IP addresses.
	#
	#Listen 12.34.56.78:80
	Listen 8080
	...
	```
7. Делаем релоад systemd  
		[root@localhost vagrant]# systemctl daemon-reload  

8. Запустим два инстанса  
		systemctl start httpd@first  
		systemctl start httpd@second  

9. Проверим запуск первого экземпляра  
		[root@localhost vagrant]# systemctl status httpd@first  
	```
	● httpd@first.service - The Apache HTTP Server
	   Loaded: loaded (/etc/systemd/system/httpd@.service; disabled; vendor preset: disabled)
	   Active: active (running) since Пн 2019-12-16 18:46:28 UTC; 35s ago
	     Docs: man:httpd(8)
		   man:apachectl(8)
	 Main PID: 4176 (httpd)
	   Status: "Total requests: 0; Current requests/sec: 0; Current traffic:   0 B/sec"
	   CGroup: /system.slice/system-httpd.slice/httpd@first.service
		   ├─4176 /usr/sbin/httpd -f conf/first.conf -DFOREGROUND
		   ├─4177 /usr/sbin/httpd -f conf/first.conf -DFOREGROUND
		   ├─4178 /usr/sbin/httpd -f conf/first.conf -DFOREGROUND
		   ├─4179 /usr/sbin/httpd -f conf/first.conf -DFOREGROUND
		   ├─4180 /usr/sbin/httpd -f conf/first.conf -DFOREGROUND
		   ├─4181 /usr/sbin/httpd -f conf/first.conf -DFOREGROUND
		   └─4182 /usr/sbin/httpd -f conf/first.conf -DFOREGROUND

	дек 16 18:46:28 localhost.localdomain systemd[1]: Starting The Apache HTTP...
	дек 16 18:46:28 localhost.localdomain httpd[4176]: AH00558: httpd: Could n...
	дек 16 18:46:28 localhost.localdomain systemd[1]: Started The Apache HTTP ...
	Hint: Some lines were ellipsized, use -l to show in full.
	```
10. Проверим запуск второго экземпляра  
		[root@localhost vagrant]# systemctl status httpd@second  
	```
	● httpd@second.service - The Apache HTTP Server
	   Loaded: loaded (/etc/systemd/system/httpd@.service; disabled; vendor preset: disabled)
	   Active: active (running) since Пн 2019-12-16 18:47:53 UTC; 12s ago
	     Docs: man:httpd(8)
		   man:apachectl(8)
	 Main PID: 4216 (httpd)
	   Status: "Total requests: 0; Current requests/sec: 0; Current traffic:   0 B/sec"
	   CGroup: /system.slice/system-httpd.slice/httpd@second.service
		   ├─4216 /usr/sbin/httpd -f conf/second.conf -DFOREGROUND
		   ├─4217 /usr/sbin/httpd -f conf/second.conf -DFOREGROUND
		   ├─4218 /usr/sbin/httpd -f conf/second.conf -DFOREGROUND
		   ├─4219 /usr/sbin/httpd -f conf/second.conf -DFOREGROUND
		   ├─4220 /usr/sbin/httpd -f conf/second.conf -DFOREGROUND
		   ├─4221 /usr/sbin/httpd -f conf/second.conf -DFOREGROUND
		   └─4222 /usr/sbin/httpd -f conf/second.conf -DFOREGROUND

	дек 16 18:47:53 localhost.localdomain systemd[1]: Starting The Apache HTTP...
	дек 16 18:47:53 localhost.localdomain httpd[4216]: AH00558: httpd: Could n...
	дек 16 18:47:53 localhost.localdomain systemd[1]: Started The Apache HTTP ...
	Hint: Some lines were ellipsized, use -l to show in full.
	```
11. Проверим прослушиваемые порты  
		ss -tnulp | grep httpd  
	```
	tcp    LISTEN     0      511       *:8080                  *:*                   users:(("httpd",pid=4222,fd=3),("httpd",pid=4221,fd=3),("httpd",pid=4220,fd=3),("httpd",pid=4219,fd=3),("httpd",pid=4218,fd=3),("httpd",pid=4217,fd=3),("httpd",pid=4216,fd=3))
	tcp    LISTEN     0      511       *:80                    *:*                   users:(("httpd",pid=4182,fd=3),("httpd",pid=4181,fd=3),("httpd",pid=4180,fd=3),("httpd",pid=4179,fd=3),("httpd",pid=4178,fd=3),("httpd",pid=4177,fd=3),("httpd",pid=4176,fd=3))
	```

