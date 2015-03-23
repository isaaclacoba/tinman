.. title: Añadiendo circuitos al juego
.. slug: anadiendo-circuitos-al-juego
.. date: 2015-03-14 11:46:24 UTC+01:00
.. tags: regex
.. link:
.. description: creacion de circuitos
.. type: text

En esta entrada se van a exponer los problemas que se han tenido que
resolver para poder añadir circuitos al juego. Se explicará la
situación al empezar a estudiar el problema y después se mostrará el
proceso que se ha segudio para crear nuevos circuitos.

.. TEASER_END: click to read the rest of the article

*******************************************************
Problemas iniciales y justificacion de las elecciones
*******************************************************

En cualquier videojuego 3D se hace necesario disponer de un software
que permita modelar los objetos que se quieren añadir a la escena, en
el caso de este proyecto, el circuito. El proceso de modelado consiste en dar forma
a cuerpos geométricos simples hasta alcanzar la forma deseada.

En este proyecto se ha usado `Blender
<http://www.blender.org/>`_. Blender es una herramienta de software
libre para realizar modelado, iluminación, renderizado, animación y
creación de gráficos tridimensionales.

.. En particular, destacan páginas como `cgcookies
.. <https://cgcookie.com/blender>`_, donde se pueden encontrar
.. completísimos tutoriales orientados a artistas, y `sketchfab
.. <https://sketchfab.com/>`_ o `blenderswap
.. <http://www.blendswap.com/>`_, donde los autores pueden compartir su
.. trabajo bajo diferentes licencias de tipo Creative Commons.

La razón por la que se ha elegido blender, en lugar de otras
herramientas como Maya3D, es que se distribuye bajo una licencia de
software libre (GNU GPL license) y cuenta con una grandísima
comunidad. Además, existen proyectos profesionales que respaldan la
calidad de Blender; por ejemplo, `Sintel
<https://www.youtube.com/watch?v=eRsGyueVLvQ>`_.

Tras la elección del software de modelado el siguiente problema que se
debía solucionar es la propia creación del circuito; es decir, decidir
la manera en que se iba a construir el circuito a partir de los
objetos modelados.

La decisión que se tomó fue la de crear el circuito como si se tratase
de un Scalextric. Se crearían una serie de piezas, de tal forma, que
encajaran unas con otras perfectamente. Las ventajas de esta
aproximación son:

- reduce el tiempo de creación de nuevos circuitos una vez que se tienen modeladas las piezas base. Esto es así debido a que los nuevos circuitos consisten en diferentes combinaciones de las piezas básicas. Cuando se quieran hacer nuevos circuitos no será necesario utilizar Blender.

- El hecho de codificar el circuito en formato de texto evita tener que recompilar el juego cada vez que queramos añadir nuevos circuitos.

- Permite distribuir de una menera fácil nuevos circuitos entre usuarios sin necesidad de recompilar el videojuego, ya que un circuito se codifica como un fichero de texto.

Las mayores desventajas de este método es la de tener que parsear el
fichero del circuito y la elección de texturas que al ser concatenadas
no revelen patrones que hagan ver que existe un punto de unión entre
las piezas del circuito. Sin embargo, la dificultad de parsear un
fichero de texto con un formato simple es trivial, de modo que en este
caso los pros superan altamente los contras, mientras que el segundo
problema se trata de algo puramente artístico.


***********************
Proceso de creación
***********************

En el siguiente vídeo se muestra el proceso de creación de la curva:

.. youtube::  emC5s0C4Q54

En el minuto 6:33 se puede ver como se corrigen las piezas para que
encajen perfectamente unas con otras. El modelado 3D requiere una
larga experiencia, ya que se trata de un proceso complejo con
muchísima casuística. Por otro lado, se ha realizado un
esfuerzo por que todo los modelos del juego tengan el mínimo número de
polígonos. En el vídeo no se muestra el proceso de limpiado de la
geometría, fundiendo caras o eliminandos vértices que puedan sobrar.

Una vez creadas las piezas, el siguiente paso consiste en guardar en
un fichero de texto la información relativa al circuito.

***********************************
Parseando el fichero del circuito
***********************************

Una vez decidido que el circuito se almacenaría en un fichero de texto,
se debía elegir qué biblioteca se iba a usar para parsearlo.

Aunque en C++ existen multitud de bibliotecas de `parseo
<http://es.wikipedia.org/wiki/Analizador_sint%C3%A1ctico>`_ para los
formatos de fichero mas comunes (por ejemplo
https://github.com/open-source-parsers/jsoncpp), se decidió crear una
implementación propia haciendo uso de la `biblioteca de expresiones
regulares <http://es.cppreference.com/w/cpp/regex>`_ que incluye
C++11.

Las razones por la cuales se decidió utilizar esta nueva
características del estándar es que se trata de una forma rápida y
sencilla de implementar un parser flexible que se pudiese ajustar a
cualquier estructura de fichero, al tiempo que se reducía el número de
dependencias del proyecto, ya que la biblioteca estándar viene
incluida por defecto. Por contra, la desventaja de
usar expresiones regulares es que puede ser relativamente complejo
crear la expresión que se adapta a la estructura del fichero.

=========================
Implementación del parser
=========================

Dado que podría ser innecesariamente complejo utilizar una estructura
en cascada, se decidió que cada línea del fichero tuviese información
autocontenida.  Cada una de las líneas del fichero del circuito sigue la siguiente estructura:

.. code:: bash

   section: { position: {x, y, z} rotation: {Delta} type: {Tipo} }

Siendo:

- x, y, z, Delta  variables de tipo float.
- Tipo una variable de tipo string. Toma los valores *rect* o  *curve*.

La implementación completa del parser se puede encontrar `aquí
<https://bitbucket.org/arco_group/tfg.tinman/src/a211ee73757119ffd56193e4ac02040eb8fbed18/src/util/parser.cpp?at=master>`_. De
lo anterior, el método más importante es el que nos devuelve un vector
de resultados:

.. code:: c++
   :number-lines:

   typedef std::vector<std::string> Match;
   typedef std::vector<Match> Results;

   Parser::Results
   Parser::get_results() {
      Parser::Results results;

      std::smatch matches;
      for (std::string line; getline(file_, line);) {
          if(std::regex_search(line, matches, pattern_)) {
            Parser::Match string_matches;
            string_matches.insert(string_matches.begin(), matches.begin(), matches.end());
            results.push_back(string_matches);
          }
      }
      file_.close();

      return results;
   }

Este método lee línea por línea un fichero, y por cada línea, ejecuta
el método std::regex_search. Este método busca en *line* cualquier
ocurrencia de *pattern_* (el patrón que estamos buscando) y lo
almacena en *matches*. Si no encuentra nada, devuelve falso. Es decir,
para cada línea del fichero, se comprueba que contenga la
estructura que le hemos indicado a través de la expresión regular
*pattern_* (el patrón de búsqueda). Si es así, almacena los resultados,
finalizando haya analizado todas las líneas del fichero *file_*,
termina.

En la línea 10 se puede ver que para realizar la búsqueda
se usa la función std::regex_search. Dicha función analiza la
variable *line* (una línea del fichero) en busca del patrón *pattern_*
( la expresión regular que define la estructura de una línea del
fichero) y en caso de encontrar una coincidencia, almacena el
resultado en la variable de salida *matches*. Dicha variable está
declarada en la línea 8. La variable *matches* es de tipo `std::smatch
<http://www.cplusplus.com/reference/regex/smatch/>`_, que es una
instanciación de la plantilla `std::match_results
<http://www.cplusplus.com/reference/regex/match_results/>`_ para
variables de tipo string. Un objeto de tipo *std::match_results* (la
variable *std::smatch* es un subtipo) tiene la siguiente estructura:

.. code:: bash

                -----------------------------------
               |               m[0]                |
    -----------------------------------------------------------
    | m.prefix | m[1] | m[2] | ... |  m[m.size()]  | m.suffix() |
     ------------------------------------------------------------


- m.prefix: son los caracteres que preceden al resultado de la búsqueda.

- m[0]: es el resultado de la búsqueda al completo.

- m.suffix: son los caracteres que siguen al resultado de la búsqueda.

La biblioteca de expresiones regulares permite definir subgrupos
dentro de la expresión regular. Estos subgrupos se definen englobando
entre paréntesis los elementos que queremos almacenar, y estan
representados en el diagrama anterior debajo de m[0]. Gracias a esto
podemos acceder de forma fácil a los elementos que nos interesen,
haciendo la labor de parseo del fichero muy sencilla. La única desventaja es que, en la implementación propuesta, los subgrupos son de tipo string. Es decir, se ha de saber a priori el tipo de dato de cada subgrupo para poder hacer el casting. Por ejemplo, cuando se accede a la posición, los componentes son de tipo float, de modo que hay que hacer un casting de string a float. La expresión
regular que define la estructura de nuestro fichero es la siguiente:

.. code:: bash

   (section: \{\sposition: \{(-?\b\d*.\d*), (-?\b\d*.\d*), (-?\b\d*.\d*)\}\s*rotation: \{(-?\d{1,3})\}\s*type: \{(\w*)\} \})

Existen multitud de `tutoriales <https://msdn.microsoft.com/en-us/library/bb982727.aspx>`_ que explican detalladamente cómo usar expresiones regulares.


***************
Trabajo futuro
***************

Debido a la enorme carga de trabajo de este proyecto, existen una gran
cantidad de características que se quedarán en el tintero.

Sin embargo, gracias a la gran flexibilidad que añade el mecanismo
explicado en este post, no tiene una gran complejidad crear un editor
gráfico de circuitos, de modo que se le de al jugador la posibilidad
de añadir nuevos a placer.

Por otra parte, estaría bien contar con mas tipos de segmentos de
circuito, en especial cambiando el tipo de suelo, de modo que añada
mas baches e incluso rampas, que permitan aprovechar de una forma mas
óptima el motor de físicas.
