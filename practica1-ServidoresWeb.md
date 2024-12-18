
# PRACTICA - SERVIDOR [Servicios de red]

Lorién Borra Cruz

## 1 - Configuración de sitios web

www.web1.com, alojado en /var/www/web1. Con fichero de inicio index.html
![Texto alternativo](./imagenes/web1config.png)<br>
www.web2.com, alojado en /var/www/web2. Con fichero de inicio index.html
![Texto alternativo](./imagenes/web2config.png)<br>
www.web3.com, alojado en /var/www/web3. Con fichero de inicio index.html
![Texto alternativo](./imagenes/web3config.png)<br>
www.web4.com, alojado en /var/www/web4. Con fichero de inicio index.html
![Texto alternativo](./imagenes/web4config.png)<br>
Luego eh editado y puesto el nombre en todas als web con **sudo** y **nano**
![Texto alternativo](./imagenes/nombreenwebs.png)<br>

### - Debes configurar tu fichero hosts en el cliente para poder conectarte a ellos.
Dentro de **/etc/hosts** lso pongo apuntado a al IP del servidor<br>
![Texto alternativo](./imagenes/sudoHosts.png)<br>
![Texto alternativo](./imagenes/dominiosHosts.png)<br>
![Texto alternativo](./imagenes/hostsServer.png)<br>

## 3- Configuración de sitio

### Sitio 1 y 2
![Texto alternativo](./imagenes/terminalconf12.png)<br>
![Texto alternativo](./imagenes/daat1.png)<br>
![Texto alternativo](./imagenes/daat2.png)<br>

Después procedo a activarlos siendo en apache con **a2ensite**.
![Texto alternativo](./imagenes/activacionApache.png)<br>

### Sitio 3 
![Texto alternativo](./imagenes/http1.png)<br>

Abr0 el archivo de configuración del sitio en Apache con **sudo nano /etc/apache2/sites-available/web3.conf**

Añado el siguiente bloque de configuración dentro del archivo para habilitar HTTPS: <VirtualHost *:443> con los correspondientes enlaces del ceritificado.
![Texto alternativo](./imagenes/archibuneno.png)<br>

### Sitio 4

Me situo dentro de la carpeta  **/etc/nginx/sites-available/** y hago `sudo nano web4` y dentro hago la edición de la foto<br>
![Texto alternativo](./imagenes/web4data.png)<br>
Luego procedo a activarla con el comando `sudo ln -s /etc/nginx/sites-available/web4 /etc/nginx/sites-enabled/` seguido de `sudo systemctl reload nginx.service`<br>
![Texto alternativo](./imagenes/activacionweb4.png)<br>


## 4 - Proxy nginx
Situado ya dentro de nginx sites-available creo el archivo y lo edito y activo finalmente.<br>
![Texto alternativo](./imagenes/inversocomando.png)<br>
![Texto alternativo](./imagenes/inversoData.png)<br>
![Texto alternativo](./imagenes/activoproxy.png)<br>


