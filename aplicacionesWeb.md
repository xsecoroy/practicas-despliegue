# APUNTES APLICACIONES WEB

## MYSQL

### Autenticación en MySQL 

En las versiones actuales de MySQL (como MySQL 5.7 y 8.0), el proceso de autenticación ha cambiado en comparación con versiones anteriores, al instalarlo se crea automáticamente un usuario root para MySQL, el cual está configurado para usar el método de autenticación **auth_socket**. Este método permite que el usuario root del sistema operativo acceda a MySQL sin necesidad de una contraseña, siempre y cuando se ejecute el comando con privilegios de superusuario (sudo).<br><br>
¿Qué es auth_socket y cómo funciona?
**auth_socket** es un complemento de autenticación que verifica la identidad del usuario a través del socket Unix. En lugar de usar una contraseña, MySQL verifica si el usuario del sistema operativo que intenta acceder coincide con el usuario configurado en MySQL.<br><br>

Puedo verificar el método de autenticación configurado para el usuario root ejecutando este comando dentro del cliente MySQL:<br><br>

`SELECT user, host, plugin FROM mysql.user;`<br><br>
El resultado mostrará algo como esto:<br><br>

+------+-----------+-----------------------+
| user | host      | plugin                |
+------+-----------+-----------------------+
| root | localhost | auth_socket           |
+------+-----------+-----------------------+<br><br>

Esto confirma que el usuario root usa el complemento auth_socket.<br><br>

Acceso como root sin contraseña (auth_socket):
En sistemas Ubuntu, el **usuario root** de MySQL está configurado para usar el complemento **auth_socket** de manera predeterminada. Esto significa que puedo acceder a MySQL como root sin necesidad de una contraseña, siempre y cuando use el mismo usuario root del sistema operativo.
Para acceder, simplemente ejecuto:
`sudo mysql`
Me llevará directamente al cliente MySQL como usuario root.<br><br>

**Método de autenticación predeterminado en Mysql**:<br><br>
Si no especifico explícitamente un método de autenticación, MySQL utiliza el método predeterminado configurado en el servidor. En la mayoría de las instalaciones recientes de MySQL, este método es **caching_sha2_password** (a partir de MySQL 8.0) o **mysql_native_password** (en versiones anteriores o si se ha configurado manualmente).
El método predeterminado depende de la variable del sistema default_authentication_plugin. Puedo verificarlo ejecutando:<br><br>

`SHOW VARIABLES LIKE 'default_authentication_plugin';`<br><br>

Aquí al crear un user se hace con la autenticación default:<br><br>
`CREATE USER 'nombre_usuario'@'%' IDENTIFIED BY 'contraseña';`<br><br>
Aquí al crear un user se hace con el método de autenticación explicito que indico:<br><br>
`CREATE USER 'usuario'@'%' IDENTIFIED WITH mysql_native_password BY 'password';`<br><br>

Usuario especial y archivo ***/etc/mysql/debian.cnf***:
Durante la instalación, MySQL crea un usuario especial para tareas administrativas automatizadas. Las credenciales de este usuario están almacenadas en el archivo ***/etc/mysql/debian.cnf***.
Puedo consultar este archivo para ver el usuario y la contraseña generados automáticamente:
`sudo cat /etc/mysql/debian.cnf`
Sin embargo, este usuario no suele ser necesario para tareas manuales, ya que puedo usar el usuario root con auth_socket.

Cambiar el método de autenticación a contraseña (opcional):
Si necesito usar una contraseña para el usuario root (por ejemplo, para conectarme desde herramientas como phpMyAdmin), puedo cambiar el método de autenticación de **auth_socket** a **mysql_native_password**:<br><br>
- Primero accedo a MySQL como root:<br><br>
`sudo mysql`<br><br>
- Cambio el método de autenticación:<br><br>
`ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'tu_contraseña';`<br><br>
`FLUSH PRIVILEGES;`<br><br>
- Ahora puedo conectartme como root usando la contraseña que configure:<br><br>
`mysql -u root -p`<br><br>


Por defecto, puedes acceder como root sin contraseña usando sudo mysql gracias al complemento auth_socket.
Si necesitousar una contraseña para el usuario root, puedes cambiar el método de autenticación a mysql_native_password.
Las credenciales del usuario especial están en /etc/mysql/debian.cnf, pero rara vez necesitas usarlas manualmente.

**Volver al método de autenticación auth_socket después de usar contraseña**
Se puedes volver al método de autenticación auth_socket después de haber cambiado el método de autenticación del usuario root a mysql_native_password. Se trata cuando decido que ya no necesito usar una contraseña para el usuario root y prefiero volver al comportamiento predeterminado en Ubuntu.<br><br>

- Accede a MySQL como root usando la contraseña configurada:<br><br>
`mysql -u root -p` <br><br>
- Luego, ingreso la contraseña que configure.<br><br>
- Cambio el método de autenticación a auth_socket: Una vez dentro del cliente MySQL, ejecuto el siguiente comando para cambiar el método de autenticación:<br><br>
`ALTER USER 'root'@'localhost' IDENTIFIED WITH auth_socket;`<br><br>
`FLUSH PRIVILEGES;`<br><br>
- Verifico el cambio: Para confirmar que el usuario root está usando nuevamente auth_socket, ejecuto:
`SELECT user, host, plugin FROM mysql.user;`<br><br>
Debería ver algo como esto:<br><br>

+------+-----------+-------------+
| user | host      | plugin      |
+------+-----------+-------------+
| root | localhost | auth_socket |
+------+-----------+-------------+
<br><br>
Pruebo el acceso sin contraseña: Ahora puedo salir del cliente MySQL y probar el acceso sin contraseña usando:<br><br>

`sudo mysql`<br><br>

### Conectarse a mysql como usuario
**¿Qué hace el comando mysql -u root -p y para qué sirve?**<br><br>
El comando mysql -u root -p se utiliza para conectarte al servidor MySQL desde la línea de comandos como el usuario root si le pusimos contraseña. <br><br>
- **mysql**: Es el cliente de línea de comandos de MySQL que te permite interactuar con el servidor MySQL.
- **u root**: Especifica que deseo conectarme como el usuario root.
- **p**: Indica que deseo usar una contraseña para autenticarme. Después de ejecutar el comando, se me pedirá que ingrese la contraseña del usuario root.<br><br>

**¿Por qué usar este comando en lugar de sudo mysql?**<br><br>
- **sudo mysql**: Este comando utiliza el método de autenticación auth_socket, que no requiere contraseña y me permite acceder como root si soy el usuario root del sistema operativo.
- **mysql -u root -p**: Este comando es útil si he configurado una contraseña para el usuario root y necesito autenticarte con ella. También es necesario si estoy accediendo desde una máquina remota o desde una herramienta externa como phpMyAdmin.<br><br>


### Diferencias entre los usuarios en MySQL según su host

En MySQL, los usuarios no solo se identifican por su nombre de usuario, sino también por el host desde el cual se conectan. Esto significa que puedo tener diferentes configuraciones y permisos para el mismo nombre de usuario dependiendo de la máquina desde la que se conecte.<br><br>

1. Diferencias entre los ejemplos de creación de usuarios<br><br>

- **Usuario específico para una máquina cliente**:<br><br>

`CREATE USER 'nombre_usuario'@'IP_MÁQUINA_CLIENTE' IDENTIFIED BY 'contraseña';`<br><br>
Este usuario solo puede conectarse al servidor MySQL desde una máquina específica, identificada por su dirección IP (IP_MÁQUINA_CLIENTE). Si intento conectarme desde otra máquina, incluso con el mismo nombre de usuario y contraseña, la conexión será rechazada.<br><br>

- **Usuario para conexiones locales**:<br><br>

`CREATE USER 'nombre_usuario'@'localhost' IDENTIFIED BY 'contraseña';`<br><br>
Este usuario solo puede conectarse al servidor MySQL desde la misma máquina donde está instalado MySQL. Esto incluye conexiones realizadas desde el servidor local usando herramientas como mysql o aplicaciones que se ejecutan en el mismo servidor.<br><br>

- **Usuario para conexiones desde cualquier dirección**: <br><br>

`CREATE USER 'nombre_usuario'@'%' IDENTIFIED BY 'contraseña';`<br><br>

Este usuario puede conectarse al servidor MySQL desde cualquier máquina remota (es decir, cualquier dirección IP). El carácter % es un comodín que representa "cualquier host". Este tipo de configuración es útil para usuarios que necesitan acceso remoto, pero puede ser menos seguro si no se configura adecuadamente.<br><br>

2. Métodos de autenticación<br><br>

`CREATE USER 'usuario'@'%' IDENTIFIED WITH mysql_native_password BY 'password';`<br><br>
Aquí, el usuario se autentica usando el método mysql_native_password, que requiere una contraseña. Esto es útil para conexiones remotas o cuando no se desea usar métodos como auth_socket.<br><br>

3. ¿Qué pasa si me conecto por SSH a la máquina del servidor?<br><br>

Cuando me conecto por SSH a la máquina del servidor y luego accedo a MySQL, estoy considerado como "localhost". Esto se debe a que, una vez que estoy dentro del servidor a través de SSH, cualquier conexión que hago a MySQL se realiza desde la misma máquina donde está instalado el servidor MySQL. Me conectas por SSH a tu servidor:<br><br>

`ssh usuario@IP_SERVIDOR`<br><br>
Una vez dentro del servidor, ejecuto:<br><br>

`mysql -u root -p`<br><br>
En este caso, estoy accediendo como **root@localhost**, porque la conexión a MySQL se realiza desde la máquina local.<br><br>

4. ¿Qué pasa si me conecto desde tu máquina cliente directamente a MySQL?<br><br>

Si intento conectarme a MySQL desde mi máquina cliente sin usar SSH, entonces estaría considerado como un usuario remoto. Por ejemplo:<br><br>

`mysql -u nombre_usuario -p -h IP_SERVIDOR`<br><br>
En este caso:<br><br>
**-h IP_SERVIDOR** indica que me estpy conectando a un servidor remoto.
MySQL verificará si existe un usuario con el formato **'nombre_usuario'@'IP_DE_TU_CLIENTE'** o **'nombre_usuario'@'%'**. <br><br>
5. Resumen<br><br>
Si me conecto por SSH a la máquina del servidor y luego accedo a MySQL, estoy considerado como "localhost", porque la conexión se realiza desde la misma máquina donde está instalado MySQL.
Si te conectas directamente desde tu máquina cliente a MySQL (sin usar SSH), entonces estás considerado como un usuario remoto, y MySQL verificará si tienes permisos como 'nombre_usuario'@'IP_MÁQUINA_CLIENTE' o 'nombre_usuario'@'%'.


### PROBLEMA PHPMYADMIN NO ACCESIBLE - SOLUCIONES

1. Verifico si phpMyAdmin está instalado correctamente<br><br>
Me aseguro de que el paquete phpmyadmin se haya instalado correctamente:<br><br>
`dpkg -l | grep phpmyadmin`
Si no aparece en la lista, significa que phpMyAdmin no se instaló correctamente, deberé hacer:<br><br>
`sudo apt install phpmyadmin`<br><br>

2. Verifico si phpMyAdmin está configurado en Apache
Durante la instalación de phpMyAdmin, se me pregunto qué servidor web deseaba configurar (por ejemplo, Apache). 
Si no seleccioné Apache o hubo un error en este paso, phpMyAdmin no estará disponible en mi servidor web.
Solución: Configurar phpMyAdmin manualmente en Apache, creo un enlace simbólico para que Apache pueda encontrar phpMyAdmin:<br><br>
`sudo ln -s /usr/share/phpmyadmin /var/www/html/phpmyadmin`<br<br>
Esto crea un enlace simbólico desde el directorio donde está instalado phpMyAdmin (/usr/share/phpmyadmin) al directorio raíz de Apache (/var/www/html). Reinicio Apache para aplicar los cambios:
`sudo systemctl restart apache2`<br><br>
Intento acceder nuevamente a http://ip-servidor/phpmyadmin.<br><br>

3. Verificar los permisos del directorio<br><br>
Me asegúro de que Apache tenga permisos para acceder al directorio de phpMyAdmin. Ejecuto:<br><br>
`sudo chmod -R 755 /usr/share/phpmyadmin`

Esto asegura que el servidor web pueda leer los archivos de phpMyAdmin.<br><br>

4. Verifica la configuración de Apache
Si el enlace simbólico no funciona, puede ser necesario configurar manualmente un archivo de configuración para phpMyAdmin en Apache.
Crea un archivo de configuración para phpMyAdmin:
javascript


sudo nano /etc/apache2/conf-available/phpmyadmin.conf
Agrega el siguiente contenido al archivo:
javascript


Alias /phpmyadmin /usr/share/phpmyadmin

<Directory /usr/share/phpmyadmin>
    Options FollowSymLinks
    DirectoryIndex index.php

    <IfModule mod_php.c>
        AddType application/x-httpd-php .php
    </IfModule>

    <FilesMatch ".+\.php$">
        SetHandler application/x-httpd-php
    </FilesMatch>

    <IfModule mod_deflate.c>
        AddOutputFilterByType DEFLATE text/html text/plain text/xml text/css application/xml application/xhtml+xml application/rss+xml application/javascript application/x-javascript
    </IfModule>

    <IfModule mod_headers.c>
        Header set X-Content-Type-Options "nosniff"
        Header set X-Frame-Options "SAMEORIGIN"
        Header set X-XSS-Protection "1; mode=block"
    </IfModule>
</Directory>
Habilita la configuración:
javascript


sudo a2enconf phpmyadmin
Reinicia Apache:
javascript


sudo systemctl restart apache2
Intenta acceder nuevamente a http://ip-servidor/phpmyadmin.
5. Verifica los módulos de Apache
Asegúrate de que los módulos necesarios de Apache estén habilitados:
javascript


sudo a2enmod rewrite
sudo a2enmod php
sudo systemctl restart apache2
6. Verifica errores en los logs de Apache
Si el problema persiste, revisa los logs de Apache para obtener más información sobre el error:
javascript


sudo tail -f /var/log/apache2/error.log
Esto puede darte pistas sobre qué está fallando.
Resumen de pasos
Verifica que phpMyAdmin esté instalado correctamente.
Crea un enlace simbólico para que Apache pueda encontrar phpMyAdmin.
Asegúrate de que Apache tenga permisos para acceder al directorio de phpMyAdmin.
Configura manualmente phpMyAdmin en Apache si es necesario.
Habilita los módulos necesarios de Apache.
Revisa los logs de Apache para identificar problemas específicos.
Con estos pasos, deberías poder solucionar el error y acceder a phpMyAdmin en http://ip-servidor/phpmyadmin.


### CREAR UNA BBDD Y OTORGAR PERMISOS

1. Lista las bases de datos disponibles:<br><br>
`SHOW DATABASES;`<br><br>
Esto mostrará todas las bases de datos en el servidor MySQL, incluida las creadas correctamente en phpmyadmin.<br><br>

2. Crear la base de datos desde terminal:<br><br>
`CREATE DATABASE demo;`<br><br>
Esto creará una base de datos llamada demo.<br><br>

3. Crear el usuario demo con contraseña<br><br>
Creo el usuario demo con la contraseña password:<br><br>
`CREATE USER 'demo'@'%' IDENTIFIED BY 'password';`
**'demo'@'%'**: El usuario demo podrá conectarse desde cualquier dirección IP (%).<br><br>
**'password'**: Es la contraseña del usuario.<br><br>

Ejemplos de alcance en MySQL:<br><br>
demo.*: Aplica a todas las tablas dentro de la base de datos demo.<br><br>
demo.mi_tabla: Aplica solo a la tabla específica mi_tabla dentro de la base de datos demo.<br><br>
*.*: Aplica a todas las bases de datos y todas las tablas en el servidor MySQL.<br><br>

4. Otorgar permisos al usuario demo<br><br>
Otorgo todos los privilegios sobre la base de datos demo al usuario demo:<br><br>
`GRANT ALL PRIVILEGES ON demo.* TO 'demo'@'%';`
demo.*:  Esto aplica los privilegios a todas las tablas (*) dentro de la base de datos demo.
GRANT ALL PRIVILEGES: Otorga todos los permisos necesarios (lectura, escritura, modificación, etc.).

5. Aplicar los cambios
Ejecuto el siguiente comando para recargar los privilegios y asegurarme de que los cambios surtan efecto:<br><br>
`FLUSH PRIVILEGES;`<br><br>

6. Verificar la configuración<br><br>
Puedo verificar que la base de datos y el usuario se hayan creado correctamente:<br><br>
 - Listar bases de datos:<br><br>
`SHOW DATABASES;`<br><br>
Deberías ver demo en la lista.<br><br>
Verificar los privilegios del usuario demo:<br><br>

`SHOW GRANTS FOR 'demo'@'%';`<br><br>
Esto mostrará los permisos otorgados al usuario demo.<br><br>


### IMPORTAR ARCHIVO .SQL EN LA BBDD CON TERMINAL

1. Coloco el archivo .sql en una ubicación accesible desde la terminal (por ejemplo, en el directorio de inicio o en /tmp).<br><br>

2. Me conecto al servidor MySQL.<br><br>
`mysql -u tu_usuario -p`<br><br>

2. Selecciono la base de datos demo:<br><br>
Una vez dentro de MySQL, selecciona la base de datos demo:<br><br>
`USE demo;`

3. Importo el archivo SQL: <br><br>
Salgo de la consola MySQL (escribo exit) y ejecuta el siguiente comando desde la terminal:

`mysql -u tu_usuario -p demo < /ruta/al/archivo.sql` <br><br>

Reemplazo /ruta/al/archivo.sql con la ruta completa al archivo SQL que deseo importar.
Por ejemplo, si el archivo está en el directorio de inicio y se llama archivo.sql, el comando sería:<br><br>
`mysql -u tu_usuario -p demo < ~/archivo.sql`<br><br>

4. Verifico la importación:<br><br>
Vuelvo a conectarte a MySQL y selecciono la base de datos demo:<br><br>
`mysql -u tu_usuario -p`<br><br>
`USE demo;`<br><br>
`SHOW TABLES;`<br><br>
Esto mostrará las tablas importadas en la base de datos.<br><br>


### PROCESO SERVIR SITIO SOLO CON APACHE

1. El navegador solicita demo.local<br><br>

Cuando escribp **demo.local** en el navegador, este intenta resolver el dominio demo.local a una dirección IP. Como demo.local no es un 
dominio real en internet, el navegador consulta el archivo ***/etc/hosts*** para encontrar una entrada que asocie demo.local 
con una dirección IP.<br><br>
**127.168.0.1 demo.local**<br><br>
Entonces el navegador sabe que debe enviar la solicitud a 127.168.0.1 (la dirección IP local, es decir, el propio servidor).<br><br>
2. Apache recibe la solicitud<br><br>
Apache, que está escuchando en el puerto 80 (por defecto), recibe la solicitud para demo.local. Ahora Apache necesita decidir qué sitio web debe servir. 
Aquí es donde entran los archivos de configuración de los Virtual Hosts.<br><br>

3. Apache busca el Virtual Host correspondiente<br><br>

Apache revisa los archivos de configuración de los Virtual Hosts para encontrar uno que coincida con el dominio solicitado (demo.local). 
Estos archivos suelen estar en el directorio **/etc/apache2/sites-available/**.
En mi caso, el archivo de configuración es demo.conf y contiene:<br><br>

<VirtualHost *:80>
    ServerName demo.local
    DocumentRoot /var/www/demo
</VirtualHost>
Apache busca el ServerName que coincida con demo.local. Si lo encuentra, utiliza este Virtual Host para manejar la solicitud.<br><br>

4. Apache verifica el DocumentRoot<br><br>
Una vez que Apache encuentra el Virtual Host correcto, revisa la directiva DocumentRoot. Esta directiva le dice a Apache dónde están los archivos 
del sitio web que debe servir.Por ejemplo, si el DocumentRoot está configurado como: **DocumentRoot /var/www/demo** <br><br>
Apache buscará los archivos del sitio web en el directorio **/var/www/demo**.<br><br>

5. Apache busca el archivo predeterminado con DirectoryIndex<br><br>
Si  no especifico un archivo en la URL (por ejemplo, solo escribe http://demo.local en lugar de http://demo.local/index.php), Apache utiliza la 
directiva DirectoryIndex para determinar qué archivo cargar.<br><br>
Si tiengo configurado:<br><br>
**DirectoryIndex index.php** <br><br>
Apache buscará un archivo llamado **index.php** en el directorio especificado por DocumentRoot (en este caso, /var/www/demo).<br><br>
Si no configure DirectoryIndex, Apache usará su configuración predeterminada, que generalmente incluye index.html y index.php. <br><br>
Si no encuentra ninguno de estos archivos, puede mostrar un listado del contenido del directorio (si está habilitado Options Indexes) o devolver un error.<br><br>

6. El archivo .conf debe estar habilitado<br><br>
Para que Apache utilice el archivo de configuración del Virtual Host (por ejemplo, demo.conf), este debe estar habilitado. Esto se hace creando 
un enlace simbólico en el directorio **/etc/apache2/sites-enabled/**.
Cuando ejecutp: **sudo a2ensite demo.conf** <br><br>
Apache crea un enlace simbólico desde **/etc/apache2/sites-available/demo.conf** a **/etc/apache2/sites-enabled/demo.conf**.<br><br>
Apache solo lee los archivos de configuración que están en /etc/apache2/sites-enabled/. Por eso, aunque el archivo esté en sites-available, 
no tendrá efecto hasta que lo habilite con a2ensite.<br><br>

7. Apache sirve el archivo al navegador<br><br>
Finalmente, Apache encuentra el archivo especificado (por ejemplo, /var/www/demo/index.php), lo procesa (si es un archivo PHP, lo pasa al intérprete de PHP),
y envía el resultado al navegador.<br><br>

**Resumen del proceso**
- El navegador solicita demo.local.
- El sistema resuelve demo.local a 127.0.0.1 usando /etc/hosts.
- Apache recibe la solicitud y busca un Virtual Host con ServerName demo.local.
- Apache encuentra el Virtual Host y revisa el DocumentRoot para saber dónde buscar los archivos.
- Apache usa DirectoryIndex para determinar qué archivo cargar si no se especifica uno en la URL.
- Apache procesa el archivo y envía la respuesta al navegador.


### APLICACIÓN CON NODE
No necesito Apache para ejecutar y ver una aplicación creada con Node.js. Node.js incluye su propio servidor HTTP, lo que significa que puedo 
manejar solicitudes directamente sin necesidad de un servidor web como Apache o Nginx. 
Por lo tanto, no necesitas crear un archivo .conf en sites-available ni colocar los archivos en /var/www/.<br><br>

- Opción 1: Clonar directamente en el directorio actual<br><br>
Si estoy en el directorio donde quiero que se cree el repositorio, simplemente ejecuto el comando git clone y Git creará un subdirectorio con el nombre del repositorio.
`isard@isard:~/proyectonode$ git clone https://github.com/rafacabeza/demoapinode`<br><br>
Esto creará un subdirectorio llamado **demoapinode** dentro de **~/proyectonode**.<br><br>

- Opción 2: Clonar en un directorio específico
Si quiero que el repositorio se clone en un directorio con un nombre diferente, puedes especificarlo como un argumento adicional en el comando git clone.
Clonao el repositorio en un directorio con un nombre personalizado:
`isard@isard:~/proyectonode$ git clone https://github.com/rafacabeza/demoapinode mi-aplicacion`<br><br>
Esto creará un subdirectorio llamado **mi-aplicacion** en lugar de demoapinode.<br><br>

**¿Qué pasa después de clonar el repositorio?**
Accedo al directorio clonado `cd demoapinode` e instalo las dependencias del proyecto: Si el proyecto tiene un archivo package.json (como en este caso),
instalo las dependencias necesarias con `npm install`.<br><br>
Configuro la base de datos y otros parámetros.<br><br>
Ejecuto la aplicación con `npm start`<br><br>

**Problema al hacer `npm install`** lo solucione dando permisos al usuario actual sobre el proyecto:<br><br>
`sudo chown -R $USER:$USER /home/proyectonode/demoapinode`

**Pasos para importar el archivo SQL del proyecto node**<br><br>
Creo la base de datos demo en MySQL, primero, accede a MySQL con mi usuario (por ejemplo, king):<br><br>
`mysql -u root -p`<br><br>
Luego, crea la base de datos demo:<br><br>
`CREATE DATABASE demo;`<br><br>
Importo el archivo SQL, Si estás en el directorio donde se encuentra demo.sql, simplemente ejecuto:<br><br>
`mysql -u root -p demo < demo.sql`<br><br>
ruta relativa:<br><br>
`mysql -u root -p demo < /home/proyectonode/demoapinode/demo.sql`<br><br>
Esto importará las tablas y datos definidos en el archivo demo.sql a la base de datos demo.<br><br>
Eso si le debo dar todos los permisos a king `GRANT ALL PRIVILEGES ON demo.* TO 'king'@'%';` y `FLUSH PRIVILEGES;`<br><br>

### NGINX COMO PROXY INVERSO DE NODE Y APACHE

1. Nginx actuará como un proxy inverso que redirige las solicitudes entrantes a diferentes servidores backend 
según el dominio o subdominio que se utilice. Por ejemplo:<br><br>
- demonode.local → Redirige a tu aplicación Node.js (en localhost:3000).<br><br>
- demo.local → Redirige al servidor Apache (en localhost:8080 o el puerto que Apache esté usando).<br><br>

2. **En APACHE debo cambiar el puerto predeterminado y de los proyectos** <br><br>
Para cambiar el puerto de Apache debo editar el archivo de configuración de Apache:<br><br>
`sudo nano /etc/apache2/ports.conf`<br><br>

Después he de cambiar los puertos en los sitios de **apache2/sites-available/** pues los archivos .conf en 
esta carpeta definen la configuración de los sitios virtuales (Virtual Hosts). Cada sitio puede especificar un puerto 
en el que debe funcionar. Por ejemplo:<br><br>
<VirtualHost *:80>
    ServerName example.com
    DocumentRoot /var/www/example
</VirtualHost>
<br><br>
Si el puerto especificado aquí (*:80) no coincide con un puerto que Apache está escuchando (definido en ports.conf), el sitio no funcionará.
Cambio Listen **80** a Listen **8080** y guardo el archivo para luego reiniciar Apache:<br><br>

`sudo systemctl restart apache2`
<br><br>

Después de realizar los cambios, verifico que la configuración de Apache es válida: `sudo apachectl configtest`<br><br>

3. Configurar el archivo **/etc/hosts** en el cliente<br><br>
Agrega los dominios demo.local y demonode.local al archivo /etc/hosts:<br><br>
`sudo nano /etc/hosts`<br><br>
**127.0.0.1 demo.local**<br><br>
**127.0.0.1 apache.local**<br><br>

4. Configuración de Nginx como proxy inverso
Creo un archivo en sites-available de nginx y en el bloques de servidor para cada dominio, por tanto Editare el archivo de 
configuración de Nginx o creo uno nuevo para manejar los dominios. Por ejemplo:<br><br>
`sudo nano /etc/nginx/sites-available/proxy-config` <br><br>
En el archivo agrego los bloques de cada dominio:<br><br>

***Bloque para la aplicación Node.js*** <br><br>
server {
    listen 80;
    server_name demonode.local;

    location / {
        proxy_pass http://localhost:3000; # Redirige a la aplicación Node.js
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}

***Bloque para el servidor Apache*** <br><br>
server {
    listen 80;
    server_name demo.local;

    location / {
        proxy_pass http://localhost:8080; # Redirige al servidor Apache
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
<br><br>

Después debo habilitar la configuración, es decir que aparezca correcto en sites-enabled, para ello
creo un enlace simbólico para habilitar la configuración:<br><br>
`sudo ln -s /etc/nginx/sites-available/proxy-config /etc/nginx/sites-enabled/`<br><br>
Verifico que la configuración de Nginx no tenga errores:<br><br>
`sudo nginx -t`<br><br>
Reinicia Nginx para aplicar los cambios:<br><br>
`sudo systemctl restart nginx`<br><br>

Solo si deria errores de firewall hacer esto "Firewall:
`sudo ufw allow 'Nginx Full`<br><br>

### APACHE COMO PROXY INVERSO DE NODE
¿Qué pasa si quieres usar Apache como proxy inverso?
Si decides usar Apache como proxy inverso para redirigir solicitudes a tu aplicación Node.js, entonces sí necesitarás crear un archivo .conf en sites-available. Aquí tienes un ejemplo de configuración:
Crea un archivo .conf para tu aplicación:
javascript


sudo nano /etc/apache2/sites-available/demo-node.conf
Agrega la configuración del proxy inverso:
apache


<VirtualHost *:80>
    ServerName demo.local

    ProxyPreserveHost On
    ProxyPass / http://localhost:3000/
    ProxyPassReverse / http://localhost:3000/

    ErrorLog ${APACHE_LOG_DIR}/demo-node_error.log
    CustomLog ${APACHE_LOG_DIR}/demo-node_access.log combined
</VirtualHost>
Habilita los módulos necesarios en Apache:
javascript


sudo a2enmod proxy
sudo a2enmod proxy_http
Habilita el sitio y reinicia Apache:
javascript


sudo a2ensite demo-node.conf
sudo systemctl restart apache2
Accede a la aplicación:
Ahora puedes acceder a tu aplicación en http://demo.local (asegúrate de que demo.local esté configurado en el archivo /etc/hosts de tu máquina cliente).