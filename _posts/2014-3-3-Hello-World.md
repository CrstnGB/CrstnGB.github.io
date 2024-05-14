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


vamonos que nos vamos

### Subsección de Otra sección
