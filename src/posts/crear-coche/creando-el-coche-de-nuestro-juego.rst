.. title: Creando el coche de nuestro juego
.. slug: creando-el-coche-de-nuestro-juego
.. date: 2015-02-13 16:50:59 UTC+01:00
.. tags:
.. link:
.. description:
.. type: text

.. _logo_tinman:
.. image:: tinman.jpg
   :align: center



En esta entrada se va a hablar acerca del componente de dinámica de
vehículos que incorpora Bullet Physics, el motor de físicas que se
está usando en este proyecto. Se explicará como inicializar el
vehículo y los elementos mas relevantes de éste módulo de Bullet. Por
último, mostraremos un pequeño ejemplo que servirá para ejemplificar
lo hablado en esta entrada.

******************
Un poco de teoría
******************

*******************
Vehículos en Bullet
*******************

El componente de dinámica de vehículos de Bullet ofrece una
implementación basada en rayqueries, de tal manera que se lanza un
rayo por cada rueda del coche. Usando como referencia el punto de
contacto del rayo contra el suelo, calculamos la longitud y la fuerza
de la suspensión. La fuerza de la suspensión se aplica sobre el
chassis de forma que no choque contra el suelo. De hecho, el chasis
del vehículo flota sobre el suelo sustentándose sobre los rayos. La
fuerza de fricción se calcula por cada rueda que esté en contacto con
el suelo. Esto se aplica como una fuerza hacia los lados y adelante.

Como cualquier cuerpo físico en Bullet, el chasis está formado como un
cuerpo rígido y una forma de colisión que modela el comportamiento del
cuerpo rígido. Es importante señalar que el punto de origen de los
rayos debe situarse dentro de la forma de colisión del chasis ya que,
de otro modo, las ruedas no tendrían tracción al no estar enganchadas al
chassis.

Roll influence effectively lowers the vehicles centre of mass, reducing the chance of the vehicle rolling over.
