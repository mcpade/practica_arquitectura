# Big Data Architecture. Práctica - María Araceli Paredes Delgado (mariceli.paredes@gmail.com)

## Enunciado

*Diseñar, especificar y desplegar un datalake para el procesamiento de datos provenientes de fuentes de datos no estructurados extraídos mediante técnicas de scraping/crawling de sitios de dominio público.*

El objetivo del diseño de este DataLake será tomar como entrada una serie de datos de un apartamento turístico de la ciudad de Madrid (dirección, nº dormitorios, nº baños, metros cuadrados, fotografias...) y mostrar de forma gráfica una evolución del precio estimado para el alquiler a dos meses vista.

La fuente principal de datos será el dataset de Airbnb pero voy a enriquecer estos datos con otros obtenidos a través de Crawling y Scraping.



## Parte 1

*Utilizar una herramienta de diagramado como Google Draw o DIA para diseñar y especificar el flujo de datos y herramientas utilizadas*

El fichero "Diagrama_Practica_Arquitectura" de este repositorio contiene el diagrama solicitado. 

![Diagrama_Practica_Arquitectura](https://raw.githubusercontent.com/mcpade/practica_arquitectura/master/Diagrama_Practica_Arquitectura.jpg)



Utilizaré un Cluster de Hadoop en Google Cloud que se encargará de procesar datos extraidos de Airbnb y de una serie de ficheros adicionales que conseguiré usando Crawling y Scraping. El objetivo de este procesamiento será dar de forma gráfica una estimación del precio de alquilé a dos meses vista para un apartamento turístico en Madrid, conociendo detalles del apartamento como dirección, nº de dormitorios, nº baños, metros cuadrados, fotografías..
Además utilizaré HIVE también en Cloud para tener todos los datos extraidos dentro de tablas a las que poder realizar consultas SQL cuando estemos realizando el procesamiento y análisis de esos datos.

De momento como datos adicionales a los de Airbnb se me ha ocurrido usar datos de noticias de los diferentes barrios, localización de parkings, eventos culturales previstos, localización de monumentos y museos.

En principio, para esta arquitectura, la información del apartamento sobre el que queremos realizar la estimación de precio estará en un fichero que se subirá al Cloud Storage de forma manual, pero la idea final sería tener un interfaz web en el que el usuario pudiera introducir los datos y que mediante la API REST de Google Cloud se enviaran esos datos y se lanzaran las tareas de procesamiento.


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


### Carga de datos en Hadoop - Staging

Al utilizar Hadoop en Cloud y el Cloud Storage, la fase de Staging queda reducida únicamente a subir los ficheros csv al Cloud Storage. Los mismos scripts de Python que obtienen los datos se encargarán en esta práctica de guardarlos en el Cloud Storage y ya estarán disponibles para HADOOP.

Para un proyecto final habría que meter una fase más antes de la carga de los ficheros en el Cloud Storage. Esta fase sería la de limpieza datos de forma que una vez estén subidos a HADOOP ya tengamos estos ficheros en los formatos adecuados y perfectamente preparados para el procesamiento.


### Hadoop

En Google Cloud tendré un cluster de Hadoop con tres contenedores. Dentro de Hadoop se realizarán el/los proceso/s que se encargue/n de la estimación temporal de precios (de momento será un simple Wordcount)


### HIVE

Voy a montar también un HIVE en la nube de Google. La idea es que los datos csv que se han obtenido se acaben cargando en tablas de HIVE de forma que el/los proceso/s que se encargue/n de calcuar los precios pueda/n realizar consultas SQL a HIVE. Por ejemplo que me ocurre que si el piso del que quiero saber los precios estimados está en el barrio SOL se pueda realizar una consulta SQL para ver cuales son las actividades de ocio programadas en ese barrio en los dos meses siguientes o si hay parking municipales en ese barrio, etc...

La carga de los datos en las tablas de HIVE de los ficheros csv se puede hacer desde el Cloud Storage. La carga de las noticias locales se hará de forma diaria a las 0:30 y el resto de datos los domingos a las 01:00


### JAR

El proceso JAR (que no sé de momento si será uno o varios) se encargará de realizar todo el procesamiento y las consultas necesarias a HIVE para ser capaz de hacer la estimación de precios a dos meses vista. Tomará como entrada los datos que se aporten del apartamento del usuario y el resultado se representaría de forma gráfica con alguna herramienta de visualización.
De momento en esta práctica este proceso va a realizar solo un wordcount de los ficheros y almacenará el resultado en una carpeta output del Cloud Storage


## Parte 2

*Crear un scraper en Google Colaboratory a partir de un API o de un crawler con scrapy, que descargue los datos a un archivo de formato estructurado*

### Colaboratory 1. Dataset de Airbnb

El Dataset de Airbnb lo tenemos en esta dirección [Airbnb](https://public.opendatasoft.com/explore/dataset/airbnb-listings/export/?disjunctive.host_verifications&disjunctive.amenities&disjunctive.features&q=Madrid&dataChart=eyJxdWVyaWVzIjpbeyJjaGFydHMiOlt7InR5cGUiOiJjb2x1bW4iLCJmdW5jIjoiQ09VTlQiLCJ5QXhpcyI6Imhvc3RfbGlzdGluZ3NfY291bnQiLCJzY2llbnRpZmljRGlzcGxheSI6dHJ1ZSwiY29sb3IiOiJyYW5nZS1jdXN0b20ifV0sInhBeGlzIjoiY2l0eSIsIm1heHBvaW50cyI6IiIsInRpbWVzY2FsZSI6IiIsInNvcnQiOiIiLCJzZXJpZXNCcmVha2Rvd24iOiJyb29tX3R5cGUiLCJjb25maWciOnsiZGF0YXNldCI6ImFpcmJuYi1saXN0aW5ncyIsIm9wdGlvbnMiOnsiZGlzanVuY3RpdmUuaG9zdF92ZXJpZmljYXRpb25zIjp0cnVlLCJkaXNqdW5jdGl2ZS5hbWVuaXRpZXMiOnRydWUsImRpc2p1bmN0aXZlLmZlYXR1cmVzIjp0cnVlfX19XSwidGltZXNjYWxlIjoiIiwiZGlzcGxheUxlZ2VuZCI6dHJ1ZSwiYWxpZ25Nb250aCI6dHJ1ZX0%3D&location=16,41.38377,2.15774&basemap=jawg.streets)

Podemos escoger entre bajarnos una versión resumida de 14.780 registros o el dataset completo. Para esta práctica voy a utilizar la versión reducida. Podemos bajarnos el archivo directamente desde este enlace y luego subirlo al Cloud Storage. De cara a una automatización en el futuro el código python para bajarse este archivo se muestra en este Colaboratory: 

- [Ver Código](https://github.com/mcpade/practica_arquitectura/blob/master/code/Airbnb%20Download.ipynb)



### Colaboratory 2. Crawling Noticas Locales Madrid

Mediante el siguiente código en Python voy a realizar un Crawling de una página de noticias locales de Madrid. Como resultado tendré un archivo "datanoticias.csv" que contendrá los siguientes datos de las noticias: fecha, distrito, titular.
En principio para la prática me quedaré solo con las 32 primeras noticias

- [Ver Código](https://colab.research.google.com/drive/1EoVTmGQbznHsgctedHsgZDgowXDXx1Px)

### Colaboratory 3. API de la EMT de Madrid

Mediante el siguiente código en Python hago consultas a la API de la EMT de Madrid para conocer datos de aparcamientos municipales. Como resultado tendré un archivo "Parkings.csv" que contendrá los siguientes datos de cada aparcamiento: id, nombre, código postal, coordenadas geográficas y dirección

- [Ver Codigo](https://colab.research.google.com/drive/1tlLzm6tJhjdMsp1QP98ArnjjmKL67ey8)

### Collaboratory 4. API Datos Abiertos de Madrid

Mediante el siguiente código en Python hago consultas a la API de Datos Abiertos de Madrid para conocer datos sobre:
- **Actividades Culturales y de Ocio Municipal en los próximos 100 días**. Como resultado obtengo un archivo "actividades.csv" que contendrá los siguientes datos de las actividades: id, Fecha de inicio, Fecha de fin, Título de la actividad, Descripción de la actividad, Distrito, Latitud, Longitud
- **Monumentos de la ciudad de Madrid**. Como resultado obtengo un archivo "monumentos.csv" que contendrá los siguientes datos de los monumentos: id, Titulo, Barrio, Distrito, Codigo Postal, Direccion, Latitud, Lontitud, Descripcion
- **Museos de la ciudad de Madrid** Como resultado obtengo un archivo "museos.csv" qque contendrá los siguientes datos de los monumentos: id, Titulo, Barrio, Distrito, Codigo Postal, Direccion, Latitud, Lontitud, Descripcion
- [Ver Codigo](https://colab.research.google.com/drive/15GBd3Z4aQHxVjWo4Fs11xcaEZkFyK_dm)

## Parte 3

*Utilizar un proveedor de Cloud para montar un cluster de al menos 3 contenedores configurados correctamente*

Para montar el cluster de Hadoop en Cloud uso GCP (Google Cloud Computing) Dataproc. Describo a continuación los pasos seguidos para montar un cluster de HADOOP en GCP Dataproc con un nodo master y tres slaves

Tengo ya una cuenta de google y me he dado de alta en https://cloud.google.com/ 

#### Paso 1

Entramos en esta dirección https://console.cloud.google.com/getting-started y hacemos click en Compute Engine:

![Paso 1](https://raw.githubusercontent.com/mcpade/practica_arquitectura/master/Paso_1_GCP.png)

#### Paso 2

Lo siguiente será crear un proyecto nuevo haciendo click en Nuevo Proyecto

![Paso 2](https://raw.githubusercontent.com/mcpade/practica_arquitectura/master/Paso_2_GCP.png)

Le doy un nombre al Proyecto: "MiProyectoEjemplo" y hago click en Crear

![Paso 2b](https://raw.githubusercontent.com/mcpade/practica_arquitectura/master/Paso_2b_GCP.png)

#### Paso 3

En el menú de GCP buscamos la opción DataProc y entramos en Clústeres

![Paso 3](https://raw.githubusercontent.com/mcpade/practica_arquitectura/master/Paso_3_GCP.png)

#### Paso 4

Lo siguiente será la creación del Cluster, para ello hacemos click en Crear Cluster

![Paso 4](https://raw.githubusercontent.com/mcpade/practica_arquitectura/master/Paso_4_GCP.png)

A continuación relleno los datos del cluster que quiero crear:

Región: europe-west1  (para que esté físicamente lo más cercano posible)

Modo del clúster: Estándar

Nodo Maestro: Le pongo 4 CPU con 15 GB

Nodos de trabajo: Les pongo 2 CPU con 8 GB   (de momento no se va a nececitar mucho procesamiento)

Nodos (mínimo 2): 3 (estos son los nodos slaves)

![Paso 4b](https://raw.githubusercontent.com/mcpade/practica_arquitectura/master/Paso_4b_GCP.png)


Tras esto ya podré ver mi cluster creado:

![Paso 4c](https://raw.githubusercontent.com/mcpade/practica_arquitectura/master/Paso_4c_GCP.png)

#### Paso 5

A continuación voy a crear las reglas de cortafuegos o firewall. GCP funciona a través de los puertos 8088 y 8970 (voy a abrir ya de camino también el 10000 para HIVE que usaré posteriormente)

En el menú de GCP buscamos la opción Red de VPC y entramos en Reglas de cortafuegos

![Paso 5](https://raw.githubusercontent.com/mcpade/practica_arquitectura/master/Paso_5_GCP.png)

Damos a Crear Regla de Cortafuegos

![Paso 5b](https://raw.githubusercontent.com/mcpade/practica_arquitectura/master/Paso_5b_GCP.png)

Y rellenamos los datos correspondientes a esta regla:

Nombre: abrir-hadoop-casa

Descripción: Cortafuegos para entrar en yarn, hdfs y hive

Destinos: Todas las instancias de la red

Filtro de origen: Intervalos de IPs

Intervalos de IPs de origen: Mi IP Pública local/32

Protocolos y puertos:  tcp:8088, 9870, 10000

![Paso 5c](https://raw.githubusercontent.com/mcpade/practica_arquitectura/master/Paso_5c_GCP.png)


#### Paso 6

Voy a comprobar que puedo acceder al cluster a través de la IP del master

Para encontrar la IP del master tenemos que entrar dentro del cluster y hacer click en Instancias de VM

![Paso 6](https://raw.githubusercontent.com/mcpade/practica_arquitectura/master/Paso_6_GCP.png)

Aquí podemos ver la máquina que funciona como master y los tres slaves. Debemos entrar dentro del master

![Paso 6b](https://raw.githubusercontent.com/mcpade/practica_arquitectura/master/Paso_6b_GCP.png)

En la sección de Interfaces de red encontramos la IP externa del master

![Paso 6c](https://raw.githubusercontent.com/mcpade/practica_arquitectura/master/Paso_6c_GCP.png)


Podemos acceder a través de:
http://IP_externa_master:8088

![Paso 6d](https://raw.githubusercontent.com/mcpade/practica_arquitectura/master/Paso_6d_GCP.png)

http://IP_externa_master:9870

![Paso 6e](https://raw.githubusercontent.com/mcpade/practica_arquitectura/master/Paso_6e_GCP.png)



## Parte 4

*Subir los archivos extraídos durante la parte 2 al cluster de Hadoop e insertarlos en el HDFS. Indicar pasos necesarios para realizar esto, dependiendo de la opción elegida en el Sprint 3*

Al utilizar arquitectura Cloud para Hadoop la fase de staging se reduce a la subida de datos a un segmento de Cloud Storage. Al crear el cluster en GCP automáticamente se crea un Storage Segment y los ficheros subidos a él estarán disponibles para ejecutar jar o cargar datos en hdfs. En esta práctica dentro de los propios Colaboratory  que se usan para extraer los datos se termina subiendo al Cloud Storage de Google usando la API de Python (se pueden consultar en la Parte 2)

Muestro a continuación una imagen donde se ve como quedan subidos los diferentes archivos csv en mi segmento del Cloud Storage. 

![Segmento](https://raw.githubusercontent.com/mcpade/practica_arquitectura/master/Segmento_Cloud_Storage.png)

*Realizar la tarea de procesamiento de datos sobre los datos extraídos utilizando WordCount*

En este punto es donde habría que procesar los datos para obtener el dato que queremos que es el de la estimación de precios. Dado que aun no hemos pasado por el módulo de procesamiento voy a realizar un proceso sencillo como es el WordCount de los ficheros subidos al Cloud Storage. Los resultados los guardo en una carpeta ouput que creo dentro del segmento. Muestro a continuación los pasos y el resultado final

#### Paso 1

En el menú de GCP buscamos la opción Dataproc y entramos en Tareas

![Paso_1_Wordcount](https://raw.githubusercontent.com/mcpade/practica_arquitectura/master/Paso_1_Wordcount.png)

#### Paso 2

Paso a crear la tarea, para ello damos a ENVIAR TAREA

![Paso_2_Wordcount](https://raw.githubusercontent.com/mcpade/practica_arquitectura/master/Paso_2_Wordcount.png)

Y rellenamos los datos de la Tarea pulsando al final en Enviar

ID de tarea: WordCount_Airbnb

Region: europe-west1 (la región donde tengamos nuestro cluster)

Clúster: escogemos el clúster creado

Tipo de tarea: Hadoop

Clase principal o jar: file:////usr/lib/hadoop-mapreduce/hadoop-mapreduce-examples.jar 

Argumentos:  

  wordcount 

  gs://dataproc-fe7b85fd-43dc-4de0-a520-a824fcd432de-europe-west1/airbnb.csv   (fichero que está en el Google                                                                                               Storage)
              
  gs://dataproc-fe7b85fd-43dc-4de0-a520-a824fcd432de-europe-west1/output/airbnb_result    (resultado)
              
![Paso_2b_Wordcount](https://raw.githubusercontent.com/mcpade/practica_arquitectura/master/Paso_2b_Wordcount.png)              
![Paso_2c_Wordcount](https://raw.githubusercontent.com/mcpade/practica_arquitectura/master/Paso_2c_Wordcount.png)   

#### Paso 3

Podemos ver que el resultado obtenido se ha guardado en la carpeta airbnb_result dentro de la carpeta output en el segmento del Cloud Storage

![Paso_3_Wordcount](https://raw.githubusercontent.com/mcpade/practica_arquitectura/master/Paso_3_Wordcount.png)

Podemos bajarnos el resultado para ver el contenido


![Paso_3b_Wordcount](https://raw.githubusercontent.com/mcpade/practica_arquitectura/master/Paso_3b_Wordcount.png)


El mismo procesamiento lo puedo realizar para el resto de ficheros .csv

![Paso_3c_Wordcount](https://raw.githubusercontent.com/mcpade/practica_arquitectura/master/Paso_3c_Wordcount.png)

![Paso_3d_Wordcount](https://raw.githubusercontent.com/mcpade/practica_arquitectura/master/Paso_3d_Wordcount.png)





## Parte 5

*Utilizar HIVE para insertar los datos extraídos durante el Sprint 2 y realizar queries con los mismos. Indicar los pasos y las decisiones de diseño respecto a cómo organizar los datos.*

GCP Dataproc ya viene con HIVE instalado, configurado y en ejecución. Solo hay que añadir el puerto sobre el que va HIVE (que es el 10000) a la regla de cortafuegos. Esto ya lo hice en la Parte 3 de la práctica, Paso 5. Cuando abrí los puertos para HADOOP también añadí el 10000 para HIVE

La idea para el diseño sería generar en HIVE unas tablas a las que volcar el contenido de los .csv extraidos (que ahora tenemos en el Cloud Storage), para posteriormente poder realizar queries sobre ellos. Antes de realizar esa carga los ficheros tendrían que estar limpios. Este es un paso que no se ha realizado y que se comentó al principio de la práctica que se tendría que realizar antes de proceder a subirlos al Cloud Storage. Supongo para la práctica que tengo los ficheros ya limpios (es algo que se verá en el módulo de procesamiento).

#### Paso 1

Lo primero que voy a hacer es entrar por SSH desde GCP en la máquina con HIVE y lanzar el cliente beeline. Para ello entro en el clúster y me voy a Instancias de VM. Junto a la máquina Master sale un enlace SSH que nos abre una consola con la que tenemos acceso a esa máquina


![Paso_1_HIVE](https://raw.githubusercontent.com/mcpade/practica_arquitectura/master/Paso_1_HIVE.png)

![Paso_1b_HIVE](https://raw.githubusercontent.com/mcpade/practica_arquitectura/master/Paso_1b_HIVE.png)

Para lanzar el cliente beeline ejecuto:

`beeline -u jdbc:hive2://localhost:10000`

![Paso_1c_HIVE](https://raw.githubusercontent.com/mcpade/practica_arquitectura/master/Paso_1c_HIVE.png)

#### Paso 2

Una vez dentro del cliente voy a crear las tablas con los campos necesarios para cada uno de los .csv que tengo como datos de entrada.

actividades.csv

`CREATE TABLE actividades (id INT, Fecha_inic TIMESTAMP, Fecha_fin TIMESTAMP, Titulo STRING, Descripcion STRING, Distrito STRING, Latitud FLOAT, Longitud FLOAT) ROW FORMAT DELIMITED FIELDS TERMINATED BY ';';`

monumentos.csv

`CREATE TABLE monumentos (id INT, Titulo STRING, Barrio STRING, Distrito STRING, CP INT, Direccion STRING, Latitud FLOAT, Lontitud FLOAT, Descripcion STRING) ROW FORMAT DELIMITED FIELDS TERMINATED BY ';';`

museos.csv

`CREATE TABLE museos (id INT, Titulo STRING, Barrio STRING, Distrito STRING, CP INT, Direccion STRING, Latitud FLOAT, Lontitud FLOAT, Descripcion STRING) ROW FORMAT DELIMITED FIELDS TERMINATED BY ';';`

parking.csv

`CREATE TABLE parking (id INT, Nombre STRING, CP INT, Coordenadas STRING, Direccion STRING) ROW FORMAT DELIMITED FIELDS TERMINATED BY ';';`

datanoticias.csv

`CREATE TABLE datanoticias (Fecha STRING, Distrito STRING, Titular STRING) ROW FORMAT DELIMITED FIELDS TERMINATED BY ';';`

airbnb.csv

`CREATE TABLE airbnb (ID INT, Listing_Url STRING, Scrape_ID STRING, Last_Scraped STRING, Name STRING, Summary STRING, Space STRING, Description STRING, Experiences_Offered STRING, Neighborhood_Overview STRING, Notes STRING, Transit STRING, Access STRING, Interaction STRING, House_Rules STRING, Thumbnail_Url STRING, Medium_Url STRING, Picture_Url STRING, XL_Picture_Url STRING, Host_ID STRING, Host_URL STRING, Host_Name STRING, Host_Since STRING, Host_Location STRING, Host_About STRING, Host_Response_Time STRING, Host_Response_Rate STRING, Host_Acceptance_Rate STRING, Host_Thumbnail_Url STRING, Host_Picture_Url STRING, Host_Neighbourhood STRING, Host_Listings_Count STRING, Host_Total_Listings_Count STRING, Host_Verifications STRING, Street STRING, Neighbourhood STRING, Neighbourhood_Cleansed STRING, Neighbourhood_Group_Cleansed STRING, City STRING, State STRING, Zipcode STRING, Market STRING, Smart_Location STRING, Country_Code STRING, Country STRING, Latitude STRING, Longitude STRING, Property_Type STRING, Room_Type STRING, Accommodates STRING, Bathrooms STRING, Bedrooms STRING, Beds STRING, Bed_Type STRING, Amenities STRING, Square_Feet STRING, Price STRING, Weekly_Price STRING, Monthly_Price STRING, Security_Deposit STRING, Cleaning_Fee STRING, Guests_Included STRING, Extra_People STRING, Minimum_Nights STRING, Maximum_Nights STRING, Calendar_Updated STRING, Has_Availability STRING, Availability_30 STRING, Availability_60 STRING, Availability_90 STRING, Availability_365 STRING, Calendar_last_Scraped STRING, Number_of_Reviews STRING, First_Review STRING, Last_Review STRING, Review_Scores_Rating STRING, Review_Scores_Accuracy STRING, Review_Scores_Cleanliness STRING, Review_Scores_Checkin STRING, Review_Scores_Communication STRING, Review_Scores_Location STRING, Review_Scores_Value STRING, License STRING, Jurisdiction_Names STRING, Cancellation_Policy STRING, Calculated_host_listings_count STRING, Reviews_per_Month STRING, Geolocation STRING, Features STRING) ROW FORMAT DELIMITED FIELDS TERMINATED BY ';'; `


![Paso_2_HIVE](https://raw.githubusercontent.com/mcpade/practica_arquitectura/master/Paso_2_HIVE.png)

![Paso_2b_HIVE](https://raw.githubusercontent.com/mcpade/practica_arquitectura/master/Paso_2b_HIVE.png)

#### Paso 3

A continuación habría que cargar el contenido de los .csv en las tablas creadas en el paso anterior. Esta carga la voy a realizar en la práctica desde el cliente beeline pero en producción habría que tener un procedimiento mediante el cual se cargaran estos datos de forma automática y periódica. Por ejemplo, podría tener un equipo en el que podría instalar PyHIVE para poder conectarme a esta maquina HIVE del GCP. En ese equipo ya podría programar una tarea que se ejecutara de forma periódica y que usando usando PyHIVE se conectara al HIVE y ejecutara las queries de carga de datos

`LOAD DATA INPATH 'gs://dataproc-fe7b85fd-43dc-4de0-a520-a824fcd432de-europe-west1/actividades.csv' OVERWRITE INTO TABLE actividades;`

![Paso_3_HIVE](https://raw.githubusercontent.com/mcpade/practica_arquitectura/master/Paso_3_HIVE.png)

`LOAD DATA INPATH 'gs://dataproc-fe7b85fd-43dc-4de0-a520-a824fcd432de-europe-west1/monumentos.csv' OVERWRITE INTO TABLE monumentos;`

![Paso_3b_HIVE](https://raw.githubusercontent.com/mcpade/practica_arquitectura/master/Paso_3b_HIVE.png)

`LOAD DATA INPATH 'gs://dataproc-fe7b85fd-43dc-4de0-a520-a824fcd432de-europe-west1/museos.csv' OVERWRITE INTO TABLE museos;`

![Paso_3c_HIVE](https://raw.githubusercontent.com/mcpade/practica_arquitectura/master/Paso_3c_HIVE.png)

`LOAD DATA INPATH 'gs://dataproc-fe7b85fd-43dc-4de0-a520-a824fcd432de-europe-west1/parking.csv' OVERWRITE INTO TABLE parking;`

![Paso_3d_HIVE](https://raw.githubusercontent.com/mcpade/practica_arquitectura/master/Paso_3d_HIVE.png)

`LOAD DATA INPATH 'gs://dataproc-fe7b85fd-43dc-4de0-a520-a824fcd432de-europe-west1/datanoticias.csv' OVERWRITE INTO TABLE datanoticias;`

![Paso_3e_HIVE](https://raw.githubusercontent.com/mcpade/practica_arquitectura/master/Paso_3d_HIVE.png)

`LOAD DATA INPATH 'gs://dataproc-fe7b85fd-43dc-4de0-a520-a824fcd432de-europe-west1/airbnb.csv' OVERWRITE INTO TABLE airbnb;`

![Paso_3f_HIVE](https://raw.githubusercontent.com/mcpade/practica_arquitectura/master/Paso_3f_HIVE.png)

#### Paso 4

Una vez que ya tengo todos los datos introducidos se pueden realizar consultas. Por ejemplo supongamos que la vivienda para la que queremos calcular la estimación de precios está en el distrito de "Retiro" podemos lanzar consultas a las tablas para ver la actividades que hay previstas en ese distrito o las noticias recientes:

`SELECT d.titular, d.distrito FROM datanoticias d where d.distrito='RETIRO';`

`SELECT a.titulo  FROM actividades a where a.distrito='Retiro';`

![Paso_4_HIVE](https://raw.githubusercontent.com/mcpade/practica_arquitectura/master/Paso_4_HIVE.png)

















