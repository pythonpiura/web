.. title: Web Scrapping con Python y Selenium
.. slug: web-scrapping-con-python-y-selenium
.. date: 2016-05-07 20:58:06
.. tags: pillow,pytesseract,Python,Selenium
.. description:

El día de hoy tuvimos el agrado de ser invitados por el `Círculo UEL`_
a dar una charla en el marco del FLISOL 2016 en la Universidad Los
Ángeles de Chimbote Sede Piura y el tema fue Web Scrapping con Python
y Selenium, hicimos algunos ejemplos muy interesantes y nos encantó la
acogida del público y el interés suscitado, vamos a compartir los
artículos que usamos:

`Selenium con Python`_

`Romper Captchas con Pytesseract y Selenium`_

Las diapositivas:

`Descargar`_

El script que fue la estrella de la mañana:

.. code-block:: python

	# -*- coding: utf-8 -*-
	from selenium import webdriver
	from selenium.webdriver.common.by import By
	from selenium.webdriver.support.wait import WebDriverWait
	from selenium.webdriver.support import expected_conditions as EC
	import os

	try:
		import Image
	except ImportError:
		from PIL import Image
	import pytesseract

	def decodifica_campo(campo):
		return u"%s" % campo

	def ir_sunat_web(ruta):
		fp = webdriver.FirefoxProfile()
		fp.set_preference("browser.download.manager.showWhenStarting",False)
		fp.set_preference("browser.download.manager.closeWhenDone", True);
		fp.set_preference("browser.download.manager.showAlertOnComplete", False);
		fp.set_preference("browser.helperApps.neverAsk.saveToDisk", "application/zip");
		driver = webdriver.Firefox(fp)
		driver.get("http://www.sunat.gob.pe/cl-ti-itmrconsmulruc/jrmS00Alias")
		try:
			WebDriverWait(driver,5).until(EC.presence_of_element_located((By.NAME, "imagen")))
		except:
			print "No carga imagen"
		driver.save_screenshot("screenshot.png")
		img=Image.open('screenshot.png')
		img_recortada = img.crop((700,309,800,361))
		img_recortada.save("recorte.png")
		try:
			captcha = pytesseract.image_to_string(img_recortada)
			codigo = driver.find_element_by_name("codigoA")
			codigo.send_keys(captcha)
		except:
			pass
		archivo = driver.find_element_by_name("archivo")
		archivo.send_keys(ruta)
		form = driver.find_element_by_name("frmConsMulRucArchivo")
		botones=form.find_elements_by_class_name("form-button")
		botones[1].click()
		try:
			WebDriverWait(driver,5).until(EC.presence_of_all_elements_located((By.CLASS_NAME, "form-table")))
			enlace=driver.find_element(By.TAG_NAME,"a")
			enlace.click()
		except:
			print "Element is not present"
			driver.close()

	def main():
		ruta = "/home/usuario/carpeta"
		ruta_archivo=os.path.join(ruta,"rucs.zip")
		ir_sunat_web(ruta_archivo)

	main()


Así como también un vídeo de su funcionamiento.

.. youtube:: mUaVFYTlibM

Saludos y muchas gracias.

.. _Selenium con Python: https://pythonpiura.wordpress.com/2014/09/18/selenium-con-python/
.. _Romper Captchas con Pytesseract y Selenium: https://pythonpiura.wordpress.com/2016/01/26/romper-captchas-con-pytesseract-y-selenium/
.. _Círculo UEL: https://cideuel.wordpress.com/
.. _Descargar: /recursos/charlapythonflisol2016.pdf


