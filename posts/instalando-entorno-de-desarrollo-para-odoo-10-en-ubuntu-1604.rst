.. title: Instalando Entorno de Desarrollo para Odoo 10 en Ubuntu 16.04
.. slug: instalando-entorno-de-desarrollo-para-odoo-10-en-ubuntu-1604
.. date: 2017-01-20 17:33:16 UTC-05:00
.. tags: odoo, ubuntu
.. category: 
.. link: 
.. description: 
.. type: text

Continuando nuestra serie de tutoriales sobre Odoo esta vez nos toca hacer la instalación de un entorno de desarrollo para Odoo sobre Ubuntu 16.04.

Primero instalamos las dependencias necesarias:

.. code-block:: bash

	sudo apt-get install git python2.7 postgresql nano python-virtualenv

Descargamos e instalamos wkhtmltox, esto es necesario para los reportes en pdf.

.. code-block:: bash

	wget http://nightly.odoo.com/extra/wkhtmltox-0.12.1.2_linux-jessie-amd64.deb
	dpkg -i wkhtmltox-0.12.1.2_linux-jessie-amd64.deb

Ahora instalamos las dependencias de desarrollo:

.. code-block:: bash

	sudo apt-get install gcc python2.7-dev libxml2-dev \
	libxslt1-dev libevent-dev libsasl2-dev libldap2-dev libpq-dev \
	libpng12-dev libjpeg-dev

Instalamos NodeJS y su manejador de paquetes:

.. code-block:: bash

	sudo apt-get install npm
	ln -s /usr/bin/nodejs /usr/bin/node

Instalamos el compilador less, a partir de la versión 9.0, el cliente web Odoo requiere el preprocesador less CSS para que las páginas web se representen correctamente, esto depende de Node.js y npm, que hemos instalado en el paso anterior.

.. code-block:: bash
	
	npm install -g less less-plugin-clean-css

Configuramos postgresql, primero creamos un usuario con posibilidad de crear bases de datos y que tenga nuestro mismo nombre de usuario de Ubuntu:

.. code-block:: bash
	
	sudo -u postgres createuser --createdb $(whoami)

Creamos la base de datos odoo10

.. code-block:: bash

	createdb odoo10

Configuramos git para poder descargar el código fuente de odoo:

.. code-block:: bash

	git config --global user.name "Tu nombre"
	git config --global user.email "Tu email"

Creamos la carpeta donde vamos a clonar odoo 10:

.. code-block:: bash

	mkdir ~/odoo-dev-10
	cd ~/odoo-dev-10
	git clone -b 10.0 --depth=1 https://github.com/odoo/odoo.git odoo
	cd odoo

Creamos el entorno virtual para odoo 10 y lo activamos:

.. code-block:: bash

	virtualenv ~/odoo-10
	source ~/odoo-10/bin/activate

Instalamos las python dependencias de Odoo en el entorno virtual:

.. code-block:: bash

	pip install -r requirements.txt

Ahora levantamos odoo con el siguiente comando:

.. code-block:: bash
	
	python odoo-bin -d odoo10 --addons-path=addons

- El script para inicializar odoo se llama odoo-bin. 
- El parámetro -d es para indicarle que vamos a trabajar con la base de datos odoo10.
- El parámetro --addons-path es para indicarle las rutas que contendrán los addons de Odoo.

Probamos el correcto funcionamiento de odoo usando nuestro navegador web y poniendo la siguiente dirección en la barra de direcciones:

http://localhost:8069

Nos logueamos con las siguientes credenciales:

- Usuario: admin
- Password: admin

En los siguientes tutoriales empezaremos a crear addons para Odoo, hasta la próxima.

Saludos.