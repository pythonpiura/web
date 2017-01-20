.. title: Instalación de Odoo 10 en Producción
.. slug: instalacion-odoo-10-en-produccion
.. date: 2017-01-16 23:56:00 UTC-05:00
.. tags: odoo
.. category: 
.. link: 
.. description: 
.. type: text

Ultimamente estamos trabajando con Odoo 10 y queremos compartir con ustedes la instalación de este genial software sobre un servidor Debian Jessie listo para ponerlo en producción, este es el primer artículo sobre el tema, el segundo estará siendo lanzado en los proximos días. Para escribir este artículo hemos tenido en cuenta los libros "Odoo Development Cookbook" y "Odoo	10	Development	Essentials"

Lo primero que debemos hacer es instalar Debian, los pasos para esto no los vamos a tener en cuenta en este artículo, ya que hay muy buenos tutoriales en la red.

Una vez instalado Debian es ingresar como root y actualizar el software:

.. code-block:: bash

	apt update && apt upgrade

Ahora instalamos los paquetes necesarios que nos permitiran el correcto funcionamiento de Odoo

.. code-block:: bash

	apt-get install git python2.7 postgresql nano python-virtualenv \
	gcc python2.7-dev libxml2-dev libxslt1-dev \
	libevent-dev libsasl2-dev libldap2-dev libpq-dev \
	libpng12-dev libjpeg-dev node-less node-clean-css \
	xfonts-75dpi xfonts-base

En odoo los reportes se generan usando la librería wkhtmltox, por lo que es necesario primero descargarla:

.. code-block:: bash

	wget http://nightly.odoo.com/extra/wkhtmltox-0.12.1.2_linux-jessie-amd64.deb

Y luego instalarla:

.. code-block:: bash

	dpkg -i wkhtmltox-0.12.1.2_linux-jessie-amd64.deb

Creamos el usuario odoo:

.. code-block:: bash

	adduser odoo

Instalamos sudo para poder realizar algunas operaciones necesarias con permisos de superusuario:

.. code-block:: bash

	apt install sudo

Creamos el usuario de base de datos odoo, esto lo hacemos con el usuario postgres:

.. code-block:: bash
	
	sudo -u postgres createuser odoo

Si nos aparece el sguiente mensaje:

.. code-block:: bash
	
	could not change directory to "/root": Permiso denegado

No debemos preocuparnos y procedemos a crear la base de datos odoo-project y la asignamos al usuario odoo:

.. code-block:: bash

	sudo -u postgres createdb -O odoo odoo-project

Ahora nos cambiamos al usuario odoo:

.. code-block:: bash

	su odoo

Creamos la carpeta odoo-prod donde vamos a guardar todos los archivos y carpetas necesarias de odoo:

.. code-block:: bash

	mkdir ~/odoo-prod

Fijémonos que antes del nombre de la carpeta tenemos los caracteres	"~/", con esto indicamos que no importa la carpeta donde estemos situados actualmente, siempre la creará en el home del usuario en sesión.

Ahora ingresamos a la carpeta creada:

.. code-block:: bash

	cd ~/odoo-prod

Y dentro de esta creamos las carpetas project y src dentro de project:

.. code-block:: bash

	mkdir -p project/src

Ingresamos a la carpeta src donde guardaremos el código de odoo:

.. code-block:: bash

	cd project/src

Ahora clonamos desde github el código fuente de odoo:

.. code-block:: bash

	git clone -b 10.0 --depth=1 https://github.com/odoo/odoo.git odoo

Observemos el parámetro --depth=1, este parámetro ignora la historia de los cambios y recupera sólo la última revisión de código, haciendo la descarga mucho más rápida.

Creamos un entorno virtual para trabajar con Odoo:

.. code-block:: bash
	
	virtualenv ~/env-odoo-10.0

Activamos el entorno virtual:

.. code-block:: bash

	source ~/env-odoo-10.0/bin/activate

Instalamos las dependencias necesarias para odoo 10:

.. code-block:: bash

	pip install -r odoo/requirements.txt

Hasta aquí ya podemos correr odoo sin ningún problema usando el script odoo-bin ubicado en el código fuente de odoo, pero continuaremos haciendo algunos pasos adicionales para obtener una mejor performance.

Procedemos a crear la carpeta bin dentro de project:

.. code-block:: bash

	mkdir ~/odoo-prod/project/bin

Ahora vamos a generar nuestro archivo de configuración de odoo, llamado production.conf, utilizando la siguiente orden:

.. code-block:: bash
	
	~/odoo-prod/project/src/odoo/odoo-bin --save --config ~/odoo-prod/project/production.conf --stop-after-init

Los parámetros --save y --config nos permiten crear y guardar el archivo de configuración en la ruta indicada, con el parámetro --stop-after-init paramos la ejecución de odoo y volvemos a la terminal.

Ahora vamos a crear el archivo que arrancará nuestro odoo, le vamos a denominar start-odoo y va a estar ubicado en la carpeta bin:

.. code-block:: bash

	nano ~/odoo-prod/project/bin/start-odoo

Este archivo es un script que contendrá lo siguiente:

.. code-block:: bash

	#! /bin/sh
	PYTHON=~odoo/env-odoo-10.0/bin/python
	ODOO=~odoo/odoo-prod/project/src/odoo/odoo-bin
	CONF=~odoo/odoo-prod/project/production.conf
	${PYTHON} ${ODOO} -c ${CONF}

El detalle de este archivo se explica de la siguiente manera:

-Tenemos el intérprete del entorno virtual, con esto nos despreocupamos de estar activando el entorno virtual cada vez que querramos arrancar el odoo.

-El script que viene en el código de odoo y que inicia la aplicación, se llama odoo-bin.

-El archivo de configuración de odoo, llamado production.conf

-Y finalmente el llamado a la orden que arranca odoo utilizando los parámetros contenidos en el archivo de configuración.

Le damos permisos de ejecución al script start-odoo

.. code-block:: bash

	chmod +x ~/odoo-prod/project/bin/start-odoo

Para probarlo arranquemos el script:

.. code-block:: bash

	~/odoo-prod/project/bin/start-odoo

Y probemos que funciona correctamente poniendo la IP del servidor y el puerto 8069:

http://IP_SERVER_ODOO:8069

Tuneando PostgreSQL
-------------------

La configuración predeterminada de PostgreSQL es generalmente muy conservadora y tiene la intención de evitar que el servidor de base de datos acapare todos los recursos del sistema. En servidores de producción, se pueden aumentar algunos parámetros en el archivo postgresql.conf para obtener un mejor rendimiento.

Vamos a editar el archivo postgresql.conf:

.. code-block:: bash
	
	nano /etc/postgresql/9.4/main/postgresql.conf 

Estas son algunas de las opciones que utilizaremos:

.. code-block:: bash

	max_connections = 100
	shared_buffers = 256MB
	effective_cache_size = 768MB
	work_mem = 10MB
	maintenance_work_mem = 64MB
	checkpoint_segments = 16
	wal_buffers = 8MB
	checkpoint_completion_target = 0.9

Para modificar el archivo procedemos a buscar cada uno de los parámetros y lo editamos o lo descomentamos según sea el caso.

Ahora reiniciamos el servicio de base de datos:

.. code-block:: bash
	
	service postgresql restart

Editando Archivo de Configuración de Odoo
-----------------------------------------

Ahora vamos a editar el archivo production.conf para modificar los parámetros necesarios para obtener un mejor rendimiento:

.. code-block:: bash

	nano ~/odoo-prod/project/production.conf

Los parámetros a modificar serán los siguientes:

Cambio del directorio de datos:

.. code-block:: bash
	
	data_dir = /home/odoo/odoo-prod/project/data

Cambio del log del servidor:

.. code-block:: bash

	logfile = /home/odoo/odoo-prod/project/logs/odoo.log

Configuración de la rotación del log, esto permite la rotación de registros. Esto hará que Odoo configure el módulo de registro para archivar los registros del servidor diariamente y mantener los registros antiguos durante 30 días. Esto es útil en servidores de producción para evitar registros que eventualmente consumen todo el espacio disponible en el disco.:

.. code-block:: bash
	
	logrotate = True

Configuración de los manejadores de login, esto configura el nivel de registro. La configuración propuesta es muy conservadora y sólo registrará mensajes con al menos el nivel WARNING, excepto werkzeug (CRITICAL) y openerp.service.server (INFO):

.. code-block:: bash

	log_level = warn
	log_handler = :WARNING,werkzeug:CRITICAL,openerp.service.server:INFO

Adaptación de los parámetros de conexión, esto funcionará si está ejecutando el servidor de base de datos PostgreSQL localmente y lo ha configurado como se explicó en anteriormente. Si se está ejecutando PostgreSQL en un servidor diferente, se tendrá que reemplazar los valores False con la configuración de conexión adecuada para su instancia de base de datos:

.. code-block:: bash

	db_host = False
	db_maxconn = 64
	db_name = odoo-project
	db_password = False
	db_port = False
	db_template = template1
	db_user = False

Configuración del filtro de bases de datos y del listado de bases de datos, esto restringe las bases de datos disponibles para la instancia configurando un filtro de base de datos.
Se desactiva el listado de las base de datos, esto no es estrictamente necesario dado que la expresión regular que establecemos en dbfilter sólo puede coincidir con una base de datos única. Sin embargo esto se hace para evitar mostrar la lista de bases de datos y que los usuarios se conecten a la base de datos incorrecta:

.. code-block:: bash

	dbfilter = odoo-project$
	list_db = False

Cambio del master password, la contraseña maestra se utiliza para la administración de la base de datos a través de la interfaz de usuario, y algunos addons de la comunidad también lo utilizan para añadir seguridad adicional antes de realizar acciones que pueden conducir a la pérdida de datos.:

.. code-block:: bash

	admin_password = mypassword

Configuración de los workers, Odoo creará una serie de procesos de trabajo (en este ejemplo, 4) para manejar las solicitudes HTTP. Esto tiene varias ventajas sobre la configuración por defecto en la cual el manejo de la petición se realiza en hilos separados:

.. code-block:: bash
	
	workers = 4
	limit_memory_hard = 4294967296 # 4 GB
	limit_memory_soft = 671088640 # 640MB
	limit_request = 8192
	limit_time_cpu = 120
	limit_time_real = 300

Si queremos que el sistema cargue datos de prueba debemos mantener el parámetro without_demo en False y si queremos lo contrario lo ponemos a True:

.. code-block:: bash
	
	without_demo = False

Arranque Automático
-------------------

Ahora vamos a configurar systemd para iniciar Odoo automáticamente.

Si aún seguimos logueados con el usuario odoo, debemos salir y regresar a trabajar con el usuario root:

.. code-block:: bash
	
	exit

Creamos el archivo llamado odoo.service en /lib/systemd/system/ de la siguiente manera:

.. code-block:: bash
	
	nano /lib/systemd/system/odoo.service

Este archivo tendrá el siguiente contenido:

.. code-block:: bash
	
	[Unit]
	Description=Odoo 10.0
	After=postgresql.service
	
	[Service]
	Type=simple
	User=odoo
	Group=odoo
	ExecStart=/home/odoo/odoo-prod/project/bin/start-odoo
	
	[Install]
	WantedBy=multi-user.target

Registramos el servicio:

.. code-block:: bash

	systemctl enable odoo.service

Iniciamos el servicio:

.. code-block:: bash
	
	service odoo start

Revisamos que al menos esté corriendo:

.. code-block:: bash
	
	service odoo status

Y lo paramos:

.. code-block:: bash
	
	service odoo stop

Para probar el funcionamiento reiniciemos el servidor y veamos que odoo esté corriendo correctamente:

.. code-block:: bash
	
	reboot

Ahora probemos nuevamente:

http://IP_SERVER_ODOO:8069

Y debemos ver la página de inicio del odoo.

El siguiente artículo nos permitirá configurar odoo con nginx y estará muy bueno, hasta la próxima.

Saludos.