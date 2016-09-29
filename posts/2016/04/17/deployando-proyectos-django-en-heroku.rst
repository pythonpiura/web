.. title: Deployando Proyectos Django en Heroku
.. slug: deployando-proyectos-django-en-heroku
.. date: 2016-04-17 16:44:49
.. tags: Django,Python
.. description: 


`Heroku`_ es una plataforma que brinda servicios en la nube y soporta
varios lenguajes de programación, funciona muy bien con Python y
Django y en esta ocasión haremos un pequeño tutorial basado en el
tutorial oficial, que se puede encontrar en la página del proyecto.
Para empezar a trabajar debemos tener una cuenta en Heroku, hay planes
de varios tipos, nosotros escogeremos el plan Free. Antes de empezar
nos vamos a cerciorar de tener instaladas algunas cosas básicas, que
ya hemos visto antes pero que siempre es bueno recordar:


+ Pip.
+ Virtualenv
+ Git.
+ Y en nuestro caso la base de datos PostgreSQL.


El primer paso es instalar Heroku Toolbelt, esta aplicación provee
acceso a la interfaz de linea de comandos(CLI) de Heroku, que es usada
para administrar y escalar nuestras aplicaciones y sus añadidos, para
tenerlo instalado debemos poner el siguiente comando en nuestra
terminal:

.. code-block:: bash

	wget -O- https://toolbelt.heroku.com/install-ubuntu.sh | sh


El script nos pedira permisos de superusuario para continuar, se los
damos y esperamos que la instalación se efectue.

Una vez que hemos terminado de instalar debemos loguearnos utilizando
el usuario y password que hemos registrado en la página de Heroku de
la siguiente manera:

.. code-block:: bash

	heroku login


Si es la primera vez que lanzamos la aplicación se demorará un poquito
instalando y configurando todo lo necesario, despues nos pedirá
usuario y password, si ingresamos las credenciales correctamente nos
saldrá un mensaje como este:

.. code-block:: bash

	Logged in as miguel.amaya99@gmail.com


Una vez que hemos hecho esto el manual oficial de Heroku nos indica
que debemos clonar una aplicación de prueba para deployar, nosotros ya
tenemos una aplicación desarrollada por lo que debemos hacer algunos
cambios a esta para poder subirla sin ningún problema.

Primero debemos agregar dos archivos en la raiz del proyecto:

El archivo procfile que debe tener el siguiente contenido:

.. code-block:: python

	web: gunicorn volpox.wsgi -log-file -


En la parte donde dice “volpox.wsgi” se debe reemplazar por el nombre
de su proyecto.

El archivo requirements.txt debe tener todos las aplicaciones que
nuestro proyecto necesita para funcionar correctamente, esto se puede
averiguar facilmente ejecutando el comando pip freeze y guardando su
contenido en el archivo, en nuestro caso el contenido del archivo es
el siguiente:

.. code-block:: python

	dj-database-url==0.4.0
	Django==1.9.2
	gunicorn==19.4.5
	psycopg2==2.6.1
	whitenoise==2.0.6


Debemos tener en cuenta que este es el contenido básico para
garantizar el correcto funcionamiento de nuestra aplicación en Heroku,
hemos adicionado el paquete psycopg2 para poder trabajar con
postgresql, si su proyecto usa algún paquete de software adicional,
póngalo en este archivo.

Ahora debemos modificar el archivo settings.py agregando las
siguientes lineas:

.. code-block:: python

	#Al inicio

	import dj_database_url

	PROJECT_ROOT = os.path.dirname(os.path.abspath(__file__))


En la parte donde está:

.. code-block:: python

	ALLOWED_HOSTS = []


Lo cambiamos por:

.. code-block:: python

	ALLOWED_HOSTS = ['*']


En la parte final agregamos esto:

.. code-block:: python

	db_from_env = dj_database_url.config(conn_max_age=500)
	DATABASES['default'].update(db_from_env)

	# Honor the 'X-Forwarded-Proto' header for request.is_secure()
	SECURE_PROXY_SSL_HEADER = ('HTTP_X_FORWARDED_PROTO', 'https')
	# Static files (CSS, JavaScript, Images)
	# https://docs.djangoproject.com/en/1.8/howto/static-files/

	STATIC_ROOT = os.path.join(PROJECT_ROOT, 'staticfiles')

	STATIC_URL = '/static/'

	# Extra places for collectstatic to find static files.
	STATICFILES_DIRS = (
	os.path.join(BASE_DIR, 'static'),
	)
	STATICFILES_STORAGE =
	'whitenoise.django.GzipManifestStaticFilesStorage'


Ahora debemos cambiar el contenido del archivo wsgi.py

.. code-block:: python

	import os
	os.environ.setdefault("DJANGO_SETTINGS_MODULE", "volpox.settings")

	from django.core.wsgi import get_wsgi_application
	from whitenoise.django import DjangoWhiteNoise

	application = get_wsgi_application()
	application = DjangoWhiteNoise(application)


También debemos crear un archivo .gitignore, que es usado por git para
no trabajar con las extensiones de archivos mencionados aquí en
nuestro caso serán las siguientes:

.. code-block:: python

	venv
	*.pyc
	staticfiles
	.env
	db.sqlite3


Ahora debemos crear un repositorio git con el siguiente comando:

.. code-block:: bash

	git init


Añadimos los archivos:

.. code-block:: bash

	git add *


Y hacemos nuestro primer commit:

.. code-block:: bash
	
	git commit -m "Deployando proyecto"


Creamos nuestro proyecto en Heroku:

.. code-block:: bash

	heroku create volpox


Ahora si procedamos a subir nuestro proyecto a Heroku:

.. code-block:: bash
	
	git push heroku master


Si tuviesemos algún error, revisar los archivos de la carpeta static,
generalmente la mayoría de errores se presentan por rutas no
encontradas o cosas así.

Si todo ha salido correctamente vamos a proceder a asegurarnos de que
al menos una instancia de la aplicación se está ejecutando:

.. code-block:: bash

	heroku ps:scale web=1


Ahora abrimos el proyecto con el navegador por defecto:

.. code-block:: bash
	
	heroku open


Listo, ya tenemos nuestra aplicación funcionando correctamente, ahora
falta configurar la base de datos, para ello vamos a utilizar el
siguiente comando:

.. code-block:: bash
	
	heroku addons


Con esto añadimos una base de datos PostgreSQL en el plan Free de
nuestra cuenta.
Los siguiente comandos:

.. code-block:: bash

	heroku config
	heroku pg


Nos permiten ver la configuración de nuestra base de datos.

Procedemos a crear nuestras tablas en la base de datos creada, usando
el comando migrate de la siguiente manera:

.. code-block:: bash

	heroku run python manage.py migrate


Si es necesario procedemos a crear nuestro superusuario:

.. code-block:: bash

	heroku run python manage.py createsuperuser


Listo ya tenemos nuestra aplicación desplegada. Hasta la próxima.

.. _Heroku: https://www.heroku.com/
