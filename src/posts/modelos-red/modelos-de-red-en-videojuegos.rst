.. title: Modelos de red en videojuegos
.. slug: modelos-de-red-en-videojuegos
.. date: 2015-03-25 11:38:59 UTC+01:00
.. tags: networking
.. link:
.. description:
.. type: text

Este post pretende explicar de forma breve y concisa los diferentes modelos de red, así como las técnicas utilizadas para reducir los efectos de la latencia en los videojuegos actuales.

.. TEASER_END: click to read the rest of the article


**************
Juegos en red
**************

Utilizar la red para comunicar las diferentes instancias de un
videojuego implica una limitación física insalvable: la `latencia
<http://es.wikipedia.org/wiki/Latencia>`_. Esto es especialmente grave
cuando esa red es Internet, ya que involucra computadores que pueden
estar repartidos en cualquier parte del planeta. Eso significa que la
latencia será muy diferente entre dos jugadores cualesquiera debido a
que la calidad y características concretas de su conexión a la red
pueden ser muy diferentes.

La latencia afecta muy negativamente a la experiencia de juego debido
a la propia naturaleza de un videojuego. Un videojuego es un programa
que en esencia está formado por un bucle que ejecuta una serie de
instrucciones que permiten al jugador interaccionar con la lógica. La
frecuencia del bucle de juego es de al menos 30 iteraciones por
segundo. Esto supone que cada frame se genera cada 34 milisegundos
aproximadamente. Para poder renderizar la escena, se requiere
información de los demás jugadores, de modo que un retraso en la
recepción de dicha información supone un empeoramiento de la calidad
del juego.  En cualquier juego en red, existen dos propiedades clave
relacionadas con la latencia: la consistencia y la inmediatez[1].

La consistencia es un indicador de las diferencias entre la percepción
que cada jugador tiene del *mundo* del juego. Si un jugador recibe un
mensaje de actualización en un instante diferente a la de otro
jugador, obtendrán una representación diferente del entorno durante
ese intervalo. De modo que, a mayor latencia menos consistencia. Por
ejemplo, en un juego de carreras, es común que la posición dentro del
circuito de un determinado coche no corresponda exactamente para el
jugador que lo pilota que para los contrincantes. Esto puede tener
implicaciones negativas sobre la jugabilidad.

Por otro lado, la inmediatez mide el tiempo que le lleva al juego
representar las acciones de los usuarios. Por ejemplo, en un juego FPS
(First Person Shooter), el jugador espera oír inmediatamente el sonido
del disparo de su propia arma. Si el juego necesita confirmar que la
acción del jugador y sus consecuencia, tendrá que esperar una
validación (remota) y, por tanto, el resultado puede perder mucho
realismo. Como esa comprobación implica comunicación con otras
instancias del juego, una mayor latencia reduce la inmediatez.

Además, aunque es lo deseable, no es posible maximizar a la vez
consistencia e inmediatez. Maximizar la consistencia implica comprobar
todas las acciones y eso reduce la inmediatez. Al mismo tiempo,
aumentar la inmediatez implica crear potenciales inconsistencias. En
función del tipo de juego, será más adecuado sacrificar una sobre la
otra.

====================
Modelo peer-to-peer
====================

El `primer modelo de red utilizado
<http://gafferongames.com/networking-for-game-programmers/what-every-programmer-needs-to-know-about-game-networking>`_
en los juegos multijugador estaba basado en el modelo
peer-to-peer. Este modelo se caracteriza por no tener ningún elemento
central, sino que cada nodo de la red sirve y solicita información a
los demás. Aunque todavía hoy día es posible encontrar juegos que usan
este modelo de red , presenta una serie de problemas que desaconsejan
su uso en la mayoría de juegos actuales. Actualmente el modelo
Peer-to-Peer es usado principalmente en juegos de estrategia en tiempo
real(RTS). Como ejemplo representativo, podemos nombrar Starcraft I.

En este modelo, cada par ejecuta una instancia del juego y envía el
estado de su *mundo* a todos los demás pares. Para poder renderizar la
escena, antes todos los pares deben recibir los mensajes de los
demás. La latencia de todos los jugadores tiene un impacto
determinante en la partida, ya que el jugador con mayor latencia
tardará mas en recibir los mensajes de los demás jugadores y en enviar
los suyos, retrasando la ejecución del juego para todos los
participantes; es decir, la latencia de los jugadores tiende a
igualarse a la del jugador más «lejano». Esto afecta negativamente a
la inmediatez, lo que no es aceptable en juegos donde ésta sea una
propiedad básica (tipo FPS). Por otro lado, este modelo tiene
dificultades para mantener la consistencia de una partida, ya que
comunicar el estado entre todos los jugadores consume mucho ancho de
banda, al requerir un gran número de mensajes.

========================
Modelo cliente servidor
========================

Un modelo de red cliente-servidor se caracteriza por tener una entidad
central, llamada servidor, que se encarga de enviar información a los
demás participantes en la partida, normalmente llamados clientes, que
se la solicitan. Este modelo reduce el problema de la latencia debido
a que ahora la latencia para un cliente dado solo depende de su
conexión con el servidor, no viéndose afectado (al menos no
directamente) por la latencia de los demás jugadores.

El primer modelo de red basado en una arquitectura `cliente-servidor
<https://developer.valvesoftware.com/wiki/Latency_Compensating_Methods_in_Client/Server_In-game_Protocol_Design_and_Optimization#Basic_Architecture_of_a_Client_.2F_Server_Game>`_
implementaba el cliente de forma que las únicas funciones que tenía
eran muestrear los eventos de entrada del jugador (pulsaciones de
teclado/ratón), enviarlos al servidor y renderizar el *mundo*. El
servidor por su parte ejecutaba la única instancia real del juego,
recibía mensajes de los clientes y actualizaba el estado del *mundo*
basándose en los mensajes recibidos. Este método conseguía mejorar la
consistencia, ya que el servidor se encarga de comprobar la
*legalidad* de las acciones de cada jugador y de enviarles mensajes
con el estado actualizado del juego.

Aún con estos cambios, la latencia seguía suponiendo un problema si se
quería mejorar la inmediatez. Sin embargo, se desarrollaron una `serie
de técnicas
<https://developer.valvesoftware.com/wiki/Source_Multiplayer_Networking>`_
que conseguirían reducir y enmascarar la latencia. Estas técnicas se
pueden resumir en: mecanismos de predicción en el lado del cliente,
desarrollados por John Carmack y de compensación de la latencia en el
lado del servidor, desarrollados por Yahn Bernier.

Actualmente, la gran mayoría de juegos en red utilizan un modelo
cliente servidor ya que reduce el problema de la latencia, ayudando a
enmascararla, lo que mejora la inmediatez, al tiempo que reduce las
exigencias de ancho de banda.

=======================================
Mecanismos de reducción de la latencia
=======================================

En un modelo cliente-servidor actual típico, el servidor envía al
cliente una media de `20 mensajes por segundo
<http://goo.gl/r3zBEp>`_. Si el cliente renderizase el mundo
únicamente con la información que recibe del servidor, no se
alcanzaría un *framerate* (cantidad de cuadros por segundo) aceptable,
haciendo que el vídeo del juego no fuese fluido. El mínimo ratio de
renderizado aceptado por la industria del videojuego es de 30 frames
por segundo. Eso supone 30 mensajes por segundo, por lo que el
servidor debería enviar al menos esta cantidad de mensajes para que
los clientes pudiesen renderizar el mundo sin implementar ningún tipo
de técnica de compensación, todo esto suponiendo que los paquetes no
sufren perdidas, no se viesen afectados por factores como el
jittering, altas latencias, etcétera.

Para poder alcanzar una tasa de renderizado aceptable en esta
situación se usa una técnica que se conoce como *interpolación*. La
*interpolación* pasa por retrasar el proceso de renderizado en el
cliente una cantidad de tiempo aproximadamente igual a la latencia,
mientras que se van almacenando en un buffer los snapshots que envía
el servidor. En el ejemplo anterior, si el servidor envía 20 mensajes
por segundo, el cliente recibiría uno cada 50ms. Al retrasar 50ms el
renderizado de la escena, el estado de los objetos del juego podrá ser
interpolado tomando el rango de valores comprendido entre el último
mensaje recibido y el previo a ese. Este proceso requiere seguir
recibiendo mensajes del servidor, no siendo posible llevarse a cabo si
se pierden mas de dos mensajes consecutivos, debido a que el cliente
se quedaría sin snapshots en el buffer. La forma de solucionar este
problema pasaría por aumentar el tiempo de interpolación. Sin embargo,
esto aumentaría aún mas la latencia, de modo que se puede aumentar
indefinidamente el tiempo de interpolación.

Otra técnica consiste en optar por aceptar como bueno, de forma
temporal, la ejecución de un comando del jugador local (recordemos que
el servidor debe autorizar todas las acciones de cada jugador), en
lugar de esperar la recepción de la confirmación del servidor. Esto
permite ejecutar las acciones asociadas a dicho comando
inmediatamente. Este proceso sólo es aplicable a las entidades del
mundo que se vean directamente afectada por los comandos del usuario
local. Esto es así debido a que se necesita la información de los
demás usuarios para poder saber qué acciones tomaron. Si el cliente
comete un error al predecir sus propias acciones (por ejemplo, porque
ocurrió una interacción con otro jugador que el cliente no pudo tener
en cuenta durante la predicción) deberá rectificar en base a los
mensajes procedentes del servidor. Para poder realizar estas
correcciones, el cliente guarda todos las acciones no confirmadas por
el servidor en un buffer, de modo que cuando el servidor le informa de
su error, el cliente puede «rebobinar» sus acciones y corregir su
error con la información recibida del servidor.

El servidor por su parte realiza compresión de datos y `compensación
de la latencia
<https://developer.valvesoftware.com/wiki/Latency_Compensating_Methods_in_Client/Server_In-game_Protocol_Design_and_Optimization#Lag_Compensation>`_. La
compresión de datos se realiza enviando snapshots, es decir, resúmenes
del estado del juego en un determinado momento, en lugar de enviar el
estado con cada cambio que se produzca.

La compensación de latencia consiste en tener en cuenta la latencia
del cliente a la hora de procesar los mensajes. Comprender cómo
funciona la interpolación es importante en el diseño de la
compensación de latencia debido a que la interpolación es otro tipo de
latencia en la experiencia de un usuario. En la medida en que un
jugador está interaccionando a otros objetos que han sido
interpolados, el tiempo de interpolación debe ser tomado en
consideración de modo que se puedan conseguir un mayor determinismo.

La compensación de latencia es un método de normalización del estado
del mundo por parte del servidor, que se lleva a cabo para cada
comando de cada jugador. Se podría pensar en la compensación de
latencia como en dar un paso atrás en el tiempo para mirar a la
situación del mundo en el instante exacto en que el usuario realiza
alguna acción Por ejemplo, si el cliente tiene 150ms de latencia, los
mensajes que reciba el servidor tendrían sentido con el estado que el
tuviese el *mundo* hace 150ms, de modo que el servidor «rebobina» el
estado del juego para este jugador a 150ms antes de recibir un mensaje
de dicho cliente, de modo que pueda ejecutar los comandos de usuario
en el contexto en el que el cliente estaba inmerso cuando los ejecutó.

**************
Conclusiones
**************

En este artículo he intentado resumir de forma breve y concisa los
principales modelos de red y las tecnicas más importantes que se usan
actualmente en los juegos en red.

En el sector de los videojuegos está muy candente el tema del "Cloud
Gamming"; es decir, ejecutar el juego en servidores dedicados, de modo
que lo que recibe el cliente es un flujo de video. De esta forma, se
puede jugar a los títulos mas punteros en dispositivos que no tienen
la capacidad para ejecutarlos. Como ejemplo representativo, se pueden
nombrar `Nvidia's GRID cloud
<http://www.nvidia.com/object/cloud-gaming.html>`_ o el servicio de
Sony `PlayStation Now
<http://www.playstation.com/en-us/explore/psnow/>`_. Existen otras
empresas que ofrecen estos servicios como `OnLive
<https://games.onlive.com/>`_.

*************
Referencias
*************

[1] Algorithms and Networking for Computer Games, Smed, J. and Hakonen, H.
