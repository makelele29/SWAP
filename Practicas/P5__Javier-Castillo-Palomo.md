Práctica 5
===================
:bust_in_silhouette: Javier Castillo Palomo

:bust_in_silhouette: Cristóbal Rodríquez Reina

----------


Replicación de bases de datos MySQL.
==================

> **Cuestiones a resolver**

> - Realizar copias de seguridad de una BD completa usando mysqldump.
> - Restaurar dicha copia en la segunda máquina (clonado manual de la BD).
> - Realizar la configuración maestro-esclavo de los servidores MySQL para que la
replicación de datos se realice automáticamente.

Crear una BD e insertar datos
-------------------

###:page_facing_up:  Código
El código que hemos puesto es el mismo que el del guión de la práctica pero hemos añadido dos campos más con nuestro nombre.

    # mysql -uroot -p
	Enter password: 
		Welcome to the MySQL monitor.  Commands end with ; or \g.
	Your MySQL connection id is 40
	Server version: 5.5.24-0ubuntu0.12.04.1 (Ubuntu)
	
	Copyright (c) 2000, 2014, Oracle and/or its affiliates. All rights reserved.
	
	Oracle is a registered trademark of Oracle Corporation and/or its
	affiliates. Other names may be trademarks of their respective
	owners.
	
	Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
	mysql> create database maquina1;
	Query OK, 1 row affected (0,00 sec)
	mysql> use maquina1;
	Database changed
	mysql> show tables;
	Empty set (0,00 sec)
	mysql> create table datos(nombre varchar(100),tlf int);
	Query OK, 0 rows affected (0,01 sec)
	mysql> show tables;
	+---------------------+
	| Tables_in_contactos |
	+---------------------+
	| datos |
	+---------------------+
	1 row in set (0,00 sec)
	mysql> insert into datos(nombre,tlf) values ("pepe",95834987);
	Query OK, 1 row affected (0,00 sec)
	mysql> insert into datos(nombre,tlf) values ("Cristobal",95834988);
	Query OK, 1 row affected (0,00 sec)
	mysql> insert into datos(nombre,tlf) values ("Javier",95834989);
	Query OK, 1 row affected (0,00 sec)
	mysql> select * from datos;
	+---------+-----------+
	| nombre    | tlf |
	+---------+-----------+
	| pepe      | 95834987 |
	| Cristobal | 95834988 |
	| Javier    | 95834989 |
	+---------+-----------+
	3 rows in set (0,00 sec)

Replicar una BD MySQL con mysqldump
--------------------


### :pencil2: Pasos seguidos
####Bloquear tablas.

Para que los datos no se pierdan al hacer la copia de seguridad, ya que puede haber actualizaciones constantes de la BD debemos hacer:

    # mysql -u root –p
	mysql> FLUSH TABLES WITH READ LOCK;
	mysql> quit
####Guardar datos.

Ahora debemos exportar la BD para luego pasarla a la máquina 2:

    # mysqldump maquina1 -u root -p > /root/maquina1db.sql
####Desbloquear tablas
Una vez se haya exportado la BD desbloqueamos las tablas que bloqueemos anteriormente.

    # mysql -u root –p
	mysql> UNLOCK TABLES;
	mysql> quit
####Copiar BD a máquina 2

    scp /root/maquina1bd.sql root@192.168.1.120:/root/
	The authenticity of host '192.168.1.120 (192.168.1.120)' can't be established.
	ECDSA key fingerprint is ff:4a:e2:40:66:80:45:f5:48:8d:20:1b:a7:6c:2a:71.
	Are you sure you want to continue connecting (yes/no)? yes
	Warning: Permanently added '192.168.1.120' (ECDSA) to the list of known hosts.
	root@192.168.1.120's password: 
	maquina1bd.sql                                100% 1909     1.9KB/s   00:00  
####Importar la BD a la máquina 2
Una vez que ya tenemos la BD importamos para insertarle las tablas de la máquina 1, antes de eso debemos crear la BD.

    # mysql -u root –p
	mysql> CREATE DATABASE maquina2;
	mysql> quit
	
	# mysql -u root -p maquina2 < /root/maquina1db.sql

Replicación de BD mediante una configuración maestro-esclavo
-------------------

### :pencil2: Pasos seguidos
####Máquina Maestro.
Para la configuración debemos modificar el archivo /etc/mysql/my.cnf, debemos estar en root.
> **Datos a cambiar**

> - #bind-address 127.0.0.1
> - log_error = /var/log/mysql/error.log
> - server-id = 1
> - log_bin = /var/log/mysql/mysql-bin.log

Se guarda el archivo y se reinicia el archivo:

    /etc/init.d/mysql restart
Como la versión de mysql es superior a 5.5 debemos crear un usuario y darle permisos de acceso para la replicación.

    CREATE USER esclavo IDENTIFIED BY 'esclavo';
	GRANT REPLICATION SLAVE ON *.* TO 'esclavo'@'%' IDENTIFIED BY 'esclavo';
	FLUSH PRIVILEGES;
	FLUSH TABLES;
	FLUSH TABLES WITH READ LOCK;
Ahora utilizamos SHOW MASTER STATUS para mostrar los datos de la BD que vamos a replicarlos.

    mysql> show master status;
	+------------------+----------+--------------+------------------+
	| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
	+------------------+----------+--------------+------------------+
	| mysql-bin.000001 |      501 |              |                  |
	+------------------+----------+--------------+------------------+
	1 row in set (0.00 sec)


#### Máquina Esclavo
Para la configuración debemos modificar el archivo /etc/mysql/my.cnf como lo hicimos en la máquina maestro pero con la diferencia de que el server-id=2.
Se guarda el archivo y se reinicia el archivo:

    /etc/init.d/mysql restart
Una vez guardado el archivo debemos abrir mysql y añadir la siguiente sentencia:

    CHANGE MASTER TO MASTER_HOST='192.168.1.117',
	MASTER_USER='esclavo', MASTER_PASSWORD='esclavo',
	MASTER_LOG_FILE='mysql-bin.000001', MASTER_LOG_POS=501,
	MASTER_PORT=3306;

Por último, arrancamos el esclavo y ya está todo listo para replicar los datos que se introduzcan/modifiquen/borren en el servidor maestro:

    START SLAVE;
  Después de esto solo debemos irnos al maestro y ejecutar UNLOCK TABLES;
  Una vez realizado esto debemos irnos al esclavo y ejecutar “SHOW SLAVE STATUS\G” y viendo que el valor de la variable “Seconds_Behind_Master” es distinto de null todo funciona perfectamente
###Imágenes de configuración maestro-esclavo
![enter image description here](https://github.com/cr13/SWAP2015/blob/master/Practicas_Swap/P5/SHOW_MASTER_STATUS.png?raw=true)
![enter image description here](https://github.com/cr13/SWAP2015/blob/master/Practicas_Swap/P5/FUNCIONANDO.png?raw=true)
 
