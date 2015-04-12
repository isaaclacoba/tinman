.. title: Roadmap del proyecto
.. slug: roadmap-del-proyecto
.. date: 2015-02-22 17:15:49 UTC+01:00
.. tags:
.. link:
.. description:
.. type: text

En esta entrada se va a mostrar cuál es el estado actual del proyecto, qué objetivos se han cumplido y en qué porcentaje. Además se hablará en qué se está trabajando actualmente.

.. TEASER_END: click to read the rest of the article


********************
Estado del proyecto
********************

En primer lugar, recordemos cuál es la lista de hitos que se pretenden cumplir en este proyecto:

- Desarrollar un videojuego utilizando gráficos 3D.
- Implementar mecanismos de instrumentación que permitan aplicar técnicas de testing automático.
- Diseñar e implementar un modelo de red para permitir modo multijugador.

Como subobjetivo, se pretende que dicho videojuego pueda servir a futuros programadores para aprender técnicas básicas y fundamentos esenciales del desarrollo de un videojuego. Por esta razón se hará énfasis en los siguientes aspectos:

- Claridad del código.
- Uso de patrones de diseño.
- Técnicas de introspección de objetos.
- Acceso al código fuente y al propio proyecto. Por esta razón se distribuirá bajo una licencia libre.

========================================
Grado de cumplimiento de los objetivos
========================================

Hasta la fecha se han cumplido los siguientes objetivos:

1. Desarrollo de un videojuego utilizando gráficos 3D: Se ha conseguido desarrollar una primera versión del juego de carreras. En el siguiente vídeo se ve una muestra de lo desarrollado hasta la fecha. Aunque el vídeo no tiene sonido, el motor de juego que se ha desarrollado para este proyecto sí que cuenta con el soporte para incluirlos.

   - *Trabajo futuro*: animaciones y efectos de partículas, así
     como mejorar el aspecto visual de los menús, seleccionar sonidos
     adecuados para este juego(música y efectos). En cuanto al gameplay, se
     pretende dar la oportunidad al jugador de mejorar su coche mediante la
     compra de mejoras.

.. youtube::  Rn_WwsMEW_4


2. Implementar mecanismos de instrumentación que permitan aplicar técnicas de testing automático: gran parte de la arquitectura del motor de juego que se ha desarrollado para este proyecto ha surgido a partir de las pruebas que se han escrito. Son pruebas muy sencillas basadas en log escritas a modo de pequeños `ejemplos <https://bitbucket.org/arco_group/tfg.tinman/src/e56b57a12b1661caa19d066f3127827e28a36186/examples/?at=master>`_. Dichos ejemplos son en realidad pruebas de comportamiento; es decir, lo que se persigue no es comprobar que el valor de retorno de una determinada función es correcto, sino que el funcionamiento en conjunto del sistema es el adecuado. La ventaja es que las pruebas se han escrito con la hoja de especificación de requisitos en la mano, lo que nos permite afirmar que se están implementando al pie de la letra. Las pruebas sacan obtienen la información del sistema a través de un sistema de logs que se ha añadido al motor, el cuál facilita la labor de depuración. Además, dado que la salida de las pruebas se registra en ficheros de texto, es posible automatizar la ejecución de las pruebas y mas tarde comprobar los resultados.

   - *Trabajo futuro*: La limitación del sistema de log que usamos es que sólo puede proporcionar información relativa a los objetos del juego, la traza de ejecución o las excepciones lanzadas. Cuándo queremos comprobar cualquier aspecto referente a la interfaz gráfica, nos encontramos con que no tenemos el soporte necesario. Una posible forma de hacer frente a esta limitación consistiría en hacer uso de una biblioteca de analisis de imagen, a la cuál le pasaríamos una captura de pantalla. En teoría, este tipo de pruebas nos permitirían asegurarnos que la construcción de los escenarios es correcta, así como los menús y demás elementos gráficos, aunque queda mucho trabajo de investigación por delante para ver hasta qué punto es realmente útil la implementación de pruebas de este tipo.

3. Diseñar e implementar un modelo de red para permitir modo multijugador: actualmente se está trabajando en el modelo de red del juego. Para ello, se está utilizando `ZeroC Ice <https://www.zeroc.com/>`_, un middleware de red orientado a objetos. Cuenta con implementación en python, C++, java, etcétera.

   - *Trabajo futuro*: Generalmente se suele decir que los middleware de
     red añaden demasiada sobrecarga y, por esta razón, no son adecuados
     para juegos que requieren reacciones rápidas por parte de los
     jugadores, como son los juegos de carreras. Con el uso de este
     middleware se pretende probar que el uso de un middleware de red, si
     es lo suficientemente flexible, permite adecuarse a las necesidades de
     los desarrolladores y, aunque añada una mayor sobrecarga que una
     implementación que sólo ofrezca soporte para sockets, los mecanismos
     de alto nivel que proporciona hace que merezca la pena.
