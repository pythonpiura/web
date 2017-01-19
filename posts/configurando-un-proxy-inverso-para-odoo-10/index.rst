.. title: Configurando un Proxy Inverso para Odoo 10
.. slug: configurando-un-proxy-inverso-para-odoo-10
.. date: 2017-01-19 11:09:29 UTC-05:00
.. tags: 
.. category: 
.. link: 
.. description: 
.. type: text

Teniendo en cuenta la configuración realizada en nuestro anterior artículo `Instalación de Odoo 10 en Producción </posts/instalacion-odoo-10-en-produccion/>`_
en esta oportunidad, vamos a configurar un proxy inverso para poder acceder a nuestro odoo mediante el puerto 80, en otro artículo haremos lo mismo para el protocolo https usando el puerto 443, debemos recalcar que en esta oportunidad hemos utilizado como guía el libro "Odoo 10 Development Essentials" de Daniel Reis, entonces manos a la obra.

Odoo puede servir páginas web por si mismo, pero se recomienda tener un proxy Inverso en frente. Un proxy inverso actúa como un intermediario que gestiona el tráfico entre los clientes que envían solicitudes y el servidor Odoo que les responde. El uso de un proxy inverso tiene varios beneficios.

A nivel de seguridad:

- Manejar (y hacer cumplir) los protocolos HTTPS para cifrar el tráfico
- Ocultar las características de la red interna
- Actuar como un cortafuegos que limita las URL aceptadas para el procesamiento

A nivel de rendimiento:

- Caché de contenido estático, reduciendo así la carga en los servidores Odoo
- Comprimir contenido para acelerar los tiempos de carga
- Actuar como un equilibrador de carga, distribuyendola entre varios servidores.

Apache es una opción popular para usar como un proxy inverso, pero en esta oportunidad usaremos Nginx, Nginx es una alternativa reciente con buenas características técnicas y fácil de configurar.

Configurando Nginx como Proxy Inverso
-------------------------------------

Nginx, escucha en los puertos HTTP predeterminados(80), por lo que debemos asegurarnos que ninguna otra aplicación esté usando estos puertos, probaremos esto con la herramienta curl:

.. code-block:: bash

	apt-get install curl
	curl http://localhost

Si nos sale un mensaje como el siguiente entonces el puerto está libre:

.. code-block:: bash

	curl: (7) Failed to connect to localhost port 80: Conexión rehusada

Ahora procedemos a instalar nginx:

.. code-block:: bash
	
	apt-get	install	nginx

Para confirmar que está funcionando correctamente debemos ver una página de bienvenida cuando visitemos el servidor con un navegador web o usando curl:

http://IP_SERVER

Los archivos de configuración de Nginx siguen el mismo enfoque que Apache se almacenan en:

.. code-block:: bash

	/etc/nginx/available-sites/ 

Y se activan añadiendo un enlace simbólico en:

.. code-block:: bash

	/etc/nginx/enabled-sites

También debemos desactivar la configuración predeterminada proporcionada por la instalación de Nginx, borrando el archivo default:

.. code-block:: bash

	rm /etc/nginx/sites-enabled/default

Ahora creamos el archivo para odoo:

.. code-block:: bash

	nano /etc/nginx/sites-available/odoo

y le agregamos el siguiente contenido:

.. code-block:: bash

	upstream backend-odoo{
		server 127.0.0.1:8069;	
	}

	server{
		location / {
			proxy_pass http://backend-odoo;	
		}
	}

Primero, agregamos los upstreams, y los servidores backend Nginx redirigirán el tráfico al servidor Odoo en que está escuchando en el puerto 8069

Añadimos el enlace simbólico:

.. code-block:: bash

	ln -s /etc/nginx/sites-available/odoo /etc/nginx/sites-enabled/odoo

Para testear si la configuración es correcta hacemos lo siguiente:

.. code-block:: bash

	nginx -t

Si hay errores revisar si hemos escrito correctamente la configuración en el archivo odoo.

Recargamos nginx:

.. code-block:: bash

	/etc/init.d/nginx reload

Y visitamos nuestro servidor a través del navegador web:

.. code-block:: bash
	
	http://IP_SERVER

Este nos redirigirá automáticamente a Odoo, eso es todo por hoy. 

Saludos.