---
layout: post
title: Recorrido completo por el ciberataque a la OPM de EEUU.
categories: Ataque
---

La fuga de información de la Oficina de Gestión de Personal (OPM) fue una **masiva filtración de datos pertenecientes al gobierno federal de Estados Unidos**. El ataque ocurre sobre un sistema de misión crítica. La fuga de datos solo de este ataque ya supone un alto riesgo por su contenido, pero podremos ver como relacionando estos datos filtrados con otros ataques similares se obtiene información aún más relevante sobre las víctimas. También está presente en este caso el ciberespionaje y los ciberataques entre Estados.

Estas es una de las filtraciones más escandalosas relacionadas con el Gobierno de Estados Unidos. La filtración se realiza sobre la OPM. **La OPM es la agencia que proporciona fuerza laboral al gobierno y está a cargo de los recursos humanos, recursos médicos, seguros de vida o jubilación de empleados del Gobierno Federal o jubilados, entre otros.** 

Inicialmente fueron robados datos de 4 millones de personas, pero las investigaciones siguientes ascienden este número hasta las **21.5 millones de personas**. 

La información robada era tanto de trabajadores como personas que han solicitado trabajar. Había información usada para dibujar perfiles psicológicos de las personas, es decir, **datos relacionados con la orientación sexual, religión o drogas, además del número de la seguridad social.** 

Entre los datos robados hay **información de 1.8 millones de personas que no tienen relación directa con la OPM**, pero estaban ahí porque pertenecían al círculo social de algún trabajador o solicitante de trabajo, es decir, esposos, amigos, compañeros de piso…

Muchos de estos datos no estaban encriptados, aunque conociendo como de sofisticado fue el ataque probablemente no hubiese sido relevante que los datos estuviesen encriptados. 

### Objetivos potenciales y motivación

Detrás de este ataque estaba China. No hay confirmación oficial de qué grupo exactamente pero, como podremos ver posteriormente, todo indica que fue el grupo conocido como **Comment Crew (APT-1).** Según el CSIS, China está involucrada en un cuarto de los más importantes ciberataques de 2019. El principal objetivo es el espionaje entre China y Estados Unidos. 

**Con el robo de datos de ciudadanos estadounidenses es posible realizar un perfil de “vulnerabilidades humanas”.** Las “vulnerabilidades humanas” pueden resumirse con el acrónimo MICE (Money, Ideology, Coercion and Ego) y sirve para atraer personas hacia el espionaje. 

Que el espionaje chino conozca la situación financiera de ciudadanos estadounidenses puede suponer un riesgo cuando sean capaces de cruzar datos y conocer sus trabajos, deudas económicas, personas que quieren mejorar su calidad de vida o gastos inmorales con los que puedan ser chantajeados. Toda esta información es obtenida a través de múltiples intrusiones. Por ejemplo, a través del robo del número de la seguridad social y cruzándolo con otras bases de datos robadas se pueden obtener las deudas de las personas, o cruzándolo con la información robado en la filtración de datos de Ashley Madison se podría obtener las infidelidades de la víctima, una **información de utilidad para el chantaje.**

En este caso, además, se robó la información de los círculos sociales de las víctimas, haciendo posible conocer si hay alguna persona vulnerable en el entorno de la víctima objetivo con la que poder chantajearlo, pudiendo ser el objetivo principal un empleado federal o diplomático.

La red social más utilizada para el reclutamiento de espías es LinkedIn, donde hay una gran cantidad de perfiles falsos. 

En cuanto al atacante, a pesar del silencio del Gobierno de Estados Unidos sobre este asunto, **la revista Wired en 2016 cedió la responsabilidad de la filtración a China**. Años más tarde, en 2018, el asesor de seguridad de la Casa Blanca, **John Bolton, dijo que los datos robados estaban en Beijing**. Oficialmente nunca se ha confirmado la autoría del ataque. 

Extraoficialmente, todo indica que el ataque fue liderado por el grupo APT1, conocido como Comment Crew. **APT (Advanced Persistent Threat) es una nomenclatura que se utiliza para definir grupos que llevan a cabo sofisticados ciberataques a largo plazo y normalmente están financiados por algún Estado.** Las APT tienen estrategias y técnicas muy específicas, así como un objetivo al que atacar. El hecho de utilizar una nomenclatura es para facilitar su catalogación, ya que, por ejemplo, APT1 es una amenaza china atribuida a la Unidad 61398 del Ejército Popular de Liberación.

El grupo detrás de APT1 se conoce como Comment Crew o Byzantine Candor, que fue el nombre en clave que le asignó la inteligencia estadounidense. **Según un informe de Mediant de verano de 2016, se estima que este grupo está formado por miles de personas con grandes conocimientos de ciberseguridad y un alto nivel de inglés. Estarían trabajando en unas instalaciones de más de 12000 metros cuadrados equipadas con una infraestructura de fibra óptica independiente ofrecida por China Telecom. El Partido Comunista Chino parece estar apuntando a parte de las fuerzas de su Ejército Popular de Liberación a la lucha contra el ciberespionaje.**

**Se calcula que este grupo ha podido robar cientos de terabytes de información de al menos 141 organizaciones de los 20 sectores más relevantes**, pudiendo realizar robos de forma paralela y simultánea a diversas empresas y gobiernos. 

### ¿Qué ocurrió?

**El 15 de abril de 2015 el ingeniero de seguridad de la OPM Brendan Saulsbury** estaba desencriptando tráfico SSL para ver qué tipo de datos se movían por la red de la agencia. .**En este proceso de monitorización encontró tráfico saliente extraño hacia un dominio aparentemente legítimo e interno de la agencia. Este dominio era opmsecurity.org.** Saulsbury, quien llevaba trabajando mucho tiempo en la agencia, observó que el dominio realmente no pertenecía a la OPM y comenzó a investigar. 

Haciendo uso de un producto de seguridad de Cylance anaĺizó lo que había encontrado para localizar la fuente del tráfico. **El origen estaba en una biblioteca llamada mcutil.dll que está asociada con McAfee.** El punto aquí estaba en que la agencia no tenía ningún producto de McAfee contratado, por lo que esta biblioteca debía estar relacionada con el malware que había filtrado datos. 

Una vez que **el malware** estaba localizado se le aplicó ingeniería inversa para descubrir que este **era una variante de PlugX**, la cual es una herramienta de acceso remoto relacionada con cibercriminales chinos. 

Mientras la investigación se llevaba a cabo, otro analista encontró extraños archivos .rar con contraseñas. Esta era una técnica muy común de filtrado de datos, que se encarga de dividir y comprimir la información antes de filtrarla. 

Después de analizar la red completa se encontraron más de 2000 malware (virus, adware…) y 10 máquinas infectadas por el malware principal. Una de las máquinas infectadas era un “jump server” que daba acceso a la red del servidor, esto convertía la situación en extremadamente grave. 

También se realizó una investigación sobre el dominio **opmsecurity.org. Este dominio estaba registrado desde abril de 2014, lo que indica que el malware podría haber estado en la agencia durante casi un año, y estaba registrado a nombre de Steve Rogers**, personaje de Capitán América. 

Ese mismo año también hubo otros ciberataques relacionados con este. **La compañía privada de seguros de salud Anthem puso en manos de la Fiscalía una brecha de seguridad** con la que le robaron datos de 78 millones de personas. Durante la investigación **se encontró emails en los que se utilizaban personajes de Ironman para pasar la información**, táctica ya conocida y vinculada al grupo Deep Panda (probablemente APT19, pero aún sin confirmar) el cual estaba vinculado con el Gobierno chino. Lo importante de esta investigación es que **se descubrió el dominio opm-learning.com, el cual no tenía vinculación con OPM o Anthem y estaba a nombre de Tony Stark**, personaje de Ironman. 

En estas mismas fechas **se producen importantes intrusiones sobre otras compañías con relaciones laborales con el Gobierno estadounidense**, como CareFirst o Premera. CareFirst, Premera y Anthem son parte del BlueCross BlueShield, que es una federación estadounidense de aseguradoras médicas. Todo esto indica que fue un ataque sofisticado y dirigido. 

Volviendo a la investigación de OPM, **se hizo un seguimiento a los accesos de usuarios**, especialmente los inicios de sesión realizados por una cuenta de una persona que, en el periodo investigado, teóricamente se encontraba de vacaciones. A través de esta información lograron mapear los diferentes movimientos y conocer, más o menos, qué venían haciendo los atacantes en el sistema. Realizaron un marco temporal y **localizaron el primer acceso externo a través de un usuario perteneciente a la compañía KeyPoint Government Solutions, que había reportado una intrusión en diciembre de 2014.**

Curiosamente, **KeyPoint ganó muchos proyectos a costa de la pérdida de contratos por parte de USIS** (US Investigation Services). USIS era un antiguo proveedor de informes de antecedentes penales con el que OPM había cancelado sus vínculos comerciales porque en agosto de 2014 habían sufrido un ciberataque que condujo a sospechas de fraude y otros muchos problemas. 

Es decir, **los cibercriminales entraron en el sistema de USIS pero fueron detectados y se les fastidió el plan cuando rompieron relaciones laborales con OPM. OPM les dió el contrato a KeyPoint pero estos cibercriminales consiguieron entrar en KeyPoint y, a través de un usuario de KeyPoint, lograron llegar a la red de OPM para instalar el malware en el “jump server” que les permitió acceder a la red del servidor.**

Una vez dentro de la red de OPM, **consiguieron realizar una escalada de privilegios con la que obtuvieron una cuenta de administración desde la cual pudieron instalar una variante del malware PlugX** en el “jump server”.

**El 4 de julio de 2014 los cibercriminales comenzaron a preparar los datos para la filtración, comprimiéndolos y moviéndolos.** Es decir, mientras los estadounidenses celebraban el Día de la Independencia cantando el himno nacional y disparando al aire, los chinos iniciaban una de las intrusiones más poderosas en una de las oficinas de seguridad del Gobierno de los Estados Unidos. 

Lo primero en ser filtrado fueron los datos de verificación de antecedentes penales, ya que OPM almacenaba 2 millones de verificaciones de antecedentes penales al año. **También filtraron 18 millones de copias del formulario Estándar 86.**

El formulario Estándar 86 es un cuestionario de 127 páginas con información privada, que va desde finanzas personales hasta abusos de sustancias, atención psiquiátrica, conducta sexual o entrevistas realizadas con detector de mentiras. Toda esta información era debido a los requisitos necesarios para trabajar para el Gobierno. Y la información era tanto de trabajadores como personas que habían intentado trabajar allí. 

También **se robaron 5600000 huellas dactilares de empleados gubernamentales**. Con esta información los atacantes podían saltarse el sistema de acceso biométrico de la compañía. 

Los analistas sabían que, después de la enorme cantidad de información que se había llevado, sacar a los ciberdelincuentes del sistema era una pequeña pero necesaria victoria. **Al principio ellos tocaban lo mínimo posible y solo limitaron el ancho de banda de salida.** Sabían que un paso en falso en la desinfección y desinstalación del malware podría hacer que los atacantes entraran en modo reposo, volviendo a ingresar meses después a través de una “backdoor”. Además, los ciberdelincuentes llevaban más de un año en la agencia, por lo que conocían bastante bien a la empresa. 

A finales de abril, aprovechando que se iban a realizar unos cortes de luz en el edificio para renovar el sistema eléctrico, **se aprovechó ese momento sin vigilancia externa de los atacantes para expulsarlos definitivamente.**

### Vector de ataque, tecnología y herramientas utilizadas

Dentro de la secuencia de eventos que causaron la brecha de información en OPM el primer paso fue robar las credenciales de acceso de KeyPoint Government Solutions, y para ello realizaron una **campaña de phishing.**

Las credenciales de acceso de KeyPoint les daba acceso no administrativo sobre la red de OPM, por lo que realizaron una **escalada de privilegios en Active Directory.**

A través del **software Mimikatz** explotaron una vulnerabilidad en el LSASS de los Windows Servers para obtener un volcado de memoria de todas las credenciales de los usuarios logueados. Entre las credenciales obtenidas estaban las de usuarios con acceso de administrador. Las contraseñas obtenidas no estaban en texto plano sino que eran hashes NTLM. 

**Se explotó otra vulnerabilidad de Windows Server** haciendo uso de los nombres de usuarios y los hashes NTLM de cuentas de administrador. El protocolo de autenticación NTLM no verificaba la contraseña en texto plano, por lo que era posible autenticarse haciendo uso del nombre de usuario en texto plano y la contraseña como un hash NTLM. Este método se conoce como **ataque Pass-the-Hash** y se utilizó para obtener acceso de administrador en el sistema. 

El último paso, ya con acceso de administrador, fue la **instalación de un troyano de acceso remoto, concretamente una variante del malware PlugX.** Este malware se encargaba de enviar las filtraciones al exterior y funcionaba escondiéndose bajo el nombre de mcutil.dll, una biblioteca del software de McAfee. 

PlugX es un troyano de acceso remoto con funciones como la carga, descarga y modificación de archivos, registro de pulsaciones de teclas, control de cámara web o acceso a una shell remota. En esta ocasión fue utilizado para navegar por los sistemas de OPM y comprimir y filtrar datos. 

**Otra herramienta que también instalaron fue Sakula, que es también un RAT (Remote Access Trojan) que abre una “backdoor” y descarga archivos potencialmente maliciosos en el equipo comprometido.** En 2017, Yu Pingan, profesor de la Escuela Comercial de Shanghai, fue arrestado en relación con los ciberataques de OPM y Anthem. No se pudo demostrar ninguna conexión con estos ciberataques pero fue condenado por conspiración contra empresas estadounidenses por ser el desarrollador de Sakula y haber utilizado esta herramienta para otros tres ciberataques.

### Prevención, detección y respuesta

Existen medidas preventivas que podrían haber dificultado el ataque. Por ejemplo, **tanto OPM como KeyPoint no hacían uso de un segundo factor de autenticación.** OPM comenzó usando un segundo factor de autenticación en 2015, después del ataque. Esto podría haber dificultado el acceso a las cuentas de usuario de la empresa. 

Otro grave error relacionado con las credenciales fue que **KeyPoint no eliminó ni modificó las credenciales de acceso de sus usuarios después de la intrusión de diciembre de 2014.** Por lo tanto, debería haber un plan de respuesta a ciberataques que incluya la modificación de todas las credenciales de acceso después de sufrir una intrusión. 

El proceso de detección de amenazas tampoco fue demasiado bueno, ya que la detección se debió a la casualidad de que el ingeniero de seguridad conocía la empresa y pudo darse cuenta de que el dominio opmsecurity.org realmente no pertenecía a OPM, por lo que el tráfico hacia ella era sospechoso.

**Un buen trabajo de ciberinteligencia podría haber vinculado los sitios web opmsecurity.org a nombre de Steve Rogers y opm-learning.org a nombre de Tony Stark a través del cruce de datos para comenzar una investigación.** Teniendo en cuenta que afectaba a OPM, es decir, al Gobierno de Estados Unidos, se podría haber esperado un mejor proceso de detección de amenazas.

La respuesta fue claramente tardía pero, al menos, fue efectiva. Se logró controlar el ciberataque hasta que, llegado el momento y aprovechando la oportunidad, fueron expulsados del sistema. 

Para prevenir y neutralizar este tipo de ciberataques están los Blue Teams, equipos de defensas, y en concreto los campos de Threat Detection y de Threat Hunting, además de los datos obtenidos por la ciberinteligencia. Para realizar estas tareas existen herramientas como ATT&CK de MITER. ATT&CK es un framework de conocimiento que se encarga de centralizar las tácticas y técnicas de ciberataques reales, ofreciendo una normalización de todos los grupos cibercriminales activos para relacionar determinados patrones de comportamiento con ciberataques y ATP ya conocidos.

### Conclusiones

Este caso sirve para ver la **importancia del ciberespionaje para los Estados.** Obtener información privada sobre ciudadanos es de una gran relevancia para poder manipularlos.

**Los Estados se gastan millones de dólares en espionaje y contraespionaje**, pero aún así hemos visto cómo las empresas encargadas del almacenamiento de datos altamente sensibles tenían deficiencias extremadamente graves en sus sistemas informáticos, al tener equipos inseguros y obsoletos en la red de una empresa gubernamental.

Los protocolos de actuación tras un ciberataque son fundamentales para garantizar la expulsión absoluta de los ciberdelincuentes. **Proteger las credenciales de acceso de todos los usuarios haciendo uso de múltiples factores de autenticación es vital para garantizar la seguridad.** Una simple contraseña de acceso no garantiza la seguridad. 

También existe una tendencia a sobrevalorar los sistemas biométricos como forma de autorización, seguramente debido a que es un sistema muy sencillo y cómodo para el usuario. **Pero hemos visto cómo el robo de millones de huellas dactilares destruyó por completo la seguridad de todo el sistema biométrico de la empresa**, ya que la huella dactilar de los usuarios no puede ser modificada. 

### Referencias

Conference by Marta López Pardal at C0r0n4Con.

[Inside the OPM Hack, the Cyberattack That Shocked the US Government](https://www.wired.com/2016/10/inside-cyberattack-shocked-us-government/)

[APT1: Exposing One of China's Cyber Espionage Units | Mandiant](https://www.fireeye.com/content/dam/fireeye-www/services/pdfs/mandiant-apt1-report.pdf)

[Motives for spying](https://en.wikipedia.org/wiki/Motives_for_spying)

[¿Qué es una APT? – Kaspersky Daily | Blog oficial de Kaspersky](https://www.kaspersky.es/blog/que-es-una-apt/966/)

[The OPM hack explained: Bad security practices meet China's Captain America](https://www.csoonline.com/article/3318238/the-opm-hack-explained-bad-security-practices-meet-chinas-captain-america.html)

[OPM Data Breach](https://asamborski.github.io/cs558_s17_blog/2017/04/20/opm.html)

[Take a Deep Dive into PlugX Malware](https://logrhythm.com/blog/deep-dive-into-plugx-malware/)

[APT1, Comment Crew, Comment Group, Comment Panda, Group G0006 | MITRE ATT&CK](https://attack.mitre.org/groups/G0006/)

[Chinese man behind US hacks back to teaching computing in Shanghai](https://www.scmp.com/news/world/united-states-canada/article/3043451/chinese-man-behind-us-hacks-back-teaching-computing)

[Bolton Confirms China was Behind OPM Data Breaches](https://www.fedsmith.com/2018/09/21/bolton-confirms-china-behind-opm-data-breaches/)

[Chinese Military Group Linked to Hacks of More Than 100 Companies](https://www.wired.com/2013/02/chinese-army-linked-to-hacks/)

[OPM Terminates Controversial Background Check Contractor](https://www.govexec.com/management/2014/09/opm-terminates-controversial-background-check-contractor/93680/)

[Hackers Took Fingerprints of 5.6 Million U.S. Workers, Government Says](https://www.nytimes.com/2015/09/24/world/asia/hackers-took-fingerprints-of-5-6-million-us-workers-government-says.html)

[QUESTIONNAIRE FOR NATIONAL SECURITY POSITIONS](https://www.opm.gov/forms/pdf_fill/sf86-non508.pdf)
