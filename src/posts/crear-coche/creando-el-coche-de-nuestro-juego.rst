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

Pero para empezar, vamos a introducir brevemente los principios
físicos que permiten el movimiento de un coche.

******************
Un poco de teoría
******************

A grandes rasgos, el movimiento de un coche radica en un conjunto de
fuerzas que se aplican sobre las ruedas y el chasis del vehículo. En la dirección del movimiento del coche se aplica una fuerza longitudinal, compuesta por la fuerza que aplican las ruedas, la fuerza de frenado, la resistencia que oponen los neumáticos y la resistencia del aire. Por otro lado, en giros existen fuerzas laterales causadas por la fricción lateral de las ruedas, además del momento angular del coche y el esfuerzo de torsión causado por las fuerzas laterales.

========================
Movimientos rectilíneos
========================

- fuerza de tracción = u * Fmotor
  - resistencia del aire = -Cdrag * v * |v| (constante de resistencia del aire)
  - resistencia de los neumáticos = -Crr * v (constante de rozamiento)
-Flongitudinal = Ftracción + Raire + Rneumáticos

- aceleración = F / Masa
  - velocidad = v0 + aceleración * dt (incremento de tiempo entre las
   llamadas al motor de físicas)
  - posición = p0 + v*dt

 - Flongitudinal = Ffrenado + Raire + Rneumáticos
 La fuerza de frenado sustituye a la de tracción
 - Ffrenado = -u * Cfrenado

- Transferencia de peso
  - Fmax = mu * Pesocoche (mu coeficiente de rozamiento del neumático)




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


..
Hola, parece que tienes algo de curiosidad.
Como recompensa, aquí tienes la traducción del artículo completo sobre el que me he basado para escribir el apartado teórico de este post

// -*- coding:utf-8; tab-width:4; mode:cpp -*-

Original: http://www.asawicki.info/Mirror/Car%20Physics%20for%20Games/Car%20Physics%20for%20Games.html

****************
Introducción
****************

Este tutorial trata el tema de la simulación de coches en
videojuegos. Se tratará las propiedades físicas que modelan el
comportamiento de un coche orientándolo a su aplicación a videojuegos.

Uno de los puntos clave de la simulación en videojuegos consiste en
simplificar las físicas para gestionar fuerzas laterales y
logitudinales de forma separada. Las fuerzas logitudinales operan en
la dirección del cuerpo del coche. La logitudinal está
compuesta por la fuerza que aplican las ruedas, la de frenado, la
resistencia de giro y la resistencia del aire. Estas fuerzas juntas
controlan la aceleración y desaceleración del coche, así como su
velocidad. Por otro lado, las fuerzas laterales permiten al coche
girar. Estas fuerzas son causadas por la fricción lateral de las
ruedas. Tambien hay que tener en cuenta el momento angular del coche y
el esfuerzo de torsión causado por las fuerzas laterales.

***********************************
Físicas en movimientos rectilíneos
***********************************

El primer caso a considerar es el de un coche moviendose en línea
recta. La primera fuerza que entra en juego es la fuerza de tracción;
es decir, la que proporcina el motor a través de las ruedas. El motor
gira las ruedas hacia adelante(aplicando una fuerza de torsión), de
tal forma que las ruedas empujan hacia atrás contra la superficie de la
carretera y, en reacción, se genera una fuerza hacia adelante. Esto
implica que la fuerza de tracción es equivalente a la fuerza del
motor, que es controlada directamente por el usuario.

    Ftraccion = u * FMotor,
     donde u es un vector unitario con la dirección del coche.

Si esta fuera la única fuerza que influye en el movimiento, el coche
aceleraría hasta alcanzar una velocidad infinita. Aquí es donde entran
en juego las fuerzas de resistencia. La primera sería la resistencia
del aire. Esta fuerza es muy importante porque es proporcional al
cuadrado de la velocidad. Al conducir a altas velocidades ésta es la
mayor resitencia que encuentra el coche.

   Fdrag = - Cdrag * v * |v|
    donde Cdrag es una constante de resistencia del aire,
    v es el vector de velocidad y
    |v| el módulo del vector.

El módulo del vector velocidad es la velocidad a la que nos referimos
comunmente, expresada en km/h cuando hablamos de vehículos.

  speed = sqrt(v.x*v.x + v.y*v.y);
  fdrag.x = - Cdrag * v.x * speed;
  fdrag.y = - Cdrag * v.y * speed;


La siguiente resistencia que encontramos es la resistencia al giro. Es
causada por la fricción entre la goma del neumático y la superficie de
contacto debido al desplazamiento de las ruedas.


   Frr = -Crr Frr = - Crr * v
    donde Crr es una constante de rozamiento y
    v el vector de velocidad.

A bajas velocidades la resistencia al giro es la mayor resistencia que
encuentra el coche, mientras que a altas velocidades sería la
resistencia del aire. A 100km/h (aproximadamente 30m/s) son
equivalentes [http://www.gdconf.com/2000/library/homepage.htm]. Esto
significa que el coeficiente resistencia de giro debe ser
aproximadamente 30 veces el valor del coeficiente de resistencia
aerodinámica.

La fuerza logitudinal total es la suma de estas tres fuerzas:

    Flong =   Ftraction + Fdrag   + Frr

Hay que señalar que si se conduce en línea recta las fuerzas de
resistencia tiene sentido contrario al que toma el coche, oponiéndose
al movimiento. De esta forma, dentro de la fórmula tomarían valores
negativos, mientras que la fuerza de tracción toma valores
positivos. Cuando el coche se mueve a una velocidad constante las
fuerzas se encuentran en equilibrio, por lo que Flong es cero.

La aceleración del coche(expresada en m/s) se calcula a partir de la
fuerza neta(Newtons) y la masa del coche (kg) usando la segunda ley de
Newton:

   a = F/Métrico


La velocidad del coche se calcula integrando la aceleración en el
tiempo:

   v = v0 + aceleración * dt
    donde dt es el incremento de tiempo en segundos entre las
    subsiguientes llamadas al motor de físicas.

La posición del coche se calcula integrando la velocidad a lo largo
del tiempo:

  p = p + dt * v


Con estas tres fuerzas se puede simular la aceleración del coche de
una forma bastante precisa. Juntas también determinan la velocidad
máxima del coche para una potencia de motor dada. No hay necesidad de
definir una velocidad máxima en ninguna parte del código ya que es
algo que viene dado por estas ecuaciones. Esto es así debido a que las
ecuaciones interaccionan entre ellas. Por ejemplo, si la tracción
sobrepasa a las resistencias dentro de la fórmula de la fuerza
longitudinal, el coche acelerará. La velocidad del coche se
incrementará, lo que incrementará las resistencias. La fuerza neta
decrementará y por tanto la aceleración. En algún punto las
resistencias y la fuerza de tracción se igualarán, cancelándose
mútuamente, lo que hará que el coche alcance la velocidad punta para
esa potencia de motor determinada.

file:///home/isaac/Documentos/tfg/fisicas/Car%20Physics_files/ctgraph.jpg

En el diagrama el eje de las x denota la velocidad del coche en metros
por segundo y el eje de las y el valor de las fuerzas. La fuerza de
tracción( azul oscuro) se configura a un valor aleatorio, ya que no
depende de la velocidad del coche. La resistencia de giro (línea
morada) es una función lineal de la velocidad y la resistencia
aerodinámica(línea amarilla) es una función cuadrática de la
velocidad. A velocidades bajas la resistencia de giro sobrepasa a la
resistencia del aire. A 30m/s las dos funciones se cruzan. A
velocidades altas la resistencia del aire es la mayor de las
resistencias. La suma de las dos resistencias se muestra en la línea
azul claro. A 37m/s la suma de las resitencias iguala la línea
horizontal (potencia del motor). Esta es la velocidad punta para ese
valor particular de la potencia del motor.

*******************
Constantes mágicas
*******************

Hasta ahora, hemos introducido dos constantes mágicas, Cdrag y Crr. Si
no se persigue conseguir realismo en la simulación física, los valores
que hemos dado a estas constantes son suficientemente buenos para tu
juego. Por ejemplo, en un juego arcade se podría querer que el coche
acelerase mas rápido que el la vida real. Sin embargo, si se busca
el realismo, es importante dar a estas constantes valores precisos.

La resistencia del aire está modelada, aproximadamente, por la
siguiente fórmula [Fluid Mechanics by Landau and Lifshitz, [Beckham]
chapter 6, [Zuvich]]

  Fdrag =  0.5 * Cd * A * rho * v2

    donde  Cd = coeficiente de fricción
    A es el area frontal del coche
    rho (Greek symbol)= densidad del aire.
    v = velocidad del coche

La densidad del aire(rho) es 1.29kg/m³, el area frontal
aproximadamente 2.2m², Cd depende de la forma del coche y se determina
con test en tuneles de viento. Para un Corvette: 0.3. Esto nos da para
Cdrag:

   Cdrag = 0.5 * 0.3 * 2.2 *1.29 = 0.4257

Crr es aproximadamente 30 veces Cdrag, lo que nos da:

    Crr = 30 * 0.4257 = 12.8

Este último valor no es 100% correcto.

**********
Frenado
**********

Cuando el coche frena, la fuerza de tracción se ve reemplazada por la
fuerza de frenado, la cuál está orientada en sentido opuesto al del
movimiento. La fuerza longitudinal total es el vector que resulta de
la suma de las tres fuerzas:

   Flong =   Fbraking + Fdrag   + Frr

La fuerza de frenado de forma simplificada es igual a:

   Fbraking = -u * Cbraking

   u es el vector unitario de movimiento y
   Cbraking una constante de frenado.

En esta fórmula la fuerza de frenado es constante, de modo que hay que
dejar de aplicarla cuando la velocidad del coche llegue a cero, para
que el coche no empiece a avanzar en sentido contrario al del
movimiento.

************************
Transferencia de peso
************************

Un efecto importante cuando se acelera o frena es el efecto de la
transferencia dinámica de peso. Cuando se frena el coche baja el morro
hacia adelante. Durante la aceleración, el coche se inclina hacia
atrás. Esto es debido a que el centro de gravedad el coche cambia. El
efecto de esto es que el peso sobre las ruedas traseras aumenta
durante la aceleración, mientras que las ruedas delanteras deben
soportar menos peso.


El efecto de la transferencia de peso es importante por dos
razones. La primera es que el efecto visual del coche "cabeceando" en
respuesta a las acciones del usuario aporta gran realismo. De repente
el usuario se siente mas inmerso en la simulación.

Por otra parte, la distribución de peso afecta dramáticamente a la
tracción máxima por rueda. Esto es debido a que el límite de fricción
es proporcional a la carga en esa rueda:

    Fmax = mu * W

    donde mu es el coeficiente de fricción en el neumático y
    W es el peso del coche.

Para neumáticos de calle m utoma el valor de 1.0, mientras que para
neumáticos de carrera puede tomar valores superiores a 1.5.

Para vehiculos estacionados el peso total del coche (W = M*g) se
distribuye sobre las ruedas delanteras y traseras de acuerdo a la
distancia entre la parte el eje delantero y trasero al centro de masa:

     Wdelantero = (c/L)*W
     Wtrasero = (b/L)*W
       donde b y c son la distancia al centro de gravedad de los ejes delanteros y traseros y L es la base de las ruedas.
file:///home/isaac/Documentos/tfg/fisicas/Car%20Physics_files/ctwd.jpg

Si el coche acelera o desacelera en un factor a, el peso frontal y
trasero se calculan como sigue:

    Wf = (c/L)*W - (h/L)*M*a
    Wr = (b/L)*W + (h/L)*M*a
       donde h es la altura del centro de gravedad, M es la masa del coche y a la aceleración

Para simplificar las fórmulas, se puede asumir una distribución
estática de 50-50 sobre la parte frontal y trasera. En otras palabras,
asumimos b = c = L/2. En ese caso, Wf = 0.5W -(h/L) * M * a y Wr =
0.5*W + (h/L)*M*a.

*****************
Fuerza del motor
*****************

Hasta ahora hemos hecho una pequeña simplificación diciendo que el
motor da una cantidad de fuerza. El motor aporta par motor o momento
torsor. El par motor es fuerza por distancia. Si aplicas una fuerza de
10 Newton 0.3 metros en el eje de rotación, obtienes 10*0.3 = 3N.m (
Newton metro). Es elo mismo cuando aplicas un par motor de 1 Newton a
3 metros del eje. En ambos casos el momento es el mismo.

El momento torsor que puede entregar el motor depende de la velocidad
a la cuál este gira, típicametne expresado en rpm. La relación momento
torsor/rpm no es lineal, pero se representa normalmente como una curva
llamada función del momento torsor( La curva exacta de cada motor
viene determinada por los test a los que son sometidos estos
motores). Aquí vemos un ejemplo para el motor de un Corvette de 1997 a
2000: el LS1(5.7 litros V8)

file:///home/isaac/Documentos/tfg/fisicas/Car%20Physics_files/cttorq.gif

Nota que la curva del par motor alcanza el máximo alrededor de las
4400 rpm con un par motor de 475 N.m y la curva de los caballos de
potencia alcanza el máximo a 5600rpm a 345 caballos de potencia( 257
kW). Las curvas sólo están definidas en el rango de los 1000 a los
6000 rpm debido a que es el rango operativo del motor. Cualquier valor
inferior hará que el motor se detenga. Cualquier valor superior lo
dañaría.

Los valores mencionados anteriormente hacen referencia al máximo par
motor que puede entregar el motor paraa unas rpm dadas. El par real que
entrega el motor depende de la posición del acelerador y es una
fracción entre 0 y 1 de este máximo.

Nuestro interés se centra principalmente en la curva del par, aunque
algunas personas encuentran interesante tambien la de potencia. A
continuación se puede ver la misma gráfica en unidades del SMI.

file:///home/isaac/Documentos/tfg/fisicas/Car%20Physics_files/cttorqsi.gif

Ahora, el par de torsión desde el motor (es decir, en el cigüeñal) se
convierte a través del engranaje diferencial y antes de que sea
aplicada a las ruedas traseras. El engranaje multiplica el par de
torsión por un factor que depende de las relaciones de transmisión
(las marchas).

Desafortunadamente se pierde energía en el proceso. Hasta un
30% se puede perder en forma de calor. Esto da una eficiencia de
transmisión del 70%, aunque el valor concreto en cada coche varía.

El par motor se convierte en una fuerza a través del giro de la rueda
sobre la carretera, dividido por el radio de la rueda( Fuerza = par
motor / distancia)

A continuación podemos ver la formula que convierte par motor en
fuerza de "conducción": la fuerza longitudinal que ejercen las dos
ruedas traseras sobre la carretera.

    Fdrive = u * Tengine * xg * xd * n / Rw
    donde u es el vector unitario que refleja la orientación del coche
    Tengine es el par motor en rpm
    xg es la relación de las marchas
    xd es el coeficiente diferencial
    n es la eficiencia de la transmisión
    Rw es el radio de la rueda.

*************************
Relación de transmisión
*************************

Los siguientes ratios se aplican al Corvette C5 hardtop:


First gear	        g1          2.66
Second gear	        g2          1.78
Third gear	        g3          1.30
Fourth gear	        g4          1.0
Fifth gear	        g5          0.74
Sixth gear	        g6          0.50
Reverse	            gR          2.90
Differential ratio 	xd          3.42

EL máximo par motor es 475 N.m a 4400 rpm, la masa = 1439 kg(ignorando
la del conductor por ahora). En la primera marcha, con el máximo par
nos da 475*2.66*3.42*0.7/0.33 = 9166 N de fuerza. Esto haría que el
coche acelerase los 1439 kg del coche a 6.4 m/s² que es igual a 0.65
g.

La combinación de las marchas y el diferencial actua como un
multiplicador del par motor en el cigueñal sobre el par de torsión que
se aplica a las ruedas. Por ejemplo, el Corvette en la primera marcha
tiene un multiplicador de 2,66 * 3,42 = 9,1. Esto significa que cada
metro Newton del par motor en el cigüeñal resulta en 9,1 Nm de par
motor en el eje trasero. Considerando un 30% de perdida de energía,
esto deja 6.4 N.m. Dividiendo esto por el radio de las ruedas
obtenemos la fuerza ejercida por las ruedas. Suponiendo un radio de 34
cm, tenemos 6.4 N.m/0.34m = 2.2N de fuerza por N.m de par motor. Sin
embargo, la ganancia obtenida como par motor tiene como contrapunto
velocidad angular. Se intercambia fuerza por velocidad. Por cada rpm
de las ruedas, el motor debe dar 9.1 rpm. La velocidad de rotación de
cad rueda es directamente proporcional a la velocidad del coche. Una
rpm está 1/60th de una revolución por segundo. Cada revolución hace
avanzar a la rueda 2 pi * R hacia adelante; es decir, 2 * 3.14 * 0.34
= 2.14 m. De esta forma, 4400 rpm en la primera marcha equivalen a 483
rpm en las ruedas, lo que son 8.05 rotaciones por segundo = 17.2 m/s (
alrededor de 62 km/h).

En marchas bajas el ratio de las marchas es alto, de modo que obtienes
mucho par motor pero poca velocidad. En velocidades altas, obtienes
mas velocidad que par motor. Esto se puede observar en las siguietne
gráfica.

file:///home/isaac/Documentos/tfg/fisicas/Car%20Physics_files/ctgrcrvs.gif

La gráfica asume una eficiencia del 100%. El par motor se representa
como la línea negra.

***************************************
Aceleración (Drive wheel acceleration)
***************************************

El par motor que obtenemos para una rpm dada es el máximo par motor a
esa rpm. Cuanto par motor se aplica realmente a las ruedas depende de
la posición del acelerador. Esta posición se determina por las
entradas del usuario (a través del pedal) y varía de 0 a 100%.

**********************************
Como obtener el valor de los rpm
**********************************

Se necesita calcular el valor máximo del par motor y a partir de ese
valor obtener el valor real del par motor aplicado; es decir, hay que
conocer cuán rápido gira el cigüeñal.

Una forma en que se puede calcular este valor es obteniendo la
velocidad de rotación de las ruedas. Despues de todo, si el motor no
está desembragado, el cigueñal y las ruedas estarán físicamente
conectadas a través de la transmisión. Conociendo los rpm del motor
podemos conocer la velocidad de rotación de las ruedas y viceversa.

   rpm = Ratio de giro de las ruedas * marcha * ratio  del diferencial * (60 / 2 pi)

El multiplicando 60/2 * pi es un factor de conversión de rad/s a
rpm. Hay 60 segundos en un minuto y 2pi radianes por revolución. De
acuerdo a esta fórmula el cigueñal gira más rápido que las
ruedas. Supongamos que está girando a 17 rad/s:

  Las ruedas giran a 17 rad /s.  El ratio de la primera marcha es
  2.66, el ratio differential es 3.42 por lo que el cigueñal rota a
  153 rad/s.  Eso significa que el motor gira a => 153*60 = 9170
  rad/minute = 9170/2 pi = 1460 rpm

Debido a que la curva del par motor no está definido por debajo de
ciertas rpm, hay que hacer que el gestor de físicas contemple caso:

if( rpm < 1000 )
	rpm = 1000;

Esto es necesario para poder modelar el motor del coche cuando éste
esté parado. Ya que calculamos los rpm a partir de las rpm de las
ruedas y éstas estarán paradas, los rpm serán 0.

Hay dos formas de obtener la velocidad de rotación de las ruedas. La
primera es un truco y la segunda involucra hacer un seguimiento a lo
largo del tiempo de varias variables.

La forma más fácil es pretender que la rueda está girando y derivar la
velocidad de rotación de la velocidad del coche y el radio de la
rueda. Por ejemplo, digamos que el coche se mueve a 20 km/h = 20,000 m
/ 3600 s = 5.6 m/s.  el radio de las ruedas es 0.33 m, por lo que la
velocidad angular de las ruedas es 5.6/0.33 = 17 rad/s

Usando las formulas anteriores para obtener rpm, obtenemos que el
valor es 1460 rpm, de lo que podemos calcular el par motor a partir de
la curva del par motor.

Una forma más avanzada es hacer que la simulación realice un
seguimiento de la velocidad de rotación de la rueda y de cómo cambia
con el tiempo, debido al par motor que actúan sobre dichas ruedas. En
otras palabras, calculamos la velocidad de rotación mediante la
integración de la aceleración rotacional en el tiempo. La aceleración
rotacional en cualquier instante particular depende de la suma de
todos los pares de torsión en el eje y es igual al par neto dividido
por la inercia del eje (al igual que la aceleración es la fuerza
dividida por la masa). El par neto es el par motor que vimos antes,
menos los pares de rozamiento que lo contrarrestan (par de frenado si
se está frenado y par de tracción a partir del contacto con la
superficie de la carretera).

***********************************************
Relación de deslizamiento y fuerza de tracción
***********************************************

Calcular la velocidad angular de las ruedas a partir de la velocidad
del coche sólo es posible si la rueda está girando, es decir, no hay
desplazamiento lateral entre el neumatico y la carretera. Esto es
cierto para las ruedas delanteras, pero para las ruedas motrices esto
no se suele cumplir.  Por ejemplo, cuando estas derrapan no se produce
transferencia de energia para hacer que el coche avance

En una situación típica en la que el coche se desplaza a una velocidad
constante, las ruedas traseras giran levemente más rápido que las
ruedas delanteras. Dado que las ruedas delanteras no derrapan, se
puede calcular su velocidad angular con sólo dividir la velocidad del
coche por 2 pi veces el radio de la rueda. Sin embargo, dado que las
ruedas traseras giran más rápido, eso significa que la superficie del
neumático se estará deslizando contra respecto a la superficie de la
carretera. Este deslizamiento causa una fuerza de fricción en la
dirección opuesta a la de deslizamiento. Por tanto, la fuerza de
fricción estará apuntando a la parte delantera del coche. De hecho,
esta reacción a la rueda que patina es lo que empuja al coche. Esta
fuerza de fricción se conoce como tracción o fuerza longtitudinal. La
tracción depende de la cantidad de deslizamiento. La forma
estandarizada de expresar la cantidad de deslizamiento es como la
denominada relación de deslizamiento:

file:///home/isaac/Documentos/tfg/fisicas/Car%20Physics_files/cteq_sr.gif

where
      Ww (omega) es la velocidad angular de las ruedas (in rad/s)
      Rw es el radio de las ruedas ( en metros)
      vlong es la velocidad del coche; la velocidad longitu
      dinal.

Nota: hay una serie de definiciones ligeramente diferentes de relación
de deslizamiento en uso. Esta definición particular también funciona
para los coches de tracción delantera.  La relación de deslizamiento
es cero para una rueda que no gira. Para un frenazo del coche con las
ruedas bloqueadas la relación de deslizamiento es -1, y un coche
acelerando tiene una relación de deslizamiento positivo, pudiendo
alcanzar valores mayores a 1 cuando existen una gran cantidad de
deslizamiento.

La relación entre la fuerza longitudinal y el ratio de desplazamiento
puede ser descrita por una curva como la del siguiente gráfico:

file:///home/isaac/Documentos/tfg/fisicas/Car%20Physics_files/ctsrcurve.gif

La gráfica muestra cómo la fuerza es cero si el ratio de deslizamiento
es 0, mientras que ésta alcanza su máximo para un valor del ratio de
desplazamiento del 6%, donde la fuerza longitudinal supera levemente
la carga de las ruedas. La curva exacta puede variar dependiendo del
tipo de neumático, de la superficie, la temperatura, etcetera. Esto
significa que las ruedas obtienen un mejor agarre con un poco de
deslizamiento. Mas hallá de ese óptimo, el agarre disminuye. Por esa
razón un derrape no da mayor aceleración. Habría tanto deslizamiento
que la fuerza longitudinal estaría por debajo de su valor máximo. La
disminución del desplazamiento da lugar a una mayor tracción y una
mejor aceleración.

La fuerza longitudinal es directamente proporcional a la carga de las
ruedas, como vimos cuando se discutió la transferencia de carga. Por
esta razón en lugar de dibujar una gráfica para cada valor particular
de la carga, podemos crear una curva normalizada dividiendo la fuerza
por la carga.

Para obtener la fuerza longitudinal a partir de la fuerza logitudinal
normalizada debemos multiplicarla por la carga:

     Flong = F(n, long) * Fz
      donde Fn,long es la fuerza longitudinal normalizada para una relación de desplazamiento dada y Fz es la carga del neumático.

Para simplificar la simulación se puede aproximar a la siguiente fórmula:

     Flong = Ct * slip ratio

     donde Ct es la constante de tracción; es decir, la pendiente de la curva de
     relación de desplazamiento en el origen.  Es interesante limitar
     la fuerza a un valor máximo para que no sobrepase dicho valor
     cuando la curva de desplazamiento sobrepase el valor máximo. La
     siguiente gráfica representa dicha curva:

     file:///home/isaac/Documentos/tfg/fisicas/Car%20Physics_files/ct_srapprox.gif

***********************************
Par motor sobre el eje de tracción
***********************************

Para recapitular, la fuerza de tracción es la fuerza de fricción que
la superficie de la carretera aplica sobre la superficie de las
ruedas. Obviamente, esta fuerza es causada por el par motor que aplica
el motor sobre los ejes de cada rueda.

    Par motor = Ftracción * Rruedas

Este par motor se opone al momento de torsión entregado por el motor a
cada rueda(que hemos llamado par motor de "conducción"). Si se frena,
tambien se causará momento de torsión. Para el freno, se va a suponer
que se entrega un par motor constante en la direccion opuesta a la
rotación de las ruedas. Hay que tener en cuenta esto para poder frenar
cuando se va marcha atrás. El siguiente diagrama ilustra estos
conceptos para un coche acelerando. El par motor es amplificado por
las marchas y el diferencial, proporcionando par a las ruedas
traseras. La velocidad angular de las ruedas es suficientemente alta
como para provovar deslizamiento entre la superficie del neumático y
la carretera, lo que puede ser expresado como un ratio de
deslizamiento positivo.  Esto resulta en una fuerza de fricción
reactiva, conocida como fuerza de tracción, que es lo que empuja el
coche hacia adelante. La fuerza de tracción tambien se traduce en un
par de tracción en las ruedas traseras que se opone al par de
impulso. En este caso, el par neto sigue siendo positivo y dará lugar
a una aceleración de la velocidad de rotación de las ruedas
traseras. Esto incremetnará los rpm y el ratio de deslizamiento.

file:///home/isaac/Documentos/tfg/fisicas/Car%20Physics_files/tc_torques.png

El par neto en el eje trasero es la suma de los siguientes pares:

ParMotorTotal = Par motor + par motor en ambas ruedas + par motor de frenado

Hay que recordar que los momentos de torsión son magnitudes con signo,
el momento de impulso normalmente tendrá signo opuesto a los de
tracción y de frenado. Si el conductor no frena, el momento de frenado
es cero.

El par total genera una velocidad angular sobre las ruedas que tienen
tracción, tal y como una fuerza aplicada sobre una masa hace que dicha
masa acelere:

   Aceleraciónangular = Par motor total / inercias de las ruedas de tracción.

La inercia de un cilindro sólido alrededor de un eje puede ser
calculado con la siguiente fórmula:

   InerciaCilindror = Masa * Radio^2 / 2

Así que para una rueda de 75 kg con un radio de 33 cm su inercia es de
75 * 0.33 * 0.33 / 2 = 4.1 kg.m2. Multiplicando por dos se obtiene la
inercia total de las dos ruedas del eje trasero, para mayor precisión
habría que añadir la inercia del propio eje, la inercia de los
engranajes y la del motor.

Una aceleracioń angular positiva incrementará la velocidad angular de
las ruedas traseras en el tiempo. Como la velicidad del coche depende
de la aceleración lineal, podemos simular esto realizando integración
lineal en cada simulación que realice nuestro gestor de físicas:

    rear wheel angular velocity += rear wheel angular acceleration * time step

Donde time step es la cantidad de tiempo entre llamadas al simulador
físico. De esta forma se puede determinar cuán rápido están girando
las ruedas de tracción y por lo tanto las rpm del motor.

***********************
El huevo y la gallina
***********************

Algunos lectores podrían estar confusos en este punto. Necesitamos los
rpm para calcular el par motor, pero el número de revoluciones depende
de la velocidad de rotación de las ruedas traseras, que a su vez
depende del par motor. Sin duda, esta es una definición circular.

Este es un ejemplo de una ecuación diferencial: tenemos ecuaciones
para las distintas variables que dependen mutuamente la una de la
otra. Pero ya hemos visto un ejemplo más de esto antes: la resistencia
del aire depende de la velocidad, sin embargo, la velocidad depende de
la resistencia del aire, ya que influye en la aceleración.

Para resolver ecuaciones diferenciales en los programas de ordenador
utilizamos la técnica de integración numérica: si conocemos todos los
valores en el tiempo t, podemos trabajar los valores en el tiempo t +
delta. En otras palabras, en lugar de tratar de resolver estas
ecuaciones mutuamente dependientes, tomamos instantáneas en tiempo y
resolvemos las ecuaciones para estos valores. Utilizamos los valores
de la iteración anterior para calcular los de la siguiente. Si el paso
de tiempo es lo suficientemente pequeño, este método funcionará
correctamente.

Existe multitud de teoría relacionada con el cálculo de ecuaciones
diferenciales e integración numérica. Uno de los problemas de la
integracion numérica es que un integrador puede "estallar" si el
intervalo de tiempo no es lo suficentemente pequeño. En lugar de dar
valores correctos, se disparán al infinito, debido a que estos
pequeños errores se multiplican rápidamente. La alternativa pasa por
usar integradores mas inteligentes; por ejemplo, RK4.

*******
Giros
*******

Una cosa a tener en cuenta cuando estamos simulando giros es que la
simulación de las propiedades física a baja velocidad es diferente de
la simulación a alta velocidad. A velocidades bajas (aparcamiento,
maniobras), las ruedas giran mas o menos en la dirección en la que
éstas apuntan. Para simular estos giros no se necesita considerar las
fuerzas y ni la masas. En otras palabras, es un problema de cinética
no de dinámica.

A velocidades más altas, puede ocurrir que las ruedas apunten en una
dirección mientras que se muevan en otra. En otras palabras, las
ruedas a veces pueden tener una velocidad que no esté alineada con la
orientación de la rueda. Esto significa que hay una componente de
velocidad que está en un ángulo recto a la rueda. Por supuesto, esto
causa mucha fricción. Después de todo una rueda está diseñado para
rodar en una dirección particular sin demasiado esfuerzo.  En giros a
alta velocidad, las ruedas están siendo empujadas hacia los lados y
tenemos que tomar estas fuerzas en cuenta.

Veamos el caso de giros a bajas velocidades. Podemos suponer que las
ruedas se están moviendo en la dirección que apuntan. En este caso,
las ruedas están rodando pero no se deslice hacia los lados. Si las
ruedas delanteras están giradas en un ángulo delta y el coche se está
moviendo a una velocidad constante, entonces el coche describirá una
trayectoria circular. Imagínese líneas que se proyectan desde el
centro de los hubcabs de la rueda delantera y trasera en el interior
de la curva. Cuando estas dos líneas se cruzan definen el centro de la
circuferencia que está realizando el giro del coche.

Esto está muy bien ilustrado en la siguiente figura. Note cómo las
líneas verdes se cruzan en un punto, el centro alrededor del cual el
vehículo está girando. También se puede notar que las ruedas
delanteras no están giradas en el mismo ángulo, la rueda exterior se
volvió un poco menos que la rueda interior. Esto es también lo que
sucede en la vida real, el mecanismo de dirección diferencial de un
automóvil está diseñado para girar las ruedas en un ángulo
diferente. Para una simulación de un coche puede que esta sutileza sea
tan importante. Se va a centrar la explicación en el ángulo de
dirección de la rueda delantera en el interior de la curva y se
ignorará la rueda en el otro lado.

El radio del círculo se puede determinar a través de cálculos
geométricos, como se ve en el siguiente diagrama:

file:///home/isaac/Documentos/tfg/fisicas/Car%20Physics_files/ctangles.jpg

La distancia entre el eje delantero y el trasero se calcula desde la base de
la rueda y denota como L. El radio del círculo que describe el coche
(para ser preciso el círculo que describe la rueda delantera) se llama
R. El diagrama muestra un triángulo con un vértice en el centro del
círculo y uno en el centro de cada rueda. El ángulo en la rueda
trasera es de 90 grados por definición. El ángulo en la rueda
delantera es de 90 grados menos delta. Esto significa que el ángulo en
el centro del círculo también es delta (la suma de los ángulos de un
triángulo es siempre 180 grados). El seno de este ángulo es la base de
la rueda dividido por el radio del círculo, por lo tanto:

file:///home/isaac/Documentos/tfg/fisicas/Car%20Physics_files/cteq_r.gif

Tenga en cuenta que si el ángulo de dirección es cero, entonces el
radio del círculo es infinito, es decir, que está conduciendo en línea
recta.  De esta forma podemos derivar el radio del círculo del ángulo
de dirección. Bien, el siguiente paso consiste en calcular la velocidad
angular, es decir, la velocidad a la que el coche gira. La velocidad
angular se suele representar mediante la letra griega omega (), y se
expresa en radianes por segundo. (Un radián es un círculo completo,
dividido por 2 pi). Es bastante sencillo de determinar: si estamos
conduciendo en círculos a una velocidad constante v y el radio del círculo
es R, ¿cuánto tiempo se tarda en completar un círculo? Esa es la
circunferencia dividida por la velocidad. En el momento en que el
coche ha descrito una trayectoria circular también ha girado alrededor
de su eje exactamente una vez. En otras palabras:

file:///home/isaac/Documentos/tfg/fisicas/Car%20Physics_files/cteq_av.gif

Mediante el uso de estas dos últimas ecuaciones, sabemos lo rápido que
el coche debe acudir en busca de un ángulo de giro dado a una
velocidad específica. Eso es todo lo que necesitamos para giros a
bajas velocidades. El ángulo de dirección se determina a partir de la
entrada del usuario. La velocidad del coche se determina de la misma
forma en que se calcula en movimientos rectilíneos (el vector de
velocidad siempre apunta en la dirección del coche). A partir de éste
se calcula el radio del círculo y la velocidad angular. La velocidad
angular se utiliza para cambiar la orientación del coche a una tasa
específica. La velocidad del coche no se ve afectado por el cambio, el
vector de velocidad sólo rota para que coincida con la orientación del
coche.

*****************************
Giros a altas velocidades
*****************************

Por supuesto, no hay muchos juegos que involucran autos que circulan
alrededor tranquilamente (aparte de la legendaria Trabant Granny
Racer;-). Los jugadores son impaciente y por lo general quieren llegar
a algún lugar a toda prisa, añadiendo derrapes, y destrozo de
mobiliario vario. El objetivo es encontrar un modelo de físicas que
permita vueltas subvirajes, sobreviraje, derrape, freno de mano, etc.

A altas velocidades, ya no podemos asumir que las ruedas se están
moviendo en la dirección que apuntan. Están unidas a la carrocería del
vehículo que tiene una cierta masa y lleva un toma tiempo al coche
reaccionar a las fuerzas de dirección. El cuerpo del coche también
puede tener una velocidad angular. Al igual que con la velocidad
lineal, lleva tiempo que ésta tome los valores que nosotros queremos
para que el coche gire hacia donde queramos. La velocidad angular
depende de la aceleración angular que es a su vez dependiente del par
de torsión y de la inercia (que son los equivalentes de rotación de la
fuerza y ​​de la masa).

Además, el propio vehículo no siempre se mueve en la dirección en que
quiere el conductor. Piense en pilotos de rally que pasan por una
curva. El ángulo entre la orientación del coche y vector de velocidad
del coche se conoce como el ángulo de deslizamiento lateral (beta).

file:///home/isaac/Documentos/tfg/fisicas/Car%20Physics_files/ctbeta.jpg

Ahora echemos un vistazo a alta velocidad en curva desde el punto de
vista de la rueda. En esta situación tenemos que calcular la velocidad
lateral de los neumáticos. Dado que las ruedas grian, tienen
relativamente baja resistencia al movimiento hacia adelante o hacia
atrás. sin embargo, las ruedas oponen resistencia a movimientos
perpendiculares a la dirección en la que apuntan. Pruebe empujando un
neumático del coche de lado. Esto es muy difícil porque hay que vencer
la fuerza máxima fricción estática para conseguir que la rueda se deslice.

En las curvas de alta velocidad, los neumáticos sufren unas las
fuerzas laterales también conocida como la fuerza de viraje. Esta
fuerza depende del ángulo de deslizamiento (alfa), que es el ángulo
entre el rumbo del neumático y su dirección de desplazamiento. A
medida que el ángulo de deslizamiento crece, también lo hace la fuerza
de viraje. La fuerza de viraje por neumático también depende del peso
sobre el neumático. En ángulos de deslizamiento bajos, la relación
entre el ángulo de deslizamiento y fuerza de viraje es lineal, en
otras palabras:

      Flateral = Ca * alpha
      donde la constante de Ca se conoce como la rigidez en las curvas.

Si desea ver esta explicado en una imagen, tenga en cuenta la
siguiente. El vector de velocidad de la rueda tiene un ángulo alfa con
respecto a la dirección en la que la rueda apunta. Podemos dividir el
vector velocidad v en dos componentes. El vector longtitudinal = cos
magnitud (alfa) * v. El movimiento en esta dirección se corresponde con
la dirección en la que giro la rueda. El vector lateral tiene
magnitud sen (alfa) * v y provoca una fuerza de resistencia en la
dirección opuesta: la fuerza de viraje.

file:///home/isaac/Documentos/tfg/fisicas/Car%20Physics_files/ctimage8.gif

Hay tres componentes que definen el ángulo de deslizamiento de las
ruedas: el ángulo de deslizamiento lateral del coche, la rotación
angular del coche alrededor del eje hacia arriba (velocidad de
derrape) y, para las ruedas delanteras, el ángulo de dirección.

El ángulo de deslizamiento lateral b (beta) es la diferencia entre la
orientación del vehículo y la dirección del movimiento. En otras
palabras, es el ángulo entre el eje longtitudinal y la dirección real
de viaje. Así que es similar en concepto a lo que el ángulo de
deslizamiento es para los neumáticos. Debido a que el coche puede
moverse en una dirección diferente a donde está apuntando, experimenta
un movimiento hacia los lados. Esto es equivalente a la componente
perpendicular del vector de velocidad.

file:///home/isaac/Documentos/tfg/fisicas/Car%20Physics_files/ctbeta.gif

Si el coche está girando alrededor del centro de masas (CG) a una tasa
omega (en rad / s!), esto significa que las ruedas delanteras
describen una trayectoria circular alrededor del centro de gravedad
CG. Si el coche realiza un círculo completo, la rueda delantera
habrá descrito una trayectoria circular de 2 * pi * b la  distancia alrededor de CG
en 1 / (2.pi.omega) segundos, donde b es la distancia desde el eje
delantero al CG. Esto se traduce en una velocidad lateral de omega *
b. Para las ruedas traseras, esto es -omega * c. Tenga en cuenta la
inversión del signo. Para expresar esto como un ángulo, se debe tomar el arco
tangente de la velocidad lateral dividida por la velocidad
longtitudinal (tal como lo hicimos para la beta). Para ángulos
pequeños podemos aproximar arctan (vy / vx) por vx / vy.

file:///home/isaac/Documentos/tfg/fisicas/Car%20Physics_files/ctav.gif

El ángulo de dirección (delta) es el ángulo que las ruedas delanteras
hacen en relación a la orientación del coche. No hay ángulo de dirección
de las ruedas traseras, ya que siempre están alineadas con la orientación
del cuerpo del coche. En el caso de coches con tracción delantera, el efecto de la
dirección invierte.

file:///home/isaac/Documentos/tfg/fisicas/Car%20Physics_files/ctdeltapic.gif

Los ángulos de deslizamiento para las ruedas delanteras y traseras
están dadas por las siguientes ecuaciones:

file:///home/isaac/Documentos/tfg/fisicas/Car%20Physics_files/ct_alphas.gif

La fuerza lateral ejercida por el neumático es una función del ángulo
de deslizamiento. De hecho, para los neumáticos reales es una función
bastante complejo una vez mejor descrito por diagramas de curvas,
tales como las siguientes:

file:///home/isaac/Documentos/tfg/fisicas/Car%20Physics_files/ctsacurve.gif

El diagrama anterior muestra cómo se comporta la fuerza lateral para
cualquier valor particular del ángulo de deslizamiento. Este tipo de
diagrama es específico para un tipo particular de neumático, siendo el
diagrama anterior un ejemplo ficticio. El pico está alrededor de los 3
grados. En ese punto la fuerza lateral supera incluso ligeramente la
carga de 5 kN en el neumático.

Este diagrama es similar a la curva de relación de deslizamiento visto
anteriormente lo que puede llevar a confusión. La curva de relación de
deslizamiento nos da la fuerza de avance en función de cantidad de
deslizamiento longtitudinal. La curva anterior nos da la fuerza
lateral en función del ángulo de deslizamiento.


La fuerza lateral no sólo depende del ángulo de
deslizamiento, sino también de la carga en el neumático. La
gráfica anterior muestra una gráfica donde el valor máximo de
la fuerza lateral asciende a 5000N; es decir, la fuerza
ejercida por 500 kg de masa empujando contra la superficie del
neumático. Diferentes curvas de fuerza aplican diferentes
fuerzas debido a que el peso cambia la forma del neumático y
por lo tanto sus propiedades. Pero la forma de la curva es muy
similar, aparte de la escala, por lo que se puede aproximar a
que la fuerza lateral es lineal con la carga y creamos un
diagrama de fuerza lateral normalizada dividiendo la fuerza
lateral por el 5 kN de carga.

        Flateral = Fn, lat * Fz
         donde Fnlat es la fuerza lateral normaliazda para un angulo de deslizamiento dado y
         Fz es la carga del neumático.

Para ángulos muy pequeños (por debajo del máximo) la fuerza lateral
puede ser aproximado por una función lineal:

        Flateral = Ca * alpha
        La constante de Ca se conoce con el
        nombre de la rigidez de las curvas. Esta es la pendiente del
        diagrama en ángulo de deslizamiento 0.

Si desea una mejor aproximación de la relación entre el ángulo de
deslizamiento y la fuerza lateral debe usar la fórmula mágica Pacejka
, desarrollada en la Universidad de Delft. Dicha fórmula es la que
usan los físicos para modelar el comportamiento de los
neumáticos. Es un conjunto de ecuaciones con una gran cantidad de
constantes "mágicas". Al elegir las constantes adecuadas estas
ecuaciones proporcionan una muy buena aproximación de curvas que se
encuentran a través de pruebas de neumáticos. El problema es que los
fabricantes de neumáticos son muy reservado acerca de los valores de
estas constantes toman. Así, por un lado, es una técnica de modelado
muy precisas. Por otro lado, usted tendrá un tiempo para encontrar
valores adecuados a los neumáticos que se estén usando.

Las fuerzas laterales de los cuatro neumáticos tienen dos resultados:
una fuerza neta en las curvas y un par alrededor del eje de giro. La
fuerza de viraje es la fuerza sobre el centro de gravedad en un ángulo
recto con la orientación del coche y sirve como la fuerza centrípeta
que es necesaria para describir una trayectoria circular. La
contribución de las ruedas traseras a la fuerza de viraje es la misma
que la fuerza lateral. Para las ruedas delanteras, multiplicar la
fuerza lateral por cos (delta) para permitir el ángulo de dirección.

Fcornering = Flat, rear + cos(delta) * Flat, front

Como punto de interés, podemos encontrar el radio del círculo ahora
que sabemos la fuerza centrípeta utilizando la siguiente ecuación

     Fcentripetal = M v2 / radius

La fuerza lateral también introducir un par que hace que el cuerpo del
coche para encender. Después de todo, sería muy tonto si el coche está
describiendo un círculo, pero sigue apuntando en la misma
dirección. La fuerza de viraje se asegura la CG describe un círculo,
pero ya que opera sobre una masa puntual no hace nada sobre la
orientación coche. Eso es lo que necesitamos el par alrededor del eje
de guiñada para.  El torque es la fuerza multiplicada por la distancia
perpendicular entre el punto donde se aplica la fuerza y el punto de
pivote. Así que para las ruedas traseras de la contribución a la par
es -Flat, trasera * c y para las ruedas delanteras es cos (delta) *
Piso *, delante b. Tenga en cuenta que el signo es diferente.

La aplicación de par de torsión sobre la carrocería del vehículo
introduce la aceleración angular. Al igual que la segunda ley de
Newton F = ma, hay una ley para el par y aceleración angular:

Torque = Inertia * angular acceleration.

La inercia de un cuerpo rígido es una constante que depende de su masa
y la geometría (y la distribución de la masa dentro de su
geometría). Manuales de ingeniería proporcionan fórmulas para la
inercia de las formas comunes tales como esferas, cubos, etc.
