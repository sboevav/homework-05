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
4. Убеждаемся что все успешно работает   
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

# Реализуем возможность запуска нескольких инстансов сервера apache httpd с разными конфигами 


