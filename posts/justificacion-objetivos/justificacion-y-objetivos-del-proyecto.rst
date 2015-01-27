.. title: Justificación y objetivos del proyecto
.. slug: justificacion-y-objetivos-del-proyecto
.. date: 2015-01-24 11:06:19 UTC+01:00
.. tags:
.. link:
.. description:
.. type: text

.. _logo_tinman:
.. image:: tinman.jpg
           :align: center


En esta entrada se explicará brevemente qué es el proyecto Tinman, se
justificarán las razones por las cuales se está realizando este
proyecto, y por último, se listarán los objetivos que se pretenden
cumplir con el desarrollo del mismo.

*****************
¿Qué es Tinman?
*****************
*Tinman* es el nombre corto de mi trabajo fin de Grado(TFG), que se titula: "Instrumentación de un videojuego 3D con fines docentes". El videojuego será un clon del famoso juego de carreras arcade "`Super Off Road <http://es.wikipedia.org/wiki/Super_Off_Road>`_", y de aquí es de donde surge *Tinman* como nombre de mi proyecto, a modo de broma, dado que el juego original llevó el nombre de Ivan "Ironman" Stewart, un famoso piloto del estilo *Off-Road*.

Este proyecto nació con el espíritu de crear un videojuego que fuese útil como caso práctico sobre el que estudiar toda una serie de técnicas relacionadas con el desarrollo de videojuegos. Es obvio que se trata de un proyecto muy ambicioso y por ello se ha de acotar el alcance del mismo. Del título, lo que más puede llevar a confusión sea la última parte, con fines docentes. Debido a que desarrollar un *Serius Game* específicamente orientado a desarrolladores se desmarca del objeto de evaluación dentro de un TFG, e incluso en una tesis de máster, el esfuerzo en este proyecto se ha centrado en desarrollar un código lo más limpio posible, sobre el que poder ejemplificar el uso de técnicas avanzadas como patrones de diseño o como `idioms <http://programmers.stackexchange.com/a/94567>`_ en C++, por poner algunos ejemplos.

Para que lo anterior fuese posible se decidió licenciar el código del proyecto, así como todo el contenido generado, bajo una licencia libre. En nuestro caso, el proyecto se licencia bajo GPLv3.Se decidió usar una licencia libre porque si se pretendía enseñar a quien lo deseara cómo fue creado el proyecto, la forma mas sencilla pasaba por dar libertad para leer el código, modificarlo y distribuir tanto modificaciones del proyecto original como copias del mismo. Creo que la mejor forma de aprender a programar es programando, y a mí me resulta mas sencillo si tengo algo sobre lo que empezar.

Por otra parte, no abundan proyectos libres en el mundo de los videojuegos y los que existen, o bien no se ha dado al código fuente el mimo que merece, o los creadores son ingenieros con una larga trayectoria, por lo que el código, aunque mantenible, flexible y bien estructurado, se hace difícil de estudiar para quien está comenzando a aprender y más complicado si estos proyectos están escritos en C++. Este lenguaje se ha convertido en estándar para la industria del videojuego, por múltiples razones. No sólo ofrece tanto mecanismos de muy alto nivel, tambien permite trabajar a nivel de dirección de memoria, de registro,...con lo que ofrece al ingeniero una flexibilidad que no se encuentra en otros lenguajes. Además, al ser compilado, permite optimizaciones de bajo nivel que no ofrecen otros lenguajes basados en máquinas virtuales o interpretados.

Pero alguien podría estar preguntándose: ¿realmente un videojuego se justifica como trabajo fin de Grado?

Según la "`Normativa sobre la elaboración y defensa del *Trabajo Fin
de Grado* <http://webpub.esi.uclm.es/archivos/336/Normativa-TFGs>`_"
de la Escuela Superior de Informática de Ciudad Real, su artículo 2º
*Naturaleza del Trabajo Fin de Grado*, estipula lo siguiente:

"El Trabajo Fin de Grado supone la realización por parte del estudiante, y de forma
individual, de un proyecto, memoria o estudio bajo la supervisión de
uno o más tutores/as, en el que se integren y desarrollen los
contenidos formativos recibidos, capacidades, competencias y
habilidades adquiridas durante el periodo de docencia del Grado en
Ingeniería Informática"

El apaertado anterior señala que el TFG consiste en desarrollar, de manera individual, un proyecto, memoria o estudio; es decir, que no es requerida implementación alguna. El proyecto Tinman requiere la aplicación de técnicas avanzadas de ingeniería del Software, sigue una metodología concreta de desarrollo (*desarrollo orientado por pruebas*), requiere la capacidad para resolver problemas complejos, hace uso de un software de control de versiones, que ayuda a realizar un seguimiento del progreso del proyecto, etc. Todo esto hace que se cumpla  lo previamente citado para considerar este proyecto como TFG.

Por otra parte, en todo proyecto debe contemplarse, de una forma u otra, el contexto económico. En ese sentido, el sector de los videojuegos está en pleno auge. Por aportar algunas cifras, la empresa NewZoo en su `publicación <http://www.newzoo.com/insights/global-games-market-will-reach-102-9-billion-2017-2/>`_ sobre la situación mundial del mercado del videojuego señalaba que si la progresión de crecimiento actual se mantiene, se espera que la cifra de facturación mundial ascienda desde los 81.400 millones de dólares actuales en 2014 hasta los 102.900 millones en 2017. No hace es necesario ser economista para darse cuenta que el volumen de facturación es impresionante y que por sí sólo justifica la creación de nuevos proyectos orientados a este sector.

.. _grafico_mercado:
.. image:: mercado-videojuego.png
           :align: center

Para finalizar, se listarán los objetivos que se pretenden cumplir.

***********
Objetivos
***********
En primer lugar se pretende desarrollar de un videojuego, utilizando
gráficos en 3D. Aunque el videojuego en sí ya es un objetivo de envergadura, este proyecto pretende utilizarlo como base para dos objetivos adicionales:

- Diseño e implementación de mecanismos de instrumentación del juego que permitan
  aplicar técnicas de testing automático y al mismo tiempo exponer los mecanismos internos con fines didácticos y de depuración.
- Diseño e implementación de un modelo de red para una modalidad multijugador, que
  aprovechará los citados mecanismos de instrumentación con los mismos fines.

Otro objetivo no menos importante es crear un ejemplo completo de videojuego que pueda servir a futuros programadores a aprender las técnicas básicas y los fundamentos técnicos esenciales del desarrollo de un videojuego. Por ese motivo se hará énfasis en los siguientes aspectos:

- Claridad del código.
- Uso de patrones de diseño.
- Técnicas de introspección de objetos.
- Acceso al código fuente y al propio proyecto. Por esta razón se
  distribuirá bajo una licencia libre.
