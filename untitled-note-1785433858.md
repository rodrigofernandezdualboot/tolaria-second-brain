---
type: Note
_width: wide
---
#

Ok en esta nota vamos a hablar un poco de las disconexiones entre lo que es el approach propuesto por EY-P.Comparado con lo que nosotros haríamos, la propuesta de la consultora es arrancar con una actualización o modernización del frame [1.net](http://1.net), lo cual es llevarlo de 4.5 a 4.481. En un rango entre 400 y 700 horas, eso son entre 10 y 18 semanas. Creo que es demasiado tiempo eso y una persona con experiencia [en.net](http://en.net) podría ser capaz de redondear ese trabajo en un entorno de las cuatro semanas, o sea, 160, quizás un mes y medio. No creo que se precisen más.

Segundo la modernización de la aplicación del frontera, la mezcla que tienen de stack tecnológico es variada. La estimación de entre 820 y 1,000 horas es imposible de definir solo o comparar solo con una única imagen que tenemos en la propuesta. No me animo a decir si está bien o si está mal.

En cuanto al re factor de la lógica de negocio, o sea, extraer estos procedimientos de la base de datos en una serie de layer, su estimación es de entre 1,600 y 2,200 horas. Creo que el resultado de eso no sé si prepara la solución para lo que se viene porque es simplemente mover lo que está en la base de datos a una lógica de negocio. Como no existe quien sepa lo que tiene que hacer, lo que hacen estos procedimientos, no hay pruebas unitarias ni pruebas de integración y mucho menos. No hay forma de validar que ese re factor sigue produciendo los resultados esperados.

Luego la modernización de la infraestructura es gestionar el valor de la migración de ayer on-prem a un servicio clave. Yo creo que está súper justificada. Utilizando modelos como Azure Hyperstate, donde tenés bases de datos red only, tenés resuelta toda la parte de encriptación at rest. Tenés resuelto los backups, los point in time, el recovery, un backup de una semana, días, horas o con estrategias mayores. Todo eso resuelto en Azure.
