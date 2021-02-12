---
layout: post
title: SonarQube - Evaluación de código fuente
categories: Herramienta
---

SonarQube es una plataforma de software libre para evaluar código fuente. En este artículo se describe brevemente su instalación y se hará uso de esta herramienta sobre un par de proyectos para verla en funcionamiento.

### Instalación

Para realizar esta práctica he utilizado el Sistema Operativo Debian 10.

La instalación fue sencilla. Simplemente hay que descargar de la página web la aplicación y ejecutar el comando start en el script sonar.sh situado en la carpeta de linux dentro de la carpeta bin del SonarQube.

Una vez arrancado se accede desde el navegador al puerto 9000 del localhost y se tendrá acceso al panel de SonarQube. Desde ahí se podrá crear un nuevo proyecto para analizar. El propio SonarQube nos va dando indicaciones de lo que hay que hacer para proceder a analizar el código.

En mi caso, ha sido necesario descargar el SonarScanner y configurarlo tal como indica en su página web. Una vez configurado, se ejecuta desde la raíz del proyecto el comando sonar-scanner pasándole algunos datos como la ‘key’ del proyecto, el login o la url donde está alojado el SonarQube. Esto arrancará el análisis y los resultados estarán disponible en el panel de SonarQube.


### Introducción al análisis realizado

He utilizado mi Trabajo de Fin de Grado el cual está compuesto por dos proyectos, una aplicación web hecha con CodeIgniter y un script en Python. Tras la primera pasada del SonarScanner por ambas carpetas del proyecto obtengo los siguientes resultados en el SonarQube:

![SonarQube 01]({{ site.baseurl }}/images/sonarqube01.png)

Pasaré a explicar con más profundidad los resultados de ambos proyectos.

### Script Python

Al SonarQube le pasé a escanear todo el proyecto subido a mi GitHub personal, el cual contaba principalmente con tres archivos, que son el código python final (WID.py), un código anterior del script (TFG.py) y una implementación del framework unittest para realizar pruebas de funcionamiento sobre el código. Para la calidad del código también utilicé Pylint, por eso se notarán las diferencias entre la primera vez que le paso el SonarScanner y la siguiente, después de eliminar el script antiguo (TFG.py).

Al pasar por primera vez obtengo esto:

![SonarQube 02]({{ site.baseurl }}/images/sonarqube02.png)

En primer lugar tenemos las 74 **‘security hotspots’**, que no son necesariamente problemas sino líneas que conviene revisar para verificar que todo está correcto. Entro en su verificación y veo que hay dos indicaciones, pero mayoritariamente es uno el problema indicado:

![SonarQube 03]({{ site.baseurl }}/images/sonarqube03.png)

La advertencia más indicada en los **‘security hotspots’** es la que me advierte de que el código obtiene datos desde los argumentos de la línea de comando, algo que en el caso de este proyecto es intencionado ya que es un script que se ejecuta desde la línea de comandos y tiene sus opciones correspondientes. El resto de advertencias eran sobre el uso de ‘http’ en lugar de ‘https’, en este caso el script acepta ambas opciones y también es algo intencionado.

A continuación nos vamos a las **‘Code smell’**, que nos indica correcciones necesarias para obtener un código más limpio, es decir, una mayor calidad de código. La mayor cantidad de fallos en este aspecto estaba en el código antiguo (TFG.py) que tenía de antes de pasarle el framework Pylint, pero aún así, el SonarQube mi realizó también indicaciones que llevé a cabo:

![SonarQube 04]({{ site.baseurl }}/images/sonarqube04.png)

Algunas de las correcciones más frecuentes han sido el renombre de variables o de funciones, por el tema de mayúsculas y guiones bajos, y la eliminación de código comentado. También he corregido algún grupo de ‘if’ que podía agruparse en un único if:

![SonarQube 05]({{ site.baseurl }}/images/sonarqube05.png)

Después de eliminar del proyecto el script antiguo y de realizar diversas correcciones he vuelto a pasar el escáner y el resultado ha sido el siguiente:

![SonarQube 06]({{ site.baseurl }}/images/sonarqube06.png)

Donde los 4 ‘Code Smells’ son iguales:

![SonarQube 07]({{ site.baseurl }}/images/sonarqube07.png)

Este error lo que nos indica es que hay en una misma función muchos ‘if’, y se debería agrupar en funciones independientes para facilitar su lectura. El script, por el manejo de errores y por la funcionalidad de modo ‘verbose’ con la que cuenta utiliza muchísimos if, un total de 84 hay en el código, por lo que es entendible esta advertencia.

Por último tenemos el 5.3% de **código duplicado**. Todo el código duplicado aparece en la clase de test dentro del propio código Pyhton para hacer uso del framework unittest. Vemos cómo en determinadas funciones de esta clase al inicializar ciertos parámetros la inicialización se repite en cuanto a código:

![SonarQube 08]({{ site.baseurl }}/images/sonarqube08.png)

El SonarQube nos indica las diferentes partes del código donde se repite, para poder ir eliminándolo y dejar una única inicialización.

### Aplicación Web

La aplicación web está hecha con CodeIgniter y utiliza diversos elementos de terceros, como el tema visual o librerías (PHPExcel por ejemplo), y esto será muy determinante en el resultado obtenido. Al pasar el escáner obtengo el siguiente resultado:

![SonarQube 09]({{ site.baseurl }}/images/sonarqube09.png)

Entro a analizar el resultado obtenido. En primer lugar entro en los aproximadamente 2700 **bugs**:

![SonarQube 10]({{ site.baseurl }}/images/sonarqube10.png)

Observo que la primer ristra de bugs que muestra SonarQube se debe a los ficheros ‘index.html’ que trae por defecto CodeIgniter en la mayoría de sus carpetas para mostrar en determinados lugares un HTML con el mensaje ‘403 Forbidden’. Observo también que **SonarQube etiqueta todos los mensajes**, y estos mensajes están etiquetados con el tag ‘wcag2-a’ el cual significa Web Content Accessibility Guidelines 2.0, es decir, pautas de accesibilidad para el contenido web. Por lo que entro en esta etiqueta y obtengo lo siguiente:

![SonarQube 11]({{ site.baseurl }}/images/sonarqube11.png)

Es decir, veo que hay 2363 bugs de este tipo, el cual son indicaciones de código HTML al cual le faltarían definir atributos en las etiquetas.

Volviendo a la imagen anterior, la de los 2700 bugs, puedo observar más información interesante. Por ejemplo, arriba a la derecha **SonarQube nos hace una estimación del tiempo de resolución** de los problemas indicados, en este caso serían 9 días. Y otro filtro muy interesante, además del ya visto de las etiquetas, es el de **‘Severity’, que nos indica la gravedad asignada a los bugs**, la cual puede ser, de menos a mayor importante, ‘Info’, ‘Minor’, ‘Major’, ‘Critical’ y ‘Blocker’.

Procedo a seleccionar los bugs de gravedad ‘Blocker’ y ‘Critical’:

![SonarQube 12]({{ site.baseurl }}/images/sonarqube12.png)

Procedo a corregir dichos bugs:

- El primero es un fallo en el CSS donde aparece ‘positon’ en lugar de ‘position’. Es decir, está mal escrito y fallaría.
- El segundo bug es en el mismo CSS anterior, en el que se da valores a ‘margin-left’ primero y después a ‘margin’, y debería ser al revés.
- Y por último hay una función de JavaScript que al ser llamada se le envía más parámetros de los que tiene definido.

Corrijo estos tres bugs y paso nuevamente el escáner para comprobar que han desaparecido, y vemos que ya no aparece ningún bug con severidad tipo ‘Blocker’ o ‘Critical’:

![SonarQube 13]({{ site.baseurl }}/images/sonarqube13.png)

Además podemos comprobar como la puntuación en aspectos de **‘Fiabilidad’** ha pasado de ser ‘E’ a ser ‘C’:

![SonarQube 14]({{ site.baseurl }}/images/sonarqube14.png)

Ahora pasamos a las vulnerabilidades, que en la pantalla inicial hemos visto que tenemos 11 y tienen una calificación general de ‘E’. Al entrar vemos las siguientes vulnerabilidades:

![SonarQube 15]({{ site.baseurl }}/images/sonarqube15.png)

Principalmente son vulnerabilidades que afectan a código externo, es decir, a código del propio CodeIgniter o de la librería PHPExcel, excepto una. La vulnerabilidad más frecuente es la de almacenar contraseñas en variables llamada ‘password’. Aunque también hay otras vulnerabilidades relacionadas con la encriptación, con protocolos, o la de agregar rel="noopener noreferrer" a la etiqueta HTML cuando queramos abrir una nueva ventana para evitar que la ventana abierta no modifique la vista anterior.

Después de solucionar estas vulnerabilidades procedo a ejecutar el escáner y el resultado es el siguiente:

![SonarQube 16]({{ site.baseurl }}/images/sonarqube16.png)

Podemos observar como ya no encuentra ninguna vulnerabilidad y la seguridad la puntúa como ‘A’.

Del resto de elementos creo que se han visto más claramente con el proyecto anterior. Remarcar que la duración para corregir las 7400 ‘Code Smells’ es de 175 días, y que de las 7400 hay marcadas 14 como ‘Blocker’ y las 14 son del código de CodeIgniter de variables que habría que eliminar.

### Conclusión

Me parece que SonarQube es una herramienta de gran utilidad para ir usando durante la realización de proyectos y asegurarse de que se mantienen buenas prácticas durante su desarrollo así como de que no se cometen errores de gravedad.

Además, en el caso concreto analizado aquí, se ve el peligro que puede conllevar el agregar códigos de terceros a un proyecto. También resulta de interés combinar SonarQube con otras herramientas como ocurre con el primer proyecto mostrado, el de Python, en el cual utilicé Pylint para analizar el código, y unittest para realizar otro tipo de pruebas.

