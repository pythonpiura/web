.. title: Modificación de Datos con Django Formsets
.. slug: modificacion-de-datos-con-django-formset
.. date: 2017-10-22 19:32:00 UTC-05:00
.. tags: 
.. category: 
.. link: 
.. description: 
.. type: text

Escriba su publicación aquí.

Siguiendo con nuestro `ejemplo anterior`_ ahora vamos a hacer la modificación de la compra usando formsets:

Como primer paso debemos agregar la opción de modificación en nuestro listado de compras, por lo que editamos nuestro archivo compras.html:

.. code-block:: html

	<a href="{% url 'productos:modificar_compra' compra.pk %}" class="btn">
	    <span class="glyphicon glyphicon-edit"></span>
	</a>

Este cambio se hace en la parte donde se obtiene el listado de las compras:

.. code-block:: html

	<tbody>
	{% for compra in compras %}
        <tr>
            <td>{{ compra.proveedor }}</td>
            <td>{{ compra.fecha }}</td>
            <td class="text-center">
                <a href="#" class="btn">
                    <span class="glyphicon glyphicon-eye-open"></span>
                </a>
                <a onclick="return abrir_modal('{% url 'productos:modificar_compra' compra.pk %}')" class="btn">
                    <span class="glyphicon glyphicon-edit"></span>
                </a>
            </td>
		</tr>
	{% endfor %}
	</tbody>

Creamos la url "modificar_compra":

.. code-block:: python

	from productos.views import ..., ModificarCompra

	urlpatterns = [
		...
		url(r'^modificar_compra/(?P<pk>.+)/$',ModificarCompra.as_view(), name="modificar_compra"),
	]

Creamos la vista ModificarCompra que use el formulario CompraForm:

.. code-block:: python

	class ModificarCompra(UpdateView):
	    model = Compra
	    template_name = 'compra.html'
	    form_class = CompraForm
	    success_url = reverse_lazy('productos:listado_compras')

Es bastante parecido a la vista de creación de la compra, tenemos el modelo, la plantilla, el formulario y la url donde se redirigirá cuando la operación haya concluido con éxito. 

Agregamos el método get para poder obtener los datos a modificar:

.. code-block:: python

    def get(self, request, *args, **kwargs):
    	#Obtenemos el objeto Compra
        self.object = self.get_object()
        #Obtenemos el formulario 
        form_class = self.get_form_class()
        form = self.get_form(form_class)
        #Obtenemos los detalles de la compra
        detalles = DetalleCompra.objects.filter(compra=self.object).order_by('pk')
        detalles_data = []
        #Guardamos los detalles en un diccionario
        for detalle in detalles:
            d = {'producto': detalle.producto,
                 'cantidad': detalle.cantidad,
                 'precio_compra': detalle.precio_compra}
            detalles_data.append(d)
        #Ponemos como datos iniciales del formset el diccionario que hemos obtenido
        detalle_compra_form_set = DetalleCompraFormSet(initial=detalles_data)
        #Renderizamos el formulario y el formset
        return self.render_to_response(self.get_context_data(form=form,
                                                             detalle_compra_form_set=detalle_compra_form_set))

Modificamos la plantilla compra.html para poder utilizarla también en la modificación y no crear otra:

.. code-block:: html

	{% if object %}
	<form role="form" action="{% url 'productos:modificar_compra' object.pk %}" method="post">
		<h3>Modificar Compra</h3>
	{% else %}
	<form role="form" action="{% url 'productos:crear_compra' %}" method="post">
		<h3>Crear Compra</h3>
	{% endif %}

Al consultar por la variable "object" estamos comprobando si el objeto compra existe, si es así entonces es una modificación y sino es una creación, esto es muy útil para reutilizar plantillas.

Con lo que tenemos ahora ya podemos ver un resultado como el siguiente:

.. image:: /images/blog/modificar_compra.png

Debemos implementar el método post para poder guardar las modificaciones que hagamos:

.. code-block:: python

    def post(self, request, *args, **kwargs):
    	#Obtenemos el objeto compra
        self.object = self.get_object()
        form_class = self.get_form_class()
        form = self.get_form(form_class)
        detalle_compra_form_set = DetalleCompraFormSet(request.POST)
        #Verificamos que los formularios sean validos
        if form.is_valid() and detalle_compra_form_set.is_valid():
            return self.form_valid(form, detalle_compra_form_set)
        else:
            return self.form_invalid(form, detalle_compra_form_set)


    def form_valid(self, form, detalle_compra_form_set):
        #Guardamos el objeto compra
        self.object = form.save()
        detalle_compra_form_set.instance = self.object
        #Eliminamos todas los detalles de la compra
        DetalleCompra.objects.filter(compra = self.object).delete()
        #Guardamos los nuevos detalles de la compra
        detalle_compra_form_set.save()
        return HttpResponseRedirect(self.success_url)

    def form_invalid(self, form, detalle_compra_form_set):
    	#Renderizamos los errores
        return self.render_to_response(self.get_context_data(form=form,
                                                             detalle_compra_form_set = detalle_compra_form_set))

Para comprobar que los cambios estén funcionando debemos guardarlos y volver a abrirlos usando la misma modificación:

.. image:: /images/blog/modificar_compra_datos.png


Como podemos ver, por ahora no existe la posibilidad de hacer eliminación de los formularios que nos vienen por defecto, pero ya corregiremos eso en las pŕoximas publicaciones.

Recuerden que el código del proyecto lo pueden encontrar en la siguiente ubicación:

`Proyecto Modales`_

Saludos.

.. _ejemplo anterior: http://pythonpiura.org/posts/implementando-django-formsets/
.. _Proyecto Modales: https://github.com/pythonpiura/modales