---
layout: post
title: Stuxnet - Análisis de malware
categories: Malware
---

En este artículo se va a analizar un volcado de memoria RAM de un equipo infectado por **Stuxnet**, y para ello se hará uso de la herramienta **Volatility.** 

### Stuxnet

Disponemos del volcado de la memoria RAM situado en el archivo stuxnet.vmem.

Sacamos la información del perfil del sistema:

![Stuxnet 01]({{ site.baseurl }}/images/stuxnet01.png)

Podemos comprobar que el perfil que estamos analizando pertenece a WinXPSP3x86 y el huso horario es de -0400.

Ahora observamos los procesos activos:

![Stuxnet 02]({{ site.baseurl }}/images/stuxnet02.png)

A simple vista no hay nada que llame notablemente la atención. Así que pasamos a analizar las conexiones de red:

![Stuxnet 03]({{ site.baseurl }}/images/stuxnet03.png)

Parece que tampoco hay nada por esta parte. Así que vamos a utilizar la funcionalidad malfind de Volatility la cual busca código malicioso dentro de los procesos:

![Stuxnet 04]({{ site.baseurl }}/images/stuxnet04.png)

De aquí obtenemos un listado de procesos con contenido detectado como malicioso, que son csrss.exe, services.exe, svchost.exe, explorer.exe y dos procesos de lsass.exe, tanto el 868 como el 1928, aunque este último aparece en más ocasiones. También nos devuelve los correspondientes volcados de memoria, que podemos analizar con el comando file:

![Stuxnet 05]({{ site.baseurl }}/images/stuxnet05.png)

Podemos ver que algunos ficheros mencionan UPX compressed, una técnica habitual de empaquetado de malware. Ahora habría que analizar a través de VirusTotal la información obtenida. Por ejemplo, vamos a sacar el MD5 de uno de los volcados qué “file” nos indica que ha sido comprimido con UPX:

![Stuxnet 06]({{ site.baseurl }}/images/stuxnet06.png)

![Stuxnet 07]({{ site.baseurl }}/images/stuxnet07.png)

Podemos observar que al meter el MD5 en VirusTotal este nos lo detecta como malware en 59 de las 71 herramientas, y muchas de ellas nos informa de que se trata del Stuxnet.

Ahora ya conocemos los procesos implicados. Podemos volver a ver la lista de procesos para analizar por otro camino alguno de estos procesos. Usamos pstree:

![Stuxnet 08]({{ site.baseurl }}/images/stuxnet08.png)

Vamos a analizar, por ejemplo, el proceso lsass.exe que ya nos informó el malfind sobre él. El pid del lsass que vamos a analizar es 1928 y su proceso padre es el 668 services.exe, que también fue notificado por malfind.

Lo primero que vamos a hacer es un dlllist sobre ambos procesos:

![Stuxnet 09]({{ site.baseurl }}/images/stuxnet09.png)

![Stuxnet 10]({{ site.baseurl }}/images/stuxnet10.png)

Lo que más llama la atención es el “command line” del proceso lsass.exe, el cual está entre comillas y la ruta utiliza doble barra vertical en lugar un una barra simple como suele ser lo habitual.

Ahora entramos en los handles de este proceso:

![Stuxnet 11]({{ site.baseurl }}/images/stuxnet11.png)

Vemos que cuenta con secciones críticas y zonas de exclusión mutua. También nos fijamos en algunos eventos, como WkssvcShutdownEvent2, y en los múltiples ficheros relacionados que tiene.

Lo siguiente que podemos hacer, por ejemplo, es un filescan buscando ficheros lsass:

![Stuxnet 12]({{ site.baseurl }}/images/stuxnet12.png)

De aquí tenemos dos .exe que podemos seleccionar, por lo que le hacemos a uno de ellos un dumpfiles lo cual nos genera un par de archivos, uno img y otro dat:

![Stuxnet 13]({{ site.baseurl }}/images/stuxnet13.png)

Le calculamos el MD5:

![Stuxnet 14]({{ site.baseurl }}/images/stuxnet14.png)

Y comprobamos qué ocurre en VirusTotal al subir el archivo .img:

![Stuxnet 15]({{ site.baseurl }}/images/stuxnet15.png)

Podemos ver que este archivo es detectado como malware por 4 de 68 herramientas.

Otra tarea que podemos realizar es hacer un memdump de este proceso y de su proceso padre:

![Stuxnet 16]({{ site.baseurl }}/images/stuxnet16.png)

Ambos procesos podríamos analizarlo con strings, por ejemplo si analizamos el lsass.exe podemos encontrar una línea que parece una consulta  en las que incluye un uid (WinCCConnect) y un pwd (2WSXcder):

![Stuxnet 17]({{ site.baseurl }}/images/stuxnet17.png)

Pero vamos a sacar los MD5 de cada volcado:

![Stuxnet 18]({{ site.baseurl }}/images/stuxnet18.png)

Y comprobar los resultados que nos devuelve VirusTotal:

![Stuxnet 19]({{ site.baseurl }}/images/stuxnet19.png)

El proceso lsass.exe es detectado como malware por 2 herramientas (Avast y AVG).

![Stuxnet 20]({{ site.baseurl }}/images/stuxnet20.png)

Y exactamente lo mismo ocurre con el proceso padre, el services.exe.

Y para finalizar vamos a sacar la línea temporal de procesos. Para ello sacamos con el Volatility el timeliner, el mftparser y el shellbags. Y posteriormente unimos todo en un único fichero:

![Stuxnet 21]({{ site.baseurl }}/images/stuxnet21.png)

Y por último, conociendo el huso horario, podemos hacer uso de mactime y more para ir viendo la información de la línea temporal:

![Stuxnet 22]({{ site.baseurl }}/images/stuxnet22.png)

![Stuxnet 23]({{ site.baseurl }}/images/stuxnet23.png)

De la cual podríamos filtrar con grep o egrep para buscar información más concreta en la que queramos profundizar.
