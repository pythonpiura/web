.. title: Uso de DetailView
.. slug: uso-de-detailview
.. date: 2016-04-21 20:48:56
.. tags: Django
.. description: 

Luego de haber visto el uso de TemplateView, CreateView y ListView,
estos dos últimos los vimos de pasada casi sin mencionarlos, ahora
vamos a agregar un par de nuevas funcionalidades a nuestro proyecto
tutorial, para ello empezaremos modificando el archivo personas.html
para agregarle un par de enlaces que nos van a permitir editar y ver
el detalle de una persona en cuestión, en este post solamente
cubriremos la funcionalidad de ver el detalle, posteriormente en otro
post nos encargaremos de la funcionalidad de editar a la persona:

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
	                        <div class="col-lg-10">   
	                                                                                 
	                        </div>
	                        <div class="col-lg-1">
	                            <a id="crear_detalle" href="{% url 'personas:reporte_personas_excel' %}" class="btn btn-info btn-block">
	                                <span class="glyphicon glyphicon-list-alt"></span>
	                            </a>
	                        </div>    
	                        <div class="col-lg-1">                            
	                            <a id="crear_detalle" href="{% url 'personas:crear_persona' %}" class="btn btn-info btn-block">
	                                <span class="glyphicon glyphicon-plus"></span>
	                            </a>                          
	                        </div>                        
	                    </div>
	                </div>
	                <div class="row">
	                    <div class="col-lg-12">
	                        <table id="tabla" class="table table-striped table-bordered" cellspacing="0" width="100%">
	                            <thead>
	                                <tr>
	                                    <th class="text-center">DNI</th>
	                                    <th class="text-center">NOMBRE</th>
	                                    <th class="text-center">APELLIDO PATERNO</th>
	                                    <th class="text-center">APELLIDO MATERNO</th>
	                                    <th class="text-center">ACCIONES</th>                                   
	                                </tr>
	                            </thead>
	                            <tbody>
	                            {% for persona in personas %}
	                                <tr>
	                                    <td>{{ persona.dni }}</td>
	                                    <td>{{ persona.nombre }}</td>
	                                    <td>{{ persona.apellido_paterno }}</td>
	                                    <td>{{ persona.apellido_materno }}</td>
	                                    <td class="text-center">
	                                        <a class="btn btn-small" href="#">
	                                            <span class="glyphicon glyphicon-folder-open"></span>
	                                        </a>
	                                        <a class="btn btn-small" href="#">
	                                            <span class="glyphicon glyphicon-pencil"></span>
	                                        </a>                                                                                                                      
	                                    </td>                                                                     
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


Nótese que en la cabecera de la tabla hemos puesto el encabezado
acciones y que dentro del cuerpo de la tabla se ha agregado una
columna que contiene dos enlaces: uno con el ícono de un libro y otro
con el ícono de un lapiz, ambos son, para ver el detalle de la persona
y el otro para editarlo.

.. image:: /images/blog/tabla_nueva.jpg

Ahora vamos a implementar la funcionalidad de ver el detalle de la
persona, para ello usamos la clase DetailView, esta clase nos
simplifica la vida ya que nos permite mostrar el detalle de un modelo
en particular, debiendo definir el modelo al que haremos referencia y
la plantilla donde se renderizará el contenido:

.. code-block:: python

	from django.views.generic.detail import DetailView
	
	class DetallePersona(DetailView):
		model = Persona
		template_name = 'detalle_persona.html'


Ahora debemos crear la plantilla detalle_persona.html:

.. code-block:: html

	{% extends "base.html" %}
	{% block cuerpo %}
	<div class="row">
	    <div class="col-lg-12">
	        <h1 class="page-header">Personas</h1>
	    </div>    
	</div>
	<div class="row">
	    <div class="col-lg-12">
	        <div class="panel panel-info">
	            <div class="panel-heading">
	                Detalle de Persona
	            </div>
	            <div class="panel-body">
	                <div class='form-group'>
	                    <div class="row">
	                        <div class="col-md-12">
	                            <label>DNI:</label>
	                            <p>{{ object.dni }}</p>
	                            <label>NOMBRE:</label>
	                            <p>{{ object.nombre }}</p>
	                            <label>APELLIDO PATERNO:</label>
	                            <p>{{ object.apellido_paterno }}</p>
	                            <label>APELLIDO MATERNO: </label>
	                            <p>{{ object.apellido_materno }}</p>
	                        </div>
	                    </div>
	                </div>
	            </div>
	        </div>
	    </div>
	</div>
	{% endblock cuerpo %}


Modificamos el archivo urls.py para crear el url detalle_persona:

.. code-block:: python

	from django.conf.urls import patterns, url
	from personas.views import Personas, CrearPersona,
	ReportePersonasExcel,\
	Bienvenida, DetallePersona

	urlpatterns = patterns(",
		url(r'^$',Bienvenida.as_view(), name="bienvenida"),
		url(r'^personas/$',Personas.as_view(), name="personas"),
		url(r'^crear_persona/$',CrearPersona.as_view(), name="crear_persona"),
		url(r'^reporte_personas_excel/$',ReportePersonasExcel.as_view(),
		name="reporte_personas_excel"),
		url(r'^detalle_persona/(?P<pk>\d+)/$', DetallePersona.as_view(),
		name="detalle_persona"),
	)


Ahora hacemos que el enlace con el ícono del libro abierto apunte a
nuestra url detalle_persona, debemos tener en cuenta que el enlace
debe estar de acuerdo a la definición de la expresión regular en la
url, en este caso se le pasa como argumento la primary key que debe
ser un conjunto de dígitos:

.. code-block:: html

	<a class="btn btn-small" href="{% url 'personas:detalle_persona' persona.pk %}">
		<span class="glyphicon glyphicon-folder-open"></span>
	</a>


Ahora cuando hagamos click en el ícono del lápiz nos mostrará lo
siguiente:

.. image:: /images/blog/detalle_persona.jpg

Eso es todo.

Saludos.



