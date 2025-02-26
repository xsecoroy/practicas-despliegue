
# PRÁCTICA 3 LORIEN BORRA

### Creación de usuario en MySQL para la práctica

Primero accedo con sudo a mysql: `sudo mysql`<br><br>

- **Creo un usuario demo para la practica y  para conexiones desde cualquier dirección**: <br><br>

`CREATE USER 'demo'@'%' IDENTIFIED BY 'password';`<br><br>
![Texto alternativo](./imagenes/mysqlnuevouser.png)<br><br>



### CREACIÓN DE UNA BBDD Y OTORGAR PERMISOS
1. Creo la base de datos desde terminal:<br><br>
`CREATE DATABASE demo;`<br><br>
Esto creará una base de datos llamada demo.<br><br>

2. Otorgo permisos al usuario demo<br><br>
Otorgo todos los privilegios sobre la base de datos demo al usuario demo:<br><br>
`GRANT ALL PRIVILEGES ON demo.* TO 'demo'@'%';`<br><br>

3. Aplico los cambios
Ejecuto el siguiente comando para recargar los privilegios y asegurarme de que los cambios surtan efecto:<br><br>
`FLUSH PRIVILEGES;`<br><br>

![Texto alternativo](./imagenes/mysqlprivilegios.png)<br><br>


### PROCESO SERVIR SITIO SOLO CON APACHE

- Clono directamente en el directorio actual<br><br>
Estoy en el directorio donde quiero que se cree el repositorio, en **/var/www/** simplemente ejecuto el comando git clone y Git creará un subdirectorio con el nombre del repositorio pero añado el personalziado **depliegue1**.
`git clone https://github.com/rafacabeza/demoappphp depliegue1`<br><br>
Esto crea un subdirectorio llamado **depliegue1** en vez de **demoppphp**.<br><br>

1. Importo el archivo SQL: <br><br>
`mysql -u usuario -p BBDD < /ruta/al/archivo.sql` <br><br>
`mysql -u demo -p demo < /var/www/depliegue1/demo.sql`<br><br>
![Texto alternativo](./imagenes/importarbbdd.png)<br><br>


2. Verifico la importación:<br><br>
Vuelvo a conectarme a MySQL y selecciono la base de datos demo para ver que esta correcto:<br><br>
`mysql -u demo -p`<br><br>
`USE demo;`<br><br>
`SHOW TABLES;`<br><br>
![Texto alternativo](./imagenes/mostrartabla.png)<br><br>

3. Configuro los dominios en **/etc/hosts**<br><br>
![Texto alternativo](./imagenes/dominiosenhosts.png)<br><br>

4. Apache recibe la solicitud<br><br>
Apache, que está escuchando en el puerto 80 (por defecto), pero yo lo he cambiado al 8080, en el archivo **/etc/apache2/ports.conf** . <br><br>
![Texto alternativo](./imagenes/puertoapacheportconf.png)<br><br>

5. Apache busca el Virtual Host correspondiente<br><br>
Apache revisa los archivos de configuración de los Virtual Hosts para encontrar uno que coincida con el dominio solicitado (php.local). Estos archivos están en el directorio **/etc/apache2/sites-available**.
En mi caso, creo ese archivo en ese directorio, el archivo de configuración es **despliegue1.conf** y contiene estos datos:<br><br>
![Texto alternativo](./imagenes/despliegue1conf.png)<br><br>


6. El archivo .conf que he creado antes debe estar habilitado<br><br>
Para que Apache utilice el archivo de configuración del Virtual Host debe estar habilitado. Esto se hace creando un enlace simbólico en el directorio **/etc/apache2/sites-enabled/**.
Ejecuto: **sudo a2ensite despliegue1.conf** <br><br>
Apache crea un enlace simbólico desde **/etc/apache2/sites-available/demo.conf** a **/etc/apache2/sites-enabled/demo.conf**.<br><br>
![Texto alternativo](./imagenes/apachesitienable.png)<br><br>

7. Configuro el archivo index.php dentro de ***/var/www/depliegue1/app/index.php*** con las nuevas credenciales que hice en mysql.<br><br>
![Texto alternativo](./imagenes/confiindexphp.png)<br><br>

8. Apache sirve el archivo al navegador<br><br>
Finalmente, Apache encuentra el archivo especificado (por ejemplo, /var/www/demo/index.php), lo procesa (si es un archivo PHP, lo pasa al intérprete de PHP), y envía el resultado al navegador.<br><br>
![Texto alternativo](./imagenes/phplocal8080.png)<br><br>




### APLICACIÓN CON NODE


- Clonar directamente en el directorio actual<br><br>
Si estoy en el directorio donde quiero que se cree el repositorio, simplemente ejecuto el comando git clone y Git creará un subdirectorio con el nombre del repositorio.
`isard@isard:~/proyectonode$ git clone https://github.com/rafacabeza/demoapinode despliegue2`<br><br>
Esto creo un subdirectorio llamado **despliegue2** dentro del directorio raiz **~/**.<br><br>
![Texto alternativo](./imagenes/despliegue2node.png)<br><br>


**¿Qué pasa después de clonar el repositorio?**
Accedo al directorio clonado `cd despliegue2` e instalo las dependencias del proyecto, las instalo las con `npm install`.<br><br>
Configuro la base de datos con las nuevas credenciales y otros parámetros.<br><br>
![Texto alternativo](./imagenes/rutabasenode.png)<br><br>
![Texto alternativo](./imagenes/basedenode.png)<br><br>

Ejecuto la aplicación con `npm start`<br><br>

**Problema al hacer `npm install`** lo solucioné dando permisos al usuario actual sobre el proyecto, para ello verifque si isard es el dueño o es root:<br><br>
`ls -ld /home/isard/demoapinode`<br><br> 
Como efectivamente es root cambio el ownership y doy permisos:<br><br>
`sudo chown -R $USER:$USER /home/proyectonode/demoapinode`<br><br>
`sudo chmod -R 775 /home/isard/demoapinode`<br><br>

**Pasos que he seguido para importar el archivo SQL del proyecto node en la misma BBDD demo**<br><br>
![Texto alternativo](./imagenes/importarbbdd.png)<br><br>



### NGINX COMO PROXY INVERSO DE NODE Y APACHE

1. Configuración de Nginx como proxy inverso
Creo un archivo para el proxy inverso en el directorio **sites-available** dentro  a su vez del directorio **nginx** y establezo un bloque de servidor para cada dominio, por tanto, edito el archivo de 
configuración de Nginx o creo uno nuevo para manejar los dominios. Hice:<br><br>
`sudo nano /etc/nginx/sites-available/proxyinverso` <br><br>
En el archivo agrego los bloques de cada dominio:<br><br>

![Texto alternativo](./imagenes/confproxyinverso4.png)<br><br>

Lo releventa es **listen** donde pongo un puerto distinto al de apache y node, el ^**80** y luego **server_name** donde pongo el nombre de dominio, y **proxy_pass** donde pongo la url con el puerto de su servidor ya sea el de apache o node.<br><br>

Después debo habilitar la configuración, es decir que aparezca correcto en sites-enabled, para ello
creo un enlace simbólico para habilitar la configuración:<br><br>
`sudo ln -s /etc/nginx/sites-available/proxy-config /etc/nginx/sites-enabled/`<br><br>

![Texto alternativo](./imagenes/enableddelinverso.png)<br><br>

`sudo systemctl restart nginx`<br><br>

Resultados en puerto 80:<br><br>
![Texto alternativo](./imagenes/resultadophp80.png)<br><br>
![Texto alternativo](./imagenes/resultadonode80.png)<br><br>

