# APUNTES Servidores Web

## 1 - 

**/var/www/sitioweb** Es un estándar en la configuración de Apache, en ese directorio **/var/www/** se crea el directorio del sitio web que yo cree como 
sería **sitio1**  de forma que dentro de **/var/www/sitio1** tendré el contenido público de mi sitio web (archivos HTML, CSS, imágenes, etc.) y la configuración del servidor para ese sitio.<br><br>

**/etc/apache2/sites-available/sitioweb.conf** Este es el directorio donde almacenano los archivos de configuración de los Virtual Hosts de Apache. Cada uno de estos archivos define cómo Apache debe manejar las solicitudes para el dominio específico, por tanto al crear dentro de **/etc/apache2/sites-available/** el archivo **sitio1.conf** ahora tendre ese sitio web configurados pero no necesariamente activos. Este archivo aún no está activo porque está en sites-available. Aquí es donde Apache almacena todas las configuraciones de sitios que podrían habilitarse, pero que no necesariamente se están sirviendo aún.<br><br>
Lógica: En sites-available tengo todos los archivos de configuración, pero solo los que habilite con a2ensite aparecerán en sites-enabled y estarán activos.
Así con **a2ensite** activo los sitios y con **a2dissite** los deshabilito temporalmente un sitio sin borrarlo, simplemente elimino su enlace simbólico.
Después: `sudo systemctl reload apache2`<br><br>

## 2 - Crear un sitio web

Para mi web, www.sitio1.com.<br>

Primero - Creo el contenido del sitio en **/var/www/sitio1** :<br>

`sudo mkdir -p /var/www/sitio1`<br>
`echo "<h1>Bienvenido a mi sitio</h1>" | sudo tee /var/www/sitio1/index.html`<br>

Segundo - Creo un archivo de configuración para sitio1 en **/etc/apache2/sites-available/sitio1.conf**:<br>

`sudo nano /etc/apache2/sites-available/sitio1.conf`<br>

El contenido que debe tener es así:<br><br>
<VirtualHost *:80><br>
    ServerAdmin webmaster@localhost<br>
    ServerName sitio1.com<br>
    ServerAlias www.sitio1.com sitio1<br>
    DocumentRoot /var/www/tusitio<br>
    ErrorLog ${APACHE_LOG_DIR}/error.log<br>
    CustomLog ${APACHE_LOG_DIR}/access.log combined<br>
</VirtualHost><br>

Tercero - Debo habilitar el sitio web con los comandos:<br><
`sudo a2ensite sitio1.conf`<br>
`sudo systemctl reload apache2`<br>

**IMPORTANTE**
Como no tengo un dominio real debo modificar la configuración del archivo hosts( en el servidor ):<br>

En **/etc/hosts** añado:<br>
`127.0.0.1    sitio1.com www.sitio1.com`<br>
Ahora, al entrar en www.sitio1.com desde el navegador, apuntará al servidor local (localhost).<br><br>

## A2ensite
El comando a2ensite (abreviatura de "Apache2 Enable Site") crea un enlace simbólico del archivo de configuración del sitio, ubicado en /etc/apache2/sites-available/tusitio.conf, dentro del directorio /etc/apache2/sites-enabled/.
Esto le indica a Apache que debe cargar y usar la configuración definida en tusitio.conf para servir ese sitio.
Recarga de Apache:<br><br>

Después de habilitar el sitio, generalmente se ejecuta el comando sudo systemctl reload apache2 para recargar la configuración del servidor Apache. Esto asegura que la nueva configuración del Virtual Host tome efecto sin necesidad de reiniciar completamente el servicio.
Resultado:<br><br>

A partir de este punto, Apache servirá el sitio web configurado en el archivo tusitio.conf. El servidor reconocerá las solicitudes enviadas al dominio o subdominio especificado (por ejemplo, www.tusitio.com) y servirá los archivos correspondientes desde el directorio indicado en DocumentRoot.<br><br>

El comando a2ensite ("Apache2 Enable Site") habilita tu sitio web. Lo que hace internamente es:<br><br>

Crea un enlace simbólico en /etc/apache2/sites-enabled/ que apunta al archivo original en /etc/apache2/sites-available/<br><br>

## Reglas puertos Apache<br><br>
En el archivo `/etc/apache2/ports.conf` establezco el puerto que escuchará apache,
si establezco **Listen 8080** estoy diciendo a apache ***Abre y escucha conexiones en el puerto 8080***.<br><br>
Después en el archivo conf de cada sitio como `/etc/apache2/sites-available/000-default.conf` también se configura en `<VirtualHost *:PUERTO>` debe coincidir con uno de los puertos configurados en ports.conf usando Listen. Por tanto si establezco `<VirtualHost *:8080>` estoy diciendo que  ***Este sitio web responderá únicamente a las peticiones que lleguen al puerto 8080***.<br>

Si pongo 80 en VirtualHost y 8080 en ports.conf, El VirtualHost nunca será activado porque Apache no está escuchando en el puerto 80, solo en 8080, por tanto Apache no sabrá cómo manejar las peticiones al puerto 8080 para ese VirtualHost porque no hay concordancia.<br>

En caso de querer usar ambas podria ponerlas en **ports.conf** así:<br><br>
**Listen 80**
**Listen 8080**<br><br>
Luego en el archivo del sitio así:
<VirtualHost *:80><br>
    DocumentRoot /var/www/html<br>
    ServerName localhost<br>
</VirtualHost><br>

<VirtualHost *:8080><br>
    DocumentRoot /var/www/html<br>
    ServerName localhost<br>
</VirtualHost><br><br>


## Reglas puertos Nginx
En nginx no hay un archivo como en apache generico `/etc/apache2/ports.conf` el puerto de configura en cada archivo de sitio, se encuentra en **/etc/nginx/sites-available/sitio2** (y luego se habilita en sites-enabled).
En el ejemplo anterior de sitio 2 en server añadiria una linea con **LISTEN puerto**<br><br>
server {
    listen 8080;  # Este es el puerto en el que Nginx escuchará
Si quiero que el sitio escuche en varios puertos (como 80 y 8080), debo configurar varios bloques server { ... } para cada puerto.<br>
server {<br>
    listen 80;  # Escuchar en el puerto 80<br>
    server_name sitio2.local;<br>
    root /var/www/sitio2;<br>
    index index.html;<br>
}<br>

server {<br>
    listen 8080;  # Escuchar en el puerto 8080<br>
    server_name sitio2.local;<br>
    root /var/www/sitio2;<br>
    index index.html;<br>
}<br><br>

## Proxy inverso
Primero logica sin su uso(sin configuración en Nginx):<br><br>
En el caso de que apache lo puse con puerto 8080 y lso sitios de nginx con el 80.
- Acceder a sitio2:80: Esto irá directamente a Nginx. Nginx está configurado para servir el contenido de /var/www/sitio2nginx, 
así que se verá el contenido de ese directorio.
- Acceder a sitio2:8080: Esto irá directamente a Apache. Apache está configurado para servir el contenido de /var/www/sitio2, 
así que verás el contenido de ese directorio.<br><br>

Usando el proxy inverso (con Nginx configurado para pasar las solicitudes a Apache):<br><br>
- Acceder a sitio2:80: Aquí es donde entra en juego el proxy inverso. Aunque Nginx está configurado para servir desde /var/www/sitio2nginx, 
Nginx no mostrará ese contenido directamente. En lugar de eso, pasará las solicitudes de Nginx a Apache en el puerto 8080 
(por la configuración de proxy_pass que se estableció en Nginx).

- En este caso, lo que vere en la web será el contenido de /var/www/sitio2, que es lo que Apache está sirviendo,
 no el contenido de /var/www/sitio2nginx. Así, aunque acceda a sitio2.com:80, lo que se está mostrando es el contenido que está en Apache, 
 porque Nginx está actuando como intermediario, enviando las solicitudes a Apache en el puerto 8080.<br><br>

**Resumen de comportamiento con y sin proxy inverso**:

- Sin proxy inverso:

sitio2:80 -> Nginx -> Contenido de /var/www/sitio2nginx.
sitio2:8080 -> Apache -> Contenido de /var/www/sitio2.<br><br>

- Con proxy inverso:

sitio2:80 -> Nginx -> Apache en 8080 -> Contenido de /var/www/sitio2.<br><br>

**Con el proxy inverso, Nginx no sirve el contenido desde su propio directorio /var/www/sitio2nginx,**
**sino que simplemente reenvía las solicitudes a Apache, que sirve el contenido desde /var/www/sitio2 (el directorio de Apache)**.<br><br>

## Varios puertos en nginx 
Archivo sitio2 en /etc/nginx/sites-available:<br><br>

Este archivo se encarga de servir el contenido directamente desde un directorio como /var/www/sitio2nginx (por ejemplo).
Nginx escuchará en el puerto 80 y mostrará el contenido de ese directorio.
Archivo proxy en /etc/nginx/sites-available:

Este archivo está configurado para actuar como proxy inverso, redirigiendo las solicitudes que llegan a Nginx en el puerto 80 a Apache en el puerto 8080.
En este caso, Nginx no sirve contenido directamente, sino que pasa las solicitudes a Apache, que sirve el contenido de /var/www/sitio2.<br><br>

## Varios archivos y para un mismo sitios con mismo poerto en nginx, uno inverso y otro normal

- Archivo **sitio2** en **/etc/nginx/sites-available**:
Este archivo se encarga de servir el contenido directamente desde un directorio como /var/www/sitio2nginx (por ejemplo).
Nginx escuchará en el puerto 80 y mostrará el contenido de ese directorio.<br><br>

server {
    listen 80;
    server_name sitio2.com;

    root /var/www/sitio2nginx;
    index index.html;

    # Otros parámetros como logs, etc.
}<br><br>


- Archivo proxy en /etc/nginx/sites-available:
Este archivo está configurado para actuar como proxy inverso, redirigiendo las solicitudes que llegan a Nginx en el puerto 80 a Apache en el puerto 8080.
En este caso, Nginx no sirve contenido directamente, sino que pasa las solicitudes a Apache, que sirve el contenido de /var/www/sitio2.<br><br>

server {
    listen 80;
    server_name sitio2.com;

    location / {
        proxy_pass http://localhost:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}<br><br>

**HABRÁ CONFLICTO** pues si ambos archivos están habilitados en sites-enabled (usando ln -s), Nginx no podrá decidir que archivo usar, 
porque ambos están intentando escuchar en el mismo puerto (80) para el mismo dominio sitio2.com. Esto generará un conflicto, 
ya que no se puede tener dos servidores escuchando en el mismo puerto al mismo tiempo.
**No puedes tener dos configuraciones Nginx para el mismo puerto y dominio a la vez.**
Por tanto, solo una de las configuraciones debe estar habilitada en el puerto 80.<br><br>

Si solo quiero dejar el archivo de proxie inverso deshabilito el archivo sitio2 dentro de **/etc/nginx/sites-enabled/**<br><br>
`sudo unlink /etc/nginx/sites-enabled/sitio2`<br><br>
Esto eliminará el enlace simbólico, deshabilitando el archivo sin borrar el archivo original en sites-available
Además compruebo si el archivo **proxie** esta habilitado con el comando:<br><br>
`sudo ln -s /etc/nginx/sites-available/proxy /etc/nginx/sites-enabled/`<br><br>

## Redirigir en apache.
Dentro del archivo de configuración del sitio.
Para que el tráfico de sitio2.com siempre se redirija a www.sitio2.com (o viceversa), agrego una redirección en Apache usando una configuración adicional en el archivo de VirtualHost. <br><br>

Redirigir sitio2.com a www.sitio2.com:<br><br>

<VirtualHost *:80><br>
    ServerName sitio2.com<br>
    Redirect permanent / http://www.sitio2.com/<br>
</VirtualHost><br>

<VirtualHost *:8080><br>
    ServerName www.sitio2.com<br>
    DocumentRoot "/var/www/sitio2"<br>
    ServerAdmin webmaster@localhost<br>
    ErrorLog ${APACHE_LOG_DIR}/error.log<br>
    CustomLog ${APACHE_LOG_DIR}/access.log combined<br>
</VirtualHost><br>
Con esto, cualquier solicitud a sitio2.com será automáticamente redirigida a www.sitio2.com.<br><br>

## etc/hosts hay que poenr todo tipo de dominios a usar
Si en mi configuración del sitio tengo esat linea **ServerName www.sitio2.com sitio2.com**
pero luego en **ect/hosts** sol otenga esta otra linea **127.0.0.1 sitio2.com**, si bien 
Apache está configurado para aceptar solicitudes tanto para www.sitio2.com como para sitio2.com, 
pero en mi archivo /etc/hosts solo he agregado la entrada para sitio2.com, por tanto, cuando intento 
acceder a www.sitio2.com, mi máquina no podrá resolver ese dominio porque no está definido en **/etc/hosts**.
Debo añadirlo también en local `127.0.0.1 sitio2.com www.sitio2.com`<br><br>

## Configuración de seguridad SSL
### Paso 1: Instalar OpenSSL y habilitar el módulo mod_ssl en Apache
Pasos:
- Instala OpenSSL:
`sudo apt install openssl`

- Habilita el módulo SSL en Apache:
`sudo a2enmod ssl`

- Reinicia Apache para que los cambios surtan efecto:
`sudo systemctl restart apache2`<br><br>

### Paso 2: Crear un certificado SSL autofirmado

- Crea el directorio donde se guardarán los certificados SSL:
`sudo mkdir /etc/ssl/certs /etc/ssl/private`

- Genera el certificado autofirmado (esto también generará una clave privada):
`sudo openssl req -x509 -newkey rsa:4096 -keyout /etc/ssl/private/tu-clave-privada.key -out /etc/ssl/certs/tu-certificado.crt -days 365`
Aquí se te pedirá información sobre el certificado (como el nombre del dominio, la organización, etc.). Puedes dejar las opciones en blanco si es para pruebas locales.<br><br>

### Paso 3: Configurar el archivo de VirtualHost para HTTPS

- Abre el archivo de configuración del sitio en Apache. Si estás configurando sitio2.com, puedes abrir su archivo de configuración en 
**/etc/apache2/sites-available/sitio2.conf** (o el que haya creado).
`sudo nano /etc/apache2/sites-available/sitio2.conf`

- Añade el siguiente bloque de configuración dentro de este archivo para habilitar HTTPS:
<VirtualHost *:443>
    ServerAdmin webmaster@tusitio.com
    DocumentRoot "/var/www/tusitio"
    ServerName www.sitio2.com

    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/tu-certificado.crt
    SSLCertificateKeyFile /etc/ssl/private/tu-clave-privada.key

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

- Habilito el módulo SSL:
`sudo a2enmod ssl`

## Configurar forzar HTTPS - redirigir HTTP a HTTPS
Esto significa redirigir automáticamente cualquier solicitud que llegue a tu servidor usando HTTP (puerto 80) hacia HTTPS (puerto 443).

En mi configuración de Apache, necesito dos bloques de configuración:
- Uno para el puerto 80: Este bloque maneja las conexiones HTTP no seguras.
- Otro para el puerto 443: Este bloque maneja las conexiones HTTPS seguras.

**¿Por qué se necesitan ambos?**
El puerto 80 es el puerto predeterminado para HTTP (no seguro), por lo que cuando un usuario visita tu sitio sin especificar 
el protocolo (por ejemplo, http://sitio2.com), Apache escuchará en este puerto.
El puerto 443 es el puerto predeterminado para HTTPS (seguro), y cuando un usuario visita tu sitio con el protocolo 
seguro (por ejemplo, https://sitio2.com), Apache lo manejará en este puerto.

### La configuración de redirección: *VirtualHost :80
En el bloque del puerto 80 (HTTP), redirigo a los usuarios a la versión segura del sitio (HTTPS).
Esto se hace con la directiva **Redirect permanent**.<br>
<VirtualHost *:80><br>
    ServerAdmin webmaster@localhost<br>
    ServerName www.sitio2.com<br>
    DocumentRoot "/var/www/sitio2"<br>
    # Redirigir todo el tráfico HTTP a HTTPS<br>
    Redirect permanent / https://sitio2.com/<br>
</VirtualHost><br>

### La configuración para HTTPS: *VirtualHost :443
En este bloque se incluyen las configuraciones para el certificado SSL.<br>
<VirtualHost *:443><br>
    ServerAdmin webmaster@localhost<br>
    ServerName www.sitio2.com<br>
    DocumentRoot "/var/www/sitio2"<br>
    # Activar SSL<br>
    SSLEngine on<br>
    SSLCertificateFile /etc/ssl/certs/tu-certificado.crt<br>
    SSLCertificateKeyFile /etc/ssl/private/tu-clave-privada.key<br>
    # Archivos de log<br>
    ErrorLog ${APACHE_LOG_DIR}/error.log<br>
    CustomLog ${APACHE_LOG_DIR}/access.log combined<br>
</VirtualHost><br>

## No forzar redirección
Si prefiero que los usuarios puedan acceder tanto mediante HTTP como HTTPS, entonces basta con eliminar la línea de redirección del puerto 80.
`Redirect permanent / https://sitio2.com/`
Ahora Apache ya no redirigirá automáticamente a los usuarios de HTTP a HTTPS, y en su lugar, permitiría que se conecten a tu sitio tanto 
con HTTP como con HTTPS, mostrando el mismo contenido en ambos casos.

- Con HTTP (http://sitio2.com):
Los usuarios llegarían a tu sitio utilizando el protocolo HTTP (puerto 80) sin ningún tipo de redirección.
- Con HTTPS (https://sitio2.com):
Los usuarios llegarían a tu sitio utilizando el protocolo HTTPS (puerto 443) de forma segura, ya que Apache manejará esa conexión con SSL.

## HTTP/2
Lo habilito con el comando: `sudo a2enmod http2`.
`sudo systemctl restart apache2`.

Luego añado la directiva Protocols en la configuración del servidor apache o en un archivo de configuración de un sitio específico.

Puedo añadirla en el bloque <VirtualHost *:443> para HTTPS. Debería verse algo así:<br>

<VirtualHost *:443><br>
    ServerAdmin webmaster@tu-sitio.com<br>
    DocumentRoot /var/www/tu-sitio<br>
    ServerName www.tu-sitio.com<br>
    # Configuración SSL<br>
    SSLCertificateFile /etc/ssl/certs/tu-certificado.crt<br>
    SSLCertificateKeyFile /etc/ssl/private/tu-clave-privada.key<br>
    # Activar HTTP/2<br>
    Protocols h2 http/1.1<br>
    ErrorLog ${APACHE_LOG_DIR}/error.log<br>
    CustomLog ${APACHE_LOG_DIR}/access.log combined<br>
</VirtualHost><br>

Explicación: La directiva Protocols h2 http/1.1 indica que Apache debe soportar HTTP/2 (h2) y 
también seguir soportando HTTP/1.1 (http/1.1) para clientes que no son compatibles con HTTP/2.

Por último `sudo systemctl restart apache2`.

## Activación de sitios Apache vs Nginx
En Apache, los sitios se activan con el comando **a2ensite** este comando automáticamente crea un enlace simbólico desde el directorio 
**sites-available** hacia **sites-enabled**, activando la configuración del sitio. No necesito hacer esto manualmente, 
ya que a2ensite se encarga de crear el enlace simbólico por mi.
Ativacion:
`sudo a2ensite mi_sitio.conf`<br>
`sudo systemctl reload apache2`<br>
Desactivación:
`sudo a2dissite mi_sitio.conf`<br>
`sudo systemctl reload apache2`<br>

En Nginx, el proceso es manual. Cuando creo un archivo de configuración para un sitio en **/etc/nginx/sites-available/** necesito crear 
un enlace simbólico hacia **/etc/nginx/sites-enabled/** para activarlo.
Activación:
`sudo ln -s /etc/nginx/sites-available/mi_sitio /etc/nginx/sites-enabled/`<br>
`sudo systemctl reload nginx`<br>
Desactivación:
`sudo rm /etc/nginx/sites-enabled/mi_sitio`<br>
`sudo systemctl reload nginx`<br>
Esto es necesario porque Nginx no tiene un comando similar a a2ensite de Apache. Entonces, debes hacerlo manualmente
usando **ln -s** para crear el enlace simbólico.<br>

Borrar de ambos definitvamente:<br>
`sudo rm /etc/apache2/sites-available/mi_sitio.conf`<br>
`sudo rm /etc/nginx/sites-available/mi_sitio`

**SIN NETPLAN EN EL SERVER**
`sudo apt update`
`sudo apt install netplan.io`

Otro forma  de conectarme

## Permisos

`sudo chown -R $USER:$USER /var/www/misitio`<br><br>
`sudo chmod -R 755 /var/www`<br><br>

## Crear directorio e index

`sudo mkdir -p /var/www/sitio`
`echo "<h1>Bienvenido a mi sitio web</h1>" | sudo tee /var/www/misitio/index.html`
