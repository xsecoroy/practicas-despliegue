# APUNTES DOCKER

## CONCEPTOS

### IMAGÉN

Una imagen es un archivo estático que contiene todo lo necesario para ejecutar una aplicación: 
el sistema operativo base, las dependencias, las configuraciones y el código de la aplicación. <br><br>
Es como una plantilla, como una receta de cocina. Contiene todos los pasos e ingredientes necesarios, 
pero no es la comida en sí.<br><br>
Una imagen es el punto de partida. Para que algo funcione, necesito "instanciar" esa imagen, lo que crea un contenedor.<br><br>
Puedo crear varios contenedores a partir de una misma imagen. Por ejemplo, si tiengo una imagen de MySQL, 
puedo crear múltiples contenedores que ejecuten bases de datos diferentes basadas en esa imagen.



### CONTENEDOR

 Un contenedor es una instancia en ejecución de una imagen. Es el entorno "vivo" que se crea a partir
 de la imagen y que ejecuta la aplicación o servicio.<br><br>
 Un contenedor es el plato cocinado y servido. Es el resultado de ejecutar la receta (imagen).<br><br>
 Es mutable mientras está en ejecución. Puedo interactuar con él, modificar archivos, ejecutar comandos, etc<br><br>

Características principales de un contenedor:
- **Aislamiento**: Cada contenedor opera de forma independiente, como si fuera un pequeño sistema operativo. 
- **Portabilidad**: Los contenedores pueden ejecutarse en cualquier máquina que tenga un motor de contenedores (como Docker), sin importar el sistema operativo subyacente.
- **Eficiencia**:Al compartir el núcleo del sistema operativo, los contenedores consumen menos recursos que las máquinas virtuales.
- **Reproducibilidad**: Al incluir todas las dependencias necesarias, los contenedores garantizan que una aplicación funcione igual en desarrollo, pruebas y producción.<br><br>

**Contra**: Datos efímeros: Los datos almacenados dentro del sistema de archivos del contenedor se eliminan cuando el contenedor se elimina. 
Los volúmenes resuelven este problema al proporcionar un almacenamiento persistente. <br><br>

### VOLUMENES

Los volúmenes en Docker son una forma de almacenar datos persistentes que pueden ser utilizados por los contenedores.<br><br>
A diferencia de los datos que se almacenan dentro del sistema de archivos del contenedor (que desaparecen cuando el contenedor se elimina),
los volúmenes **permiten que los datos persistan incluso después de que el contenedor se detenga o elimine**.<br><br>

Características principales de los volúmenes:
- **Persistencia de datos**: Los volúmenes permiten que los datos sobrevivan a la eliminación de un contenedor. 
- **Almacenamiento externo al contenedor**: Los volúmenes existen fuera del sistema de archivos del contenedor y están gestionados por Docker. 
En sistemas Linux, los volúmenes suelen almacenarse en la ruta `/var/lib/docker/volumes`.
- **Compartición de datos**: Los volúmenes pueden ser compartidos entre múltiples contenedores. Esto permite que 
varios contenedores accedan y utilicen los mismos datos.
- **Gestión por Docker**: Docker se encarga de gestionar los volúmenes, lo que incluye su creación, eliminación y acceso. 
Esto simplifica la administración de datos persistentes.<br><br>

Tipos de volúmenes en Docker<br<br>
- **Volúmenes gestionados por Docker**: Son creados y gestionados automáticamente por Docker. Solo Docker tiene acceso directo a ellos,
 y se almacenan en rutas específicas del sistema operativo del host.<br><br>
- **Bind mounts (montajes vinculados)**: Permiten montar una carpeta específica del sistema de archivos del host dentro del contenedor. 
A diferencia de los volúmenes gestionados por Docker, los bind mounts dependen directamente de la estructura del sistema de archivos del host.<br><br>


### DOCKERFILE

Un Dockerfile no es una imagen, sino un archivo de texto que contiene las instrucciones necesarias para construir una imagen Docker. 
Es como una "receta" que describe cómo se debe crear la imagen, incluyendo qué base usar, qué configuraciones aplicar, 
qué archivos copiar, qué comandos ejecutar, etc.<br><br>
Las instrucciones del dockerfile son ejecutadas en el orden en que aparecen en el archivo.<br><br>
Una vez que tengo configurado el Dockerfile, puedo usar el comando `docker build -t nombre-imagen` para convertirlo en una imagen Docker.<br><br>

En los Dockerfile como el de MySQL, las variables de entorno definidas con ENV asignan valores predefinidos por defecto. 
Si no especifico valores personalizados al crear el contenedor, se usarán esos valores predefinidos.<br><br>
Si en el dockerfile se encuentran estas lineas:<br><br>
- **ENV MYSQL_ROOT_PASSWORD=root_password**<br><br>
- **ENV MYSQL_DATABASE=default_database**<br><br>
- **ENV MYSQL_USER=default_user**<br><br>
- **ENV MYSQL_PASSWORD=default_password**<br><br>
Si no sobrescribo estas variables al ejecutar el contenedor, estos valores serán los que se utilicen dentro del contenedor.<br><br>

Cuando ejecutas un contenedor con el comando **docker run** y uso la opción **-e** para establecer variables de entorno, los valores que 
especifique sobrescribirán los valores definidos en el Dockerfile.Por ejemplo:<br><br>
docker run -d \
  --name mi-mysql \
  -e MYSQL_ROOT_PASSWORD=mi_contraseña \
  -e MYSQL_DATABASE=mi_base_datos \
  -e MYSQL_USER=mi_usuario \
  -e MYSQL_PASSWORD=mi_password \
  mi-mysql <br><br>

Luego accedería así: `docker exec -it mi-mysql sh` <br><br>


## PROCESO

Toda la configuración con la que lo cree el contenedor (la imagen que use, las variables de entorno, los volúmenes vinculados, etc.) se mantiene intacta.<br><br> 
No necesitas volver a ejecutar todo el comando original para usar el contenedor de nuevo. Simplemente al reiniciar el contenedor y todo estará como lo deje.<br><br>

Cuando creo un contenedor con Docker, este guarda toda la configuración asociada al contenedor (como la imagen, los volúmenes montados, las variables de entorno, etc.) en su metadata. 
Si detengo el contenedor, este queda en un estado "detenido", pero toda su configuración y datos persisten mientras no lo elimine.<br><br>

Los datos en el volumen son persistentes y están desacoplados del ciclo de vida del contenedor. Esto significa que:<br><br>
- Si detengo el contenedor, los datos permanecen.<br><br>
- Si eliminas el contenedor, los datos también permanecen mientras no elimine el volumen explícitamente.<br><br>

Si ejecuto esto:<br><br>
docker run -d \
  --name mi-mysql \
  -e MYSQL_ROOT_PASSWORD=mi_contraseña \
  -v mi-volumen:/var/lib/mysql \
  mysql <br><br>

**docker run**: Este es el comando para crear y ejecutar un contenedor.<br><br>
**-d**: Ejecuta el contenedor en segundo plano (modo "detached").<br><br>
**--name mi-mysql**: Asigna el nombre mi-mysql al contenedor.<br><br>
**-e MYSQL_ROOT_PASSWORD=mi_contraseña**: Define una variable de entorno para establecer la contraseña del usuario root de MySQL.<br><br>
**-v mi-volumen:/var/lib/mysql**: Monta un volumen llamado mi-volumen en la ruta /var/lib/mysql dentro del contenedor.<br><br>
**mysql**: Esta es la imagen que se usará para crear el contenedor.<br><br>

Docker crea un volumen **llamado mi-volumen** en su sistema de almacenamiento interno, que generalmente está ubicado en una carpeta específica
del sistema operativo del host, en Linux quedaría así: ***/var/lib/docker/volumes/mi-volumen/_data*** <br><br>
Como el volumen **mi-volumen** está montado en ****/var/lib/mysql*** del contenedor, cualquier cambio que haga dentro de esa ruta en el contenedor
(como crear el directorio baseDatosPrueba) se guarda en el volumen.<br><br>
Si creo un directorio llamado ***/var/lib/mysql/baseDatosPrueba*** (por ejemplo, usando mkdir /var/lib/mysql/baseDatosPrueba), si hago cambios en el 
volumen desde el host (por ejemplo, añado un archivo dentro de ***/var/lib/docker/volumes/mi-volumen/_data/baseDatosPrueba***), esos cambios se 
reflejarán automáticamente dentro del contenedor en la ruta ***/var/lib/mysql/baseDatosPrueba*** de la misma manera, si hago cambios dentro 
del contenedor en ***/var/lib/mysql/baseDatosPrueba***, esos cambios se reflejarán en el volumen en el host.<br><br>

Por lo tanto, para volver a usar el contenedor, solo necesito reiniciarlo con:<br><br>
`docker start mi-mysql` <br><br>

## COPY VS VOLUMENES

La instrucción **COPY** en el dockerfile es unidireccional y estática, la instrucción copia archivos o directorios desde el sistema de archivos local (host) 
al sistema de archivos del contenedor durante la construcción de la imagen.
Esto significa que los archivos se transfieren una sola vez al crear la imagen, y no hay sincronización posterior entre el host y el contenedor.
Los cambios realizados en los archivos del contenedor no se reflejan en el host, y viceversa.
Ejemplo:<br><br>
`COPY html/ /var/www/html/` <br><br>
Esto copia el contenido de la carpeta ****html/** en mi máquina local al directorio **/var/www/html/** dentro de la imagen. 
Cuando cree un contenedor basado en esta imagen, esos archivos estarán disponibles en el contenedor.

En cambio lso volumenes son bidireccionales y dinámicos, permitiendo compartir datos entre el host y el contenedor de forma persistente y sincronizada.
Los cambios realizados en los archivos del volumen dentro del contenedor se reflejan en el host, y viceversa.
Ejemplo:<br><br>
`docker run -v /ruta/local:/var/www/html nginx`<br><br>
Esto monta la carpeta **/ruta/local** de mi máquina host en el directorio **/var/www/html** del contenedor. Cualquier cambio en **/ruta/local**
se reflejará en el contenedor, y viceversa.<br><br>


## REDES
Cada contenedor tiene su propia dirección IP dentro de la red de Docker, y pueden comunicarse entre sí si están en la misma red.<br><br>
Si tengo un contenedor MySQL y uno de Apache+PHP y están conectados a la misma red llamada **mi-red** significa que el contenedor de 
Apache+PHP puede comunicarse con el contenedor de MySQL utilizando el nombre del contenedor de MySQL (**mi-db** en este caso) como si fuera un hostname.<br><br>

**Docker proporciona un sistema de resolución de nombres que permite que los contenedores se encuentren entre sí usando sus nombres.**<br><br>

***¿Puede el contenedor de Apache+PHP usar la base de datos del contenedor de MySQL?***
¡Sí, puede! Al estar en la misma red, el contenedor de Apache+PHP puede conectarse al contenedor de MySQL utilizando el nombre del contenedor (**mi-db**) 
como dirección del servidor de base de datos. Esto es posible gracias a la red de Docker, que permite la comunicación entre contenedores conectados a la misma red.<br><br>

***Mostrar los datos de las BBDD del contenedor mysql por web en el contenedor Apache+PHP***
En el contenedor de Apache, el servidor web buscará y ejecutará el archivo **index.php** que se encuentre en el directorio **/var/www/html/**, ya que este es el 
directorio raíz predeterminado para Apache en la mayoría de las configuraciones. Por lo tanto, si coloco el archivo PHP en esa ubicación, se ejecutará automáticamente 
cuando acceda al contenedor a través del navegador.<br><br>
En el archivo **index.php** esribiré el código PHP que realiza la conexión al contenedor de MySQL y ejecutará una consulta para mostrar datos. <br><br>

Como laUbicación del archivo index.php debe estar en el directorio **/var/www/html/** dentro del contenedor de Apache y en mi configuración, este directorio 
está vinculado a la carpeta **./app** en mi máquina local gracias al volumen que defini al ejecutar el contenedor:<br><br>

docker run -d \
  -p 8000:80 \
  -v ./app:/var/www/html \
  --network mi-red \
  mi-servidor-apache <br><br> 

Esto significa que cualquier archivo que coloque en la carpeta **./app** en mi máquina local estará disponible en **/var/www/html/** dentro del contenedor.<br><br>
Por lo tanto si guardo el archivo PHP como **./app/index.php** en mi máquina local, Apache lo ejecutará automáticamente cuando accedas a http://localhost:8000.<br><br>

## DOCKER COMPOSE

Docker Compose es una herramienta que facilita la definición y ejecución de aplicaciones que requieren múltiples contenedores en Docker. 
En lugar de ejecutar manualmente varios comandos `docker run` para iniciar y configurar cada contenedor, Docker Compose permite definir todos los servicios, 
configuraciones y relaciones entre contenedores en un único archivo llamado **docker-compose.yml** (o compose.yaml).<br><br>

En el archivo de configuración (docker-compose.yml) defino todos los servicios (contenedores) que forman parte de mi aplicación, puedo especificar:<br><br>
- Las imágenes de Docker que usarán los contenedores.<br><br>
- Los puertos que se expondrán.<br><br>
- Los volúmenes para persistencia de datos.<br><br>
- Las redes para la comunicación entre contenedores.<br><br>
- Variables de entorno y otras configuraciones necesarias.<br><br>

Posee Multi-contenedores, está diseñado para aplicaciones que requieren múltiples contenedores trabajando juntos. Por ejemplo:<br><br>
- Un contenedor para un servidor web (como Apache o Nginx).<br><br>
- Otro contenedor para una base de datos (como MySQL o PostgreSQL).<br><br>
- Otros servicios adicionales, como Redis o RabbitMQ.<br><br>

Automatización:<br><br>
- Con un solo comando (docker-compose up), puedo iniciar todos los contenedores definidos en el archivo docker-compose.yml, configurarlos y conectarlos automáticamente.<br><br>

Portabilidad:<br><br>
- El archivo docker-compose.yml actúa como una receta que puedo compartir con otros desarrolladores o usar en diferentes entornos (desarrollo, pruebas, producción). 
Esto asegura que todos los servicios se ejecuten de manera consistente.<br><br>


## COMANDOS

- `docker run ubuntu`: Crea y ejecuta un nuevo contenedor basado en la imagen ubuntu. Si la imagen ubuntu no está disponible localmente, Docker la descargará del repositorio de Docker Hub.<br><br>
Puedo pasar parámetros como -d (modo detached), -p (mapeo de puertos), o -it (modo interactivo).<br><br>
`docker run -d --name contenedor1 nginx`<br><br>

- `docker start <ID_del_contenedor>`: Inicia un contenedor que ya existe pero está detenido.No crea un nuevo contenedor ni descarga imágenes.
Solo funciona con contenedores que ya han sido creados previamente (por ejemplo, con docker run o docker create).<br><br>

Ejemplo: `docker run -d -p 8080:80 nginx` esto crea un nuevo contenedor (por ejemplo, llamado contenedor1) y configura el mapeo de puertos entre el host y el contenedor donde el puerto 8080 del 
host está vinculado al puerto 80 del contenedor.<br><br>
Si hago `docker stop contenedor1` y luego lo reinicio con `docker start contenedor1` el contenedor conservará su configuración original, incluyendo el mapeo de puertos (8080:80 en este caso).
No puedo cambiar los puertos al reiniciar un contenedor existente con docker start. Esto se debe a que docker start simplemente reinicia el contenedor con la configuración que tenía cuando fue creado.<br><br>

**La configuración que se define cuando se crea el contenedor (es decir, con docker run) una vez que el contenedor ha sido creado, esta configuración no se puede modificar directamente.** <br><br>

- `docker exec`: Ejecuta un comando dentro de un contenedor que ya está en ejecución. No inicia ni crea contenedores, solo interactúa con uno que ya está corriendo.<br><br>

- `docker exec -it`: Es una variante de docker exec que abre una sesión interactiva dentro del contenedor en ejecución, permite interactuar directamente con el contenedor como si estuviera dentro de él. El parámetro -it combina: <br><br>
  - **i**: Modo interactivo (mantiene la entrada estándar abierta).
  - **t**: Asigna una pseudo-terminal (útil para trabajar con shells como bash).<br><br>
`docker exec -it contenedor1 bash`<br><br>
