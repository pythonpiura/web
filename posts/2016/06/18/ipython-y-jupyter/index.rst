.. title: IPython y Jupyter
.. slug: ipython-y-jupyter
.. date: 2016-06-18 04:46:26
.. tags: ipython
.. description: 

Despues de un receso hemos regresado y esta vez para hablar de IPython
y Jupyter, como todos sabemos cuando estamos aprendiendo Python
utilizamos el interprete directamente desde una terminal y vamos
probando nuestros programas, al principio todo nos va muy bien pero
luego notamos que nos vamos quedando cortos, es alli donde aparece
`IPython`_ que tal como nos dice su pagina web nos provee una
arquitectura rica para computación interactiva con:


+ Una shell interactiva poderosa.
+ Un nucleo para Jupyter.
+ Soporte para visualización de datos interactivos y uso de
  herramientas GUI.
+ Interprete embebido flexible para cargar en nuestros propios
  proyectos.
+ Fácil de usar, herramientas de alta performance para computación
  paralela.


Vamos a instalarlo para ir conociendolo mejor, para ello vamos a
`crear un entorno virtual`_, al que llamaremos ipython:

.. code-block:: bash

	virtualenv ipython


Lo activamos:

.. code-block:: bash
	
	source ipython/bin/activate


Ahora procedemos a instalar ipython:

.. code-block:: bash

	pip install ipython


Para probar que todo ha salido bien ponemos lo siguiente en la
terminal:

.. code-block:: bash

	ipython


Y nos aparecerá algo como esto, que nos indica que todo esta
funcionando correctamente:

.. image:: /images/blog/seleccic3b3n_369.png

Escribimos exit para salir del interprete y regresar a nuestra
terminal:

.. code-block:: bash

	exit


Ahora conozcamos a `Jupyter`_, tal como nos dice su pagina web, es una
aplicación web que permite crear y compartir documentos que contienen
código vivo, ecuaciones, visualizaciones y texto explicativo. Se puede
usar para simulación numérica, simulación estadística, machine
learning y mucho mas.
Vamos a instalarlo para ir conociéndolo:

.. code-block:: bash

	pip install jupyter


Para correrlo debemos escribir lo siguiente:

.. code-block:: bash

	ipython notebook


Y automáticamente nos abrirá en una pestaña del navegador lo
siguiente:

.. image:: /images/blog/seleccic3b3n_370.png

Aqui podemos crear un nuevo notebook, dando click al boton new en la
parte superior derecha y seleccionando Python 2:

.. image:: /images/blog/seleccic3b3n_371.png

Nos aparece una especie de consola web donde podemos ir escribiendo
nuestros programas:

.. image:: /images/blog/seleccic3b3n_373.png


Y ejecutando su contenido, como si trabajáramos en nuestra vieja
terminal:


.. image:: /images/blog/seleccic3b3n_372.png


Maravilloso verdad, un interprete web, donde vamos escribiendo
nuestros programas y viendo su funcionamiento, agregando texto,
imagenes, generando graficos, podemos guardar cada noteboook,
exportarlo a pdf, intercambiarlo, etc, como veran es una herramienta
valiosisima para utilizar en la enseñanza, en el analisis de datos, en
los trabajos de investigacion, etc.

En la red podemos encontrar una gran de cantidad de notebooks listos
para descargar, hay tutoriales, tesis, trabajos de investigacion, etc.
A nosotros nos intereso el siguiente curso, que es introductorio al
leguaje Python:
`Python For Developers`_

Vamos a descargar el archivo `zip`_ conteniendo todos los notebooks y
lo guardamos en la ruta desde donde ejecutamos el comando “ipython
notebook”, lo descomprimimos y tendremos lo siguiente:

.. image:: /images/blog/seleccic3b3n_375.png

Como verán ahora ya nos aparece la carpeta donde esta el curso,
ingresamos a ella:

.. image:: /images/blog/seleccic3b3n_376.png

Ahora ingresamos a uno de los capítulos:

.. image:: /images/blog/seleccic3b3n_378.png

Seleccionamos el archivo con la extensión “ipynb” y veremos un hermoso
curso de introduccion a Python donde podemos ir interactuando con los
programas de ejemplo:

.. image:: /images/blog/seleccic3b3n_377.png


Genial no, ahora aprender Python y hacer demostraciones de código va a
ser mas divertido :-)

Eso es todo por hoy, saludos.

.. _IPython: https://ipython.org/
.. _zip: https://github.com/ricardoduarte/python-for-developers/zipball/master
.. _Python For Developers: http://ricardoduarte.github.io/python-for-developers/
.. _crear un entorno virtual: http://rukbottoland.com/blog/tutorial-de-python-virtualenv/
.. _Jupyter: http://jupyter.org/


