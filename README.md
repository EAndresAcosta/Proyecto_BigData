# Proyecto Integral Big Data
![Big Data](https://www.campusbigdata.com/media/k2/items/cache/c10c64c27e0606d1654b81b9bb482558_XL.jpg)

Durante esta practica la idea es emular un ambiente de trabajo, desde un área de innovación solicitan construir un MVP(Producto viable mínimo) de un ambiente de Big Data donde se deban cargar unos archivos CSV que anteriormente se utilizaban en un datawarehouse en MySQl, pero ahora en un entorno de Hadoop.

Desde la gerencia de Infraestructura no están muy convencidos de utilizar esta tecnología por lo que no se asigno presupuesto alguna para esta iniciativa, de forma tal que por el momento no es posible utilizar un Vendor(Azure, AWS, Google) para implementar dicho entorno, es por esto que todo el MVP se deberá implementar utilizando Docker de forma tal que se pueda hacer una demo al sector de infraestructura mostrando las ventajas de utilizar tecnologías de Big Data.

<h1 align="center">Entorno: Docker con Haddop, Spark y Hive</h1>

![Docker1](http://badgen.net/badge/Big/Data/blue?icon=docker) ![Badge en Desarollo](https://img.shields.io/badge/STATUS-EN%20DESAROLLO-green) ![GitHub Org's stars](https://img.shields.io/github/stars/camilafernanda?style=social) ![Commits](https://badgen.net/github/last-commit/micromatch/micromatch) ![Terminal](http://badgen.net/badge/icon/server-ubuntu/orange?icon=terminal&label)

## Índice

* [Proyecto Big Data](#Proyecto-Integral-BigData)

* [HDFS](#HDFS)

* [HIVE](#HIVE)

* [Formatos de Almacenamiento](#Formatos-de-Almacenamiento)

* [SQL](#SQL)

* [No-SQL](#No-SQL)

* [Autores](#Autores)

* [Expresiones de Gratitud](#Expresiones-de-Gratitud)

* [Créditos](#Créditos)


![Docker](https://www.edureka.co/blog/wp-content/uploads/2017/11/Docker-Container-C.png)

Se pesenta un entorno Docker con Hadoop (HDFS) y la implementación de:
* Spark
* Hive
* HBase
* MongoDB
* Neo4J
* Zeppelin
* Kafka

Es importante mencionar que el entorno completo consume muchos recursos de su equipo, motivo por el cuál, se propondrán ejercicios pero con ambientes reducidos, en función de las herramientas utilizadas.

**Primer paso muy importante**, debemos clonar el repositorio en la maquina virtual:

```
git clone https://github.com/soyHenry/DS-M4-Herramientas_Big_Data.git
```

![Output](/images/m1.png)

Ejecute `docker network inspect` en la red (por ejemplo, `docker-hadoop-spark-hive_default`) para encontrar la IP en la que se publican las interfaces de hadoop. Acceda a estas interfaces con las siguientes URL:

```
Namenode: http://<IP_Anfitrion>:9870/dfshealth.html#tab-overview
Datanode: http://<IP_Anfitrion>:9864/
Spark master: http://<IP_Anfitrion>:8080/
Spark worker: http://<IP_Anfitrion>:8081/	
HBase Master-Status: http://<IP_Anfitrion>:16010
HBase Zookeeper_Dump: http://<IP_Anfitrion>:16010/zk.jsp
HBase Region_Server: http://<IP_Anfitrion>:16030
Zeppelin: http://<IP_Anfitrion>:8888
Neo4j: http://<IP_Anfitrion>:7474
```

***NOTA***: Agregar usuario autorizado para no usar sudo constantemente:

```
sudo su root
```


## 1. **HDFS**

![Hadoop](https://www.railscarma.com/wp-content/uploads/2015/01/The-Tool-For-Processing-Big-Data-Hadoop.jpg)

*Primero* ingresamos a la carpeta del proyecto:

```
cd DS-M4-Herramientas_Big_Data
```
```
ls
```

![List](/images/m4.png)

Creamos los contenedores ejecutando docker-compose archivo v1:

```
docker-compose -f docker-compose-v1.yml up -d
```

![Run docker-compose](/images/m2.png)

Copiar los archivos ubicados en la carpeta Datasets, dentro del contenedor **namenode**

Ubicarse dentro del contenedor *namenode*, crear el directorio *Datasets* y copiamos los archivos en el directorio creado luego salimos del contenedor:

```
docker exec -it namenode bash
cd home
```

![enter the container ](/images/m3.png)

```
mkdir Datasets
```
![List](/images/m2.1.png)
```
exit
```
```
docker cp <path><archivo> namenode:/home/Datasets/<archivo>
```
![List](/images/m2.2.png)

Ingresamos al directorio _Datasets_ y listamos para ver si estan todas ls carpetas con los archivos de las tablas:

```
cd home
cd Datasets
ls
```

![Files](/images/m15.png)

Tambien podemos ejecutar el **script _Paso00.sh_** que contiene comandos para copiar los archivos al directorio creado anteriormente. Para ello debemos modificar los permisos para poder ejecutarlo:

```
chmod u + rwx <Paso00.sh>
```

```
./Paso00.sh
```

Ingresamos de nuevo al contenedor *namenode*:

```
docker exec -it namenode bash
```

Creamos el directorio llamado _/data_ en **HDFS**:
> *-p*: Es una opción que indica al comando que cree todos los directorios intermedios necesarios. Esto significa que si algún directorio en la ruta especificada no existe, el comando lo creará junto con el directorio final:
```
hdfs dfs -mkdir -p /data
```
```
hdfs dfs -mkdir /data
```

Copiar los archivos csv provistos a HDFS:
```
  hdfs dfs -put /home/Datasets/* /data
```

![copying files](/images/m5.png)

Podemos verificar si los archivos estan en *HDFS* ingresando en el navegador a la *interfaz de hadoop*, colocando la *IP de nuestra maquina virtual seguido de:9870*:

```
http://<IP de nuestra maquina virtual>:9870/explorer.html#/
```
![Interfaz Hadoop](/images/m8.png)
![Interfaz Hadoop](/images/m9.png)

Este proceso de creación de la carpeta data y copiado de los arhivos, debe poder ejecutarse desde un shell script _Paso01.sh_.

**Nota:** Busque dfs.blocksize y dfs.replication en _http://<IP_Anfitrion>:9870/conf_ para encontrar los valores de tamaño de bloque y factor de réplica respectivamente entre otras configuraciones del sistema Hadoop.

![blocksize](/images/m6.png)  ![replication](/images/m7.png)


## 2) **HIVE**

![Hive](https://d3f1iyfxxz8i1e.cloudfront.net/courses/course_image/d12539898106.png)

Creamos el entorno de _hive_ ejecuntando el script _docker-compose-v2.yml_:

```
docker-compose -f docker-compose-v2.yml up -d
```

![hive environment](/images/m10.png)

En caso de generar error al crear el entorno de _hive_ detenemos los contenedores del entorno de hadoop:

```
docker stop $(sudo docker ps -a -q)
```

Crear tablas en _Hive_, a partir de los csv ingestados en _HDFS_. Para poder realizar este proceso podemos ejecutar el script _Paso02.hql_.
*importante*: Para poder ejecutar el script debemos tener este archivo dentro de **HIVE**. Asi que lo copiamos en el contenedor, tambien debemos tener en cuenta que debemos estar dentro del directorio del proyecto *DS-M4-Herramientas_Big_Data*:

```
docker cp ./Paso02.hql hive-server:/opt/
```

Ingresamos al servidor de Hive:

```
docker exec -it hive-server bash
```

Ejecutamos el script que copiamos anteriormente:

```
hive -f Paso02.hql
```
![creating tables](/images/m3.1.png)
> Tamien puede ejecutarlo de esta manera ingresando a hive `hive` y escribiendo el siguiente comando:
![entering the hive environment](/images/m12.png)

```
source <path><archivo>
```
> Ejemplo: source /opt/Paso02.hql;

![creating tables](/images/m11.png)

Ingresamos al Datas

*Nota*: Para ejecutar un script de Hive, requiere el comando:
```
  hive -f <script.hql>
```

 ### Para verificar si se crearon las tablas con exito lo hacemos realizando consultas en _hive_.

Ingresamos a *Hive*:

```
hive
```

Consultas:

```
show databases;
```
![consultation](/images/m13.1.png)
```
use integrador;
```
```
SELECT * FROM venta LIMIT 5;
```
![consultation](/images/m13.png)

***Tambien podemos verificar ingresando a HUE desde nuestro navegador con la IP de la pc:8888***


## 3) Formatos de Almacenamiento

En este punto vamos a crear otra base de datos con el Datasets del punto anterior, en formato ***parquet con snappy***.

Debemos copiar el siguiente archivo dentro del contenedor de _hive_:

```
docker cp ./Paso03.hql hive-server:/opt/
```

Ingresamos al servidor de Hive:

```
docker exec -it hive-server bash
```

Ejecutamos:

```
hive -f Paso03.hql
```
Tambien como en el segundo punto ingresando a *hive*:

```
source /opt/Paso03.hql;
```

![creating tables](/images/m14.png)

### Verificamos por medio de consultas si se creo la base da datos con sus tablas

Ingresamos a *Hive*:

```
hive
```

Consultas:

```
show databases;
```
![consultation](/images/m3.2.png)
```
use integrador2;
```
```
SELECT * FROM venta LIMIT 5;
```
```
SELECT * FROM tipo_gasto;
```
![consultation](/images/m3.3.png)


## 4) SQL

![SQL](https://static.tildacdn.one/tild6262-6661-4034-b164-383063636462/What_is_SQL_Database.png)

La mejora en la velocidad de consulta que puede proporcionar un índice tiene el costo del procesamiento adicional para crear el índice y el espacio en disco para almacenar las referencias del índice.
Se recomienda que los índices se basen en las columnas que utiliza en las condiciones de filtrado. El índice en la tabla puede degradar su rendimiento en caso de que no los esté utilizando.
Crear índices en alguna de las tablas cargadas y probar los resultados:

```
CREATE INDEX index_name
 ON TABLE base_table_name (col_name, ...)
 AS index_type
 [WITH DEFERRED REBUILD]
 [IDXPROPERTIES (property_name=property_value, ...)]
 [IN TABLE index_table_name]
 [ [ ROW FORMAT ...] STORED AS ...
 | STORED BY ... ]
 [LOCATION hdfs_path]
 [TBLPROPERTIES (...)]
 [COMMENT "index comment"];
```

Ejemplo:

```
hive> CREATE INDEX index_students ON TABLE students(id) 
 > AS 'org.apache.hadoop.hive.ql.index.compact.CompactIndexHandler' 
 > WITH DEFERRED REBUILD ;
```

ALTER INDEX index_name ON table_name [PARTITION partition_spec] REBUILD;

Ejemplo:
```
hive> ALTER INDEX index_students ON students REBUILD; 
```

DROP INDEX [IF EXISTS] index_name ON table_name;
```
hive> DROP INDEX IF EXISTS index_students ON students; 
```

Vamos a ejecutar consultas en la base de datos creada en el paso 2 o 3:

```
SELECT idproducto, sum(precio) FROM venta GROUP BY idproducto;
```
![consultation](/images/m16.png)
```
SELECT v.IdCliente, SUM(v.Precio * v.Cantidad) FROM venta v JOIN cliente c USING (IdCliente) WHERE c.Localidad = 'CIUDAD DE BUENOS AIRES' GROUP BY v.IdCliente;
```
![consultation](/images/m16.1.png)
```
SELECT IdProducto, sum(Precio) AS monto FROM venta GROUP BY IdProducto ORDER BY monto DESC LIMIT 5;
```
![consultation](/images/m16.2.png)
```
SELECT year(Fecha_Entrega), count(*) AS total_vendidos FROM venta GROUP BY year(Fecha_Entrega) ORDER BY total_vendidos DESC LIMIT 3;
```
![consultation](/images/m16.3.png)
```
SELECT IdEmpleado, SUM(Precio * Cantidad) FROM venta GROUP BY IdEmpleado;
```
![consultation](/images/m16.4.png)

Creamos la indexacion de las tablas que usamos en el paso anterior, esto para mejorar el rendimiento de las consultas:

```
CREATE INDEX index_venta_IdProducto ON TABLE venta(IdProducto) AS 'org.apache.hadoop.hive.ql.index.compact.CompactIndexHandler' WITH DEFERRED REBUILD;
```
  *Esta es la forma completa de crear un index pero no me funciono lo dejo solo como para ver la estructura 👇* 
> ```
> CREATE INDEX index_venta_idProducto ON TABLE venta(IdProducto) AS 'org.apache.hadoop.hive.ql.index.compact.CompactIndexHandler' WITH DEFERRED REBUILD IDXPROPERTIES ('index.created.by'='Andres AG', 'index.created.on'='2024-04-20') IN TABLE venta TBLPROPERTIES ('index.type'='compact', 'index.version'='1.0') COMMENT "Indice de idProducto para mejorar el rendimiento de las consultas";
>```

```
CREATE INDEX index_venta_idCliente ON TABLE venta(idCliente) AS 'org.apache.hadoop.hive.ql.index.compact.CompactIndexHandler' WITH DEFERRED REBUILD;
```
  *👇*
>```
>CREATE INDEX index_venta_idCliente ON TABLE venta(idCliente) AS 'org.apache.hadoop.hive.ql.index.compact.CompactIndexHandler' WITH DEFERRED REBUILD IDXPROPERTIES ('index.created.by'='Andres AG', 'index.created.on'='2024-04-20') IN TABLE venta TBLPROPERTIES ('index.type'='compact', 'index.version'='1.0') COMMENT "Indice de idCliente para mejorar el rendimiento de las consultas";
>```

```
CREATE INDEX index_venta_IdEmpleado ON TABLE venta(IdEmpleado) AS 'org.apache.hadoop.hive.ql.index.compact.CompactIndexHandler' WITH DEFERRED REBUILD;
```
  *👇*
>```
>CREATE INDEX index_venta_IdEmpleado ON TABLE venta(IdEmpleado) AS 'org.apache.hadoop.hive.ql.index.compact.CompactIndexHandler' WITH DEFERRED REBUILD IDXPROPERTIES ('index.created.by'='Andres AG', 'index.created.on'='2024-04-20') IN TABLE venta TBLPROPERTIES ('index.type'='compact', 'index.version'='1.0') COMMENT "Indice de IdEmpleado para mejorar el rendimiento de las consultas";
>```
```
CREATE INDEX index_venta_Fecha_Entrega ON TABLE venta(Fecha_Entrega) AS 'org.apache.hadoop.hive.ql.index.compact.CompactIndexHandler' WITH DEFERRED REBUILD;
```
  *👇*
>```
>CREATE INDEX index_venta_Fecha_Entrega ON TABLE venta(Fecha_Entrega) AS 'org.apache.hadoop.hive.ql.index.compact.CompactIndexHandler' WITH DEFERRED REBUILD IDXPROPERTIES ('index.created.by'='Andres AG', 'index.created.on'='2024-04-20') IN TABLE venta TBLPROPERTIES ('index.type'='compact', 'index.version'='1.0') COMMENT "Indice de Fecha_Entrega para mejorar el rendimiento de las consultas";
>```

Volvemos a ejecutar las query de antes y verificamos si el tiempo de consulta disminuye:

```
SELECT idproducto, sum(precio) FROM venta GROUP BY idproducto;
```
![consultation](/images/m17.png)
```
SELECT v.IdCliente, SUM(v.Precio * v.Cantidad) FROM venta v JOIN cliente c USING (IdCliente) WHERE c.Localidad = 'CIUDAD DE BUENOS AIRES' GROUP BY v.IdCliente;
```
![consultation](/images/m17.1.png)
```
SELECT IdProducto, sum(Precio) AS monto FROM venta GROUP BY IdProducto ORDER BY monto DESC LIMIT 5;
```
![consultation](/images/m17.2.png)
```
SELECT year(Fecha_Entrega), count(*) AS total_vendidos FROM venta GROUP BY year(Fecha_Entrega) ORDER BY total_vendidos DESC LIMIT 3;
```
![consultation](/images/m17.3.png)
```
SELECT IdEmpleado, SUM(Precio * Cantidad) FROM venta GROUP BY IdEmpleado;
```
![consultation](/images/m17.4.png)


## 5) No-SQL

![NOSQL](https://geekflare.com/cdn-cgi/image/width=800,height=257,fit=crop,quality=90,gravity=auto,sharpen=1,metadata=none,format=auto,onerror=redirect/wp-content/uploads/2020/06/NoSQL.png)

Se puede utilizar el entorno docker-compose-v3.yml

#### 1) HBase:

![HBase](https://media.licdn.com/dms/image/C5612AQHra2pTFgtFOQ/article-cover_image-shrink_600_2000/0/1564190745001?e=2147483647&v=beta&t=4ILz9WNuMe1QhmUvya2z8Nv5M99AWumNzIPWmVmTddI)

<a id='Instrucciones'>Instrucciones:</a>

```
	1- sudo docker exec -it hbase-master hbase shell

		create 'personal','personal_data'
		list 'personal'
		put 'personal',1,'personal_data:name','Juan'
		put 'personal',1,'personal_data:city','Córdoba'
		put 'personal',1,'personal_data:age','25'
		put 'personal',2,'personal_data:name','Franco'
		put 'personal',2,'personal_data:city','Lima'
		put 'personal',2,'personal_data:age','32'
		put 'personal',3,'personal_data:name','Ivan'
		put 'personal',3,'personal_data:age','34'
		put 'personal',4,'personal_data:name','Eliecer'
		put 'personal',4,'personal_data:city','Caracas'
		get 'personal','4'

	2-En el namenode del cluster:

		hdfs dfs -put personal.csv /hbase/data/personal.csv

	3-sudo docker exec -it hbase-master bash
		
    hbase org.apache.hadoop.hbase.mapreduce.ImportTsv -Dimporttsv.separator=',' -Dimporttsv.columns=HBASE_ROW_KEY,personal_data:name,personal_data:city,personal_data:age personal hdfs://namenode:9000/hbase/data/personal.csv
		hbase shell
		scan 'personal'
		create 'album','label','image'
		put 'album','label1','label:size','10'
		put 'album','label1','label:color','255:255:255'
		put 'album','label1','label:text','Family album'
		put 'album','label1','image:name','holiday'
		put 'album','label1','image:source','/tmp/pic1.jpg'
		get 'album','label1'
```		

Ejecutamos el ***Script docker-compose-v3.yml***
![hive environment](/images/m18.png)

Seguimos las instruccones: 
[siga las instrucciones](#instrucciones)



#### 2) MongoDB

![MongoDB](https://s3.amazonaws.com/info-mongodb-com/_com_assets/cms/kuzt9r42or1fxvlq2-Meta_Generic.png)

Instrucciones:
```
	1) 	sudo docker cp iris.csv mongodb:/data/iris.csv
		  sudo docker cp iris.json mongodb:/data/iris.json

	2)  sudo docker exec -it mongodb bash

	3) 	mongoimport /data/iris.csv --type csv --headerline -d dataprueba -c iris_csv
		  mongoimport --db dataprueba --collection iris_json --file /data/iris.json --jsonArray

	4) mongosh
		use dataprueba
		show collections
		db.iris_csv.find()
		db.iris_json.find()
	
	5) 	mongoexport --db dataprueba --collection iris_csv --fields sepal_length,sepal_width,petal_length,petal_width,species --type=csv --out /data/iris_export.csv
		mongoexport --db dataprueba --collection iris_json --fields sepal_length,sepal_width,petal_length,petal_width,species --type=json --out /data/iris_export.json
				
	6) 	Descargar desde https://search.maven.org/search?q=g:org.mongodb.mongo-hadoop los jar:
		https://search.maven.org/search?q=a:mongo-hadoop-hive
		https://search.maven.org/search?q=a:mongo-hadoop-spark
		
		sudo docker cp mongo-hadoop-hive-2.0.2.jar hive-server:/opt/hive/lib/mongo-hadoop-hive-2.0.2.jar
		sudo docker cp mongo-hadoop-core-2.0.2.jar hive-server:/opt/hive/lib/mongo-hadoop-core-2.0.2.jar
		sudo docker cp mongo-hadoop-spark-2.0.2.jar hive-server:/opt/hive/lib/mongo-hadoop-spark-2.0.2.jar
		sudo docker cp mongo-java-driver-3.12.11.jar hive-server:/opt/hive/lib/mongo-java-driver-3.12.11.jar
		
	7) 	sudo docker cp iris.hql hive-server:/opt/iris.hql
		sudo docker exec -it hive-server bash

	8) 	hiveserver2
		chmod 777 iris.hql
		hive -f iris.hql
```

## Autores ✒️

- Henry - *Trabajo Inicial* - [Soy Henry](https://github.com/soyHenry)
- EAndresAcosta - *Documentación* - [EAndresAcosta](https://github.com/EAndresAcosta)

## Expresiones de Gratitud 🎁

* Invitacion a una 🍺 o un ☕ a [Jesus Parra](https://github.com/ing-jhparra) - [Santos Iparraguirre](https://github.com/SantosIparraguirre) - [Facundo Corvalan](https://github.com/facu-corvalan) 
* Quiero expresar mi sincero agradecimiento por la excelente labor y enseñanza que han proporcionado. Su compromiso con la calidad educativa ha hecho una diferencia significativa en mi experiencia de aprendizaje. 🤓.
* etc.

## Créditos

Copyright (c) 2024 [EAndresAcosta](https://github.com/EAndresAcosta)
