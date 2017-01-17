.. title: Aplicar Permisos por Usuario en Django
.. slug: permisos-basicos-en-django
.. date: 2016-04-28 22:33:04
.. tags: Django
.. description: 

Hasta ahora solo hemos trabajado con un usuario(el superusuario que
creamos al principio) en nuestro proyecto, pero en la vida real son
muchos los usuarios que interactuan con el software y todos no cuentan
con los mismos permisos, hay algunos que pueden crear, ver, editar o
eliminar elementos y otros no. Para empezar a trabajar con permisos
primero debemos crear algunos usuarios adicionales, para ello vamos a
ingresar a la consola de administración de Django, poniendo lo
siguiente en nuestra barra de direcciones:

http://localhost:8000/admin

A continuación veremos una pantalla como esta:

.. image:: /images/blog/login_administracion.jpg

Aquí nos logueamos usando el superusuario con el que hemos venido
trabajando y tendremos una pantalla como esta:

.. image:: /images/blog/admin_django.jpg

En el enlace donde dice Users vamos a dar click en Add para crear
nuestro nuevo usuario, al que llamaremos usuario1:

.. image:: /images/blog/crear_usuario.jpg

Completamos los campos y le damos click a save:

.. image:: /images/blog/usuario_creado.jpg

Por ahora con esto es suficiente y ya tenemos nuestro usuario creado y
listo para usarlo, salimos dando click en logout y nos vamos al login
del proyecto para ingresar con nuestro nuevo usuario y la contraseña
que le hemos asignado:

.. image:: /images/blog/nuevo_logueo.jpg

En nuestra aplicación personas, con este usuario podemos hacer lo
mismo que con el usuario anterior, pero que pasa si nosotros queremos
restringir algunas cosas a este nuevo usuario, por ejemplo, no dejar
que este usuario cree ni modifique personas, pero que si pueda ver su
detalle y exportar el reporte en una hoja de calculo ¿Cómo hacemos
esto?
Con Django otra vez esto es muy sencillo y podemos definir diferentes
niveles de acceso, restringiendo las opciones en la plantilla html y
negando el acceso a la vista en cuestión, hagamos lo primero:

Si queremos restringir el acceso a las opciones desde la plantilla
html, lo mas rápido que se nos ocurre es eliminar los íconos que nos
brindan esas opciones, esto lo podemos hacer usando los permisos
proporcionados por django de la siguiente manera:
Primero para restringir la visualización del ícono de crear personas:
personas.html

.. code-block:: html

	{% if perms.personas.add_persona %}

		<div class="col-lg-1">
			<a id="crear_detalle" href="reporte_personas_excel' %}" class="btn btn-info btn-block">
			<span class="glyphicon glyphicon-list-alt"></span>
			</a>
		</div>

	{% endif %}

Si observamos detenidamente hemos agregado en la parte superior del
div del enlace un condicional que utilizar una variable llamada perms,
en esta variable se almacenan los permisos de los que dispone el
usuario que ha iniciado sesión, luego de perms podemos observar el
nombre de la aplicación, en nuestro caso personas, y finalmente
tenemos el permiso llamado add y el modelo persona que configuran el
permiso add_persona, esta condificional nos dice, si el usuario tiene
el permiso add_persona, entonces se debe renderizar el div, sino no
hace nada.
Si ejecutamos nuestra aplicación observamos lo siguiente:

.. image:: /images/blog/no_crear1.jpg

El ícono de agregar ya no aparece, pero todavía no podemos cantar
victoria, un usuario avispado puede recordar la url de creación de
personas y tipearla directamente en la barra de direcciones:

.. image:: /images/blog/crear_persona.jpg

Mmmm, vemos que si puede ingresar ¿Cómo hacemos para restringir eso?
Nuestros viejos amigos los decoradores vuelven nuevamente a nuestro
auxilio, para ello debemos editar el archivo views.py:

.. code-block:: python

	from django.contrib.auth.decorators import permission_required
	from django.utils.decorators import method_decorator
	class CrearPersona(CreateView):
		model = Persona
		fields =['dni','nombre','apellido_paterno','apellido_materno']
		template_name = 'crear_persona.html'
		success_url = reverse_lazy('personas:personas')

		@method_decorator(permission_required('personas.add_persona',reverse_lazy('personas:personas')))
		def dispatch(self, *args, **kwargs):
			return super(CrearPersona, self).dispatch(*args, **kwargs)


Expliquemos esto que está un poco endiablado:
Un método sobre una clase no equivale realmente a una función
independiente, por lo que se debe transformar en un decorador primero,
el decorador @method_decorator transforma un decorador de una función
en un decorador de un método a fin de que puede ser usado sobre una
instancia de un método, en el caso de las vistas basadas en clase a
quien debemos aplicar el decorador es al método dispatch, en este
ejemplo, cada instancia de CrearPersona tendrá protección de
permission_required.
El punto de entrada as_view() crea una instancia de la clase y llama
al método dispatch(), (el despachador o resolvedor de URL) que busca
la petición para determinar si es un GET, POST, etc, y releva la
petición a un método que coincida con uno definido, o levante una
excepción HttpResponseNotAllowed si no encuentra coincidencias.
A la función permission_required se le deben pasar dos argumentos:
El primero es el permiso a verificar que en este caso es add_persona y
tiene la misma notación, primero la aplicación y luego el permiso.
El segundo es la url a donde debe ser direccionado el usuario en caso
de no tener el permiso necesario, en este caso a la url personas.

Ahora pongamos nuevamente la url anterior y veamos que nos sale:

.. image:: /images/blog/permiso_denegado.jpg

Bueno con esto ya nos aseguramos que el usuario “usuario1″ no va a
poder entrar de ninguna manera a la funcionalidad de creación de
personas, ahora lo que falta es aplicar lo mismo a la vista
ModificarPersona las demás vistas tendrá un tratamiento especial en un
artículo posterior:

personas.html

.. code-block:: html

	<tbody>
	{% for persona in personas %}
		<tr>
			<td>{{ persona.dni }}</td>
			<td>{{ persona.nombre }}</td>
			<td>{{ persona.apellido_paterno }}</td>
			<td>{{ persona.apellido_materno }}</td>
			<td class="text-center">
			<a class="btn btn-small" href="{% url personas:detalle_persona' persona.pk %}">
				<span class="glyphicon glyphicon-folder-open"></span>
			</a>
			{% if perms.personas.change_persona %}
				<a class="btn btn-small" href="{% url 'personas:modificar_persona' persona.pk %}">
				<span class="glyphicon glyphicon-pencil"></span>
				</a>
			{% endif %}
			</td>
		</tr>
	{% endfor %}
	</tbody>



views.py

.. code-block:: python

	class ModificarPersona(UpdateView):
		model = Persona
		template_name = 'modificar_persona.html'
		fields = ['dni','nombre','apellido_paterno','apellido_materno']
		success_url = reverse_lazy('personas:personas')

		@method_decorator(permission_required('personas.change_persona',reverse_lazy('personas:personas')))
		def dispatch(self, *args, **kwargs):
			return super(ModificarPersona, self).dispatch(*args, **kwargs)


Notamos que el permiso ahora se llama change_persona, detengamonos un
poco aquí y expliquemos esto, por defecto django aplica tres permisos
a cada uno de nuestros modelos: add, change y delete, que especifican
si un usuario puede crear, modificar o borrar un elemento de un modelo
dado, en este caso el modelo es persona.

Si corremos nuestra proyecto tenemos lo siguiente:

.. image:: /images/blog/sin_accesos.jpg

Nótese que ya no aparece el lápiz de edición y tampoco se puede
acceder con el url de modificar directamente en la barra de
direcciones.

Ahora vamos a crear un nuevo usuario llamado usuario2, al que si le
vamos a dar los permisos de crear y modificar, repetimos los pasos
para crear el usuario en la interfaz de administración y nos detenemos
en la ventana posterior a la creación del usuario, en la opción de
permisos:

.. image:: /images/blog/permisos.jpg

En esta opción tenemos un sinnumero de permisos, busquemos los
relacionados a la aplicación personas y al modelo persona y los
seleccionamos:

.. image:: /images/blog/seleccionar_permisos.jpg

Seleccionamos la flechita entre los dos cuadros y le damos al botón
Save:

.. image:: /images/blog/escoger.jpg

Con esto ya tenemos que el usuario “usuario2″ tiene los permisos
asignados. Ahora salgamos de la interfaz de administración e
ingresemos a nuestro proyecto con el “usuario2″:

.. image:: /images/blog/con_permisos.jpg

Listo ahora si el usuario2, ya tendrá estos permisos básicos y el
usuario1 no, el usuario mamaya es superusuario por lo tanto puede
ingresar donde quiera.

Eso es todo.
Saludos.



