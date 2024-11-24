# APUNTES REDES

## 1 - Lógica creación
Al configurar la dirección IP de mi tarjeta de red enp3s0 en el servidor ubuntu con IP 192.168.2.1, lo que hago es "definir" implícitamente que ese servidor está en la subred 192.168.2.0/24. <br>

No es que tenga que crear la red de manera explícita, sino que, al asignar una dirección IP dentro de ese rango, ahora mi ubuntu server si forma parte de esa subred, me conviertes en parte de esa red local (LAN) y automáticamente estás utilizando las direcciones dentro del rango de 192.168.2.0 a 192.168.2.254.

**La subred 192.168.2.0/24**:

Al asignar la IP 192.168.2.1 al servidor, también estoy estableciendo que todos los dispositivos que tengan direcciones dentro de este rango (como 192.168.2.2, 192.168.2.3, etc.) podrán comunicarse entre sí localmente.

La subred 192.168.2.0/24 está implícita y no necesito declararla explícitamente. Cuando asigno una dirección IP dentro de ese rango, todos los dispositivos que tengan direcciones dentro de este rango también están dentro de la misma red y pueden comunicarse entre sí sin necesidad de un router (si no necesitas salir a internet).

Mi servidor no es el "dueño" de la red. Simplemente está configurado para estar dentro de ella con una IP estática. En una red local, todos los dispositivos con direcciones IP dentro de la misma subred pueden comunicarse entre sí, independientemente de cuál sea el "servidor" o el "cliente". Sin embargo, si tiengo un dispositivo configurado como puerta de enlace (gateway), este dispositivo podría facilitar el acceso a internet o a otras redes.

La puerta de enlace (gateway): Es la dirección IP del dispositivo (generalmente un router) que conecta la red local a otras redes, como internet, como no queremos que se conecte a 
otras redes no lo establecemos. Misma lógica con el DNS no establecemos addresses de servidores DNS en nameservers.

**Red con direccione IP estaticas**
`dhcp4: false`
dhcp4: false desactiva el protocolo DHCP (Dynamic Host Configuration Protocol) para IPv4.
Esto significa que la interfaz no obtendrá automáticamente una dirección IP ni la configuración de red desde un servidor DHCP (como un router).
Por lo tanto, la dirección IP, la máscara de red, la puerta de enlace y los servidores DNS deben configurarse manualmente.

Por tanto, se debe configurar, así:
addresses:
  - 10.10.10.2/24

**addresses:**
Es una clave que define las direcciones IP estáticas que se asignarán manualmente a la interfaz de red.
Puede incluir una o más direcciones IP.

**10.10.10.2** o en mi caso **192.168.2.1**:
Es la dirección IP estática asignada a la interfaz. Esta dirección identifica al dispositivo en la red local.

**/24**:
Es la longitud del prefijo de la máscara de subred. Este prefijo indica cuántos bits de la dirección IP pertenecen a la red.
En este caso, /24 equivale a la máscara de subred 255.255.255.0. Esto significa que los primeros 24 bits de la dirección (los tres primeros octetos) definen la red, y los últimos 8 bits se usan para identificar dispositivos dentro de la red. En mi caso,
la red sería 192.168.2.0/24, con un rango de direcciones de 192.168.2.1 a 192.168.2.254.

## DHCP

Un servidor DHCP es responsable de asignar direcciones IP de manera automática a los dispositivos que se conectan a la red. El servidor no solo asigna una dirección IP, 
sino que también proporciona otras configuraciones necesarias para que los dispositivos funcionen correctamente, como la puerta de enlace predeterminada (gateway) y los servidores DNS.

Al configurar un servidor DHCP, hay que  definiendo varias cosas clave:

- Subnet: Esto define la red, en mi caso fué 192.168.102.0 . El servidor DHCP asignará direcciones dentro de esta red.

- Rango de direcciones IP (range): El servidor DHCP asignará direcciones dentro de este rango a los dispositivos que se conecten. Por ejemplo, un rango de direcciones entre  100 y 200 sería -> 192.168.102.100 y 192.168.102.200 para que los clientes reciban una IP dentro de ese rango.

- Máscara de subred (netmask): Especifico la máscara de subred que se aplica a la red. Por ejemplo una máscara de subred 255.255.255.0, lo que significa que las direcciones IP de la red tienen los primeros tres octetos iguales (por ejemplo, 192.168.102.x) y la x para clientes.

- Puerta de enlace predeterminada (gateway): Esta es la dirección IP del dispositivo que los clientes usarán para acceder a redes fuera de la red local, es decir, el router o gateway. Por ejemplo, si el router tiene la dirección 192.168.102.1, esa sería la puerta de enlace 
predeterminada que se proporciona a los clientes.

- Servidores DNS: Opcionalmente, se puede especificar los servidores DNS que los clientes usarán para resolver nombres de dominio.

¿Qué pasa cuando un cliente se conecta a la red?

Cuando un cliente se conecta a la red dchp y tiene configurado el DHCP (dhcp4: true en su archivo de configuración), el cliente envía una solicitud DHCP para obtener una dirección IP.
El servidor DHCP (configurado en mi servidor Ubuntu) recibe esta solicitud y asigna una IP disponible dentro del rango especificado (en este caso, entre 192.168.102.100 y 192.168.102.200).
El servidor también envía información adicional al cliente, como la puerta de enlace predeterminada (192.168.102.1) y los servidores DNS.
El cliente ahora puede usar esta IP para comunicarse dentro de la red local y, si necesita, acceder a Internet a través de la puerta de enlace (192.168.102.1).
