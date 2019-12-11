# Создание сервиса мониторинга лога

1. Создаем файл кофигурации watchlog, в котором зададим переменным название файла для мониторинга и слово,которое будет искать сервис  
		[root@localhost sysconfig]# > watchlog  
		[root@localhost sysconfig]# vi watchlog  
		[root@localhost sysconfig]# cat watchlog  
	```
	# File and word that we will be monit
	WORD="ALERT"
	LOG=/var/log/watchlog.log
	```

2. Создаем тестовый файл логов /var/log/watchlog.log и наполняем его данными, включае отлавливаемое слово ‘ALERT’  
		[root@localhost sysconfig]# > /var/log/watchlog.log  
		[root@localhost sysconfig]# vi /var/log/watchlog.log  
		[root@localhost sysconfig]# cat /var/log/watchlog.log  
	```
	log1
	log2
	alert
	log3
	Alert
	log4
	...
	```

3. Создаем файл скрипта /opt/watchlog.sh, который будет искать заданное слово  
		[root@localhost sysconfig]# > /opt/watchlog.sh
		[root@localhost sysconfig]# vi /opt/watchlog.sh
		[root@localhost sysconfig]# cat /opt/watchlog.sh
	```
	#!/bin/bash

	WORD=$1
	LOG=$2
	DATE= `date`

	if grep $WORD $LOG &> /dev/null
	then
	  logger "$DATE: I found word, Master!"
	else
	  exit 0
	fi
	```

4. Создадим юнит /etc/systemd/system/watchlog.service для сервиса watchlog  
		[root@localhost sysconfig]# > /etc/systemd/system/watchlog.service  
		[root@localhost sysconfig]# vi /etc/systemd/system/watchlog.service  
		[root@localhost sysconfig]# cat /etc/systemd/system/watchlog.service  
	```
	[Unit]
	Description=My watchlog service

	[Service]
	Type=oneshot
	EnvironmentFile=/etc/sysconfig/watchdog
	ExecStart=/opt/watchlog.sh $WORD $LOG
	```
5. Создадим юнит /etc/systemd/system/watchlog.timer для таймера watchlog  
		[root@localhost sysconfig]# > /etc/systemd/system/watchlog.timer
		[root@localhost sysconfig]# vi /etc/systemd/system/watchlog.timer  
		[root@localhost sysconfig]# vi /etc/systemd/system/watchlog.timer  
		[root@localhost sysconfig]# cat /etc/systemd/system/watchlog.timer  
	```
	[Unit]
	Description=Run watchlog script every 30 second

	[Timer]
	# Run every 30 second
	OnUnitActiveSec=30
	Unit=watchlog.service

	[Install]
	WantedBy=multi-user.target
	```

