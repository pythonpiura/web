.. title: Guardando la Configuración de una Instancia de Odoo en un Archivo
.. slug: guardando-la-configuracion-de-una-instancia-de-odoo-en-un-archivo
.. date: 2017-01-21 11:35:33 UTC-05:00
.. tags: odoo
.. category: 
.. link: 
.. description: 
.. type: text

Seguimos configurando un entorno de desarrollo para Odoo 10 y tal como hemos visto en el `artículo anterior </posts/instalando-entorno-de-desarrollo-para-odoo-10-en-ubuntu-1604/>`_ podíamos levantar una instancia de Odoo con algunos parámetros agregados:

.. code-block:: bash

	python odoo-bin -d odoo10 --addons-path=addons

Odoo tiene múltiples parámetros de configuración, para ver que parámetros podemos utilizar ejecutamos el siguiente comando:

.. code-block:: bash

	python odoo-bin --help | less

Estar cargando todos estos parámetros cada vez que levantamos nuestra instancia es un poco tedioso, por eso tenemos la opción de guardar un archivo de configuración donde tengamos todos nuestros parámetros establecidos.

Suponiendo que estamos ubicados en la carpeta que contiene el código fuente de Odoo, para generar el archivo de configuración debemos correr el siguiente comando:

.. code-block:: bash

	python odoo-bin --save --config myodoo.cfg --stop-after-init

De esta manera generamos un archivo de configuración llamado myodoo.cfg(el nombre puede ser cualquiera) que se guardará en la misma carpeta donde está el script odoo-bin. 

Es posible guardar este archivo en cualquier ubicación, eso ya depende de nosotros y de como nos ubiquemos para utilizar el script odoo-bin, en nuestro caso siempre creamos el archivo de configuración en la carpeta superior a la que contiene el código fuente de odoo(si seguimos la secuencia del artículo anterior esta carpeta será odoo-dev-10), por lo que haremos lo siguiente:

.. code-block:: bash
	
	cd ~/odoo-dev-10
	python odoo/odoo-bin --save --config myodoo.cfg --stop-after-init

Si editamos el archivo myodoo.cfg y cambiamos algunos de los parámetros, es posible tomar estos cambios la siguiente vez que iniciemos nuestra instancia usando el parámetro -c, que es una opción abreviada de --config:

.. code-block:: bash
	
	python odoo/odoo-bin -c myodoo.cfg

Esto funciona de la siguiente manera, al arrancar, Odoo carga su configuración en tres pasos:

- Primero se inicializa un conjunto de valores predeterminados para todas las opciones desde el código fuente. 
- A continuación, se analiza la configuración y cualquier valor definido en el archivo reemplaza los valores predeterminados. 
- Finalmente, se analizan las opciones de la línea de comandos y sus valores anulan la configuración obtenida del paso anterior.

Los nombres de las variables de configuración se pueden transformar desde los nombres de las opciones de la línea de comandos, eliminando los guiones principales y convirtiendo los guiones medios en guiones bajos. Hay algunas excepciones en particular, pero para obtener mayor detalle de esto remitanse a la documentación de Odoo o a los libros que hemos recomendado en los artículos anteriores y que estamos usando como guía.

Saludos.