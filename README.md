# Proyecto Integral Big Data
![Big Data](https://www.campusbigdata.com/media/k2/items/cache/c10c64c27e0606d1654b81b9bb482558_XL.jpg)

Durante esta practica la idea es emular un ambiente de trabajo, desde un área de innovación solicitan construir un MVP(Producto viable mínimo) de un ambiente de Big Data donde se deban cargar unos archivos CSV que anteriormente se utilizaban en un datawarehouse en MySQl, pero ahora en un entorno de Hadoop.

Desde la gerencia de Infraestructura no están muy convencidos de utilizar esta tecnología por lo que no se asigno presupuesto alguna para esta iniciativa, de forma tal que por el momento no es posible utilizar un Vendor(Azure, AWS, Google) para implementar dicho entorno, es por esto que todo el MVP se deberá implementar utilizando Docker de forma tal que se pueda hacer una demo al sector de infraestructura mostrando las ventajas de utilizar tecnologías de Big Data.

# Entorno: Docker con Haddop, Spark y Hive

![Docker](https://www.edureka.co/blog/wp-content/uploads/2017/11/Docker-Container-C.png)  ![Spark](https://www.zdnet.com/a/img/resize/d26dbe5b8b669db1ff5e77a09668c92b558e2757/2017/11/01/d17912b7-7717-4a71-9b09-573460970f94/sparkecosystem.png?auto=webp&width=1280) 


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


## 1) **HDFS**

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
```
exit
```
```
docker cp <path><archivo> namenode:/home/Datasets/<archivo>
```

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

![Hive](/images/hive.jpg)

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
use integrador;
```
```
SELECT * FROM venta LIMIT 5;
```
![consultation](/images/m13.png)

***Tambien podemos verificar ingresando a HUE desde nuestro navegador con la IP:8888***

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
hive -f Paso02.hql
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
use integrador2;
```
```
SELECT * FROM venta LIMIT 5;
```
```
SELECT * FROM tipo_gasto;
```
