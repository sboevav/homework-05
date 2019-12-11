# Создание сервиса мониторинга лога

1. Создаем файл кофигурации watchlog, в котором зададим переменным название файла для мониторинга и слово,которое будет искать сервис  
		[root@localhost sysconfig]# > watchlog
		[root@localhost sysconfig]# 
		[root@localhost sysconfig]# vi watchlog
		[root@localhost sysconfig]# 
		[root@localhost sysconfig]# cat watchlog
	# File and word that we will be monit
	WORD="ALERT"
	LOG=/var/log/watchlog.log


