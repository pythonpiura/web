.. title: DataTables JQuery, Bootstrap y Django
.. slug: datatables-jquery-bootstrap-y-django
.. date: 2016-04-18 22:35:28
.. tags: Django
.. description: 

Muchas veces al querer mostrar un listado o un conjunto de objetos,
necesitamos hacer uso de tablas, esto lo podemos trabajar de manera
sencilla usando una simple tabla html, pero que pasa cuando queremos
tener mayores prestaciones, como búsquedas, ordenamiento paginación,
etc. En este caso tenemos que recurrir a elementos mas avanzados, en
internet podemos encontrar muchísimas aplicaciones con tablas que nos
facilitan la vida, pero en esta oportunidad vamos a trabajar con una
herramienta que nos ha parecido muy interesante y que nos ha resultado
muy fácil de configurar y además trabaja con JQuery, la aplicación se
llama `DataTables`_ y como su página dice es un plugin para JQuery,
para empezar a trabajar debemos descargarla desde el siguiente
`enlace.`_

Para poder mostrar un sitio web mas o menos presentable vamos a echar
mano de Bootstrap, como ya es bien conocido en el mundo del desarrollo
web `Bootstrap`_ es un framework que nos facilita el diseño de
nuestras aplicaciones, contiene plantillas de diseño con tipografía,
formularios, botones, cuadros, menús de navegación y otros elementos
de diseño basado en HTML y CSS, así como, extensiones de JavaScript,
la última versión la podemos descargar de esta `ubicación`_. 

Ahora si, vamos a empezar creando nuestro proyecto en django, nosotros
utilizamos como IDE Eclipse con pydev, si quieren una guía para
configurarlo dense una vuelta por `aquí`_. Nuestro proyecto se llamará
tutorial, dentro del proyecto vamos a crear la aplicación personas, de
tal manera que tengamos la siguiente estructura:

Ahora vamos a crear una carpeta llamada static en la raiz del proyecto
y dentro de ella vamos a crear las carpetas css, js, fonts, images y
localizacion.

Nos toca configurar el archivo settings para que nuestros archivos
estáticos se muestren correctamente, agregamos la siguiente linea al
final del archivo.

settings.py

.. code-block:: python

	STATICFILES_DIRS = (
		os.path.join(BASE_DIR, 'static'),
	)


Y ahora como ya tenemos descargados tanto DataTables como Bootstrap
los descomprimimos, en DataTables buscamos la carpeta media, aquí
encontraremos las carpetas css, images y js, vamos a trabajar de la
manera mas sencilla que podamos por lo que no vamos a necesitar todo
el contenido de las carpetas antes mencionadas, así que de la carpeta
css seleccionamos el archivo jquery.dataTables.min.css y lo copiamos a
la carpeta css de nuestro proyecto, de la carpeta js seleccionamos los
archivos jquery.js y jquery.dataTables.min.js y los copiamos a la
carpeta js de nuestro proyecto,el contenido de la carpeta images si lo
copiamos integro dentro de nuestra carpeta images. Lo mismo hacemos
con Bootstrap copiamos los archivos bootstrap.min.css y
bootstrap.min.css.map a nuestra carpeta css, copiamos todo el
contenido de la carpeta fonts a nuestra carpeta fonts y finalmente
copiamos el archivo bootstrap.min.js a nuestra carpeta js.
Adicionalmente a esto debemos copiar el contenido del siguiente
`enlace`_ y crear un archivo en la carpeta localización que en este
caso le hemos llamado es_ES.json, esto sirve para traducir DataTables
al español.

Finalmente tendremos lo siguiente dentro de nuestra carpeta static:

Ahora vamos a crear una carpeta llamada templates que va a contener
nuestras plantillas html dentro de la aplicación personas. Dentro de
esta carpeta creamos un archivo llamado base.html que va a ser la
plantilla madre de la que heredarán las demás, este archivo tendrá el
siguiente contenido:

.. code-block:: html

	{% load staticfiles %}
	<!DOCTYPE html>
	<html>
	<head>
	Ejemplo Círculo de Programadores de Python Piura
	<link rel="stylesheet" type="text/css" href="{% static "css/bootstrap.min.css" %}">
	<link rel="stylesheet" type="text/css" href="{% static "css/jquery.dataTables.min.css" %}" />
	<script src="{% static "js/jquery.js" %}"></script>
	<script src="{% static "js/bootstrap.min.js" %}"></script>
	<script src="{% static "js/jquery.dataTables.min.js" %}"></script>
	</head>
	<body>
	<div id="page-wrapper">
	<div class="container-fluid">
	{% block cuerpo %}
	{% endblock cuerpo %}
	</div>
	</div>
	</body>
	{% block js %}
	{% endblock js %}
	</html>

Vamos a modificar el archivo urls.py del proyecto de tal manera que al
correrlo este apunte directamente a nuestra aplicación personas:

urls.py

.. code-block:: python

	from django.conf.urls import include, url
	from django.contrib import admin

	urlpatterns = [
	url(r'^admin/', include(admin.site.urls)),
	url(r'^', include('personas.urls',namespace='personas')),
	]

Ahora crearemos nuestro modelo y unas vistas bastante simples, en el
archivo models.py de la aplicación personas, creamos el modelo
Persona:

models.py

.. code-block:: python

	from django.db import models

	# Create your models here.
	class Persona(models.Model):
	dni= models.CharField(primary_key=True,max_length=8)
	nombre = models.CharField(max_length=100)
	apellido_paterno = models.CharField(max_length=100)
	apellido_materno = models.CharField(max_length=100)

Creamos dos vistas con sus respectivas urls:
views.py

.. code-block:: python

	from personas.models import Persona
	from django.core.urlresolvers import reverse_lazy
	from django.views.generic.edit import CreateView
	from django.views.generic.list import ListView

	# Create your views here.
	class CrearPersona(CreateView):
	model = Persona
	fields =['dni','nombre','apellido_paterno','apellido_materno']
	template_name = 'crear_persona.html'
	success_url = reverse_lazy('personas:personas')

	class Personas(ListView):
	model = Persona
	template_name = 'personas.html'
	context_object_name = 'personas'


urls.py

.. code-block:: python

	from django.conf.urls import patterns, url
	from personas.views import Personas, CrearPersona

	urlpatterns = patterns(",
	url(r'^crear_persona/$',CrearPersona.as_view(), name="crear_persona"),
	url(r'^',Personas.as_view(), name="personas"),
	)

Ahora debemos crear la plantilla:

personas.html

.. code-block:: html

	{% extends "base.html" %}
	{% block cuerpo %}
	<div class="row">
	<div class="col-lg-12">
	<h1 class="page-header">Tablas</h1>
	</div>
	</div>
	<div class="row">
	<div class="col-lg-12">
	<div class="panel panel-info">
	<div class="panel-heading">
	Personas
	</div>
	<div class="panel-body">
	<div class='form-group'>
	<div class="row">
	<div class="col-lg-3">

	</div>
	<div class="col-lg-8">

	</div>
	<div class="col-lg-1">
	<a id="crear_detalle" href="{% url 'tablas:crear_persona' %}"
	class="btn btn-info btn-block">
	<span class="glyphicon glyphicon-plus"></span>
	</a>
	</div>
	</div>
	</div>
	<div class="row">
	<div class="col-lg-12">
	<table id="tabla" class="table table-striped table-bordered"
	cellspacing="0" width="100%">
	<thead>
	<tr>
	<th class="text-center">DNI</th>
	<th class="text-center">NOMBRE</th>
	<th class="text-center">APELLIDO PATERNO</th>
	<th class="text-center">APELLIDO MATERNO</th>
	</tr>
	</thead>
	<tbody>
	{% for persona in personas %}
	<tr>
	<td>{{ persona.dni }}</td>
	<td>{{ persona.nombre }}</td>
	<td>{{ persona.apellido_paterno }}</td>
	<td>{{ persona.apellido_materno }}</td>
	</tr>
	{% endfor %}
	</tbody>
	</table>
	</div>
	</div>
	</div>
	</div>
	</div>
	</div>
	{% endblock cuerpo %}
	{% block js %}
	<script>
	$(document).ready(function()
	{
	var table = $('#tabla').DataTable( {
	"language": {
	url: "/static/localizacion/es_ES.json"
	}
	} );

	$('#tabla tbody').on( 'click', 'tr', function()
	{
	if ($(this).hasClass('selected') )
	{
	$(this).removeClass('selected');

	}
	else
	{
	table.$('tr.selected').removeClass('selected');
	$(this).addClass('selected');
	}
	});

	});
	</script>
	{% endblock js %}

Como hemos visto lineas arriba es en esta plantilla donde se hace uso
de DataTables, primero se crea la tabla teniendo en cuenta que se la
ha puesto el id “tabla" y además se hace la traducción al español
invocando al archivo es_ES.json:

.. code-block:: javascript

	var table = $('#tabla').DataTable( {
	"language": {
	url: "/static/localizacion/es_ES.json"
	}
	} );

Lo siguiente es un efecto que se le da al seleccionar una de las filas
de la tabla.

Y finalmente creamos el archivo:

crear_persona.html

.. code-block:: html

	{% extends "base.html" %}
	{% block cuerpo %}

	<div class="row">
	<div class="col-lg-12">
	<h1 class="page-header">Tablas</h1>
	</div>
	</div>
	<div class="row">
	<div class="col-lg-12">
	<div class="panel panel-default">
	<div class="panel-heading">
	Crear Persona
	</div>
	<div class="panel-body">
	<form role="form" action="#" method="post">
	{% csrf_token %}
	<div class="form-group">
	{{ form.as_p }}
	</div>
	<div class='form-group'>
	<input type="submit" class="btn btn-primary" name="submit"
	value="Crear Persona">
	<button type="reset" class="btn btn-primary"
	onclick="location.href='{% url 'personas:personas' %}'">
	Cancelar
	</button>
	</div>
	</form>
	</div>
	</div>
	</div>
	</div>
	<div id="popup"></div>
	{% endblock cuerpo %}

	{% block js %}

	{% endblock js %}

Hacemos las migraciones correspondientes de nuestros modelos y
finalmente corremos el servidor de prueba, si todo ha salido bien
tendremos una aplicación como la siguiente:

.. _Bootstrap : http://getbootstrap.com/
.. _aquí: http://www.pythondiario.com/2013/06/eclipse-y-pydev-configuracion-del-ide.html
.. _enlace: https://www.datatables.net/plug-ins/i18n/Spanish
.. _enlace.: https://datatables.net/download/download
.. _ubicación: https://github.com/twbs/bootstrap/releases/download/v3.3.6/bootstrap-3.3.6-dist.zip
.. _DataTables : https://datatables.net/