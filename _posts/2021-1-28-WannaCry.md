---
layout: post
title: WannaCry - Análisis de malware
categories: Malware
---

En este artículo se va a analizar una imagen de memoria RAM afectada por el malware **Wannacry** y un binario del malware. El análisis de la memoria RAM se hará principalmente haciendo uso de la herramienta **Volatility**.

### WannaCry

El escenario para realizar este análisis es que disponemos de una imagen de la memoria RAM afectada por WannaCry en el archivo llamado malware.raw, y un binario del malware.
Vamos a comenzar analizando el binario. Para ello sacaremos información sobre él con el comando file:

![WannaCry 01]({{ site.baseurl }}/images/wannacry01.png)

Así podemos saber que es un ejecutable que, a priori, no parece estar empaquetado.
Podemos hacer uso de la herramienta rabin2 de radare2, incluida en Kali Linux, para conocer las importaciones, y así saber las funciones que utiliza y comenzar a intuir su funcionamiento e intenciones:

![WannaCry 02]({{ site.baseurl }}/images/wannacry02.png)

![WannaCry 03]({{ site.baseurl }}/images/wannacry03.png)

Por ejemplo podemos ver que hace uso de sockets y de conexiones a Internet, por las funciones socket, connect, send, closesocket y las funciones InternetOpenA o InternetOpenUrlA.
También podemos obtener más información sobre el binario a través del análisis de lo que devuelve el comando strings:

![WannaCry 04]({{ site.baseurl }}/images/wannacry04.png)

De esta forma encontramos cosas interesantes, como por ejemplo aquí podemos ver una URL

![WannaCry 05]({{ site.baseurl }}/images/wannacry05.png)

También vemos una serie de comandos interesantes como el comando de icacls para aumentar los privilegios o el cmd.exe, así como la línea WNcry@2ol7.

![WannaCry 06]({{ site.baseurl }}/images/wannacry06.png)

A partir de aquí, procedemos a realizar el análisis del RAW con la herramienta Volatility.
Lo primero que debemos hacer es conseguir información acerca del sistema que estamos analizando. Para ello, utilizamos el siguiente comando:

![WannaCry 07]({{ site.baseurl }}/images/wannacry07.png)

De aquí vamos a quedarnos con alguna información, por ejemplo, podemos saber a través de Suggested Profile y de Image Type que el sistema utiliza un WinXPS3x86. También nos quedaremos con la información del huso horario, la cual es +0530.
A continuación comprobamos todos los procesos que estaban en ejecución de la siguiente forma:

![WannaCry 08]({{ site.baseurl }}/images/wannacry08.png)

Aquí podemos observar un proceso muy sospechoso llamado @WanaDecryptor@. Vamos a anotar el Pid (identificador del proceso) y el PPid (identificador del proceso padre), los cuales son 740 y 1940 respectivamente.
Vamos a profundizar más en el proceso 1940 y para ello vamos a sacar la información de las horas en la que se comenzaron a ejecutar y finalizaron su ejecución cada proceso relacionado con el 1940. Esta información la vamos a almacenar en un fichero de texto que mostraremos ordenándolo cronológicamente:

![WannaCry 09]({{ site.baseurl }}/images/wannacry09.png)

Ahora vamos a sacar la información relacionada con las librerías DLL de cada proceso. Para ello, aplicaremos la siguiente instrucción primero con el proceso 740 y posteriormente con el proceso 1940:

![WannaCry 10]({{ site.baseurl }}/images/wannacry10.png)

![WannaCry 11]({{ site.baseurl }}/images/wannacry11.png)

A través de algunas librerías concretas, que son menos frecuentes, podemos comenzar a deducir la funcionalidad o intenciones del malware, como ya hicimos con el binario al sacar los ‘imports’. Pero aquí, con el detalle que nos vamos a quedar, será con el “Command line” relacionado con el proceso 1940. Vemos que el proceso tasksche.exe está situado en una carpeta llamada ivecuqmanpnirkt615.
Ahora vamos a proceder a analizar los handles. Primero nos centraremos en el proceso 1940 y veremos los handles de tipo ‘Key’, ‘Mutant’, ‘File’ y ‘Event’:

![WannaCry 12]({{ site.baseurl }}/images/wannacry12.png)

Entre las cosas que podemos observar es que, en el handle 0x34, vuelve a aparecer el directorio ivecuqmanpnirkt615.
Esto mismo lo hacemos también con el proceso 740:

![WannaCry 13]({{ site.baseurl }}/images/wannacry13.png)

Aquí podemos ver que de nuevo aparece el fichero ivecuqmanpnirkt615. Además, en ambos handles obtenemos handles de tipo Key, por lo que los procesos tienen secciones críticas, y hay handles de tipo Mutant, por lo que tenemos las zonas de exclusión mutua. Los de tipo Files son ficheros que debemos prestar atención porque pueden estar siendo generados por el malware, como es el caso del fichero muy sospechoso llamado ivecuqmanpnirkt615.
El siguiente paso que vamos a seguir es comprobar la persistencia. Para ello vamos a investigar en los registros de Windows, concretamente buscaremos en el directorio “Microsoft\Windows\CurrentVersion\Run”:

![WannaCry 14]({{ site.baseurl }}/images/wannacry14.png)

Vemos que un registro situado en software y con valor REG_SZ carga el ejecutable tasksche.exe de la ya conocida carpeta ivecuqmanpnirkt615.
A continuación pasamos a revisar la información relacionada con la red. En primer lugar realizamos a través de volatility los comandos connetions y connscan, pero ninguno nos devuelve información de interés. Así que pasamos a obtener un fichero pcap que podamos analizar con Wireshark. Para esto vamos a hacer uso de bulk_extractor:

![WannaCry 15]({{ site.baseurl }}/images/wannacry15.png)

Podemos comprobar que bulk_extractor nos genera varios archivos, pero nos vamos a centrar en el PCAP que analizaremos con Wireshark. En el análisis de Wireshark no obtenemos demasiada información, pero podemos conocer las IPs y los puertos implicados:

![WannaCry 16]({{ site.baseurl }}/images/wannacry16.png)

Volviendo al Volatility vamos a escanear los ficheros relacionados con el directorio sospechoso encontrado anteriormente, es decir, con ivecuqmanpnirkt615:

![WannaCry 17]({{ site.baseurl }}/images/wannacry17.png)

A partir de esta información tenemos los offset de memoria con los que, con esto, podemos obtener los ficheros .dat y .img relacionados de los ficheros que nos interese. Para ello, realizamos la siguiente función usando los offset:

![WannaCry 18]({{ site.baseurl }}/images/wannacry18.png)

Sobre estos ficheros generados podemos realizarle el comando strings combinándolo con more para ir poco a poco viendo su contenido y descubriendo cosas interesantes:

![WannaCry 18_2]({{ site.baseurl }}/images/wannacry18_2.png)

![WannaCry 19]({{ site.baseurl }}/images/wannacry19.png)

![WannaCry 19_2]({{ site.baseurl }}/images/wannacry19_2.png)

![WannaCry 20]({{ site.baseurl }}/images/wannacry20.png)

Podemos observar información interesante como un posible monedero de criptomonedas, mensajes sobre bitcoin, los mensajes de textos que irán apareciendo en cada momento durante la ejecución del malware, la intención de eliminar todos los puntos de restauración (shadowcopy) del equipo… También observamos que modifica permisos con icacls o una cadena que podría ser una especie de contraseña, WNcry@2ol7.
La línea del monedero de criptomonedas aparecía también en el strings realizado sobre el archivo binario, y junto a esta línea había otras dos más que podían ser también monederos. Para verificar estas líneas podemos hacer uso de Blockchain Explorer. Primero verificaremos el posible monedero aquí obtenido:

![WannaCry 21]({{ site.baseurl }}/images/wannacry21.png)

![WannaCry 22]({{ site.baseurl }}/images/wannacry22.png)

Podemos ver que efectivamente existe tanto como moneder de Bitcoin como de Bitcoin Cash. A partir de aquí podría ser interesante continuar investigando los movimientos para marcar una trazabilidad. Ahora comprobaremos los otros dos monederos obtenidos del análisis del binario:

![WannaCry 23]({{ site.baseurl }}/images/wannacry23.png)

![WannaCry 24]({{ site.baseurl }}/images/wannacry24.png)

El 115p7UMMngoj1pMvkpHijcRdfJNXj6LrLn también existe y, al igual que el otro, tiene monedero de Bitcoin y de Bitcoin Cash. Y lo mismo ocurre con 12t9YDPgwueZ9NyMgw519p7AA8isjr6SMw.

![WannaCry 25]({{ site.baseurl }}/images/wannacry25.png)

![WannaCry 26]({{ site.baseurl }}/images/wannacry26.png)

Volviendo a los ficheros volcados, de estos ficheros dat e img obtenidos también podemos sacar su MD5 para subirlos a VirusTotal:

![WannaCry 27]({{ site.baseurl }}/images/wannacry27.png)

Y estos md5 obtenidos los vamos subiendo a VirusTotal y analizamos la información que este nos devuelve:

![WannaCry 28]({{ site.baseurl }}/images/wannacry28.png)

En VirusTotal observamos que 58 de los 68 analizadores lo detecta como malicioso y muchos nos informa que se trata del Ransomware WannaCryptor. Desde VirusTotal se puede obtener mucha otra información como la fecha de compilación u otra información que ya hemos ido descubriendo anteriormente:

![WannaCry 29]({{ site.baseurl }}/images/wannacry29.png)

A continuación volvemos con Volatility y realizamos un volcado de las secciones de memorias relacionadas con los dos procesos en cuestión:

![WannaCry 30]({{ site.baseurl }}/images/wannacry30.png)

Los ficheros DMP obtenidos podemos pasarlos también por VirusTotal:

![WannaCry 31]({{ site.baseurl }}/images/wannacry31.png)

Aunque en esta ocasión vemos que solo dos herramientas de VirusTotal nos lo detecta como malicioso, siendo Avast y AVG los que nos indica que se trata de WanaCry.
También podemos aplicarle strings a estos ficheros, al igual que hicimos con los anteriores:

![WannaCry 32]({{ site.baseurl }}/images/wannacry32.png)

Y podemos descubrir el uso de Tor y enlaces onion, así como un monedero virtual que ya vimos al analizar el binario y que verificamos su existencia anteriormente. También podemos combinar strings con el comando grep para ir filtrando por información que podemos creer de interés, como lnk, www…:

![WannaCry 33]({{ site.baseurl }}/images/wannacry33.png)

El último paso que vamos a realizar es generar una línea temporal. Para ello debemos generar tres ficheros, que son el timeliner, el mftparser y el shellbags. Comenzamos con el timeliner:

![WannaCry 34]({{ site.baseurl }}/images/wannacry34.png)

Seguimos con el mftparser:

![WannaCry 35]({{ site.baseurl }}/images/wannacry35.png)

Y concluimos con la información sobre las shellbags:

![WannaCry 36]({{ site.baseurl }}/images/wannacry36.png)

Ahora vamos a unir en un único fichero toda esta información obtenida:

![WannaCry 37]({{ site.baseurl }}/images/wannacry37.png)

Finalmente, podemos hacer uso de mactime y egrep sobre el fichero generado para filtrar por los procesos que hemos ido viendo de interés según los análisis anteriores, traducir la hora a un lenguaje entendible y ver todo el proceso relacionado con el malware en orden cronológico:

![WannaCry 38]({{ site.baseurl }}/images/wannacry38.png)

![WannaCry 39]({{ site.baseurl }}/images/wannacry39.png)
