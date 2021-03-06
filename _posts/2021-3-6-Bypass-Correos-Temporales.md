---
layout: post
title: Bypass uso de correos temporales
categories: Ataque
---

Existen múltiples webs que nos proporcionan correos temporales que son de utilidad cuando queremos registrarnos en sitios a los que solo queremos echarles un vistazo u obtener una información concreta y puntual. El inconveniente es que, debido a la popularidad de estas webs, **en muchos sitios ya están restringiendo los dominios de estos correos temporales. Aquí vamos a ver una forma de saltarnos esta prohibición.** 

### Qué son los correos temporales

Los correos temporales son **páginas webs que nos ofrecen un email temporal o desechable, es decir, una dirección y buzón de correo electrónico con una fecha de caducidad y que se genera sin registro. Estos correos, después de un determinado tiempo, dejarán de existir. **

Hay diferentes escenarios donde nos puede ser de utilidad utilizar un correo temporal, como por ejemplo:

- Poder acceder a un foro.
- Acceder a un enlace de descarga o prueba de un servicio puntual. 
- Acceder a una web o recurso de forma puntual. 
- En general, tener que recibir una información muy puntual y no querer revelar tu correo personal. 

Hay muchas webs de correos temporales, como por ejemplo Mailinator, GuerrillaMail, 10minutemail, correotemporal, temp-email, emailondeck… 

### Restricción

Los correos temporales se han vuelto más populares con el paso del tiempo. Ello ha hecho que sea muy común encontrarnos con webs, desde Facebook hasta páginas menos populares, que **restringen el uso de determinados dominios de correos, por lo que nos imposibilitan poder utilizar correos temporales** para incitarnos a utilizar nuestro correo personal. 

### ByPass

¿Cómo saltarse esta prohibición en esas webs? Pues la idea será el **utilizar ese correo que nos proporciona el email temporal pero cambiarle el dominio por alguno menos común o más fiable.** Para el ejemplo utilizaré por un lado el correo temporal de guerrillamail.com y para probar el funcionamiento enviaré un correo a la dirección con el nuevo dominio. 

En primer lugar elegiremos un servicio de correos temporales, en este ejemplo utilizaré Guerrilla Mail, y **averiguaremos a través de qué dominio gestiona el email utilizando el comando ‘host’:**

![ByPass Correo 01]({{ site.baseurl }}/images/bypassCorreo01.png)

Como podemos observar, en el caso de Guerrilla Mail utiliza para gestionar los correos mail.guerrillamail.com, mientras que en el caso de Mailinator utiliza mail.mailinator.com y mail2.mailinator.com. 

Ahora nos creamos un buzón en Guerrilla Mail:

![ByPass Correo 02]({{ site.baseurl }}/images/bypassCorreo02.png)

El objetivo para bypassear las restricciones es poder cambiar el dominio guerrillamail.org por otro menos conocido. Para ello, **vamos a buscar a través de Shodan un servidor que tenga configurado el servicio de correos de Guerrilla Mail como servidor de correos.** 

![ByPass Correo 03]({{ site.baseurl }}/images/bypassCorreo03.png)

Encontramos un servidor que tiene esta configuración específica. **Ahora tenemos que encontrar un dominio que apunte a este servidor**, que mirando la información que nos ofrece Shodan de este servidor concreto ya podemos encontrarlo:

![ByPass Correo 04]({{ site.baseurl }}/images/bypassCorreo04.png)

Vemos que la configuración SMTP es la que buscamos, y que además hay un servicio web por el puerto 80 al que se accede a través del dominio pokemail. Esto significa que podemos utilizar pokemail en lugar de guerrillamail. Así que **escribimos un correo a esta nueva dirección, es decir, a la dirección que nos proporciona Guerrilla Mail cambiándole el dominio:**

![ByPass Correo 05]({{ site.baseurl }}/images/bypassCorreo05.png)

Y como podemos comprobar en la última imágen, **recibimos el correo correctamente en nuestro buzón de Guerrilla Mail** habiendo utilizado otro dominio:

![ByPass Correo 06]({{ site.baseurl }}/images/bypassCorreo06.png)

Hay algunos servidores de correos temporales que aparecen más y otros que aparecen menos, o no aparecen, en las búsquedas de Shodan, eso ya es cuestión del momento y del servicio concreto, por ejemplo, en mailinator suele dar mejores resultados.

Utilizando este “nuevo” correo ya podríamos registrarnos en webs que con la dirección de Guerrilla Mail no nos dejaba.
