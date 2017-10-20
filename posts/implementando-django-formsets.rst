.. title: Implementando Django Formsets
.. slug: implementando-django-formsets
.. date: 2017-10-18 19:03:29 UTC-05:00
.. tags: 
.. category: 
.. link: 
.. description: 
.. type: text

En esta oportunidad vamos a trabajar con los formsets de Django que son muy interesantes y pueden ser de mucha ayuda cuando tenemos múltiples filas con detalles a rellenar, como por ejemplo los detalles de una compra. Vamos a seguir utilizando el proyecto llamado modales que hemos venido usando en posts anteriores y que ya tiene varios modelos listos para ser usados:

Vamos a crear un nuevo modelo llamado compra:

.. code-block:: python

	class Compra(models.Model):
	    proveedor = models.ForeignKey(Proveedor)
	    fecha = models.DateField(auto_now_add=True)

Como podemos ver este modelo Compra tiene dos atributos proveedor, donde se utiliza el modelo Proveedor que ya hemos usado, y fecha, que se asigna automáticamente la fecha actual.

También creamos el modelo DetalleCompra:

.. code-block:: python

	class DetalleCompra(models.Model):
	    compra = models.ForeignKey(Compra)
	    producto = models.ForeignKey(Producto)
	    cantidad = models.DecimalField(max_digits=5, decimal_places=2)
	    precio_compra = models.DecimalField(max_digits=5,decimal_places=2)

Este modelo tiene el detalle de la compra con los siguientes atributos: "compra" que referencia al modelo "cabecera"(por llamarlo de alguna manera), "producto" que hace la referencia al modelo Producto, cantidad para guardar la cantidad del producto comprado y el precio_compra para guardar el precio del producto en esa compra.

Igual que con los ejemplos anteriores primero vamos a mostrar un listado de las compras para ello necesitamos una vista, un template y su correspondiente url.

Creamos la clase ListadoCompras basada en la clase ListView:

.. code-block:: python

	class ListadoCompras(ListView):
	    model = Compra
	    template_name = 'compras.html'
	    context_object_name = 'compras'

Creamos la plantilla compras.html:

.. code-block:: html

	{% extends "base.html" %}
	{% block cuerpo %}
	<h3>Compras</h3>
	<div class="row">
		<div class="col-lg-10">
			<a href="{% url 'productos:crear_compra' %}" class="btn btn-primary">
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
						<th class="text-center">PROVEEDOR</th>
						<th class="text-center">FECHA</th>
						<th class="text-center">ACCIONES</th>
					</tr>
				</thead>
				<tbody>
				{% for compra in compras %}
	                <tr>
	                    <td>{{ compra.proveedor }}</td>
	                    <td>{{ compra.fecha }}</td>
	                    <td class="text-center">
	                        <a href="#" class="btn">
	                            <span class="glyphicon glyphicon-eye-open"></span>
	                        </a>
	                        <a href="#" class="btn">
	                            <span class="glyphicon glyphicon-edit"></span>
	                        </a>
	                    </td>
					</tr>
				{% endfor %}
				</tbody>
			</table>
		</div>
	</div>
	
	<script>
	
	$(document).ready(function()
	{
	    var table = $('#tabla').dataTable( {
	        "language": {
	        	url: "/static/localizacion/es_ES.json"
	        }
	    } );
	});
	</script>
	{% endblock cuerpo %}

Añadimos la url correspondiente, nótese que también se usa en esa plantilla la url "productos:crear_compra" por lo que debemos incluirla y crear una clase vacia para no tener problemas:

.. code-block:: python

	class CrearCompra(CreateView):
		pass

Ahora editamos el archivo urls.py:


.. code-block:: python
	
	from productos.views import ListadoCompras, CrearCompra
	
	urlpatterns = [
		...
		url(r'^compras/$', ListadoCompras.as_view(), name="listado_compras"),
		url(r'^crear_compra/$', CrearCompra.as_view(), name="crear_compra"),
	]

Si en el navegador ponemos la url http://localhost:8000/compras/ nos debe aparecer la siguiente pantalla:

.. image:: /images/blog/listado_compras.png

Editamos el archivo forms.py para crear los formularios que necesitamos.

Creamos el formulario para la compra usando un ModelForm y con el campo proveedor:

.. code-block:: python

	class CompraForm(forms.ModelForm):

	    class Meta:
	        model = Compra
	        fields = ['proveedor']

	    def __init__(self, *args, **kwargs):
	        super(CompraForm, self).__init__(*args, **kwargs)
	        for field in iter(self.fields):
	            self.fields[field].widget.attrs.update({
	                'class': 'form-control'
	            })

Creamos nuestro formulario del Detalle de la compra usando también un ModelForm y con los campos producto, cantidad y precio_compra:

.. code-block:: python

	class DetalleCompraForm(forms.ModelForm):

	    class Meta:
	        model = DetalleCompra
	        fields = ['producto','cantidad','precio_compra']

	    def __init__(self, *args, **kwargs):
	        super(DetalleCompraForm, self).__init__(*args, **kwargs)
	        for field in iter(self.fields):
	            self.fields[field].widget.attrs.update({
	                'class': 'form-control'
	            })

	    def clean_cantidad(self):
	        cantidad = self.cleaned_data['cantidad']
	        if cantidad == '':
	            raise forms.ValidationError("Debe ingresar una cantidad valida")
	        return cantidad

	    def clean_precio_compra(self):
	        precio = self.cleaned_data['precio_compra']
	        if precio == '':
	            raise forms.ValidationError("Debe ingresar un precio valido")
	        return precio

Nótese que hemos creados dos métodos "clean" para hacer las validaciones correspondientes tanto de cantidad como de precio_compra.

Ahora si vamos a crear un formset en linea, estos son una pequeña capa de abstracción que simplifica trabajar con objetos relacionados a través de una clave foránea:

.. code-block:: python

	from django.forms.models import inlineformset_factory

	DetalleCompraFormSet = inlineformset_factory(Compra, DetalleCompra, form=DetalleCompraForm, extra=4)
 
Para hacerlo llamamos al método inlineformset_factory que usa a su vez al método modelformset_factory() pero con el argumento para borrar can_delete=True. Este método tiene como argumentos el modelo foráneo Compra, el modelo DetalleCompra que aparecerá en cada una de las filas, el formulario DetalleCompraForm que se corresponde con el modelo DetalleCompra y el parámetro extra que indica que aparecerán por defecto 4 filas.

Modificamos la vista CrearCompra para poder utilizar los formularios, para ello editamos el archivo views.py:

.. code-block:: python

	from productos.forms import CompraForm, DetalleCompraFormSet

	class CrearCompra(CreateView):
	    model = Compra
	    template_name = 'compra.html'
	    form_class = CompraForm
	    success_url = reverse_lazy('productos:listado_compras')

En esta primera parte tenemos una clase basada en CreateView para facilitarnos la vida, con el modelo Compra, la plantilla 'compra.html' y el formulario CompraForm, el atributo success_url indica la dirección donde se va a retornar una vez que la creación del objeto haya sido concretada con éxito.

A continuación pasamos a describir el método get que va a ser necesario para mostrar el formulario cuando se llama a la url, vamos a explicar cada una de las lineas con comentarios:

.. code-block:: python

	    def get(self, request, *args, **kwargs):
	    	"""Primero ponemos nuestro object como nulo, se debe tener en 
	    	cuenta que object se usa en la clase CreateView para crear el objeto"""
	        self.object = None
	        #Instanciamos el formulario de la Compra que declaramos en la variable form_class
	        form_class = self.get_form_class()
	        form = self.get_form(form_class)
	        #Instanciamos el formset
	        detalle_orden_compra_formset=DetalleCompraFormSet()
	        #Renderizamos el formulario de la compra y el formset
	        return self.render_to_response(self.get_context_data(form=form,
	                                                             detalle_compra_form_set=detalle_orden_compra_formset))

Es necesario ahora crear un método post para recibir el contenido cuando se guarde:

.. code-block:: python

		def post(self, request, *args, **kwargs):
			#Obtenemos nuevamente la instancia del formulario de Compras
		    form_class = self.get_form_class()
		    form = self.get_form(form_class)
		    #Obtenemos el formset pero ya con lo que se le pasa en el POST
		    detalle_compra_form_set = DetalleCompraFormSet(request.POST)
		    """Llamamos a los métodos para validar el formulario de Compra y el formset, si son válidos ambos se llama al método 
		    form_valid o en caso contrario se llama al método form_invalid"""
		    if form.is_valid() and detalle_compra_form_set.is_valid():
		        return self.form_valid(form, detalle_compra_form_set)
		    else:
		        return self.form_invalid(form, detalle_compra_form_set)

		def form_valid(self, form, detalle_compra_form_set):
			#Aquí ya guardamos el object de acuerdo a los valores del formulario de Compra
		    self.object = form.save()
		    #Utilizamos el atributo instance del formset para asignarle el valor del objeto Compra creado y que nos indica el modelo Foráneo
		    detalle_compra_form_set.instance = self.object
		    #Finalmente guardamos el formset para que tome los valores que tiene
		    detalle_compra_form_set.save()
		    #Redireccionamos a la ventana del listado de compras
		    return HttpResponseRedirect(self.success_url)

		def form_invalid(self, form, detalle_compra_form_set):
			#Si es inválido el form de Compra o el formset renderizamos los errores
		    return self.render_to_response(self.get_context_data(form=form,
		                                                         detalle_compra_form_set = detalle_compra_form_set))


Creamos el template para la compra en el archivo "compra.html":

.. code-block:: html

	{% extends "base.html" %}
	{% block cuerpo %}

	<form role="form" action="{% url 'productos:crear_compra' %}" method="post">
		<h3>Crear Compra</h3>	
		{% csrf_token %}
		<div class="panel panel-default">
			<div class="panel-body">
				{{ form.as_p }}
				{{ detalle_compra_form_set.management_form }}
				{% for detalle_compra_form in detalle_compra_form_set %}
					<div class="row">
						<div class="col-lg-4">
							<label>Producto</label>
							{% if detalle_compra_form.producto.errors %}
								{% for error in detalle_compra_form.producto.errors %}
								<div class="alert alert-danger alert-dismissible" role="alert">
									<button type="button" class="close" data-dismiss="alert" aria-label="Close">
										<span aria-hidden="true">&times;</span>
									</button>
									<strong>Error: </strong> {{ error|escape }}
								</div>
								{% endfor %}
							{% endif %}
							{{ detalle_compra_form.producto }}
						</div>
						<div class="col-lg-4">
							<label>Cantidad</label>
							{% if detalle_compra_form.cantidad.errors %}
								{% for error in detalle_compra_form.cantidad.errors %}
								<div class="alert alert-danger alert-dismissible" role="alert">
									<button type="button" class="close" data-dismiss="alert" aria-label="Close">
										<span aria-hidden="true">&times;</span>
									</button>
									<strong>Error: </strong> {{ error|escape }}
								</div>
								{% endfor %}
							{% endif %}
							{{ detalle_compra_form.cantidad }}
						</div>
						<div class="col-lg-4">
							<label>Precio</label>
							{% if detalle_compra_form.precio_compra.errors %}
								{% for error in detalle_compra_form.precio_compra.errors %}
								<div class="alert alert-danger alert-dismissible" role="alert">
									<button type="button" class="close" data-dismiss="alert" aria-label="Close">
										<span aria-hidden="true">&times;</span>
									</button>
									<strong>Error: </strong> {{ error|escape }}
								</div>
								{% endfor %}
							{% endif %}
							{{ detalle_compra_form.precio_compra }}
						</div>
					</div>
				{% endfor %}
			</div>
		</div>
		<div class="col-lg-12 text-right">
			<input type="submit" class="btn btn-primary" name="submit" value="Guardar">
			<a type="button" class="btn btn-default" href="{% url 'productos:listado_compras' %}">
				Cancelar
			</a>
		</div>
	</form>
	{% endblock cuerpo %}

Análicemos parte por parte esta plantilla:

Primero renderizamos el formulario de la compra:

.. code-block:: html

	{{ form.as_p }}

Luego renderizamos el management_form del formset que nos indica varios parámetros importantes del formset tales como el total de formularios dentro del formset y la cantidad mínima y máxima de formularios que se pueden crear en el formset:

.. code-block:: html

	{{ detalle_compra_form_set.management_form }}


Hacemos un recorrido al formset renderizando form por form con cada uno de sus campos y sus respectivos errores:

.. code-block:: html

	{% for detalle_compra_form in detalle_compra_form_set %}
		<div class="row">
			<div class="col-lg-4">
				<label>Producto</label>
				{% if detalle_compra_form.producto.errors %}
					{% for error in detalle_compra_form.producto.errors %}
					<div class="alert alert-danger alert-dismissible" role="alert">
						<button type="button" class="close" data-dismiss="alert" aria-label="Close">
							<span aria-hidden="true">&times;</span>
						</button>
						<strong>Error: </strong> {{ error|escape }}
					</div>
					{% endfor %}
				{% endif %}
				{{ detalle_compra_form.producto }}
			</div>
			<div class="col-lg-4">
				<label>Cantidad</label>
				{% if detalle_compra_form.cantidad.errors %}
					{% for error in detalle_compra_form.cantidad.errors %}
					<div class="alert alert-danger alert-dismissible" role="alert">
						<button type="button" class="close" data-dismiss="alert" aria-label="Close">
							<span aria-hidden="true">&times;</span>
						</button>
						<strong>Error: </strong> {{ error|escape }}
					</div>
					{% endfor %}
				{% endif %}
				{{ detalle_compra_form.cantidad }}
			</div>
			<div class="col-lg-4">
				<label>Precio</label>
				{% if detalle_compra_form.precio_compra.errors %}
					{% for error in detalle_compra_form.precio_compra.errors %}
					<div class="alert alert-danger alert-dismissible" role="alert">
						<button type="button" class="close" data-dismiss="alert" aria-label="Close">
							<span aria-hidden="true">&times;</span>
						</button>
						<strong>Error: </strong> {{ error|escape }}
					</div>
					{% endfor %}
				{% endif %}
				{{ detalle_compra_form.precio_compra }}
			</div>
		</div>
	{% endfor %}

Finalmente nos debe quedar una imagen para crear compra como la siguiente:

.. image:: /images/blog/crear_compra.png

Si llenamos los datos correspondientes:

.. image:: /images/blog/crear_compra_datos.png

Cuando le demos click al botón guardar automáticamente nos guarda los elementos y retorna a la ventana con el listado de las compras:

.. image:: /images/blog/listado_compras_datos.png


Por ahora no tenemos un detalle de lo guardado pero si lo está haciendo correctamente, en el siguiente post veremos como mostrar los datos guardados en el admin, recuerden que el código del proyecto lo pueden encontrar en la siguiente ubicación:

`Proyecto Modales`_

Hasta la pŕoxima, saludos.

.. _Proyecto Modales: https://github.com/pythonpiura/modales