.. title: Login en Django
.. slug: login-en-django
.. date: 2016-04-26 22:08:42
.. tags: Django
.. description: 

Hasta este momento hemos podido tener acceso sin ningún tipo de
restricciones a nuestro proyecto tutorial, pero todos sabemos que en
el mundo real, eso no funciona así, por lo tanto es necesario que
creemos un login para que los usuarios puedan iniciar sesión, para
ello vamos a rediseñar nuestro proyecto tutorial y vamos a agregar
nuevas cosas, para hacer este pequeño post me he basado en estos
artículos:

`Formulario Login usando Class Based Views`_

`El Libro de Django`_

Listo ahora si empecemos con todo:
Primero crearemos una aplicación llamada seguridad, donde vamos a
manejar todo lo referente a los usuarios:

.. image:: /images/blog/seguridad.jpg

Ahora vamos a decirle a nuestro settings.py que vamos a utilizar la
aplicación seguridad:

.. code-block:: python

	INSTALLED_APPS = (
		'django.contrib.admin',
		'django.contrib.auth',
		'django.contrib.contenttypes',
		'django.contrib.sessions',
		'django.contrib.messages',
		'django.contrib.staticfiles',
		'seguridad',
		'personas',
	)

Y redefinimos nuestro archivo urls.py principal de tal manera que al
ingresar ya no apunte a la aplicación personas sino a seguridad:

.. code-block:: python

	from django.conf.urls import include, url
	from django.contrib import admin

	urlpatterns = [
		url(r'^admin/', include(admin.site.urls)),
		url(r'^', include('seguridad.urls',namespace='seguridad')),
		url(r'^personas/', include('personas.urls',namespace='personas')),
	]

Dentro de nuestra aplicación seguridad vamos a crear una vista llamada
Login:

.. code-block:: python

	#Importamos la vista genérica FormView
	from django.views.generic.edit import FormView
	from django.http.response import HttpResponseRedirect
	from django.core.urlresolvers import reverse_lazy
	#Importamos el formulario de autenticación de django
	from django.contrib.auth.forms import AuthenticationForm
	from django.contrib.auth import login

	# Create your views here.
	class Login(FormView):
		#Establecemos la plantilla a utilizar
		template_name = 'login.html'
		#Le indicamos que el formulario a utilizar es el formulario de autenticación de Django
		form_class = AuthenticationForm
		#Le decimos que cuando se haya completado exitosamente la operación nos redireccione a la url bienvenida de la aplicación personas
		success_url = reverse_lazy("personas:bienvenida")

	def dispatch(self, request, *args, **kwargs):
		#Si el usuario está autenticado entonces nos direcciona a la url establecida en success_url
		if request.user.is_authenticated():
			return HttpResponseRedirect(self.get_success_url())
		#Sino lo está entonces nos muestra la plantilla del login simplemente
		else:
			return super(Login, self).dispatch(request, *args, **kwargs)

	def form_valid(self, form):
		login(self.request, form.get_user())
		return super(Login, self).form_valid(form)


Expliquemos algunas cosas:

reverse_lazy, como su nombre lo indica, es una implementación perezosa
de la URL de resolución inversa(reverse). A diferencia de la función
inversa tradicional, reverse_lazy no se ejecutará hasta que se
necesite el valor. Es útil para prevenir excepciones 'Reverse Not
Found' cuando se trabaja con direcciones URL que no pueden ser
conocidos inmediatamente.
dispatch, en el archivo urls.py el punto de entrada as_view() crea una
instancia de la clase y llama al método dispatch(), (el despachador o
resolvedor de URL) que busca la petición para determinar si es un GET,
POST, etc, y releva la petición a un método que coincida con uno
definido, o levante una excepción HttpResponseNotAllowed si no
encuentra coincidencias.
form_valid, este método es llamado cuando el formulario valida los
datos.
login, la llamada a login() acepta un objeto de la clase HttpRequest y
un objeto User y almacena el identificador del usuario en la sesión,
usando el entorno de sesiones de Django. En el caso nuestro, al usar
el formulario de autenticación de django, invocando a form.get_user()
obtenemos el objeto User.

Ahora vamos a crear la plantilla login.html y la guardaremos dentro de
la carpeta templates(que debemos crear) en nuestra aplicación
seguridad:

.. code-block:: html

	{% load staticfiles %}
	<!DOCTYPE html>
	<html>
	    <head>
	        <meta charset="utf-8"/></meta>
	        <title>CIPROPY</title>
	        <meta content="width=device-width, initial-scale=1.0" name="viewport"></meta>
	        <link rel="stylesheet" type="text/css" href='{% static "css/bootstrap.min.css" %}'></link>
	    </head>
	    <body>
	        <div class="container">       
	            <br>          
	            <div class="row">
	                <div class="col-md-4 col-md-offset-4">
	                    <div class="login-panel panel panel-default">
	                        <div class="panel-heading">
	                            <h3 class="panel-title">CIPROPY</h3>
	                        </div>
	                        <div class="panel-body">
	                            <form role="form" method="post" id='form_login'>
	                                {% csrf_token %}
	                                <div class="form-group">
	                                    <label>Usuario:</label>
	                                    {{ form.username }}
	                                </div>
	                                <div class="form-group">
	                                    <label>Contraseña:</label>
	                                    {{ form.password }}
	                                </div>                                    
	                                <button class="btn btn-lg btn-success btn-block" type="submit" name="login"/>Login</button>                             
	                            </form>
	                        </div>
	                    </div>
	                </div>
	            </div>
	        </div>
	    </body>
	</html>


Nótese que esta plantilla no extiende de ninguna otra, ya que su
comportamiento es especial, otra cosa peculiar aquí es el uso de las
etiquetas {{ form.username }} y {{ form.password }} que al
renderizarse se comportan como dos cajas de texto una normal y la otra
tipo password, esto es posible gracias a que estamos usando el
Formulario AuthenticationForm de Django y lo hemos definido
previamente en nuestra vista Login, este formulario tiene los campos
username y password y los podemos utilizar en la plantilla
anteponiendo el objeto form, este comportamiento especial es definido
por la clase FormView.

Ahora si debemos crear un archivo urls.py en nuestra aplicación
seguridad de la siguiente manera:

.. code-block:: python

	from django.conf.urls import patterns, url
	from seguridad.views import Login

	urlpatterns = patterns(",
		url(r'^$', Login.as_view(), name="login"),
	)


Y listo con eso ya tenemos que al momento de correr nuestro servidor
de prueba nos va a mostrar lo siguiente:

.. image:: /images/blog/login.jpg

Con esto ya tenemos el proyecto listo pero no tenemos ningún usuario,
debemos crear el superusuario para probar, mas adelante en próximos
tutoriales crearemos mas usuarios para hacer las pruebas
correspondientes, nos movemos hasta la carpeta donde está ubicado el
proyecto y desde una terminal escribimos lo siguiente:

.. code-block:: bash
	
	python manage.py createsuperuser


Nos pedira algunos datos, que debemos completar o presionar enter para
dejar el dato mostrado por defecto entre paréntesis:

.. code-block:: bash

	Username (leave blank to use 'mamaya'):
	Email address:
	Password:
	Password (again):
	Superuser created successfully.


En mi caso he dejado por defecto el usuario mamaya, no le he puesto
ninguna dirección de correo y he completado la password, esta no
aparece al momento de ser digitada ya que es un mecanismo de
protección de django, si todo se ha hecho correctamente nos aparecerá
el mensaje “Superuser created successfully."

Ahora si ya podemos usar el usuario mamaya para loguearnos
.. image:: /images/blog/login.jpg

Si hemos escrito correctamente la password nos redireccionará hasta
nuestra vista principal de la aplicación personas y sino nos volverá a
mostrar la ventana de login:

.. image:: /images/blog/presonas.jpg

Listo eso es todo, ya estaremos mejorando el funcionamiento de nuestro
login en próximos posts.

Saludos.

.. _El Libro de Django: http://librosweb.es/libro/django_1_0/capitulo_12/utilizando_usuarios.html
.. _Formulario Login usando Class Based Views : http://www.secnot.com/django-custom-login-cbv.html


