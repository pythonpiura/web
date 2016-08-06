.. title: Mejorando Nuestro Login
.. slug: mejorando-nuestro-login
.. date: 2016-04-28 20:50:59
.. tags: Círculo
.. description: 

Este es un post pequeñito donde vamos a demostrar como mejorar nuestro
login que se ve un poco feo, para ello debemos crear un módulo de
Python llamado forms.py en nuestra aplicación seguridad:

.. image:: /images/blog/forms.jpg

Ahora vamos a crear una clase llamada FormularioLogin que va a heredar
de AuthenticationForm, la herencia en Python se determina poniendo la
clase de la que se va a heredar entre parentesis en la definición de
la clase hija:

forms.py

.. code-block:: python

	from django.contrib.auth.forms import AuthenticationForm

	class FormularioLogin(AuthenticationForm):

		def __init__(self, *args, **kwargs):
			super(FormularioLogin, self).__init__(*args, **kwargs)
			self.fields['username'].widget.attrs['class'] = 'form-control'
			self.fields['username'].widget.attrs['placeholder'] = 'Usuario'
			self.fields['password'].widget.attrs['class'] = 'form-control'
			self.fields['password'].widget.attrs['placeholder'] = 'Contraseña'


¿Que cosa hemos hecho aquí?, sencillo hemos sobreescrito el
comportamiento de los campos del formulario AuthenticationForm,
expliquemos esto: primero se necesita acceder a cada uno de los campos
del formulario, estos son 'username' y 'password', y se accede a ellos
a través de la lista “fields”, luego accederemos a los widgets de cada
uno de estos campos, el widget es el elemento o etiqueta html que se
dibuja en el navegador, en este caso los dos son “input”, al ser
elementos html tienen atributos y cada atributo tiene un valor, en
este caso vamos a modificar los atributos “class” y “placeholder”, el
atributo class establece la clase CSS que se aplica a los estilos del
elemento y el atributo placeholder provee una ayuda a los usuarios
para indicar que cosa se debe escribir en las cajas de texto. Hemos
aplicado la clase `form-control`_ de bootstrap, que sirve para mostrar
los elementos de un formulario mejor presentados, a ambos elementos y
hemos definido la ayuda que aparecerá en cada caja de texto.

Recordemos que en nuestra vista Login el atributo form_class es
AuthenticationForm, ahora esto va a cambiar:

views.py

.. code-block:: python

	from seguridad.forms import FormularioLogin

	# Create your views here.
	class Login(FormView):
		template_name = 'login.html'
		form_class = FormularioLogin
		success_url = reverse_lazy("personas:bienvenida")

		def dispatch(self, request, *args, **kwargs):
			if request.user.is_authenticated():
				return HttpResponseRedirect(self.get_success_url())
			else:
				return super(Login, self).dispatch(request, *args, **kwargs)

		def form_valid(self, form):
			login(self.request, form.get_user())
			return super(Login, self).form_valid(form)


Finalmente tendremos la siguiente presentación:

.. image:: /images/blog/login_bonito.jpg

¿Mejor, verdad?, eso es todo por hoy.

Saludos.

.. _form-control: http://librosweb.es/libro/bootstrap_3/capitulo_5/campos_de_formulario.html


