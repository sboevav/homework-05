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

2. Создаем тестовый файл логов /var/log/watchlog.log и наполняем его данными, включае отлавливаемое слово ‘ALERT’  
		[root@localhost sysconfig]# > /var/log/watchlog.log  
		[root@localhost sysconfig]# vi /var/log/watchlog.log  
		[root@localhost sysconfig]# cat /var/log/watchlog.log  
	log1  
	log2  
	alert  
	log3  
	Alert  
	log4  
	...  


