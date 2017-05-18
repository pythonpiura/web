.. title: Modales en Django con Vistas Basadas en Clases y JQuery-UI
.. slug: modales-en-django-con-vistas-basadas-en-clases-y-jquery-ui
.. date: 2017-05-17 21:02:54 UTC-05:00
.. tags: Django, jquery-ui, modales
.. category: 
.. link: 
.. description: 
.. type: text

En este post vamos a usar ventanas modales para poder hacer operaciones de visualización, creación y modificación de entidades.
Para este fin hemos creado un proyecto llamado "modales" y una aplicación "productos" donde vamos a tener el modelo Producto, para poder mostrar los modales haremos uso de la librería JQuery-UI y de bootstrap para ayudarnos con el diseño, jquery como librería javascript y Datatables para mostrar las tablas de la manera mas cómoda. Los pormenores de creación del proyecto y la correspondiente aplicación serán obviados por haber sido tratados en artículos anteriores.

Primero creamos nuestro archivo base.html que va a contener la carga de las librerias necesarias:

base.html

.. code-block:: html

	<!DOCTYPE html>
	{% load staticfiles %}
	<html lang="en">
	<head>
	    <meta charset="UTF-8">
	    <title>Modales</title>
	    <meta charset="UTF-8">
	    <meta name="viewport" content="width=device-width, initial-scale=1">
	    <link rel="stylesheet" type="text/css" href="{% static 'css/bootstrap.min.css' %}">
	    <link rel="stylesheet" type="text/css" href="{% static 'css/jquery-ui.css' %}">
	    <link rel="stylesheet" type="text/css" href="{% static 'css/jquery.dataTables.min.css'  %}"/>
	    <script type="text/javascript" src="{% static 'js/jquery.js' %}" charset="UTF-8"></script>
	    <script type="text/javascript" src="{% static 'js/bootstrap.min.js' %}" charset="UTF-8"></script>
	    <script type="text/javascript" src="{% static 'js/jquery-ui.js' %}"></script>
	    <script type="text/javascript" src="{% static 'js/jquery.dataTables.min.js' %}"></script>
	</head>
	<body>
	<div id="page-wrapper">
	    <div class="container-fluid">
	    {% block cuerpo %}

	    {% endblock cuerpo %}
	    </div>
	</div>
	</body>
	</html>


Vamos a proceder a crear nuestras vistas en el archivos views.py:

views.py

.. code-block:: python

	# -*- coding: utf-8 -*-
	from __future__ import unicode_literals
	from django.views.generic.edit import UpdateView, CreateView
	from django.views.generic.list import ListView
	from productos.forms import ProductoForm
	from productos.models import Producto
	from django.core.urlresolvers import reverse_lazy
	from django.views.generic.detail import DetailView

	class ListadoProductos(ListView):
	    model = Producto
	    template_name = 'productos.html'
	    context_object_name = 'productos'

	class CrearProducto(CreateView):
	    template_name = 'producto.html'
	    form_class = ProductoForm
	    success_url = reverse_lazy('productos:listado_productos')

	class ModificarProducto(UpdateView):
	    model = Producto
	    template_name = 'producto.html'
	    form_class = ProductoForm
	    success_url = reverse_lazy('productos:listado_productos')


	class DetalleProducto(DetailView):
	    model = Producto
	    template_name = 'detalle_producto.html'

La vista ListadoProductos es la vista principal de nuestra aplicación desde aquí vamos a interactuar con las demás vistas, como se ve todas están basadas en clases.

Debemos crear un formulario llamado ProductoForm para utilizarlo tanto en ModificarProducto como en CrearProducto:

forms.py

.. code-block:: python

	from django import forms

	from productos.models import Producto


	class ProductoForm(forms.ModelForm):

	    class Meta:
	        model = Producto
	        fields = ['descripcion', 'marca', 'precio', 'estado']

	    def __init__(self, *args, **kwargs):
	        super(ProductoForm, self).__init__(*args, **kwargs)
	        for field in iter(self.fields):
	            if field <> 'estado':
	                self.fields[field].widget.attrs.update({
	                    'class': 'form-control'
	                })

Ĺo único nuevo en la parte de los formularios es que estamos implementando el método __init__ para poder agregar la clase 'form-control' de bootstrap.

Ahora vamos a crear nuestro archivo urls.py:

.. code-block:: python

	from django.conf.urls import url
	from productos.views import ListadoProductos, CrearProducto, ModificarProducto, DetalleProducto

	urlpatterns = [
	    url(r'^$', ListadoProductos.as_view(), name="listado_productos"),
	    url(r'^crear_producto/$', CrearProducto.as_view(), name="crear_producto"),
	    url(r'^modificar_producto/(?P<pk>.+)/$',ModificarProducto.as_view(), name="modificar_producto"),
	    url(r'^detalle_producto/(?P<pk>.+)/$',DetalleProducto.as_view(), name="detalle_producto"),
	]

Ahora si haremos nuestra plantilla principal, llamada productos.html, primero haremos la parte del html y luego la de javascript:

productos.html

.. code-block:: html

	{% extends "base.html" %}
	{% block cuerpo %}
	<h3>Productos</h3>
	<div class="row">
		<div class="col-lg-10">
			<a onclick="return abrir_modal('{% url 'productos:crear_producto' %}','Productos / Nuevo')" class="btn btn-primary">
				Crear
			</a>
		</div>
	</div>
	<hr/>
	<div class="row">
		<div class="col-lg-12">
			<table id="tabla" class="display" cellspacing="0" width="100%">
				<thead>
					<tr>
						<th class="text-center">DESCRIPCION</th>
						<th class="text-center">MARCA</th>
						<th class="text-center">PRECIO</th>
						<th class="text-center">ESTADO</th>
						<th class="text-center">ACCIONES</th>
					</tr>
				</thead>
				<tbody>
				{% for producto in productos %}
	                <tr>
	                    <td>{{ producto.descripcion }}</td>
	                    <td>{{ producto.marca }}</td>
	                    <td>{{ producto.precio }}</td>
	                    {% if producto.estado %}
	                    <td>ACTIVO</td>
	                    {% else %}
	                    <td>INACTIVO</td>
	                    {% endif %}
	                    <td class="text-center">
	                        <a onclick="return abrir_modal('{% url 'productos:detalle_producto' producto.pk %}','Productos / {{ producto.descripcion }}')" class="btn">
	                            <span class="glyphicon glyphicon-eye-open"></span>
	                        </a>
	                        <a onclick="return abrir_modal('{% url 'productos:modificar_producto' producto.pk %}','Productos / {{ producto.descripcion }}')" class="btn">
	                            <span class="glyphicon glyphicon-edit"></span>
	                        </a>
	                    </td>
					</tr>
				{% endfor %}
				</tbody>
			</table>
		</div>
	</div>
	<div id="popup"></div>

Aquí primero heredamos de base.html y luego tenemos un botón para crear un nuevo producto y una tabla donde se listan los productos ya creados, junto con dos enlaces en cada fila de producto para poder desplegar el detalle y la modificación del mismo en los modales correspondientes, esto se hace con esta linea:

.. code-block:: html
	
	onclick="return abrir_modal('{% url 'productos:crear_producto' %}','Productos / Nuevo')"

Se llama a la url correspondiente como primer argumento del método de javascript "abrir_modal" y como segundo el título del modal.

Para mostrar el modal usaremos el div con el id "popup".

Ahora vamos a ver la parte de javascript necesaria para poder mostrar el modal correspondiente:

.. code-block:: javascript
	
	var modal;

	function abrir_modal(url, titulo)
	{
	    modal = $('#popup').dialog(
	    {
	        title: titulo,
	        modal: true,
	        width: 500,
	        resizable: false
	    }).dialog('open').load(url)
	}

	function cerrar_modal()
	{
	    modal.dialog("close");
	}

	$(document).ready(function()
	{
	    var table = $('#tabla').dataTable( {
	        "language": {
	        	url: "/static/localizacion/es_ES.json"
	        }
	    } );
	});


La función abrir_modal usa el método dialog() de JQuery-UI para mostrar un modal cuyo contenido en html va a ser cargado con el método load de acuerdo a la url que estemos tratando de visualizar. Este elemento se guarda en la variable modal.

La función cerrar_modal() utiliza la variable modal, para cerrar el dialogo pasando el parámetro "close".

Finalmente tenemos los templates correspondientes a detalle_producto.html y producto.html:


detalle_producto.html

.. code-block:: html
	
	<div class="panel panel-default">
		<div class="panel-body">
			<div class="row">
				<div class="col-lg-7">
					<h3>{{ object.descripcion }}</h3>
				</div>
			</div>
			<div class="row">
				<div class="col-lg-4">
					<label>MARCA:</label>
					<p>{{ object.marca }}</p>
					<label>PRECIO:</label>
					<p>{{ object.precio }}</p>
					<label>ESTADO:</label>
					{% if object.estado %}
						<p>ACTIVO</p>
					{% else %}
						<p>INACTIVO</p>
					{% endif %}
				</div>
			</div>
		</div>
	</div>

producto.html

.. code-block:: html

	{% if object %}
	<form role="form" action="{% url 'productos:modificar_producto' object.pk %}" method="post">
	{% else %}
	<form role="form" action="{% url 'productos:crear_producto' %}" method="post">
	{% endif %}
		{% csrf_token %}
		<div class="panel panel-default">
			<div class="panel-body">
				{{ form.as_p }}
			</div>
		</div>
		<div class="row">
			<div class="col-lg-12 text-right">
				<input type="submit" class="btn btn-primary" name="submit" value="Guardar">
				<button type="button" class="btn btn-default" onclick="return cerrar_modal()">
					Cancelar
				</button>
			</div>
		</div>
	</form>

Este template tiene de especial que va a ser usado tanto para crear como para modificar el producto, por lo que se procede a preguntar si existe un objeto object, en caso de que sea así la acción a realizar será modificar el producto y sino crear un nuevo producto.

Si todo ha salido bien podemos tener una pantalla como la siguiente:

.. image:: /images/blog/modales_principal.png

Y los modales:

Creacion de Nuevo Producto:

.. image:: /images/blog/creacion_producto.png

Detalle de Producto:

.. image:: /images/blog/detalle_producto.png

Modificacion de Producto:

.. image:: /images/blog/modificacion_producto.png

Para ver los archivos de configuración del proyecto y todo lo demás que no ha sido explicado en este post, pueden acceder al repositorio:

`Proyecto Modales`_

Saludos.

.. _Proyecto Modales: https://github.com/pythonpiura/modales