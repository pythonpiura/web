.. title: Romper Captchas con Pytesseract y Selenium
.. slug: romper-captchas-con-pytesseract-y-selenium
.. date: 2016-01-26 14:17:00
.. tags: Python,Selenium
.. description: 

En muchas de las páginas que consultamos tenemos una imagen que
contiene un pequeño texto que generalmente es de 4 o de 6 letras, esta
imagen es conocida como captcha y sirve para evitar la consulta masiva
de datos y el uso de boots para hacer consultas, el problema radica en
que podemos convertir esta imagen en texto y buscar la forma de
ingresar estos datos de manera automática sin intervención humana.
Para hacer la conversión de la imagen en texto tenemos la librería
pytesseract que hace uso del programa tesseract-ocr y para el ingreso
automático de datos y el acceso a la página objetivo contamos con la
ayuda de nuestro viejo conocido selenium, cabe destacar que este
procedimiento funciona siempre y cuando la imagen del captcha no sea
tan complejo y contenga solamente texto.
Hemos creado un script de manera general sin tener una página objetivo
en específico, pero para aplicarlo se debe conocer bien la página a
testear tal como hemos explicado en posts anteriores, analizando sus
elementos y la ubicación del captcha.
Antes de empezar debemos tener instalado el programa tesseract-ocr ya
que no podremos hacer nada sin este, lo podemos descargar de la
siguiente `ubicación`_.

Posteriormente procedemos a instalar las librerias pytesseract y
pillow(para el manejo y tratamiento de imágenes)

.. code-block:: bash

	pip install pillow
	pip install pytesseract

Y finalmente hacemos nuestro script el cual está debidamente
documentado para que lo puedan adaptar y modificar a su gusto.

.. code-block:: python
	
	# -*- coding: utf-8 -*- 
	from selenium import webdriver
	from selenium.webdriver.common.by import By
	from selenium.webdriver.support.wait import WebDriverWait
	from selenium.webdriver.support import expected_conditions as EC
	 
	try:
	    import Image
	except ImportError:
	    from PIL import Image
	import pytesseract
	 
	def ir_pagina_web(cadena,direccion_web):
	    driver = webdriver.Firefox()
	    #Vamos a la dirección web de la página objetivo
	    driver.get(direccion_web)
	    try:
	        #Esperamos que se cargue correctamente la caja de texto de búsqueda
	        WebDriverWait(driver, 5).until(EC.presence_of_element_located((By.ID, "txtBusqueda")))
	    except:
	        print "El elemento no está presente"
	    #Obtenemos la caja de texto de búsqueda
	    placa = driver.find_element_by_id("txtBusqueda")
	    #Enviamos la cadena a buscar en la caja de texto 
	    placa.send_keys(cadena)
	    #Hacemos una captura de pantalla de la página objetivo
	    driver.save_screenshot("screenshot.jpg")
	    #Abrimos la captura hecha
	    img=Image.open('screenshot.jpg')
	    #Recortamos la captura de acuerdo a la ubicación del captcha.
	    #El primer argumento es la posición "x" inicial, el segundo la posición "y" inicial.
	    #El tercer argumento es la posición "x" final y el cuarto la posición "y" final.
	    img_recortada = img.crop((380,300,550,340))
	    #Guardamos el área seleccionada con el nombre de "recorte.jpg"
	    img_recortada.save("recorte.jpg")
	    try:
	        #Convertimos la imagen a cadena de texto usando la libreria pytesseract
	        captcha = pytesseract.image_to_string(img_recortada)
	        #Obtenemos la caja de texto donde se ingresa el contenido del captcha
	        codigo = driver.find_element_by_id("txtCaptcha")
	        #Enviamos el captcha obtenido
	        codigo.send_keys(captcha)
	    except:
	        print "Error al convertir imagen en texto"
	    #Obtenemos el botón buscar
	    boton=driver.find_element_by_id("btn_buscar")
	    #Hacemos click en el botón
	    boton.click()           
	    driver.close()
	 
	def main():
	    ir_pagina_web("cadena","direccion_web")
	         
	main()

Por cierto hemos realizado esta prueba en Windows, nos queda la tarea
pendiente de hacerlo en GNU/Linux.

.. _ubicación: http://en.osdn.jp/projects/sfnet_tesseract-ocr-alt/downloads/tesseract-ocr-setup-3.02.02.exe/