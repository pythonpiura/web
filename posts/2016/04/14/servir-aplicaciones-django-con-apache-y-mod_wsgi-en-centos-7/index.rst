.. title: Servir Aplicaciones Django con Apache y mod_wsgi en Centos 7
.. slug: servir-aplicaciones-django-con-apache-y-mod_wsgi-en-centos-7
.. date: 2016-04-14 22:34:33
.. tags: Centos,Django,Python
.. description: 

Para instalar un servidor de producción Django en Centos 7 usaremos
Apache y mod_wsgi, mod_wsgi es un módulo de Apache, que permite servir
aplicaciones hechas en Python, que tengan soporte para la interfaz
WSGI.
Los requisitos para esto son tener un servidor Centos 7 correctamente
instalado y configurado y los permisos de root para poder hacer las
instalaciones correspondientes. Para comenzar el proceso, vamos a
descargar e instalar todos los elementos que necesitamos de los
repositorios de la distribución. Esto incluirá el servidor web Apache,
el módulo mod_wsgi utilizado para interactuar con nuestra aplicación
Django, y pip. Para obtener pip, tendremos que habilitar el
repositorio EPEL(paquetes para Linux Empresarial):

.. code-block:: bash

	yum install epel-release

Con EPEL habilitado nosotros podemos instalar los componentes
tipeando:

.. code-block:: bash

	yum install python-pip httpd mod_wsgi

Ahora que ya tenemos instalados los paquetes necesarios, debemos crear
un entorno virtual para ellos instalaremos virtualenv usando pip:

.. code-block:: bash

	pip install virtualenv

Ahora procedemos a crear nuestro entorno virtual:

.. code-block:: bash
	
	virtualenv vproduccion

Y lo activamos:

.. code-block:: bash
	
	source vproduccion/bin/activate

Cuando activamos el entorno virtual nuestro prompt debe cambiar y
aparecer entre paréntesis el nombre del entorno virtual activado.
Una vez que esto ha sucedido debemos instalar Django:

.. code-block:: bash
	
	pip install django


Cuando ya tengamos listo esto, subimos o creamos nuestro proyecto
Django utilizando los comandos clásicos que ya hemos utilizado antes,
ahora nos toca configurar Apache.
Una vez que comprobamos que el proyecto Django está funcionando,
podemos configurar Apache como un front-end. Las conexiones de cliente
que recibe serán traducidos al formato WSGI que la aplicación Django
espera utilizando el módulo mod_wsgi. Esto debería haber sido activado
automáticamente después de la instalación anterior. Para configurar lo
anterior, tendremos que crear un nuevo archivo de configuración en el
directorio /etc/httpd/conf.d. Vamos a llamar a este archivo
django.conf:

.. code-block:: bash
	
	nano /etc/httpd/conf.d/django.conf


Y dentro de él escribiremos lo siguiente:

.. code-block:: bash
	
	#En mi caso la ruta es /home/jamaya y mi proyecto se llama siad
	#Configuración para mostrar correctamente losa archivos estáticos
	Alias /static /home/jamaya/siad/static
	<Directory /home/jamaya/siad/static>
		Require all granted
	</Directory>
	#Configuración para acceder correctamente al wsgi.py
	<Directory /home/jamaya/siad/siad/>
		<Files wsgi.py>
			Require all granted
		</Files>
	</Directory>

	WSGIDaemonProcess siad python-path=/home/jamaya/siad:/home/jamaya/vproduccion/lib/python2.7/site-packages
	WSGIProcessGroup siad
	WSGIScriptAlias /siad /home/jamaya/siad/siad/wsgi.py

A continuación, tenemos que arreglar los permisos para que el servicio
de Apache puede acceder a nuestros archivos. Por defecto CentOS
bloquea el directorio personal de cada usuario de manera muy
restrictiva. Para evitar esto, añadiremos el usuario apache al grupo
de nuestro propio usuario. Esto nos permitirá abrir los permisos sólo
lo suficiente para que pueda llegar a los archivos correspondientes.
Añadimos el usuario apache a nuestro grupo, reemplazamos la palabra
jamaya con nuestro usuario:

.. code-block:: bash

	usermod -a -G jamaya apache

Damos permisos al grupo en la carpeta personal, esto permitirá al
proceso apache acceder a los archivos.

.. code-block:: bash
	
	chmod 710 /home/jamaya

Ya podemos iniciar el servicio de apache:

.. code-block:: bash
	
	systemctl start httpd

Vamos a probar si la carga del proyecto funciona, vamos a otra
computadora de la red y colocamos la direccion ip de nuestro servidor
y la aplicación:

.. code-block:: bash

	http://172.20.30.129/siad

Si hubiese algún problema lo mas rápido y sencillo es deshabilitar
SELINUX, esto lo hacemos editando el archivo:

.. code-block:: bash

	/etc/selinux/config


Y cambiando la siguiente linea:

.. code-block:: bash
	
	SELINUX=disabled


Si todo funciona correctamente habilitamos el servicio apache para que
se inicie automaticamente:

.. code-block:: bash

	systemctl enable httpd


Recuerden tener siempre su archivo requerimientos.txt o
requirements.txt para instalar todas las dependendencias que hacen que
nuestro proyecto funcione con éxito. Este archivo se obtiene en
nuestro entorno de desarrollo escribiendo el siguiente comando:

.. code-block:: bash

	pip freeze > requirements.txt


Y para instalarlas en nuestro entorno de producción usamos el mismo
comando con algunas variantes:

.. code-block:: bash

	pip install -r requirements.txt


En el caso nuestro teniamos un par de dependencias un poco complicadas
de instalar en Centos: Pillow y psycopg2, por lo que tuvimos algunos
errores, para solucionarlos debemos instalar algunas librerias, en
centos en el caso de psycopg2 tenemos:

.. code-block:: bash

	yum install gcc
	yum install postgresql-devel python-devel

Y en el caso de Pillow:

.. code-block:: bash

	yum install libjpeg-devel zlib-devel

Y listo eso es todo, ya tenemos nuestro servidor de producción.

Saludos.