.. title: Modales en Django con Vistas Basadas en Clases y Bootstrap
.. slug: modales-en-django-con-vistas-basadas-en-clases-y-bootstrap
.. date: 2017-05-23 22:44:05 UTC-05:00
.. tags: Django, bootstrap, modales
.. category: 
.. link: 
.. description: 
.. type: text

Vamos a hacer lo mismo que en el post anterior pero con la diferencia que ahora usaremos los modales de bootstrap, para ello vamos a hacer uso de nuestro proyecto "modales" y de la aplicación "productos", que ya tenemos listos, para hacer las pruebas hemos creado un nuevo modelo llamado "proveedores":

.. code-block:: python

	class Proveedor(models.Model):
	    ruc = models.CharField(unique=True,max_length=11)
	    razon_social = models.CharField(max_length=150)
	    direccion = models.CharField(max_length=200)
	    telefono = models.CharField(max_length=15,null=True)
	    correo = models.EmailField(null=True)
	    estado = models.BooleanField(default=True)

Creamos las vistas correspondientes al listado, creación, modificación y detalle de los proveedores:

archivo: views.py

.. code-block:: python

	class ListadoProveedores(ListView):
	    model = Proveedor
	    template_name = 'proveedores.html'
	    context_object_name = 'proveedores'

	class CrearProveedor(CreateView):
	    template_name = 'proveedor.html'
	    form_class = ProveedorForm
	    success_url = reverse_lazy('productos:listado_proveedores')

	class ModificarProveedor(UpdateView):
	    model = Proveedor
	    template_name = 'proveedor.html'
	    form_class = ProveedorForm
	    success_url = reverse_lazy('productos:listado_proveedores')

	class DetalleProveedor(DetailView):
	    model = Proveedor
	    template_name = 'detalle_proveedor.html'

El formulario a utilizar tanto para modificación como para la creación del proveedor es uno solo:

archivo: forms.py

.. code-block:: python

	class ProveedorForm(forms.ModelForm):

	    class Meta:
	        model = Proveedor
	        fields = ['ruc', 'razon_social', 'direccion', 'telefono','correo', 'estado']

	    def __init__(self, *args, **kwargs):
	        super(ProveedorForm, self).__init__(*args, **kwargs)
	        for field in iter(self.fields):
	            if field <> 'estado':
	                self.fields[field].widget.attrs.update({
	                    'class': 'form-control'
	                })

Ahora debemos modificar nuestro archivo urls.py para poder agregar las nuevas urls:

archivo: urls.py

.. code-block:: python

	urlpatterns = [
	    url(r'^proveedores/$', ListadoProveedores.as_view(), name="listado_proveedores"),
	    url(r'^crear_proveedor/$', CrearProveedor.as_view(), name="crear_proveedor"),
	    url(r'^modificar_proveedor/(?P<pk>.+)/$',ModificarProveedor.as_view(), name="modificar_proveedor"),
	    url(r'^detalle_proveedor/(?P<pk>.+)/$',DetalleProveedor.as_view(), name="detalle_proveedor"),
	]

Veamos primero el archivo proveedores.html que es el template desde donde se van a realizar las demás operaciones:

archivo: proveedores.html

.. code-block:: html

	{% extends "base.html" %}
	{% block cuerpo %}
	<h3>Proveedores</h3>
	<div class="row">
		<div class="col-lg-10">
			<a onclick="return abrir_modal('{% url 'productos:crear_proveedor' %}')" class="btn btn-primary">
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
						<th class="text-center">RUC</th>
						<th class="text-center">RAZON SOCIAL</th>
						<th class="text-center">DIRECCION</th>
						<th class="text-center">ESTADO</th>
						<th class="text-center">ACCIONES</th>
					</tr>
				</thead>
				<tbody>
				{% for proveedor in proveedores %}
	                <tr>
	                    <td>{{ proveedor.ruc }}</td>
	                    <td>{{ proveedor.razon_social }}</td>
	                    <td>{{ proveedor.direccion }}</td>
	                    {% if proveedor.estado %}
	                    <td>ACTIVO</td>
	                    {% else %}
	                    <td>INACTIVO</td>
	                    {% endif %}
	                    <td class="text-center">
	                        <a onclick="return abrir_modal('{% url 'productos:detalle_proveedor' proveedor.pk %}')" class="btn">
	                            <span class="glyphicon glyphicon-eye-open"></span>
	                        </a>
	                        <a onclick="return abrir_modal('{% url 'productos:modificar_proveedor' proveedor.pk %}')" class="btn">
	                            <span class="glyphicon glyphicon-edit"></span>
	                        </a>
	                    </td>
					</tr>
				{% endfor %}
				</tbody>
			</table>
		</div>
	</div>
	<div id="popup" class="modal fade" role="dialog">

	</div>

Tenemos la tabla con el listado de los proveedores ya creados junto con dos enlaces en cada fila de proveedor para poder desplegar el detalle y la modificación del mismo en los modales correspondientes, en la parte final vemos que para mostrar el modal usaremos el div con el id "popup" y con las clases de bootstrap "modal" y "fade" para que la ventana a mostrar sea un modal con un ligero efecto al mostrarse.

Ahora vamos a ver la parte de javascript necesaria para poder mostrar el modal correspondiente:

.. code-block:: javascript

	function abrir_modal(url)
	{
		$('#popup').load(url, function()
		{
			$(this).modal('show');
		});
		return false;
	}

	function cerrar_modal()
	{
		$('#popup').modal('hide');
		return false;
	}

	$(document).ready(function()
	{
	    var table = $('#tabla').dataTable( {
	        "language": {
	        	url: "/static/localizacion/es_ES.json"
	        }
	    } );
	});	

Notamos que aquí la forma de trabajo es mas sencilla que cuando la haciamos con JQuery-UI ya que solamente usamos el método load() y mostramos el contenido de la ruta que pasamos como argumento.

Ahora veamos los templates correspondientes a la creación, modificación y detalle del proveedor, en el caso de los dos primeros es un solo template:

archivo: proveedor.html

.. code-block:: html

	<div class="modal-dialog modal-lg">
		<div class="modal-content">
			{% if object %}
			<form role="form" action="{% url 'productos:modificar_proveedor' object.pk %}" method="post">
			{% else %}
			<form role="form" action="{% url 'productos:crear_proveedor' %}" method="post">
			{% endif %}
				<div class="modal-header">
	                <button type="button" class="close" data-dismiss="modal">x</button>
	                <h3>Modificar Proveedor</h3>
	            </div>
	            <div class="modal-body">
					{% csrf_token %}
					<div class="panel panel-default">
						<div class="panel-body">
							{{ form.as_p }}
						</div>
					</div>
				</div>
				<div class="modal-footer">
					<div class="col-lg-12 text-right">
						<input type="submit" class="btn btn-primary" name="submit" value="Guardar">
						<button type="button" class="btn btn-default" onclick="return cerrar_modal()">
							Cancelar
						</button>
					</div>
				</div>
			</form>
		</div>
	</div>

archivo: detalle_proveedor.html

.. code-block:: html

	<div class="modal-dialog modal-lg">
		<div class="modal-content">
			<div class="modal-header">
				<button type="button" class="close" data-dismiss="modal">x</button>
				<h3>Detalle Proveedor</h3>
			</div>
			<div class="modal-body">
				<div class="row">
					<div class="col-lg-4">
						<label>RUC:</label>
						<p>{{ object.ruc }}</p>
						<label>RAZÓN SOCIAL:</label>
						<p>{{ object.razon_social }}</p>
						<label>DIRECCIÓN:</label>
						<p>{{ object.direccion }}</p>
						<label>TELÉFONO:</label>
						<p>{{ object.telefono }}</p>
						<label>CORREO:</label>
						<p>{{ object.correo }}</p>
						<label>ESTADO:</label>
						{% if object.estado %}
							<p>ACTIVO</p>
						{% else %}
							<p>INACTIVO</p>
						{% endif %}
					</div>
				</div>
			</div>
			<div class="modal-footer">
				<div class="col-lg-12 text-right">
					<button type="button" class="btn btn-primary" onclick="return cerrar_modal()">
						Aceptar
					</button>
				</div>
			</div>
		</div>
	</div>

Si todo ha salido bien podemos tener una pantalla como la siguiente:

.. image:: /images/blog/proveedores.png

Y los modales:

Creacion de Nuevo Proveedor:

.. image:: /images/blog/creacion_proveedor.png

Detalle de Proveedor:

.. image:: /images/blog/detalle_proveedor.png

Modificacion de Proveedor:

.. image:: /images/blog/modificacion_proveedor.png

Para ver los archivos de configuración del proyecto y todo lo demás que no ha sido explicado en este post, pueden acceder al repositorio:

`Proyecto Modales`_

Saludos.

.. _Proyecto Modales: https://github.com/pythonpiura/modales