---
layout: post
title: Migración de SQL a NoSQL-MONGODB
---
## Intro
Desde hace ya 50 años aproximadamente se han venido usando las bases de datos relacionales basadas en *SQL (Structured Query Language)*, las cuales permiten organizar datos en tablas relacionadas entre sí mediante claves primarias y foráneas, lo cual permite una gestión más eficiente y estructurada de la información.

Sin embargo, con el surgimiento del internet 2.0 a principios del siglo XXI (donde todos somos **generadores** y consumidores de contenido), las empresas de Internet comenzaron a enfrentar desafíos con las bases de datos relacionales tradicionales debido a la escala masiva y la naturaleza **no estructurada** de sus datos.

A partir de alrededor de 2009, las bases de datos *NoSQL (Not Only Structured Query Language)* han ido ganando terreno especialmente en aplicaciones web y móviles, así como en escenarios donde la escalabilidad y flexibilidad son prioritarias.

Las bases de datos **NoSQL** se pueden categorizar en cuatro grupos:
- Basada en documentos: Almacenan datos en formato de documentos, como JSON o XML, donde cada documento puede contener una estructura flexible y diferente. Ejemplos: MongoDB y Couchbase.

- Orientadas a columnas: Almacenan datos en columnas en lugar de filas, lo que permite una recuperación rápida y eficiente de datos específicos. Ejemplos: Apache Cassandra y HBase.

- Clave-valor: Almacenan datos como pares de clave-valor simples, lo que las hace ideales para aplicaciones que requieren una alta velocidad de lectura y escritura. Ejemplos: Redis y Amazon DynamoDB.

- Grafos: Modelan datos como grafos, que consisten en nodos y relaciones entre ellos, lo que facilita la representación y consulta de relaciones complejas. Ejemplos: Neo4j y Amazon Neptune. 

Cada tipo de base de datos NoSQL está optimizado para diferentes casos de uso y requisitos de escalabilidad, consistencia y disponibilidad.

En este artículo/tutorial se va a explicar cómo realizar una migración de una base de datos **SQL** a **NoSQL** de tipo **Documental** (*MongoDB*). Para realizar esta migración existen múltiples herramientas especializadas donde el proceso es prácticamente automático. Sin embargo, como este blog tiene una orientación didáctica, se utilizarán herramientas más básicas como **Google Colab** para realizar un preprocesamiento de los datos en lenguaje **Python** con el fin de transformar las *tablas SQL* a *colecciones de documentos NoSQL de Mongo*. A continuación, se ingresarán las colecciones a **MongoDB** a través del Power Shell de **MongoDB**, **Mongosh**.

## 1. Análisis del diagrama Entidad-Relación de la base de datos
![Diagrama_entidad_relacion]({{ site.baseurl }}/images/migracion_sql_mongo/entidad_relacion_blog.drawio.png)

Como se puede observar, se ha representado el modelo de entidad-relación donde se define cada tabla con la entidad que representa, sus atributos y  cuales de estos son los *primary keys* y *foreign keys*.

Además, se observa la cardinalidad de cada relación, en donde por simplicidad, se ha escogido un modelo donde solo existen relaciones uno a muchos (1:n).
			
A continuación, se analiza cada relación:
- *Cliente-Mascota*: Cada cliente puede tener una o muchas mascotas y cada mascota pertenece a un único cliente.
			 	
- *Cliente-orden*: Cada cliente puede tener una o muchas órdenes y cada orden tendrá a un único cliente. 
			 	
- *orden-detalle_orden*:  Cada orden puede tener uno o muchos detalles de orden (un mismo pedido con varios productos) y cada detalle_orden corresponderá a una única orden.
			 	
- *producto-detalle_orden*: Cada producto puede aparecer en uno o varios detalles de órdenes y cada detalle de orden debe tener un único producto.
			 	
- *producto-suministro*:  Cada producto puede aparecer en uno o varios suministros y cada suministro debe tener un único producto.
			 	
- *proveedor-suministro*:  Cada proveedor puede aparecer en uno o varios suministros y cada suministro debe tener un único proveedor.

Nótese que la tabla *detalle_orden* surge de una relación n:m entre las entidades *orden* y *producto*. Con la existencia de esta tabla se elimina este tipo de relación y se garantiza la unicidad de los datos.

## Estructura objetivo: colecciones de documentos
## Creación de **colecciones** de **documentos** a partir de la base de datos relacional
En primer lugar, se deberá de crear una colección vacía por cada entidad:
``` python
cliente = []
mascota = []
producto = []
orden = []
detalle_orden = []
proveedor = []
suministro = []
```
Con el fin de poder manipular las tablas en bucle, se incluyen todas en una lista de tuplas, conteniendo estas tuplas la *colección* en sí (objeto lista) y el nombre de la lista en string, que coincidirá con el nombre de la tabla de la base de datos.
```python
lst_colecciones = [ 
	(cliente, 'cliente'), 
	(mascota, 'mascota'), 
	(producto, 'producto'), 
	(orden, 'orden'), 
	(detalle_orden, 'detalle_orden'), 
	(proveedor, 'proveedor'), 
	(suministro, 'suministro'), 
	]
```
