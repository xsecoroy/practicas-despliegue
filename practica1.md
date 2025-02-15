
# PRACTICA - SERVIDOR [Servicios de red]

Lorién Borra Cruz

## 1 - Cambiar el nombre de los equipos

### 1. - Cambio del nombre del servidor

Accedo con el comando **sudo nano** para editar el archivo `/etc/hostname` y pongo mi número de lista que es el 2 -> **02**, después reinicio el server con `sudo reboot`.

Fotos tomadas después del cambio, mostrando los pasos y resultado.<br><br>
![Texto alternativo](./imagenes/cambionombreserver1.png)<br><br>
![Texto alternativo](./imagenes/cambionombreserver2.png)<br><br>
![Texto alternativo](./imagenes/cambionombreserver3.png)<br><br>
![Texto alternativo](./imagenes/cambionombreserver4.png)<br>

### 1.2 - Cambio del nombre del cliente ubuntu

Ejecuto el comando de `hostnamectl set-hostname` e indico el nuevo nombre, después compruebo los cambios ejecutando tan solo `hostnamectl` y veo en ***static hostname*** como el nombre ha cambiado, o bien ejecuto `hostname` y me lo indica.

Fotos tomadas después del cambio, mostrando los pasos.<br><br>
![Texto alternativo](./imagenes/cambionombreubunt1.png)<br><br>
![Texto alternativo](./imagenes/cambionombreubunt2.png)<br>

### 1.3 - Cambio de nombre del cliente windows

Dentro de configuración de sistema, en el apartado ***acerca de*** le doy al botón de 
***cambiar el nombre de este equipo*** e indico el nombre **windows02**.

Fotos tomadas después del cambio, mostrando los pasos.<br><br>
![Texto alternativo](./imagenes/cambiarnombrewindows1.png)<br>

## 2 - Congifuración de la tarjeta de red   

### 2.1 - Configuración del ubuntu server

Para configurar la tarjeta de red debo dirijirme a `/etc/netplan/01-netcfg.yaml` y como voy a editarlo lo hare con **nano** y con **sudo**.<br>
![Texto alternativo](./imagenes/configuracionubuntuserver1.png)<br>

Configuro la tercera tarjeta de red **enp3s0** para que sea estática y escribo el address que corresponde a mi número, el 02, **192.168.2.1/24**.<br><br>
![Texto alternativo](./imagenes/configuracionubuntuserver2.png)<br>

Tras aplicar con `netplan apply` veo como los cambios se han actualizado con `ip a`.<br><br>
![Texto alternativo](./imagenes/configuracionubuntuserver3.png)<br>

### 2.2 - Configuración del cliente ubuntu

Accedo a la configuración de red avanzada, ahí selecciono la tercera tarjeta de red, y en ajustes de IPV4 selecciono el método a ***Manual*** y en dirección ahora le asigno **192.168.2.2** de forma que el cliente ubuntu es el nuevo host 2 y en máscara de red **24**. <br><br>
![Texto alternativo](./imagenes/configuracionclienteubuntu1.png)<br><br>
![Texto alternativo](./imagenes/configuracionclienteubuntu2.png)<br><br>
![Texto alternativo](./imagenes/configuracionclienteubuntu3.png)<br>

Para confirmar los cambios ejecuto el comando `ip a` y veo como la IP ha cambiado en la tarjeta de red **enp3s0**.<br><br>
![Texto alternativo](./imagenes/configuracionclienteubuntu4.png)<br>

Con ambas maquinas en la red, me queda confirmar la comunicacion entre ambas haciendo ping al servidor **192.168.2.1** y compruebo si las respuestas del servidor son exitosas.<br><br>
![Texto alternativo](./imagenes/configuracionclienteubuntu5.png)<br>

### 2.3 - Configuración del cliente windows

En windows dentro de ***Configuración de red e internet*** selecciono ***Cambiar opciones de adaptador***, y a su vez dentro selecciono la tercera tarjeta de red.<br><br>
![Texto alternativo](./imagenes/configuracionclientewindows1.png)<br><br>
![Texto alternativo](./imagenes/configuracionclientewindows2.png)<br>

Al seleccionarla accedemos a ***propiedades - protocolo de internet versión 4***, y configuro de nuevo la dirección ip de este host que será el **3** de forma que pongo **192.168.2.3** en máscara de red **255.255.255.0** que es lo mismo que haber puesto 24 en ubuntu.<br><br>
![Texto alternativo](./imagenes/configuracionclientewindows3.png)<br><br>
![Texto alternativo](./imagenes/configuracionclientewindows4.png)<br>

Una vez configurado y reiniciado hago las pruebas con `ipconfig` desde el terminal y veo el correcto cambio de de la IP.
Luego pruebo la correcta comunicación haciendo `ping a 192.168.2.1``.<br><br>
![Texto alternativo](./imagenes/configuracionclientewindows5.png)<br><br>
![Texto alternativo](./imagenes/configuracionclientewindows6.png)<br>

## 3 - Configuración DHCP en ubuntu server y pruebas clientes

### Configurando DHCP en el ubuntu server
Para la instalación del servidor isc-dhcp-server hay que ejecutar este comando `apt-get install isc-dhcp-server`
> Cuando instalamos el servidor por primera se produce un error, ya que no está configurado. Para ver los errores producidos por el servidor hay que acceder fichero **/var/log/syslog** con el comando `less /var/log/syslog` o `tail /var/log/syslog`<br><br>

Dado que tengo instalado el servicio DHCP me dirijo directamente a su configuración en el archivo `/etc/dhcp/dhcpd.conf` el cual es el fichero principal de configuración de DHCP.
Dentro del archivo hago la configuración de red y su rango, al ser yo 2, la subnet indico **192.168.102.0** y en el rango del
**192.168.102.100** al **192.168.102.200**, y especifico la puerta de enlace predeterminada con ***option routers***.<br><br>
![Texto alternativo](./imagenes/configdhcpserver1.png)<br><br>
![Texto alternativo](./imagenes/configdhcpserver2.png)<br>

Estoy definiendo los parámetros necesarios para que los dispositivos conectados a la red reciban una dirección IP y otros datos automáticamente.
1. **subnet 192.168.2.0 netmask 255.255.255.0**
***subnet***: Define una red específica para la cual el servidor DHCP asignará direcciones IP.
***192.168.2.0***: Es la dirección de la subred (en este caso, la red 192.168.2.x).
***netmask 255.255.255.0***: Define la máscara de subred, que indica el rango de direcciones IP que pertenecen a esta subred. Con esta máscara (/24), las direcciones válidas van de 192.168.2.1 a 192.168.2.254<br><br>
2. **range 192.168.0.60 192.168.0.90**
***range***: Especifica el rango de direcciones IP que el servidor DHCP puede asignar dinámicamente a los dispositivos.
***192.168.2.60 192.168.2.90***: Indica que las direcciones asignables están entre 192.168.2.60 y 192.168.2.90.<br><br>
3. **option routers 192.168.2.254**
**option routers**: Especifica la dirección IP del gateway predeterminado para la red.
**192.168.2.254**: Esta es la dirección IP del router o dispositivo que conecta esta subred a otras redes (por ejemplo, Internet).<br><br>
4. **option domain-name-server 80.58.0.33, 80.58.32.9**
***option domain-name-server***: Define las direcciones IP de los servidores DNS que los dispositivos deben usar.
***80.58.0.33 y 80.58.32.9***: Son los servidores DNS que se proporcionarán a los dispositivos. Estos pueden ser, por ejemplo, servidores DNS públicos proporcionados por un proveedor de servicios de Internet (ISP).<br><br>

Después tengo que configurar `/etc/netplan/00-installer-config.yaml` en concreto voy a configurar la 4 tarjeta de red.
La configuro para garantizar, dado que tengo varias tarjetas, que se sea estatica y por tanto enp4s0 tiene la IP estática **192.168.102.1/24**, sirviendo como la puerta de enlace para el resto de clientes DHCP que se quieran conectar a esa red.
Además pongo `dhc4p0: false` pues será el servidor que he configurado quien asiganará las IP según el rango delimitado y además la IP fija sirve para que los clientes sepan donde conectarse, esa puerta de enlace que nombraba antes.<br><br>
![Texto alternativo](./imagenes/configdhcpserver3.png)<br><br>
![Texto alternativo](./imagenes/configdhcpserver6.png)<br><br>
Explicación: El servidor Ubuntu que he configurado para ofrecer el servicio DHCP es el responsable de asignar direcciones IP a los clientes en la red. Por tanto, la tarjeta de red que actúa como servidor DHCP (en este caso, la interfaz enp4s0) no debe estar configurada para recibir una IP dinámica. Debe tener una IP estática, ya que esta interfaz se usa para gestionar y distribuir las direcciones IP a otros dispositivos (clientes) en la red. Le asigne una IP estática porque esa interfaz es la que va a actuar como servidor DHCP, distribuyendo direcciones IP a los demás dispositivos de la red.

Lo siguiente será acceder a `/etc/default/isc-dhcp-server` y en **INTERFACESV4** poner el nombre de la tarjeta de red configurada, **dhc4p0**.<br><br>, es decir, configurar el interfaz de red por el que va a trabajar el servidor , en mi caso **enp4s0**.
![Texto alternativo](./imagenes/configdhcpserver4.png)<br><br>
![Texto alternativo](./imagenes/configdhcpserver5.png)<br>

Después procedo a reiniciar DHCP con `sudo systemctl restart isc-dhcp-server` o `service isc-dhcp-server restart`y con `ip a` ver la correcta asignación de la ip a la tarjeta de red. <br><br>
![Texto alternativo](./imagenes/configdhcpserver7.png)<br><br>
![Texto alternativo](./imagenes/configdhcpserver8.png)<br>

Después compruebo que está ejecutandose correctamente con `sudo systemctl status isc-dhcp-server` y que el puerto 67 está en espera.<br><br>
![Texto alternativo](./imagenes/escuchadhcp.png)<br><br>
![Texto alternativo](./imagenes/dhcppuerto67.png)<br><br>

Comando importantes:
> `service isc-dhcp-server stop`
> `service isc-dhcp-server start`
> `service isc-dhcp-server status`<br><br>

### Configurando DHCP en el cliente ubuntu

Peparo la tarjeta de red en el cliente ubuntu, de forma que el método de asignar ip sea Automático DHCP.
Comprueba la ip con `ip a show enp4s0` o bien `ip a` para ver todas, compruebo que se ha asignado la **103**, mientras que a windows fue la **102**.
Por último prueba la conexión con ping a **192.168.102.1**.<br><br>
![Texto alternativo](./imagenes/dhcpubuntu1.png)<br>
![Texto alternativo](./imagenes/dhcpubuntu2.png)<br>
![Texto alternativo](./imagenes/dhcpubuntu3.png)<br><br>

Para **liberar la conexión** ejecuto el comando `sudo dhclient -r` liberar la concesión esto significa decirle al cliente que "devuelva" la dirección IP que le asignó el servidor DHCP. El cliente deja de usar esa IP, permitiendo que el servidor DHCP la reasigne a otro dispositivo si es necesario. El comando para **renovar la conexión** es `sudo dhclient` y renovar la concesión significa pedir al servidor DHCP que asigne una nueva dirección IP o renueve la que el cliente ya tenía.<br><br>
![Texto alternativo](./imagenes/release-ubuntu.png)<br><br>

### Configurando DHCP en el cliente windows

Preparo la cuarta tarjeta de red de windows y compruebo la dirección obtenida y la conexión mediante ping, esta correcto se asgino la **102**.<br><br>
![Texto alternativo](./imagenes/pingdhcpwindows.png)<br><br>
![Texto alternativo](./imagenes/pingdhcpwindows2.png)<br><br>
![Texto alternativo](./imagenes/pingdgcpwindows3.png)<br><br>

En Windows la instrucción `ipconfig /release` **libera la concesión**, la instrucción `ipconfig /renew` la **renueva**.<br><br>

Verifico las leases o préstamos del servidor, es decir, la lista de direcciones prestadas con `dhcp-lease-list` y `/var/lib/dhcp/dhcpd.leases` .<br><br>
![Texto alternativo](./imagenes/dhcpleasing1.png)<br>
![Texto alternativo](./imagenes/dhcpleasing2.png)<br><br>


Configuro una reserva de IP fija en el servidor DHCP, la dirección 50, reinicio el servicio, y renuevo la IP en windows. 
Para ello primero copio la mac desde la terminal en windows y es la que añado en el ubuntu server en **hardware ethernet** y modifico el **fixed-address** otorgando **50**, quedando **192.168.102.50** todo ello editado en `/etc/dhcp/dhcpd.conf` donde host es  el nombre que identifica al host, el que quiera.Una vez configurado después verifico que no ha problemas.<br><br>
![Texto alternativo](./imagenes/ip501.png)<br>
![Texto alternativo](./imagenes/ip503.png)<br>
![Texto alternativo](./imagenes/ip504.png)<br>

Por último compruebo la ip en windows con ipconfig en la última tarjeta de red la ip haciendo eso si, primero /release y luego /renew.<br><br>
![Texto alternativo](./imagenes/release.png)<br>
![Texto alternativo](./imagenes/renew.png)<br>
![Texto alternativo](./imagenes/ip505.png)<br>

Por último al apagarlo sigue manteniendola.<br><br>
![Texto alternativo](./imagenes/renovadhcp1.png)<br>
![Texto alternativo](./imagenes/renovadhcp2.png)<br>








  





