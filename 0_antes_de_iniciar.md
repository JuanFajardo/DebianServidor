# 0 Antes de iniciar

## 0.1 Sobre este Guia

Este guia ilustra, paso a paso, la instalacion y configuracion de un servidor Debian GNU/Linux casero, cubriendo no solo el sistema base, sino también diversos servicios de red, como el servidor de correo, servidor de Internet, hasta los sistemas de seguridad y monitoreo del sistema.

> __Atención__
> Esta guía tiene únicamente fines educativos y de aprendizaje. Como cualquier guía, puede contener algunas imprecisiones o estar sujeto a interpretaciones incorrectas. En ningún caso se me deberá ni podrá responsabilizar por ningún tipo de daño resultante del uso de este guía.

Las instrucciones de este guía están basadas en la versión 10 'Buster' de la distribución Debian GNU/Linux.

## ¿Es este guía para ti?
__Este guía es para quienes:__
- Son entusiastas de Linux y desean instalar un pequeño servidor "casero" como ejercicio para aprender más sobre GNU/Linux.
- Tienen suficiente tiempo disponible para aprender, investigar y resolver problemas que aparecerán.
- No se rinden fácilmente ante la primera dificultad o contratiempo.

__Este guía NO es para quienes:__
- Buscan una forma fácil de impresionar al jefe con un servidor empresarial configurado con copiar/pegar y listo para usar.
- No tienen conocimientos sobre GNU/Linux y/o computadoras en general.
- Creen que pueden configurar un servidor y despreocuparse para siempre.
- Piensan que pueden autodenominarse Administradores de Sistemas solo por haber leído este manual.

__Hablemos en serio__

Instalar, configurar y mantener un servidor (ya sea GNU/Linux u otro) requiere un profesional altamente capacitado, dedicado y con mucha experiencia. Además, mantener un servidor es una tarea continua y no es, ciertamente, el trabajo más fácil del mundo.

Este guía es, básicamente, una compilación de mi proceso de aprendizaje, por lo que debe usarse únicamente como una herramienta educativa. Al ser un ejercicio de aprendizaje, responde a mis necesidades y no necesariamente a las de otras personas, por lo que debe considerarse solo como una base de trabajo para investigaciones personales.

Si esta guía genera más preguntas que respuestas, ¡felicitaciones! Estás en el camino correcto y listo para desafíos más ambiciosos.

__Cómo leer este guía__

El contenido está organizado de manera secuencial, siguiendo la lógica de configuración de un servidor y sus dependencias. Por ejemplo, la configuración de red debe completarse con éxito antes de comenzar con la instalación y configuración de un servidor web.

Por lo tanto, la mejor manera de usar este guía es seguir su estructura paso a paso. Sin embargo, para facilitar su consulta, también está dividido por temas en capítulos, secciones y artículos.

## Prueba antes de comprometerte
Una excelente forma de experimentar sin compromisos antes de trabajar seriamente es realizar una instalación virtual.

Crear un Servidor Virtual en tu computadora
Actualmente existen excelentes herramientas de virtualización. Dos de las más destacadas por su facilidad de uso y capacidad son:

- __[VirtualBox:](https://www.virtualbox.org/)__ Un software de código abierto y gratuito con una impresionante lista de funcionalidades. Es la herramienta utilizada para desarrollar este guía.

- __[VMware:](https://www.vmware.com/products/desktop-hypervisor/workstation-and-fusion)__ Un software propietario de virtualización que puede utilizarse de forma gratuita.

__Crear un Servidor Virtual Privado en la Nube__

Otra opción es crear un servidor virtual privado (VPS) a un costo bastante reducido. Durante la configuración, puedes personalizar aspectos como el número de procesadores, la cantidad de memoria RAM, el espacio de almacenamiento en disco, el volumen de tráfico mensual permitido, así como el sistema operativo y su versión:

|Proveedor|	Costo Mensual|	CPU|	RAM|	Disco|	Tráfico Mensual|
|---------|--------------|-----|-------|---------|-----------------|
|[DigitalOcean](https://www.digitalocean.com)|	$5|	1 Core|	1 GB|25 GB SSD|	1 TB|
|[Linode](https://www.linode.com/)|	$5|	1 Core|	1 GB|	25 GB SSD|	1 TB|
|[Vultr](https://www.vultr.com/)|	    $5|	1 Core|	1 GB|	25 GB SSD|	1 TB|
|[Hetzner](https://www.hetzner.com/)|	$5|	2 Cores|4 GB|	40 GB SSD|	20 TB|


> __¡Advertencia Legal!__
> Como cualquier guía, esta también puede contener algunas imprecisiones o estar sujeta a interpretaciones incorrectas. En ningún caso se me deberá ni podrá responsabilizar por ningún tipo de daño derivado del uso de este guía. ¡Quedas avisado!

__¡Diviértete!__
Aprender más sobre Linux configurando un servidor casero puede ser una experiencia esclarecedora y divertida. ¿Por qué no intentarlo?