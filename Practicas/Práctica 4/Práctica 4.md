Práctica 4
===================
:bust_in_silhouette: Javier Castillo Palomo


:bust_in_silhouette: Cristóbal Rodríquez Reina

----------


<i class="icon-wrench"></i>Comprobar el rendimiento de servidores web.
==================

> **Observaciones**

> - Para esta prueba hemos utilizado el archivo **f.php** pero con un bucle más reducido para que no tardara más de un minuto en terminar.
> - También hemos realizado un **script** para ejecutar y almacenar las 10 ejecuciones.
> - Los datos no son del todo exactos, esto es debido a que estamos haciendo pruebas en un entorno virtual.

Apache Benchmark
-------------------

#### <i class="icon-file"></i> Script

    #!/bin/bash
	for ((i = 0 ;  i <= 9;  i++))
	do
		ab -n 1000 -c "ip del balanceador o ip de un servidor"/f.php >> ab.txt
		
	done

#### <i class="icon-picture"></i> Gráficas

![Apache Benchmarck](https://github.com/makelele29/SWAP/blob/master/Practicas/Pr%C3%A1ctica%204/AB.JPG?raw=true)

#### <i class="icon-edit"></i> Conclusiones

En el time taken y en el Time per request en los balanceadores se reduce bastante lo que es normal por repartir la carga.
En cambio en el failed request y el request per second toma valores un tanto raros.

Httpref
-------------------

#### <i class="icon-file"></i> Script

    #!/bin/bash
	for ((i = 0 ;  i <= 9;  i++))
	do
		httperf --server "ip del balanceador o ip de un servidor" --port 80 --uri /f.php --rate 150 --num-conn 27000 --num-call 1 --timeout 5 >> httperf.txt
	
	done

#### <i class="icon-picture"></i> Gráficas

![Httpref]()

#### <i class="icon-edit"></i> Conclusiones



OpenWebLoad
-------------------

#### <i class="icon-file"></i> Script

    #!/bin/bash
	for ((i = 0 ;  i <= 9;  i++))
	do
		echo -ne '\n' | sleep 30s |openload "ip del balanceador o ip de un servidor"/f.php 10 >> OpenWebLoad.txt
		
	done

#### <i class="icon-picture"></i> Gráficas

![OpenWebLoad](https://github.com/makelele29/SWAP/blob/master/Practicas/Pr%C3%A1ctica%204/openWebLoad.JPG?raw=true)

#### <i class="icon-edit"></i> Conclusiones

Aqui casi todas las gráficas tienen algo de sentido ya que en el total TPS es normal que los cargadores tarden más que el servidores solo y en los otros dos que los balanceadores tarden menos.
