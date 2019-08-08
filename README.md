# Big Data Architecture. Práctica

## Enunciado

*Diseñar, especificar y desplegar un datalake para el procesamiento de datos provenientes de fuentes de datos no estructurados extraídos mediante técnicas de scraping/crawling de sitios de dominio público.*

Con un datalake se pretende centralizar datos de diferentes fuentes sobre los que ya después poder realizar análisis. La fuente principal será el dataset de Airbnb pero voy a enriquecer estos datos con otros obtenidos a través de Crawling y Scraping.
Aunque aun no tenga claro cual será el proyecto final, de momento tomo como objetivo el análizar una serie de datos para ser capaz de determinar en tiempo real cual sería el mejor precio para un vivienda Airbnb en Madrid. Para ello además de los datos de Airbnb puedo usar datos de noticias de los diferentes barrios, localización de parkings, eventos culturales previstos, localización de monumentos y museos...

## Parte 1

*Utilizar una herramienta de diagramado como Google Draw o DIA para diseñar y especificar el flujo de datos y herramientas utilizadas*

El fichero xxxx de este repositorio contiene el diagrama solicitado. 
En este diagrama se representa:
- Como se obtienen los datos que vamos a cargar en la plataforma Hadoop 
- Como se cargan los datos en Hadoop
- Como se aplican las herramientas de análisis sobre los datos que residen en los nodos clúster de Hadoop

(Ver estos puntos dependiendo de como quede el diagrama)

### Obtención de los datos

Tendré que programar un cron diario que me permita descargarme los datos de las diferentes fuentes:

- Fichero csv Airbnb. [Airbnb](https://public.opendatasoft.com/explore/dataset/airbnb-listings/export/?disjunctive.host_verifications&disjunctive.amenities&disjunctive.features&q=Madrid&dataChart=eyJxdWVyaWVzIjpbeyJjaGFydHMiOlt7InR5cGUiOiJjb2x1bW4iLCJmdW5jIjoiQ09VTlQiLCJ5QXhpcyI6Imhvc3RfbGlzdGluZ3NfY291bnQiLCJzY2llbnRpZmljRGlzcGxheSI6dHJ1ZSwiY29sb3IiOiJyYW5nZS1jdXN0b20ifV0sInhBeGlzIjoiY2l0eSIsIm1heHBvaW50cyI6IiIsInRpbWVzY2FsZSI6IiIsInNvcnQiOiIiLCJzZXJpZXNCcmVha2Rvd24iOiJyb29tX3R5cGUiLCJjb25maWciOnsiZGF0YXNldCI6ImFpcmJuYi1saXN0aW5ncyIsIm9wdGlvbnMiOnsiZGlzanVuY3RpdmUuaG9zdF92ZXJpZmljYXRpb25zIjp0cnVlLCJkaXNqdW5jdGl2ZS5hbWVuaXRpZXMiOnRydWUsImRpc2p1bmN0aXZlLmZlYXR1cmVzIjp0cnVlfX19XSwidGltZXNjYWxlIjoiIiwiZGlzcGxheUxlZ2VuZCI6dHJ1ZSwiYWxpZ25Nb250aCI6dHJ1ZX0%3D&location=16,41.38377,2.15774&basemap=jawg.streets)

- Fichero csv de noticias locales de Madrid. Lo obtendré usando Crawling en esta dirección. [Noticias Locales Madrid](https://www.eldistrito.es/distritos/)

- Fichero csv con datos de parkings de Madrid. Lo obtendré realizando cunsultas a esta API de la EMT de Madrid. [API EMT Madrid](https://apidocs.emtmadrid.es/)

- Fichero csv con Actividades Culturales y de Ocio Municipal en los próximos 100 dias. Lo obtendré realizando consultas a esta API de datos abiertos de Madrid. [API Datos abiertos Madrid](https://datos.madrid.es/portal/site/egob/menuitem.214413fe61bdd68a53318ba0a8a409a0/?vgnextoid=b07e0f7c5ff9e510VgnVCM1000008a4a900aRCRD&vgnextchannel=b07e0f7c5ff9e510VgnVCM1000008a4a900aRCRD&vgnextfmt=default)

- Fichero csv con Monumentos de la Ciudad de Madrid. Lo obtendré realiznado consultas a la misma API anterior. [API Datos abiertos Madrid](https://datos.madrid.es/portal/site/egob/menuitem.214413fe61bdd68a53318ba0a8a409a0/?vgnextoid=b07e0f7c5ff9e510VgnVCM1000008a4a900aRCRD&vgnextchannel=b07e0f7c5ff9e510VgnVCM1000008a4a900aRCRD&vgnextfmt=default)

- Fichero csv con los Museos de la Ciudad de Madrid. Lo obtendré realiznado consultas a la misma API anterior. [API Datos abiertos Madrid](https://datos.madrid.es/portal/site/egob/menuitem.214413fe61bdd68a53318ba0a8a409a0/?vgnextoid=b07e0f7c5ff9e510VgnVCM1000008a4a900aRCRD&vgnextchannel=b07e0f7c5ff9e510VgnVCM1000008a4a900aRCRD&vgnextfmt=default)


### Carga de datos en Hadoop
- Cron para la carga de datos en hadoop y paso al HDFS

(Ver video y describir un poco los pasos, dibujar el cluster de Hadoop)

### Análisis
(Pendiente de las últimas clases)

## Parte 2

*Crear un scraper en Google Colaboratory a partir de un API o de un crawler con scrapy, que descargue los datos a un archivo de formato estructurado*

### Dataset de Airbnb

El Dataset de Airbnb lo tenemos en esta dirección [Airbnb](https://public.opendatasoft.com/explore/dataset/airbnb-listings/export/?disjunctive.host_verifications&disjunctive.amenities&disjunctive.features&q=Madrid&dataChart=eyJxdWVyaWVzIjpbeyJjaGFydHMiOlt7InR5cGUiOiJjb2x1bW4iLCJmdW5jIjoiQ09VTlQiLCJ5QXhpcyI6Imhvc3RfbGlzdGluZ3NfY291bnQiLCJzY2llbnRpZmljRGlzcGxheSI6dHJ1ZSwiY29sb3IiOiJyYW5nZS1jdXN0b20ifV0sInhBeGlzIjoiY2l0eSIsIm1heHBvaW50cyI6IiIsInRpbWVzY2FsZSI6IiIsInNvcnQiOiIiLCJzZXJpZXNCcmVha2Rvd24iOiJyb29tX3R5cGUiLCJjb25maWciOnsiZGF0YXNldCI6ImFpcmJuYi1saXN0aW5ncyIsIm9wdGlvbnMiOnsiZGlzanVuY3RpdmUuaG9zdF92ZXJpZmljYXRpb25zIjp0cnVlLCJkaXNqdW5jdGl2ZS5hbWVuaXRpZXMiOnRydWUsImRpc2p1bmN0aXZlLmZlYXR1cmVzIjp0cnVlfX19XSwidGltZXNjYWxlIjoiIiwiZGlzcGxheUxlZ2VuZCI6dHJ1ZSwiYWxpZ25Nb250aCI6dHJ1ZX0%3D&location=16,41.38377,2.15774&basemap=jawg.streets)

Podemos escoger entre bajarnos una versión resumida de 14.780 registros o el dataset completo. Para este proyecto habría que tener el dataset completo y filtrar para la cuidad de Madrid.
El código python para bajarse este archivo podría ser el siguiente. Un cron diario tendría que lanzar este código en python:

```
import urllib.request

#Caso de que queramos tener la versión reducida
#link = "https://public.opendatasoft.com/explore/dataset/airbnb-listings/download/?format=csv&disjunctive.host_verifications=true&disjunctive.amenities=true&disjunctive.features=true&q=Madrid&timezone=Europe/Berlin&use_labels_for_header=true"

#Caso que queramos tener la versión completa
link = "https://public.opendatasoft.com/explore/dataset/airbnb-listings/download/?format=csv&timezone=Europe/Berlin&use_labels_for_header=true"

nfichero = "airbnb.csv"
frespuesta = urllib.request.urlopen(link)
fichero = open(nfichero,"wb")
fichero.write(frespuesta.read())
fichero.close()
```

### Colaboratory 2. Crawling Noticas Locales Madrid

Mediante el siguiente Scrapy en Python voy a realizar un Crawling de una página de noticias locales de Madrid. Como resultado tendré un archivo "datanoticias.csv" que contendrá los siguientes datos de las noticias: fecha, distrito, titular

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
(PENDIENTE)

## Parte 4

*Subir los archivos extraídos durante la parte 2 al cluster de Hadoop e insertarlos en el HDFS. Indicar pasos necesarios para realizar esto, dependiendo de la opción elegida en el Sprint 3*

(PENDIENTE)

*Realizar la tarea de procesamiento de datos sobre los datos extraídos utilizando WordCount*

(PENDIENTE)

## Parte 5

*Utilizar HIVE para insertar los datos extraídos durante el Sprint 2 y realizar queries con los mismos. Indicar los pasos y las decisiones de diseño respecto a cómo organizar los datos.*

(PENDIENTE)










