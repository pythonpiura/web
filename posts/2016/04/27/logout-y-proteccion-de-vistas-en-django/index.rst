.. title: Logout y Protección de Vistas en Django
.. slug: logout-y-proteccion-de-vistas-en-django
.. date: 2016-04-27 22:21:09
.. tags: Django
.. description:

En el post anterior habiamos creado un login con Django ahora nos toca
hacer lo mismo pero para salir de la sesión que habiamos iniciado,
para ello vamos a modificar nuestro archivo urls.py para crear la url
llamada salir que invoca al método logout definido en django:

.. code-block:: python

	from django.conf.urls import patterns, url
	from seguridad.views import Login
	from django.contrib.auth.views import logout

	urlpatterns = patterns(",
		url(r'^$', Login.as_view(), name="login"),
		url(r'^salir$', logout, name="salir", kwargs={'next_page': '/'}),
	)


Como podemos observar llamamos a la vista logout de
django.contrib.auth.views, que nos permite terminar la sesión que
hemos iniciado pasandole el argumento next_page que nos redirecciona a
la url raiz de nuestro proyecto, osea al login.

Ahora vamos a utilizar la url creada, hacemos una modificación a
nuestro archivo base.html para agregar un enlace de salida:

.. code-block:: html

	{% load staticfiles %}
	<!DOCTYPE html>
	<html>
	    <head>
	        <meta charset="utf-8">
	        <meta http-equiv="X-UA-Compatible" content="IE=edge">
	        <meta name="viewport" content="width=device-width, initial-scale=1">
	        <meta name="description" content="">
	        <meta name="author" content="">
	        <title>Ejemplo Círculo de Programadores de Python Piura</title>
	        <link rel="stylesheet" type="text/css" href="{% static 'css/bootstrap.min.css' %}">    
	        <link rel="stylesheet" type="text/css" href="{% static 'css/query.dataTables.min.css' %}"/>
	        <script src="{% static 'js/jquery.js' %}"></script>
	        <script src="{% static 'js/bootstrap.min.js' %}"></script>
	        <script src="{% static 'js/jquery.dataTables.min.js' %}"></script>
	    </head>
	    <body>
	        <div id="wrapper">
	            <nav class="navbar navbar-default navbar-static-top" role="navigation" style="margin-bottom: 0">  
	                <div class="navbar-header">
	                    <button type="button" class="navbar-toggle" data-toggle="collapse" data-target=".navbar-collapse">
	                        <span class="sr-only">Desplegar navegación</span>
	                        <span class="icon-bar"></span>
	                        <span class="icon-bar"></span>
	                        <span class="icon-bar"></span>
	                    </button>
	                    <a class="navbar-brand" href="{% url 'personas:bienvenida' %}">
	                        <span class="glyphicon glyphicon-home"></span> Inicio
	                    </a>
	                    <a class="navbar-brand" href="{% url 'personas:personas' %}">
	                        <span class="glyphicon glyphicon-user"></span> Personas
	                    </a>                  
	                </div>
	                <div class="navbar-right">
	                    <a class="navbar-brand" href="{% url 'seguridad:salir' %}">
	                        <span class="glyphicon glyphicon-log-out"></span> Salir   
	                    </a>                                                              
	                </div>                
	                {% block menu %}
	                {% endblock menu %}
	            </nav>
	            <div id="page-wrapper">
	                <div class="container-fluid">
	                    {% block cuerpo %}  
	                    {% endblock cuerpo %}
	                </div>
	            </div>
	        </div>
	    </body>
	    {% block js %}  
	    {% endblock js %}
	</html>


Con lo que tenemos lo siguiente:

.. image:: /images/blog/salir.jpg

Ahora ya podemos dar click en salir y terminar la sesión que estamos
utilizando, pero si observamos bien esto nos nos sirve de nada para
proteger nuestra aplicación de accesos no deseados, es decir que un
usuario sin estar logueado puede ingresar facilmente a cualquiera de
nuestras urls, con solo ponerla en el navegador, por ejemplo, pongamos
en la barra de direcciones:

`http://localhost:8000/personas/`_

Y observemos que podemos entrar sin ningún problema a pesar de no
estar logueados.

¿Cómo arreglamos esto? Sencillo, viene a nuestra ayuda un `decorador`_
muy útil llamado login_required, este va a proteger nuestra vista de
aquellos usuarios que pretenden ingresar sin tener una sesión
iniciada, para utilizarlo simplemente debemos modificar nuestro
archivo urls.py de la aplicación con las vistas que queremos proteger:

.. code-block:: python

	from django.conf.urls import patterns, url
	from personas.views import Personas, CrearPersona,
	ReportePersonasExcel,\
	Bienvenida, DetallePersona, ModificarPersona
	from django.contrib.auth.decorators import login_required

	urlpatterns = patterns(",
		url(r'^$',login_required(Bienvenida.as_view()), name="bienvenida"),
		url(r'^personas/$',login_required(Personas.as_view()),
		name="personas"),
		url(r'^crear_persona/$',login_required(CrearPersona.as_view()),
		name="crear_persona"),
		url(r'^reporte_personas_excel/$',login_required(ReportePersonasExcel.a
		s_view()), name="reporte_personas_excel"),
		url(r'^detalle_persona/(?P<pk>\d+)/$',
		login_required(DetallePersona.as_view()), name="detalle_persona"),
		url(r'^modificar_persona/(?P<pk>\d+)/$',login_required(ModificarPerson
		a.as_view()), name="modificar_persona"),
	)


Si probamos ahora poniendo nuevamente la dirección anterior en nuestra
barra de direcciones sin habernos logueado, nos aparecerá la siguiente
pantalla:

.. image:: /images/blog/error.jpg

¿Muy feo, no? Lamentablemente no podemos presentar esto al usuario
sino que quisieramos que cada vez que alguien intente acceder de esta
forma, nos redireccione a la pantalla de login, esto lo hacemos con
una linea agregada al archivo settings.py:

.. code-block:: python

	LOGIN_URL = '/'


Ahora probemos nuevamente:

.. image:: /images/blog/correcto.jpg

Con esto ya tenemos protegidas nuestras vistas y garantizamos que no
tendremos accesos no deseados.
Saludos.

.. _decorador: http://www.juanjoconti.com.ar/2008/07/11/decoradores-en-python-i/
.. _http://localhost:8000/personas/: http://localhost:8000/personas/


