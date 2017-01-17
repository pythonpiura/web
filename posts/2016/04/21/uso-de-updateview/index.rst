.. title: Uso de UpdateView
.. slug: uso-de-updateview
.. date: 2016-04-21 22:26:16
.. tags: Django
.. description: 

En este post vamos a terminar lo que dejamos inconcluso anteriormente,
el uso de la clase UpdateView, como ya tenemos listo nuestro archivo
personas.html, simplemente vamos a crear la clase y la url necesarias.

views.py

.. code-block:: python

	class ModificarPersona(UpdateView):
		#Especificamos que el modelo a utilizar va a ser Persona
		model = Persona
		#Establecemos que la plantilla se llamará modificar persona
		template_name = 'modificar_persona.html'
		#Determinamos los campos con los que se va a trabajar, esto es
		obligatorio sino nos saldrá un error
		fields = ['dni','nombre','apellido_paterno','apellido_materno']
		#Con esta linea establecemos que se hará despues que la operación de
		modificación se complete correctamente
		success_url = reverse_lazy('personas:personas')


Ahora creamos nuestra plantilla modificar_persona.html:

.. code-block:: html

	{% extends "base.html" %}
	{% block cuerpo %}
	 
	<div class="row">
	    <div class="col-lg-12">
	        <h1 class="page-header">Modificar Persona</h1>
	    </div>    
	</div>
	<div class="row">
	    <div class="col-lg-12">
	        <div class="panel panel-default">
	            <div class="panel-heading">
	                Por favor ingrese todos los campos necesarios.
	            </div>
	            <div class="panel-body">
	                <form role="form" method="post">
	                    {% csrf_token %}
	                    <div class="form-group">
	                        {{ form.as_p }}                         
	                    </div>    
	                    <div class='form-group'>
	                        <input type="submit" class="btn btn-primary" name="submit" value="Modificar Persona">
	                        <button type="reset" class="btn btn-primary" onclick="location.href='{% url 'personas:personas' %}'">
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


Ahora le toca el turno a urls.py:

.. code-block:: python

	from django.conf.urls import patterns, url
	from personas.views import Personas, CrearPersona,
	ReportePersonasExcel,\
	Bienvenida, DetallePersona, ModificarPersona

	urlpatterns = patterns(",
		url(r'^$',Bienvenida.as_view(), name="bienvenida"),
		url(r'^personas/$',Personas.as_view(), name="personas"),
		url(r'^crear_persona/$',CrearPersona.as_view(), name="crear_persona"),
		url(r'^reporte_personas_excel/$',ReportePersonasExcel.as_view(),
		name="reporte_personas_excel"),
		url(r'^detalle_persona/(?P<pk>\d+)/$', DetallePersona.as_view(),
		name="detalle_persona"),
		url(r'^modificar_persona/(?P<pk>\d+)/$',ModificarPersona.as_view(),
		name="modificar_persona"),
	)


Nótese que la última url añadida es la de modificar persona, y
funciona de manera parecida al detalle.

Ahora si hacemos que el enlace con el ícono del lapiz apunte a la url
de modificar_persona:

personas.html

.. code-block:: html

	<a class="btn btn-small" href="{% url 'personas:modificar_persona' persona.pk %}">
		<span class="glyphicon glyphicon-pencil"></span>
	</a>

Con lo que tenemos lo siguiente:

.. image:: /images/blog/modificar_persona.jpg

Saludos es todo por hoy.



