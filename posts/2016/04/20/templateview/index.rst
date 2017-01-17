.. title: Uso de TemplateView
.. slug: templateview
.. date: 2016-04-20 21:20:23
.. tags: Django
.. description:

Vamos a trabajar con TemplateView para hacer una vista basada en
clases, algo que ya hemos utilizado antes y que es bastante fácil de
programar, para ello seguiremos con nuestro proyecto tutorial, primero
le vamos a hacer unas pequeñas modificaciones para que se adapte a lo
que necesitamos.
Editamos el archivo base.html para agregar un pequeño menú en la parte
superior de la pantalla:

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
	        <link rel="stylesheet" type="text/css" href="{% static 'css/jquery.dataTables.min.css'  %}"  />
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
	                    <a class="navbar-brand" href="#">
	                        <span class="glyphicon glyphicon-home"></span> Inicio
	                    </a>
	                    <a class="navbar-brand" href="#">
	                        <span class="glyphicon glyphicon-user"></span> Personas
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


Tendremos algo como esto:

.. image:: /images/blog/menu.jpg

Creamos un archivo llamado bienvenida.html para poner un aviso de
bienvenida y de esta manera no ingresar directamente a nuestra tabla
de personas, para ello debemos crear una vista usando TemplateView y
agregar el url correspondiente:

bienvenida.html

.. code-block:: html

	{% extends "base.html" %}
	{% block cuerpo %}
	<div class="row">
	    <div class="col-lg-12">
	        <h1 class="page-header">Círculo de Programadores de Python Piura</h1>
	    </div>    
	</div>
	<div class="row">
	    <div class="col-lg-12">
	        <div class="panel panel-info">
	            <div class="panel-heading">
	                Bienvenido
	            </div>
	            <div class="panel-body">
	                <div class='form-group'>
	                    <div class="row">
	                        <h3 class="text-center">Bienvenido.</h3>                        
	                    </div>
	                </div>
	            </div>
	    </div>
	</div>    
	{% endblock cuerpo %}


Creamos una clase llamada Bienvenida, que hereda de TemplateView y le
decimos que la plantilla a utilizar se llamará bienvenida.html.

views.py

.. code-block:: python

	from django.views.generic.base import TemplateView

	class Bienvenida(TemplateView):
		template_name = "bienvenida.html"


En el archivo urls.py cambiamos la url raiz para que apunte a la clase
Bienvenida y creamos una url llamada personas para mostrar la tabla
personas.

urls.py

.. code-block:: python

	from django.conf.urls import patterns, url
	from personas.views import Personas, CrearPersona,
	ReportePersonasExcel,\
	Bienvenida

	urlpatterns = patterns(",
		url(r'^$',Bienvenida.as_view(), name="bienvenida"),
		url(r'^personas/$',Personas.as_view(), name="personas"),
		url(r'^crear_persona/$',CrearPersona.as_view(), name="crear_persona"),
		url(r'^reporte_personas_excel/$',ReportePersonasExcel.as_view(), name="reporte_personas_excel"),
	)


Con lo que tendriamos algo como esto:

.. image:: /images/blog/nueva.jpg

¿Y ahora como vemos nuestra tabla personas? Muy fácil haciendo que el
enlace llamado “Personas" apunte a la url personas y de paso también
hacemos que el enlace “Inicio" apunte a la url de bienvenida, para
ello modificamos el archivo base.html:

.. code-block:: html

	<a class="navbar-brand" href="{% url 'personas:bienvenida' %}">
		<span class="glyphicon glyphicon-home"></span> Inicio
	</a>
	<a class="navbar-brand" href="{% url 'personas:personas' %}">
		<span class="glyphicon glyphicon-user"></span> Personas
	</a>


Con esto hemos podido ver el uso sencillo de TemplateView mas adelante
vamos a utilizar otras vistas basadas en clases, hasta la próxima.
Saludos.



