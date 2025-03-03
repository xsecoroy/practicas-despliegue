🔹 Explicación de cada servicio
1️⃣ Proxy inverso (Nginx)
proxy:
  hostname: proxy
  image: jwilder/nginx-proxy
  container_name: proxydaw
  ports:
    - "80:80"
  volumes:
    - /var/run/docker.sock:/tmp/docker.sock:ro
    - ./nginx-proxy:/etc/nginx/vhost.d:ro
    - "./conf/my_custom_proxy_settings.conf:/etc/nginx/conf.d/my_custom_proxy_settings.conf:ro"
  networks:
    - frontend

- Usa la imagen jwilder/nginx-proxy para actuar como proxy inverso.
- Expone el puerto 80 para manejar las solicitudes HTTP.
- Mapea volúmenes para gestionar configuraciones de Nginx:
Escucha el socket de Docker (docker.sock) para detectar contenedores con variables VIRTUAL_HOST.
Usa configuraciones personalizadas en ./nginx-proxy y ./conf/my_custom_proxy_settings.conf.
- Se conecta a la red frontend.

2️⃣ Base de datos MySQL

db:
  hostname: db
  image: mysql:5.7
  container_name: dbdaw
  volumes:
    - ./data/db:/var/lib/mysql
    - ./init-db:/docker-entrypoint-initdb.d
  environment:
    - MYSQL_ROOT_PASSWORD=password
    - MYSQL_DATABASE=demo
    - MYSQL_USER=dbuser
    - MYSQL_PASSWORD=secret
  networks:
    - backend

- Usa la imagen mysql:5.7.
- Guarda los datos en ./data/db para persistencia.
- Ejecuta scripts de inicialización en ./init-db al arrancar.
- Variables de entorno:
MYSQL_ROOT_PASSWORD=password: Contraseña del usuario root.
MYSQL_DATABASE=demo: Crea una base de datos llamada demo.
MYSQL_USER=dbuser, MYSQL_PASSWORD=secret: Crea un usuario con permisos en demo.
- Se conecta a la red backend


3️⃣ Aplicación web 1 (PHP+Apache)

web1:
  build: ./dockerfiles/web1
  container_name: web1
  hostname: web1
  volumes:
    - ./data/web1:/var/www/html
  depends_on:
    - db
  environment:
    - VIRTUAL_HOST=web1.com,www.web1.com
  restart: always
  networks:
    - frontend
    - backend
Se construye desde el Dockerfile en ./dockerfiles/web1:
dockerfile
Copiar
Editar
FROM php:7.2-apache
RUN docker-php-ext-install mysqli
Usa PHP 7.2 con Apache.
Instala la extensión mysqli para conectarse a MySQL.
Usa ./data/web1 como raíz del servidor web.
Depende de la base de datos (depends_on: - db).
Configura el VIRTUAL_HOST para que nginx-proxy lo reconozca.
Se conecta tanto a la red frontend como backend.
📌 Contenido de web1:

index.php: Conexión a MySQL y consulta la tabla country.
factorial.php: Calcula el factorial de un número pasado por GET.


4️⃣ Aplicación web 2 (PHP+Apache)
yaml
Copiar
Editar
web2:
  image: php:7.3-apache
  container_name: web2
  hostname: web2.com
  volumes:
    - ./data/web2:/var/www/html
  environment:
    - VIRTUAL_HOST=web2.com,www.web2.com
  networks:
    - frontend
    - backend
Usa la imagen php:7.3-apache directamente (sin Dockerfile).
Usa ./data/web2 como raíz del servidor web.
Configura el VIRTUAL_HOST para que nginx-proxy lo reconozca.
📌 Contenido de web2:

index.php: Muestra información de PHP con phpinfo();.
5️⃣ phpMyAdmin
yaml
Copiar
Editar
phpmyadmin:
  image: phpmyadmin/phpmyadmin
  links:
    - db:db
  environment:
    - MYSQL_USERNAME=root
    - MYSQL_ROOT_PASSWORD=password
    - PMA_HOST=db
    - VIRTUAL_HOST=phpmyadmin.docker
  networks:
    - frontend
    - backend
Usa la imagen oficial de phpMyAdmin para gestionar la base de datos.
Se conecta a db.
Configura el VIRTUAL_HOST en phpmyadmin.docker.
🔹 Redes
yaml
Copiar
Editar
networks:
  frontend:
  backend:
Frontend: Para proxy, web1, web2 y phpMyAdmin.
Backend: Para db, web1, web2 y phpMyAdmin.

**¿Cómo logro mostrar los datos del contenedor bd en el contenedor web1?**
Pese a que db y web1 son contenedores diferentes, están en la misma red de Docker (backend), lo que permite que web1 se conecte a la base de datos MySQL del contenedor db.

En index.php del contenedor web1 configuro al variable **servername** así `$servername = "db"`<br><br>
Docker usa nombres de contenedor como hostnames, así que en index.php de web1, la conexión debe usar db como nombre de host, porque así se llama el servicio en docker-compose.yaml<br><br>
db:
  hostname: db <br><br>


**¿Cómo logro logró que nginx-proxy detecte web1 y le asigna un dominio local y muestre su contenido en el navegador?**
El contenedor proxy usa la imagen **jwilder/nginx-proxy**, proxy inverso el cual escucha en el puerto 80 y detecta automáticamente los contenedores que tienen la variable de entorno **VIRTUAL_HOST** redirigiendo el tráfico al contenedor correspondiente según el dominio solicitado.<br><br>
En el contenedor **web1** en **environment** establezco la variable de entorno VIRTUAL_HOST así:<br><br> `VIRTUAL_HOST=web1.com,www.web1.com`<br><br>
nginx-proxy escanea los contenedores en ejecución, detecta esta variable y actualiza su configuración automáticamente para redirigir web1.com a web1.<br><br>

Cuando nginx-proxy se inicia, monitorea el socket de Docker (/var/run/docker.sock), lo que le permite ver en tiempo real qué contenedores se están ejecutando y qué variables de entorno tienen. 
Además cada vez que un contenedor nuevo se levanta o muere, nginx-proxy se entera. Genera automáticamente un archivo de configuración Nginx con los dominios de los contenedores.

Para mostrarlos primero deben estar los contenedores en la misma red, en este caso ambos estan en la red frontend. En ellos busca contenedores con la variable VIRTUAL_HOST y 
genera un archivo de configuración de Nginx en base a esas variables y después se recarga Nginx automáticamente para aplicar la nueva configuración.
Es como si, al levantar web1, nginx-proxy creara algo como esto internamente:

server {
    listen 80;
    server_name web1.com www.web1.com;

    location / {
        proxy_pass http://web1:80;  # Redirige a Apache en el contenedor web1
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
🔹 ¿Por qué proxy_pass http://web1:80;?
Porque en Docker, web1 es accesible como http://web1 dentro de la red.
Solo necesito por último modificar /etc/hosts en mi máquina local para acceder a web1.com desde el navegador `127.0.0.1 web1.com www.web1.com`.

**¿Cómo evito conflictos de puertos entre apache y nginx?**
- web1 usa la imagen php:7.2-apache, es decir, Apache corriendo en el contenedor web1.
- proxy usa nginx-proxy, proxy inverso para enrutar las solicitudes a web1.
- Ambos contenedores están en la misma red (frontend), por tanto se pueden comunicar entre sí.

Cada contenedor de Docker tiene su propio "mundo de red", por lo que pueden usar el mismo puerto sin conflictos.Ejemplo:<br><br>

- proxy (Nginx) usa el puerto 80 dentro de su contenedor.
- web1 (Apache) usa el puerto 80 dentro de su contenedor.

Pero no chocan porque cada uno está aislado en su propio contenedor.
Sin embargo, para que puedan comunicarse, deben estar en la misma red de Docker.

**Lógica de todo el proceso**

Cliente (Navegador):
✅ Paso 1: El usuario escribe http://web1.com en el navegador.
✅ Paso 2: El navegador hace una solicitud HTTP al puerto 80 de la máquina local.
✅ Paso 3: El contenedor nginx-proxy está configurado para escuchar en el puerto 80 de la máquina local (gracias a ports: - "80:80" en docker-compose.yml).
✅ Paso 4: Nginx recibe la solicitud en su puerto 80 dentro del contenedor, es decir mapea el puerto 80 de la máquina al puerto 80 del contenedor nginx-proxy
✅ Paso 5: Nginx redrijire al contenedor de web1 al puerto 80 que es el que esta escuchando apache.
- Explicacion paso 5: En mi docker-compose.yml, en el servicio web1, tiengo esta parte:
environment:
  - VIRTUAL_HOST=web1.com,www.web1.com
**Aquí está la clave:**
✅ Nginx-proxy escanea automáticamente los contenedores en la red y busca los que tienen la variable de entorno **VIRTUAL_HOST** configurada.<br><br>
✅ Cuando detecta que el contenedor ***web1*** tiene **VIRTUAL_HOST=web1.com,www.web1.com**, nginx-proxy ya sabe que web1.com debe redirigir al contenedor web1.<br><br>
✅ Pero todavía no sabe a qué puerto de ese contenedor...
✅ En la Dockerfile que use para construir el contenedor web1 uso esta imagen **FROM php:7.2-apache**, en este caso Docker por defecto expone el puerto 80 en imágenes como php:7.2-apache.<br><br>
📌 Esta imagen ya tiene Apache instalado y configurado para escuchar en el puerto 80 dentro del contenedor, por tanto el conteendor web1 expone el puerto 80 con apache como servidor.<br><br>
✅ Cuando nginx-proxy encuentra un contenedor con VIRTUAL_HOST, busca el puerto expuesto automáticamente por Docker. Si el contenedor no tiene una variable de entorno VIRTUAL_PORT, nginx-proxy asume que el contenedor está escuchando en el puerto 80.<br><br>
✅ Crea automáticamente un "server block" en su configuración interna de Nginx para que web1.com apunte a **web1:80**, ***contenedor:puerto***.<br><br>

**¿Cómo hago para que nginx-proxy use otro puerto (como el 8000), osea apache escuche el 8000?**

✅ 1. Cambiar el puerto en el servidor Apache dentro del contenedor editando el Dockerfile de web1 y agregando:

FROM php:7.2-apache
RUN docker-php-ext-install mysqli
RUN sed -i 's/Listen 80/Listen 8000/' /etc/apache2/ports.conf
RUN sed -i 's/:80/:8000/' /etc/apache2/sites-available/000-default.conf

📌 Esto hace que Apache dentro del contenedor escuche en el puerto 8000 en lugar del 80.

✅ 2. Decirle a nginx-proxy que use ese puerto. En el docker-compose.yml de web1, agrego la variable VIRTUAL_PORT:<br><br>

environment:
  - VIRTUAL_HOST=web1.com,www.web1.com
  - VIRTUAL_PORT=8000

ports:
  - "8080:8080"

📌 Con esto, nginx-proxy redirigirá web1.com a web1:8000 en lugar de web1:80.

**La razón por la que nginx-proxy redirige automáticamente a web1:80 en vez de otro puerto (como el 8000) es porque Apache dentro del contenedor web1 está configurado para escuchar en el puerto 80 por defecto.**

El contenido de web1:

Apache en web1 sirve la página web que tiene en su directorio /var/www/html (que está vinculado con la carpeta local ./data/web1).
Apache entonces procesa y muestra el contenido de la web solicitada (en este caso, los archivos PHP que has puesto en web1).
Resumen del flujo completo:
El navegador hace una solicitud HTTP a webdeprueba1.com en el puerto 80.
Nginx en el contenedor nginx-proxy recibe la solicitud y, gracias a la configuración de server_name webdeprueba1.com, redirige la solicitud al contenedor web1 (en su puerto 80).
Dentro de web1, Apache recibe la solicitud en su puerto 80 y le responde al navegador con el contenido que tiene en /var/www/html.
Este flujo funciona porque Docker conecta todos los contenedores en una red interna. Cuando Nginx en el contenedor nginx-proxy hace referencia a web1:80, está accediendo al contenedor web1 directamente a través de la red interna de Docker, lo que lo hace posible.

## FLUJO DEL CONTENEDOR phpMyAdmin

PhpMyAdmin corre sobre un servidor web Apache dentro de su contenedor y escucha en el puerto 80 (dentro del contenedor), esto es pro que la imagen oficial ***phpmyadmin/phpmyadmin*** ya incluye Apache como servidor web.<br><br>
Cuando el contenedor se inicia, Apache arranca automáticamente y sirve la interfaz web de PhpMyAdmin en el puerto 80.<br><br>
Sin embargo, como el contenedor no expone directamente ningún puerto a la máquina host, solo es accesible a través de nginx-proxy, que lo hace disponible en http://phpmyadmin.docker.<br><br>

- Flujo con los puertos
1️⃣ El usuario accede a http://phpmyadmin.docker (sin puerto explícito porque usa el 80 por defecto).
2️⃣ nginx-proxy recibe la petición en el puerto 80 de la máquina host y la redirige a phpmyadmin.
3️⃣ Dentro del contenedor de phpmyadmin, Apache está escuchando en el puerto 80 y responde con la interfaz web.
4️⃣ El usuario ve PhpMyAdmin en el navegador.<br><br>
5️⃣  PhpMyAdmin usa PMA_HOST=db para conectarse a MySQL en el contenedor db (puerto 3306).
 Finalmente el usuario ingresa usuario/contraseña y accede a la base de datos.


## CCOMO OBTENGO LOS DATOS DEL CONTENEDOR MYSQL DESDE  EL CONTENEDOR PhpMyAdmin

**MySQL (db)**:
- Es la base de datos donde PhpMyAdmin se conecta.
- Corre en otro contenedor.
- Expone el puerto 3306 (el puerto de MySQL).
- Logramos ver los datos del contenedor mysql gracias a la variable del contenedor phpmyadmin **PMA_HOST**
**¿Qué hace PMA_HOST=db?**
Cuando PhpMyAdmin arranca, necesita saber a qué servidor MySQL conectarse.
Cada contenedor en Docker tiene su propia IP interna dentro de la red de Docker, llamada aquí ***backend***, y siendo la IP el nombre del contenedor, ***db***.
Misma red + nombre accesible + puertos expuesto = posibildiad de conexión entre contenedores. Entonces, cuando PhpMyAdmin intenta conectarse a PMA_HOST=db, en realidad está hablando con MySQL en db:3306.<br><br>
✅ PhpMyAdmin (en Apache, puerto 80) → Se conecta a MySQL (puerto 3306) usando PMA_HOST=db.
