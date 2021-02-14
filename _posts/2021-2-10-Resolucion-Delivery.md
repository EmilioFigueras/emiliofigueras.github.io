---
layout: post
title: Resolución de Delivery (WriteUp)
published: false
categories: WriteUp
---

Delivery es una máquina Linux de Hack The Box cuya dificultad es “Easy”. Las instrucciones que se utilizan en esta resolución tienen en cuenta que estamos trabajando en un entorno controlado. 

![Delivery 01]({{ site.baseurl }}/images/deliveryhtb01.png)

### Resolución de Delivery (user.txt)

En primer lugar enumeramos los puertos abiertos:

![Delivery 02]({{ site.baseurl }}/images/deliveryhtb02.png)

Para posteriormente realizar un análisis, a través de nmap, de los servicios disponibles en dichos puertos:

![Delivery 03]({{ site.baseurl }}/images/deliveryhtb03.png)

Con este primer escaneo observamos que, además del servicio SSH a través del puerto 22, también tenemos dos servicios webs corriendo a través de los puertos 80 y 8065. Con whatweb podemos obtener un primer acercamiento a cada una de estas webs:

![Delivery 04]({{ site.baseurl }}/images/deliveryhtb04.png)

Si entramos en el código fuente de la página principal observamos que se hace referencia un subdominio (helpedsk.delivery.htb) de esta página principal cuyo dominio sería delivery.htb. 

![Delivery 05]({{ site.baseurl }}/images/deliveryhtb05.png)

Agregamos ambas direcciones al /etc/hosts:

![Delivery 06]({{ site.baseurl }}/images/deliveryhtb06.png)

Ahora tenemos dos cosas importantes:

- En helpdesk.delivery.htb tenemos una plataforma de tickets que nos permite crear nuevos tickets sin necesidad de estar registrados. 
- En delivery.htb:8065 tenemos el servicio online Mattermost con un login y la posibilidad de crearnos una cuenta. 

Si creamos una cuenta se necesita aceptar el email de verificación, pero ese email no nos llegará porque el correo no sale del dominio delivery.htb. Así que para pasar ese paso va a ser necesario hacer uso del portal de tickets.
Creamos un nuevo ticket:

![Delivery 07]({{ site.baseurl }}/images/deliveryhtb07.png)

Y vemos que al crearlo correctamente nos informan de que podemos ver el estado del ticket a través del ticket id que nos dan y que podemos agregar más información al ticket haciendo uso de un email que también nos dan:

![Delivery 08]({{ site.baseurl }}/images/deliveryhtb08.png)

Por lo que vamos a aprovechar este email que nos han proporcionado para crear la cuenta en MatterMost.

Entramos en la comprobación del estado del ticket introduciendo el email que pusimos al crear el ticket y el id del ticket que nos acaban de proporcionar.

Ahora nos vamos a la creación de la cuenta de MatterMost e introducimos el email para ampliar la información del ticket para la creación de la cuenta:

![Delivery 09]({{ site.baseurl }}/images/deliveryhtb09.png)

Una vez creada la cuenta actualizamos la información en la web del estado del ticket y tendremos ahí el enlace para verificar la cuenta recién creada:

![Delivery 10]({{ site.baseurl }}/images/deliveryhtb10.png)

Accediendo a este enlace podremos verificar la cuenta e introduciendo la contraseña podremos acceder a MatterMost. 

Una vez dentro, nos saltamos el tutorial y directamente vemos unas credenciales:

![Delivery 11]({{ site.baseurl }}/images/deliveryhtb11.png)

Con estas credenciales podremos acceder por SSH y, directamente, ver la flag de user.txt:

![Delivery 12]({{ site.baseurl }}/images/deliveryhtb12.png)

### Resolución de Delivery (root.txt)

Vamos a investigar la información de configuración de MatterMost en busca de más credenciales. Para ellos debemos localizar dónde se encuentra dicha información. Usando el siguiente comando buscamos rutas de carpetas que contengan mattermost en su nombre y almacenamos dichas rutas en un fichero de texto llamado mm.txt:

    find / -name “*mattermost*” > mm.txt

La ruta localizada es “/opt/mattermost”:

![Delivery 13]({{ site.baseurl }}/images/deliveryhtb13.png)

El archivo de configuración está situado en la ruta /opt/mattermost/config/config.json. Investigamos este archivo en busca de información sensible. En el bloque de “SqlSettings” encontramos lo que podría ser unas credenciales para la base de datos:

![Delivery 14]({{ site.baseurl }}/images/deliveryhtb14.png)

Vemos que utiliza MySQL, por lo que nos intentamos conectar con dichas credenciales y vemos que funcionan:

![Delivery 15]({{ site.baseurl }}/images/deliveryhtb15.png)

Entramos en la base de datos mattermost con “use mattermost” y listamos todas las tablas con “show tables”. Hay varias tablas que podría contener información sensible, pero nos fijamos en la tabla “Users”, mostramos su contenido:

![Delivery 16]({{ site.baseurl }}/images/deliveryhtb16.png)

Hay múltiples usuarios pero nos quedamos con los campos “Username” y “Password”, observando que hay un usuario llamado “root”, por lo que nos quedaremos con la contraseña de ese usuario:

![Delivery 17]({{ site.baseurl }}/images/deliveryhtb17.png)

Con una herramienta online de detección de hashes verificamos que el algoritmo de encriptación utilizado es bcrypt:

![Delivery 18]({{ site.baseurl }}/images/deliveryhtb18.png)

Para desencriptar esta contraseña debemos usar una información que viene en la pantalla de inicio tras loguearnos en MatterMost:

![Delivery 19]({{ site.baseurl }}/images/deliveryhtb19.png)

Ahí nos indican que la contraseña es una variante de “PleaseSubscribe!”, por lo que el objetivo es usar un diccionario de variantes de esta contraseña y aplicarlo al hash haciendo uso de hashcat.

Primero buscamos qué código emplea hashcat para la encriptación bcrypt:

![Delivery 20]({{ site.baseurl }}/images/deliveryhtb20.png)

El comando que utilizaré en hashcat es el siguiente:

    hashcat -m 3200 -a 3 pass.hash ‘PleaseSubscribe!?d?d?d’ -i


![Delivery 21]({{ site.baseurl }}/images/deliveryhtb21.png)
Donde 3200 es el código de bcrypt, -a 3 es el modo de ataque (fuerza bruta), pass.hash es el fichero donde se almacena el hash de root obtenido, y ‘PleaseSubscribe!?d?d?d’ -i lo que hace es generar contraseñas desde ‘PleaseSubscribe!1’ hasta ‘PleaseSubscribe!999’ de forma iterativa. 

Una vez terminado el proceso ejecutamos el comando --show en Hashcat y vemos que la contraseña ha sido desencriptada:

![Delivery 22]({{ site.baseurl }}/images/deliveryhtb22.png)

Con esta contraseña que tenemos ya podemos loguearnos como root desde el usuario maildeliverer y obtener la flag de root.txt:

![Delivery 23]({{ site.baseurl }}/images/deliveryhtb23.png)


