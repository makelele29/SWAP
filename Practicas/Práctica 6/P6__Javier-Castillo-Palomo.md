Práctica 6
===================
:bust_in_silhouette: Javier Castillo Palomo

:bust_in_silhouette: Cristóbal Rodríquez Reina

----------


Discos en RAID
==================

> **Cuestiones a resolver**

> - Configurar dos discos en RAID 1 (los discos se añadirán a un sistema ya
instalado y funcionando).
> - Hacer pruebas de retirar y añadir un disco y comprobar que el RAID sigue funcionando correctamente.

Configuración del RAID por sofware
-------------------

###:page_facing_up:  Pasos a seguir:
Añadimos dos discos duros a la maquina que vamos a utilizar.
Instalar mdadm:

    sudo apt-get install mdadm
	
Buscamos los nombres de los dos discos duros que nos hemos creado. 

    sudo fdisk -l

Creamos el RAID 1, usando el dispositivo /dev/md0 y le indicamos el número de dispositivos a utilizar, así como sus respectivas rutas.

    sudo mdadm -C /dev/md0 --level=raid1 --raid-devices=2 /dev/sdb /dev/sdc

 Ya creado el RAID vamos a darle formato:

    sudo mkfs /dev/md0

Una vez dado formato pasamos a crear el directorio en el que se montará la unidad del RAID.

    sudo mkdir /datos 
    sudo mount /dev/md0 /datos

Probamos el estado del RAID.

    sudo mdadm --detail /dev/md0
![enter image description here](https://github.com/makelele29/SWAP/blob/master/Practicas/Pr%C3%A1ctica%206/detail.PNG?raw=true)
Automatizado del montaje del dispositivo creado al inicio del sistema
-------------------
Para llevar esto a cabo debemos editar el archivo /etc/fstab y
añadir la siguiente línea:
		
    /dev/md0 /datos ext2 defaults 0 0
   
  Al guardar y reiniciar da error al montar /datos porque automáticamente  el sistema cambia md0 por md127, por lo que tendremos que volver al archivo /etc/fstab y cambiamos md0 por md127 y cuando reiniciamos vemos que no da ningún error y ya estaría montado.

![enter image description here](https://github.com/makelele29/SWAP/blob/master/Practicas/Pr%C3%A1ctica%206/fstab.PNG?raw=true)

Tarea Opcional: Simular un fallo en uno de los discos del RAID
-------------------
###:page_facing_up:  Pasos a seguir:
Usaremos mdadm para simular el fallo.
1º Vamos a simular un fallo en un disco y lo vamos a eliminar dicho disco.
	
    mdadm /dev/md127 --fail /dev/sdb --remove /dev/sdb
 
  ![enter image description here](https://github.com/makelele29/SWAP/blob/master/Practicas/Pr%C3%A1ctica%206/fail.PNG?raw=true)
  
2º Vamos a verificarlo

    mdadm --detail /dev/md127
![enter image description here](https://github.com/makelele29/SWAP/blob/master/Practicas/Pr%C3%A1ctica%206/delete.PNG?raw=true)

3º Volvemos a añadir el disco 

    mdadm --add /dev/md127 /dev/sdb
![enter image description here](https://github.com/makelele29/SWAP/blob/master/Practicas/Pr%C3%A1ctica%206/add.PNG?raw=true)