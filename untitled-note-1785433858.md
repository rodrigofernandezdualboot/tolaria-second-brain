---
type: Note
_width: wide
---
#

Ok en esta nota vamos a hablar un poco de las disconexiones entre lo que es el approach propuesto por EY-P.Comparado con lo que nosotros haríamos, la propuesta de la consultora es arrancar con una actualización o modernización del frame [1.net](http://1.net), lo cual es llevarlo de 4.5 a 4.481. En un rango entre 400 y 700 horas, eso son entre 10 y 18 semanas. Creo que es demasiado tiempo eso y una persona con experiencia [en.net](http://en.net) podría ser capaz de redondear ese trabajo en un entorno de las cuatro semanas, o sea, 160, quizás un mes y medio. No creo que se precisen más.

Segundo la modernización de la aplicación del frontera, la mezcla que tienen de stack tecnológico es variada. La estimación de entre 820 y 1,000 horas es imposible de definir solo o comparar solo con una única imagen que tenemos en la propuesta. No me animo a decir si está bien o si está mal.

En cuanto al re factor de la lógica de negocio, o sea, extraer estos procedimientos de la base de datos en una serie de layer, su estimación es de entre 1,600 y 2,200 horas. Creo que el resultado de eso no sé si prepara la solución para lo que se viene porque es simplemente mover lo que está en la base de datos a una lógica de negocio. Como no existe quien sepa lo que tiene que hacer, lo que hacen estos procedimientos, no hay pruebas unitarias ni pruebas de integración y mucho menos. No hay forma de validar que ese re factor sigue produciendo los resultados esperados.

Luego la modernización de la infraestructura es gestionar el valor de la migración de ayer on-prem a un servicio clave. Yo creo que está súper justificada. Utilizando modelos como Azure Hyperstate, donde tenés bases de datos red only, tenés resuelta toda la parte de encriptación at rest. Tenés resuelto los backups, los point in time, el recovery, un backup de una semana, días, horas o con estrategias mayores. Todo eso resuelto en Azure.Del otro lado está el source code management: una migración de DFS a AYU. Si bien son tecnologías muy distintas, no debería ser muchísimo tiempo la migración. Basta con tener una versión local estable del código, una carpeta en directoria, crear un repositorio y subirlo. No veo cómo se pueden llegar a invertir tres semanas, 380 y 300 horas, como está propuesto. Creo que ahí podemos hacerlo mucho más rápido.

En cuanto a Pipeline, si hay ANC-D, esto sí puede llegar a ser bastante más complejo debido al stack tecnológico. Asumiendo que vamos a trabajar con 4.5 o 4.81, puede llegar a ser complejo el deployment si este tiene que ser algo que es on-prem. Probablemente tengamos que deployerlo en un AIS. Eso va a ser problemático porque, literalmente, hay que cipiar los archivos, subirlos por SFTP, extraerlos en el AIS con downtime. Nada, hay estrategias un poco más prolijas con tecnologías bastante más modernas que esto.

El Data Warehouse, estoy de acuerdo que puede ser algo que puede ser diferido a una siguiente fase. Hay algo que no se ve o no está contemplado acá, que es la existencia de Power BI y un set de reportes al cual no tenemos acceso. Power BI, los reportes están construidos sobre una base de datos viva. Es decir que cualquier cambio que hagamos nosotros a la base de datos va a impactar en los reportes. Entonces para poder modificar el esquema primero deberíamos extraer una capa, generar una capa de abstracción para los reportes y luego empezar a realizar una migración.

Después la actualización del sistema operativo. Nada esos son los problemas típicos de trabajar con un PREM: genera downtime, genera redistribución de tráfico a una diferente región. Otro gran ejemplo de las cosas que con Azure no deberíamos tener.
