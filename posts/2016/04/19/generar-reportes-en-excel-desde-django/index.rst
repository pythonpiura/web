
En ocasiones necesitamos tener nuestros reportes en alguna hoja de
cálculo, o peor aún un cliente nos pide un reporte específico en excel
¿Cómo hacemos para generar reportes desde Django en excel?
En nuestra ayuda viene una librería muy interesante que se llama
`openpyxl`_, esta libreria nos permite generar contenido en los
diferentes formatos de hojas de cálculo de Microsoft Office y soporta
desde versiones antiguas hasta la versión Office 2010, para instalarla
lo hacemos con nuestro viejo conocido pip:

.. code-block:: bash

	pip install openpyxl


Ahora vamos a trabajar usando el proyecto `tutorial`_ que creamos en
el post pasado, aquí agregamos un botón adicional que nos va a
permitir exportar el contenido de nuestra tabla de personas a excel,
por lo tanto editamos el archivo personas.html y agregamos un botón
adicional al que ya teniamos:

personas.html

.. code-block:: html

	<div class="col-lg-1">
		<a id="crear_detalle" href="#" class="btn btn-info btn-block">
			<span class="glyphicon glyphicon-list-alt"></span>
		</a>
	</div>


Nótese que aún no ponemos el enlace en "#"" para no tener ningún
problema, mas adelante modificaremos esto, ahora nuestra aplicación
debe verse así:

.. image:: /images/blog/tablas2.jpg

En el archivo views.py de la aplicación personas, vamos a crear una
clase que nos permita exportar todas las personas que tenemos
guardadas, a un archivo en excel.

views.py

.. code-block:: python

	#Vista genérica para mostrar resultados
	from django.views.generic.base import TemplateView
	#Workbook nos permite crear libros en excel
	from openpyxl import Workbook
	#Nos devuelve un objeto resultado, en este caso un archivo de excel
	from django.http.response import HttpResponse

	#Nuestra clase hereda de la vista genérica TemplateView
	class ReportePersonasExcel(TemplateView):

		#Usamos el método get para generar el archivo excel
		def get(self, request, *args, **kwargs):
			#Obtenemos todas las personas de nuestra base de datos
			personas = Persona.objects.all()
			#Creamos el libro de trabajo
			wb = Workbook()
			#Definimos como nuestra hoja de trabajo, la hoja activa, por defecto la primera del libro
			ws = wb.active
			#En la celda B1 ponemos el texto 'REPORTE DE PERSONAS'
			ws['B1'] = 'REPORTE DE PERSONAS'
			#Juntamos las celdas desde la B1 hasta la E1, formando una sola celda
			ws.merge_cells('B1:E1′)
			#Creamos los encabezados desde la celda B3 hasta la E3
			ws['B3'] = 'DNI'
			ws['C3'] = 'NOMBRE'
			ws['D3'] = 'APELLIDO PATERNO'
			ws['E3'] = 'APELLIDO MATERNO'
			cont=4
			#Recorremos el conjunto de personas y vamos escribiendo cada uno de los datos en las celdas
			for persona in personas:
				ws.cell(row=cont,column=2).value = persona.dni
				ws.cell(row=cont,column=3).value = persona.nombre
				ws.cell(row=cont,column=4).value = persona.apellido_paterno
				ws.cell(row=cont,column=5).value = persona.apellido_materno
				cont = cont + 1
			#Establecemos el nombre del archivo
			nombre_archivo ="ReportePersonasExcel.xlsx"
			#Definimos que el tipo de respuesta a devolver es un archivo de microsoft excel
			response = HttpResponse(content_type="application/ms-excel")
			contenido = "attachment; filename={0}".format(nombre_archivo)
			response["Content-Disposition"] = contenido
			wb.save(response)
			return response

Modificamos el archivo urls.py:

urls.py

.. code-block:: python

	from django.conf.urls import patterns, url
	from personas.views import Personas, CrearPersona, ReportePersonasExcel

	urlpatterns = patterns(",
		url(r'^$',Personas.as_view(), name="personas"),
		url(r'^crear_persona/$',CrearPersona.as_view(), name="crear_persona"),
		url(r'^reporte_personas_excel/$',ReportePersonasExcel.as_view(), name="reporte_personas_excel"),
	)

Finalmente nuestro archivo personas.html va a quedar de la siguiente
manera:

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
		var table = $('#tabla').DataTable({
			"language": {
				url: "/static/localizacion/es_ES.json"
			}
		});

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

Y nuestro resultado al dar click al botón de exportar en excel será el
siguiente:

.. image:: /images/blog/exportar.jpg


Y el archivo se mostrará así:

.. image:: /images/blog/reporte_excel.jpg

Hay muchísimas formas de sacarle el jugo a esta libreria, definir
formatos, bordes, colores, etc. Esta ha sido una pequeña introducción,
hasta la próxima.

Saludos.



.. _tutorial: https://pythonpiura.wordpress.com/2016/04/18/datatables-jquery-bootstrap-y-django/
.. _openpyxl: https://openpyxl.readthedocs.org/en/default/


