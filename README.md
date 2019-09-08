# Big Data Architecture. Práctica

## Enunciado

*Diseñar, especificar y desplegar un datalake para el procesamiento de datos provenientes de fuentes de datos no estructurados extraídos mediante técnicas de scraping/crawling de sitios de dominio público.*

El objetivo del diseño de este DataLake será tomar como entrada una serie de datos de un apartamento turístico de la ciudad de Madrid (dirección, nº dormitorios, nº baños, metros cuadrados, fotografias...) y mostrar de forma gráfica una evolución del precio estimado para el alquiler a dos meses vista.

La fuente principal de datos será el dataset de Airbnb pero voy a enriquecer estos datos con otros obtenidos a través de Crawling y Scraping.



## Parte 1

*Utilizar una herramienta de diagramado como Google Draw o DIA para diseñar y especificar el flujo de datos y herramientas utilizadas*

El fichero "Diagrama Practica" de este repositorio contiene el diagrama solicitado. 

Utilizaré un Cluster de Hadoop en Google Cloud que se encargará de procesar datos extraidos de Airbnb y de una serie de ficheros adicionales que conseguiré usando Crawling y Scraping. El objetivo de este procesamiento será dar de forma gráfica una estimación del precio de alquilé a dos meses vista para un apartamento turístico en Madrid, conociendo detalles del apartamento como dirección, nº de dormitorios, nº baños, metros cuadrados, fotografías..
Además utilizaré HIVE también en Cloud para tener todos los datos extraidos dentro de tablas a las que poder realizar consultas SQL cuando estemos realizando el procesamiento y análisis de esos datos.

De momento como datos adicionales a los de Airbnb se me ha ocurrido usar datos de noticias de los diferentes barrios, localización de parkings, eventos culturales previstos, localización de monumentos y museos.

Para esta arquitectura de momento los datos del apartamento será un fichero que se subirá al Cloud Storage de forma manual, pero la idea final sería tener un interfaz web en el que el usuario pudiera introducir los datos y que mediante la API REST de Google Cloud se enviaran esos datos y se lanzaran las tareas de procesamiento.


### Fuentes de datos:

- [Airbnb](https://public.opendatasoft.com/explore/dataset/airbnb-listings/export/?disjunctive.host_verifications&disjunctive.amenities&disjunctive.features&q=Madrid&dataChart=eyJxdWVyaWVzIjpbeyJjaGFydHMiOlt7InR5cGUiOiJjb2x1bW4iLCJmdW5jIjoiQ09VTlQiLCJ5QXhpcyI6Imhvc3RfbGlzdGluZ3NfY291bnQiLCJzY2llbnRpZmljRGlzcGxheSI6dHJ1ZSwiY29sb3IiOiJyYW5nZS1jdXN0b20ifV0sInhBeGlzIjoiY2l0eSIsIm1heHBvaW50cyI6IiIsInRpbWVzY2FsZSI6IiIsInNvcnQiOiIiLCJzZXJpZXNCcmVha2Rvd24iOiJyb29tX3R5cGUiLCJjb25maWciOnsiZGF0YXNldCI6ImFpcmJuYi1saXN0aW5ncyIsIm9wdGlvbnMiOnsiZGlzanVuY3RpdmUuaG9zdF92ZXJpZmljYXRpb25zIjp0cnVlLCJkaXNqdW5jdGl2ZS5hbWVuaXRpZXMiOnRydWUsImRpc2p1bmN0aXZlLmZlYXR1cmVzIjp0cnVlfX19XSwidGltZXNjYWxlIjoiIiwiZGlzcGxheUxlZ2VuZCI6dHJ1ZSwiYWxpZ25Nb250aCI6dHJ1ZX0%3D&location=16,41.38377,2.15774&basemap=jawg.streets)

- [Noticias Locales Madrid](https://www.eldistrito.es/distritos/)

- [API EMT Madrid](https://apidocs.emtmadrid.es/)

- [API Datos abiertos Madrid](https://datos.madrid.es/portal/site/egob/menuitem.214413fe61bdd68a53318ba0a8a409a0/?vgnextoid=b07e0f7c5ff9e510VgnVCM1000008a4a900aRCRD&vgnextchannel=b07e0f7c5ff9e510VgnVCM1000008a4a900aRCRD&vgnextfmt=default)

### Obtención de datos 

Tengo las fuentes de datos anteriores y mediante una serie de tareas programadas (cron) en un servidor Ubuntu voy a ejecutar scripts en Python para obtener esos datos. Exepto los datos de las noticias que voy a sacarlos de forma diaria el resto se obtendrán una vez por semana.
Estos scripts de Python generan una serie de ficheros csv con los datos. Para las noticias locales de Madrid se utilizará la técnica de Crawling y para el resto de datos utilizaré Scraping haciendo uso de APIs públicas.

- Fichero csv Airbnb. [Airbnb](https://public.opendatasoft.com/explore/dataset/airbnb-listings/export/?disjunctive.host_verifications&disjunctive.amenities&disjunctive.features&q=Madrid&dataChart=eyJxdWVyaWVzIjpbeyJjaGFydHMiOlt7InR5cGUiOiJjb2x1bW4iLCJmdW5jIjoiQ09VTlQiLCJ5QXhpcyI6Imhvc3RfbGlzdGluZ3NfY291bnQiLCJzY2llbnRpZmljRGlzcGxheSI6dHJ1ZSwiY29sb3IiOiJyYW5nZS1jdXN0b20ifV0sInhBeGlzIjoiY2l0eSIsIm1heHBvaW50cyI6IiIsInRpbWVzY2FsZSI6IiIsInNvcnQiOiIiLCJzZXJpZXNCcmVha2Rvd24iOiJyb29tX3R5cGUiLCJjb25maWciOnsiZGF0YXNldCI6ImFpcmJuYi1saXN0aW5ncyIsIm9wdGlvbnMiOnsiZGlzanVuY3RpdmUuaG9zdF92ZXJpZmljYXRpb25zIjp0cnVlLCJkaXNqdW5jdGl2ZS5hbWVuaXRpZXMiOnRydWUsImRpc2p1bmN0aXZlLmZlYXR1cmVzIjp0cnVlfX19XSwidGltZXNjYWxlIjoiIiwiZGlzcGxheUxlZ2VuZCI6dHJ1ZSwiYWxpZ25Nb250aCI6dHJ1ZX0%3D&location=16,41.38377,2.15774&basemap=jawg.streets). Lo obtengo a través de un cron programado en un servidor Ubuntu que ejecutará un script en python todos los domingos a las 00:00

- Fichero csv de noticias locales de Madrid. Lo obtendré a través de un cron programado en un servidor Ubuntu que ejecutará un spript todos los dias a las 00:00 y que se encargará de realizar Crawling en esta dirección. [Noticias Locales Madrid](https://www.eldistrito.es/distritos/)

- Fichero csv con datos de parkings de Madrid. Lo obtengo a través de un cron programado en un servidor Ubuntu que ejecutará un scrip en python todos los domingos a las 00:00 y que realizará cunsultas a esta API de la EMT de Madrid. [API EMT Madrid](https://apidocs.emtmadrid.es/)

- Fichero csv con Actividades Culturales y de Ocio Municipal en los próximos 100 dias. Lo obtengo a través de un cron programado en un servidor Ubuntu que ejecutará un scrip en python todos los domingos a las 00:00 y que realizará consultas a esta API de datos abiertos de Madrid. [API Datos abiertos Madrid](https://datos.madrid.es/portal/site/egob/menuitem.214413fe61bdd68a53318ba0a8a409a0/?vgnextoid=b07e0f7c5ff9e510VgnVCM1000008a4a900aRCRD&vgnextchannel=b07e0f7c5ff9e510VgnVCM1000008a4a900aRCRD&vgnextfmt=default)

- Fichero csv con Monumentos de la Ciudad de Madrid. Lo obtengo a través de un cron programado en un servidor Ubuntu que ejecutará un scrip en python todos los domingos a las 00:00 y realiza consultas misma API anterior. [API Datos abiertos Madrid](https://datos.madrid.es/portal/site/egob/menuitem.214413fe61bdd68a53318ba0a8a409a0/?vgnextoid=b07e0f7c5ff9e510VgnVCM1000008a4a900aRCRD&vgnextchannel=b07e0f7c5ff9e510VgnVCM1000008a4a900aRCRD&vgnextfmt=default)

- Fichero csv con los Museos de la Ciudad de Madrid. Lo obtengo a través de un cron programado en un servidor Ubuntu que ejecutará un scrip en python todos los domingos a las 00:00 y realiza consultas misma API anterior. [API Datos abiertos Madrid](https://datos.madrid.es/portal/site/egob/menuitem.214413fe61bdd68a53318ba0a8a409a0/?vgnextoid=b07e0f7c5ff9e510VgnVCM1000008a4a900aRCRD&vgnextchannel=b07e0f7c5ff9e510VgnVCM1000008a4a900aRCRD&vgnextfmt=default)


Los ficheros csv que se vayan obteniendo se cargarán en un segmento del Cloud Storage de Google


## Carga de datos en Hadoop - Staging

Al utilizar Hadoop en Cloud y el Cloud Storage la fase de Staging queda reducida únicamente a subir los ficheros csv al Cloud Storage. Los mismos scripts de Python que obtienen los datos se encargarán en esta práctica de guardarlos en el Cloud Storage y ya estará disponibles para HADOOP.

Para un proyecto final habría que meter una fase más antes de la carga de los ficheros en el Cloud Storage. Esta fase sería la de limpieza datos de forma que una vez estén subidos a HADDOP ya tengamos estos ficheros en los formatos adecuados y perfectamente preparados para el procesamiento.


## Hadoop

En Google Cloud tendré un cluster de Hadoop con tres contenedores. Dentro de Hadoop se realizarán el/los proceso/s que se encargue/n de la estimación temporal de precios (de momento será un simple Wordcount)


## HIVE

Voy a montar también un HIVE en la nube de Google. La idea es que los datos csv que se han obtenido se acaben cargando en tablas de HIVE de forma que el/los proceso/s que se encargue/n de calcuar los precios pueda/n realizar consultas SQL a HIVE. Por ejemplo que me ocurre que si el piso del que quiero saber los precios estimados está en el barrio SOL se pueda realizar una consulta SQL para ver cuales son las actividades de ocio programadas en ese barrio en los dos meses siguientes o si hay parking municipales en ese barrio, etc...

La carga de los datos en las tablas de HIVE de los ficheros csv se puede hacer desde el Cloud Storage. La carga de las noticias locales se hará de forma diaria a las 0:30 y el resto de datos los domingos a las 01:00


## JAR

El proceso JAR (que no sé de momento si será uno o varios) se encargará de realizar todo el procesamiento y las consultas necesarias a HIVE para ser capaz de hacer la estimación de precios a dos meses vista. Tomará como entrada los datos que se aporten del apartamento del usuario y el resultado se representaría de forma gráfica con alguna herramienta de visualización.
De momento en esta práctica este proceso va a realizar solo un wordcount de los ficheros y almacenará el resultado en una carpeta output del Cloud Storage


## Parte 2

*Crear un scraper en Google Colaboratory a partir de un API o de un crawler con scrapy, que descargue los datos a un archivo de formato estructurado*

### Colaboratory 1. Dataset de Airbnb

El Dataset de Airbnb lo tenemos en esta dirección [Airbnb](https://public.opendatasoft.com/explore/dataset/airbnb-listings/export/?disjunctive.host_verifications&disjunctive.amenities&disjunctive.features&q=Madrid&dataChart=eyJxdWVyaWVzIjpbeyJjaGFydHMiOlt7InR5cGUiOiJjb2x1bW4iLCJmdW5jIjoiQ09VTlQiLCJ5QXhpcyI6Imhvc3RfbGlzdGluZ3NfY291bnQiLCJzY2llbnRpZmljRGlzcGxheSI6dHJ1ZSwiY29sb3IiOiJyYW5nZS1jdXN0b20ifV0sInhBeGlzIjoiY2l0eSIsIm1heHBvaW50cyI6IiIsInRpbWVzY2FsZSI6IiIsInNvcnQiOiIiLCJzZXJpZXNCcmVha2Rvd24iOiJyb29tX3R5cGUiLCJjb25maWciOnsiZGF0YXNldCI6ImFpcmJuYi1saXN0aW5ncyIsIm9wdGlvbnMiOnsiZGlzanVuY3RpdmUuaG9zdF92ZXJpZmljYXRpb25zIjp0cnVlLCJkaXNqdW5jdGl2ZS5hbWVuaXRpZXMiOnRydWUsImRpc2p1bmN0aXZlLmZlYXR1cmVzIjp0cnVlfX19XSwidGltZXNjYWxlIjoiIiwiZGlzcGxheUxlZ2VuZCI6dHJ1ZSwiYWxpZ25Nb250aCI6dHJ1ZX0%3D&location=16,41.38377,2.15774&basemap=jawg.streets)

Podemos escoger entre bajarnos una versión resumida de 14.780 registros o el dataset completo. Para esta práctica voy a utilizar la versión reducida. Podemos bajarnos el archivo directamente desde este enlace y luego subirlo al Cloud Storage. De cara a una automatización en el futuro el código python para bajarse este archivo se muestra en este Colaboratory: 

- [Ver Código](https://colab.research.google.com/drive/1kcGH65fr6VqOjbSucdFaLpxjiC_dzzM5)


### Colaboratory 2. Crawling Noticas Locales Madrid

Mediante el siguiente código en Python voy a realizar un Crawling de una página de noticias locales de Madrid. Como resultado tendré un archivo "datanoticias.csv" que contendrá los siguientes datos de las noticias: fecha, distrito, titular.
En principio para la prática me quedaré solo con las 32 primeras noticias

- [Ver Código](https://colab.research.google.com/drive/1EoVTmGQbznHsgctedHsgZDgowXDXx1Px)

### Colaboratory 3. API de la EMT de Madrid

Mediante el siguiente código en Python hago consultas a la API de la EMT de Madrid para conocer datos de aparcamientos municipales. Como resultado tendré un archivo "Parkings.csv" que contendrá los siguientes datos de cada aparcamiento: nombre, código postal, coordenadas geográficas y dirección

- [Ver Codigo](https://colab.research.google.com/drive/1tlLzm6tJhjdMsp1QP98ArnjjmKL67ey8)

### Collaboratory 4. API Datos Abiertos de Madrid

Mediante el siguiente código en Python hago consultas a la API de Datos Abiertos de Madrid para conocer datos sobre:
- **Actividades Culturales y de Ocio Municipal en los próximos 100 días**. Como resultado obtengo un archivo "actividades.csv" que contendrá los siguientes datos de las actividades: Fecha de inicio, Fecha de fin, Título de la actividad, Descripción de la actividad, Distrito, Latitud, Longitud
- **Monumentos de la ciudad de Madrid**. Como resultado obtengo un archivo "monumentos.csv" que contendrá los siguientes datos de los monumentos: Titulo, Barrio, Distrito, Codigo Postal, Direccion, Latitud, Lontitud, Descripcion
- **Museos de la ciudad de Madrid** Como resultado obtengo un archivo "museos.csv" qque contendrá los siguientes datos de los monumentos: Titulo, Barrio, Distrito, Codigo Postal, Direccion, Latitud, Lontitud, Descripcion
- [Ver Codigo](https://colab.research.google.com/drive/15GBd3Z4aQHxVjWo4Fs11xcaEZkFyK_dm)

## Parte 3

*Utilizar un proveedor de Cloud para montar un cluster de al menos 3 contenedores configurados correctamente*

Para montar el cluster de Haddop en Cloud uso GCP (Google Cloud Computing) Dataproc. Describo a continuación los pasos seguidos para montar un cluster de HADOOP en GCP Dataproc con un nodo master y tres slaves

Tengo ya una cuenta de google y me he dado de alta en https://cloud.google.com/ 

#### Paso 1

Entramos en esta dirección https://console.cloud.google.com/getting-started y hacemos click en Compute Engine:

![Paso 1](https://raw.githubusercontent.com/mcpade/practica_arquitectura/master/Paso_1_GCP.png)

#### Paso 2

Lo siguiente será crear un proyecto nuevo haciendo en click en Nuevo Proyecto

![Paso 2](https://raw.githubusercontent.com/mcpade/practica_arquitectura/master/Paso_2_GCP.png)

Le doy un nombre al Proyecto: "MiProyectoEjemplo" y hago click en Crear

![Paso 2b](https://raw.githubusercontent.com/mcpade/practica_arquitectura/master/Paso_2b_GCP.png)



## Parte 4

*Subir los archivos extraídos durante la parte 2 al cluster de Hadoop e insertarlos en el HDFS. Indicar pasos necesarios para realizar esto, dependiendo de la opción elegida en el Sprint 3*

Al utilizar arquitectura Cloud para Hadoop la fase de staging se reduce a la subida de datos a un segmento de Cloud Storage. En esta práctica dentro de los propios Colaboratory  que se usan para extraer los datos se termina subiendo al Cloud Storage de Google usando la API de Python (se pueden consultar en la Parte 2)

Muestro a continuación una imagen donde se ve como quedan subidos los diferentes archivos csv en mi segmento del Cloud Storage. También está subida en este repositorio

- [Segmento Cloud Storage](https://drive.google.com/file/d/1j3wOPOMKehYlUeEn-KJjBsEkkaXNhw8O/view?usp=sharing)

*Realizar la tarea de procesamiento de datos sobre los datos extraídos utilizando WordCount*

(PENDIENTE)

## Parte 5

*Utilizar HIVE para insertar los datos extraídos durante el Sprint 2 y realizar queries con los mismos. Indicar los pasos y las decisiones de diseño respecto a cómo organizar los datos.*

(PENDIENTE)










