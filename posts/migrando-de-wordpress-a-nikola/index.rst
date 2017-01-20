.. title: Migrando de Wordpress a Nikola
.. slug: migrando-de-wordpress-a-nikola
.. date: 2016-07-02 05:01:23 UTC
.. tags: nikola
.. category: 
.. link: 
.. description: 
.. type: text

Despues de haber trabajado durante mucho tiempo en Wordpress decidimos usar algo mas pythonesco y encontramos el proyecto Nikola, que se adapta muy bien a lo que nosotros necesitamos como comunidad, asi que decidimos migrar, para ello seguimos la documentacion oficial del proyecto y algunos tutoriales adicionales, en este post vamos a relatar nuestra experiencia, esperando que sea util a nuestros lectores.

Primero creamos nuestro entorno virtual, en esta oportunidad usamos python3 para obtener un mejor rendimiento con Nikola,

.. code-block:: bash

	python3 -m venv nikola

Activamos el entorno virtual:

.. code-block:: bash

	source nikola/bin/activate

Antes de iniciar la instalacion de Nikola debimos instalar algunas dependencias para asegurarnos que todo funcione sin ningun problema:

.. code-block:: bash

	sudo apt-get install python3-dev
	sudo apt-get install libxml2-dev libxslt1-dev zlib1g-dev
	sudo apt-get build-dep python3-lxml python3-pil

Luego de ello debimos hacer una actualizacion de pip por un problema raro que se nos estaba presentando en la instalacion:

.. code-block:: bash

	pip install --upgrade pip

Luego procedimos a instalar Nikola y lo hicimos, de acuerdo a la sugerencia de su propia pagina, de la siguiente manera:

.. code-block:: bash

	pip install --upgrade "Nikola[extras]"

Nos demoramos un poquito, pero finalmente ya teniamos a Nikola instalado en nuestra maquina, como a nosotros no nos interesaba iniciar un sitio desde cero, sino migrar el que ya tenemos en wordpress, hicimos un backup del contenido del blog en wordpress, que es descargado en formato xml, el nuestro se llama pythonpiura.wordpress.2016-06-28.xml y las indicaciones para hacerlo son bastante sencillas y estan explicadas en la documentacion del propio wordpress.
Para hacer la migracion de Wordpress a Nikola leimos rapidamente el manual y encontramos que teniamos 3 opciones, probamos las tres para ver cual nos resultaba mejor, pero antes debimos ingresar en la ruta donde tenemos nuestro archivo xml con el backup, debemos mencionar que las tres opciones nos crean una carpeta new_site.

1-La opcion por defecto:

.. code-block:: bash

	nikola import_wordpress pythonpiura.wordpress.2016-06-28.xml

Esta opcion descarga todo el contenido del blog de wordpress e intenta hacer una conversion de cada uno de los posts y paginas al formato .md, no nos funciono correctamente ya que muchos posts se migraron a medias.

2-Convertir los posts a html:

.. code-block:: bash

	nikola import_wordpress --transform-to-html pythonpiura.wordpress.2016-06-28.xml

Para ello primero debemos instalar el plugin de conversion a wordpress

.. code-block:: bash

	nikola plugin -i wordpress_compiler
 
Esta opcion descarga el contenido y convierte los posts y paginas a html, esta fue la opcion que mejor nos funciono, salvo por el problema con los ejemplos de codigo que tenemos y que llevan la etiqueta:

.. code-block:: html

	[sourcecode language="language"]

Hasta donde hemos visto no es posible llevar a cabo la conversion de esto a un formato adecuado asi que simplemente los deja con el texto normal sin darle ningun formato, por lo que este todavia es un tema pendiente de resolver.

3-Dejar el contenido como formato de wordpress, los archivos de los posts y las paginas tienen las extension .wp

.. code-block:: bash

	nikola import_wordpress --use-wordpress-compiler pythonpiura.wordpress.2016-06-28.xml

Probamos esta opcion pensando que nos solucionaria el problema de la etiqueta [sourcecode], pero funciono igual que la opcion anterior y encima debiamos activar el plugin de wordpress en el archivo de configuracion del sitio.

Como comentamos antes usamos la segunda opcion que nos creo la carpeta new_site con el contenido listo, asi que ahora debiamos construir el sitio:

.. code-block:: bash

	nikola build

Y lanzar el servidor de pruebas:

.. code-block:: bash

	nikola serve -b

Cambiamos el tema por defecto por uno que nos parecio mas bonito llamado zen, pueden ver mas temas aqui:

https://themes.getnikola.com

Lo instalamos asi:

.. code-block:: bash
	
	nikola install_theme zen

Y lo configuramos en nuestro sitio modificando el archivo conf.py y cambiando la linea:

.. code-block:: python
	
	THEME = "bootstrap3"

por:

.. code-block:: python

	THEME = "zen"

y la linea:

.. code-block:: python

	NAVIGATION_LINKS = {
		DEFAULT_LANG: (
		    ("/archive.html", "Archives"),
		    ("/categories/index.html", "Tags"),
		    ("/rss.xml", "RSS feed"),
		),
	}

Por:

.. code-block:: python

	NAVIGATION_LINKS = {
	    DEFAULT_LANG: (
	        ('/index.html', 'Home', 'icon-home'),
	        ('/archive.html', 'Archives', 'icon-folder-open-alt'),
	        ('/categories/index.html', 'Tags', 'icon-tags'),
	        ('/rss.xml', 'RSS', 'icon-rss'),
	        ('https://getnikola.com', 'About me', 'icon-user'),
	        ('https://twitter.com/getnikola', 'My Twitter', 'icon-twitter'),
	        ('https://github.com/getnikola', 'My Github', 'icon-github'),
	    )
	}

Lo anterior es por defecto para que funcione, pero ya luego lo modificamos a nuestro gusto.

Volvimos a construir nuestro sitio:

.. code-block:: bash
	
	nikola build

Y a lanzar nuestro servidor:

.. code-block:: bash

	nikola serve -b

Ahora si nos tocaba deployar nuestro sitio en github:

Para ello creamos un repositorio con el mismo nombre de nuestro usuario "pythonpiura" pero de la siguiente manera:

pythonpiura.github.io

Clonamos este repositorio:

.. code-block:: bash

	git clone https://github.com/pythonpiura/pythonpiura.github.io.git

Lo siguiente que hicimos, fue copiar el contenido de nuestra carpeta new_site en la carpeta del repositorio recien creado.

Luego procedimos a modificar el archivo conf.py para que tenga los datos de nuetro nuevo sitio, modificando las siguientes lineas:

.. code-block:: python
	
	SITE_URL = "https://pythonpiura.github.io/"
	BLOG_EMAIL = "pythonpiura@openmailbox.org"

y tambien las siguientes para que se deploye correctamente en github:

.. code-block:: python

	GITHUB_SOURCE_BRANCH = 'sources'
	GITHUB_DEPLOY_BRANCH = 'master'
	GITHUB_REMOTE_NAME = 'origin'

Agregamos lo siguiente a nuestro archivo .gitignore para que este contenido no sea subido al repositorio:

.. code-block:: text
	
	cache
	.doit.db
	__pycache__
	output

Y corremos el comando:

.. code-block:: bash
	
	nikola github_deploy

En este paso github nos solicito nuestro usuario y password, lo ingresamos y continuamos. Tardo varios minutos en subir el contenido pero ya con eso tenemos nuestro blog en la siguiente direccion.

http://pythonpiura.org/

Saludos.
