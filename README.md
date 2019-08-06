# Big Data Architecture. Práctica

## Enunciado

*Diseñar, especificar y desplegar un datalake para el procesamiento de datos provenientes de fuentes de datos no estructurados extraídos mediante técnicas de scraping/crawling de sitios de dominio público.*

Con un datalake se pretende centralizar datos de diferentes fuentes sobre los que ya después poder realizar análisis. La fuente principal será el dataset de Airbnb pero voy a enriquecer estos datos con otros obtenidos a través de Crawling y Scraping.
Aunque aun no tenga claro aún cual será el proyecto final, de momento tomo como objetivo el análizar una serie de datos para ser capaz de determinar en tiempo real cual sería el mejor precio para un vivienda Airbnb en Madrid.


## Parte 1

*Utilizar una herramienta de diagramado como Google Draw o DIA para diseñar y especificar el flujo de datos y herramientas utilizadas*

El fichero xxxx de este repositorio contiene el diagrama solicitado. 
En este diagrama se representa:
- Como se obtienen los datos que vamos a cargar en la plataforma Hadoop 
- Como se cargan los datos en Hadoop
- Como se aplican las herramientas de análisis sobre los datos que residen en los nodos clúster de Hadoop

### Obtención de los datos

- Fichero csv Airbnb
- Fichero csv de noticias locales de Madrid. Crawling
- Fichero txt parking Madrid. Scrapy - Python
- Fichero txt Eventos Culturales. Python
- Etc

### Carga de datos en Hadoop
- Cron para la carga de datos en hadoop y paso al HDFS

### Análisis






