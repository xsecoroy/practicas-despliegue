#  Práctica 2. DNS

Servicio de DNS

## Objetivos

- Conocer el funcionamiento del servicio DNS
- Conocer los distintos registros asocidados a un dominio DNS.
- Conocer cómo suplantar el DNS de forma local.

## Qué entregar

- Toma el fichero markdown del repositorio del profesor y añádelo a tu repositorio de prácticas
- Añade los comentarios y fotos que expliquen y justifiquen tu trabajo.
- Vamos aprovechar esta práctica para aprender a añadir etiquetas en git:
  - https://git-scm.com/book/es/v2/Fundamentos-de-Git-Etiquetado
  - Definie una etiqueta P1 asocida al commit con el que terminas tu trabajo. Puedes hacerlo de una de estas maneras:
    - `git tag -a P2 <hash del commit>`
    - `git tag -a P2 -m 'Práctica 2 acabada'`
  - Sube tu trabajo y la etiqueta:
    - `git push origin P2`  //para subir la etiqueta
    - `git push origin --tags` //para subir todas las etiquetas
- Respeta la fecha de entrega.

### Herramientas de diagnóstico

- Toma un dominio público de acuerdo a tu número de alumno:
  1. gitlab.com
  **2. bitbucket.com**
  3. android.com
  4. trello.com
  5. atlassian.com
  6. ibercaja.es
  7. facebook.com
  8. twiter.com
  9. bbva.es
  10. instagram.com
  11. wikipedia.com
  12. elpais.com
  13. marca.com
  14. heraldo.es
  15. apple.com

## - Intenta resolver las siguientes preguntas con con las 3 herramientas presentadas: host,nslookup y dig.
> Primero me conecto por ssh a mi servidor con **ssh isard@192.168.2.1**<br><br>
![Texto alternativo](./imagenes/conexion-ssh.png)<br><br>
  - Busca la IP asignada
  > Ejecuto `host bitbucket.com`<br><br>
  > Obtengo 3 siendo estas: **186.166.142.21** | **186.166.142.22** |**186.166.142.23** <br><br>
  >![Texto alternativo](./imagenes/host-bitbucket.png)<br><br>
  > Ejecuto `nslookup bitbucket.com` y obtengo los mismos resultados. <br><br>
  >![Texto alternativo](./imagenes/nslookup-bitbucket.png)<br><br>
  > Ejecuto `dig bitbucket.com` y obtengo los mismos resultados en ***ANSWER SECTION***<br><br>
  >![Texto alternativo](./imagenes/dig-bitbucket.png)<br><br>
  > OJO hoy domingo hice de nuevo la consulta de `host bitbucket.com` y me dio estas tres IP distintas a las anteriores:
  > **186.166.143.48 | 186.166.143.49 | 186.166.143.50**. Según he leido puede haber múltiples razones siendo una de la más comúnes
  > por balanceo de carga, lo que puede conllevar que las IPs puedan cambiar con el tiempo, es decir o por usar CDNs dinámicos o según la carga en la red.<br><br>
  >![Texto alternativo](./imagenes/nuevohost.png)<br><br>

  ## - Quien resuelve su DNS
  > Ejecuto `dig bitbucket.com NS` y obtengo los resultados en **ANSWER SECTION**. <br><br>
  >![Texto alternativo](./imagenes/dig-dns.png)<br><br>
  > Ejecuto `host -t ns bitbucket.com` y lo mismo.<br><br>
  >![Texto alternativo](./imagenes/host-dns.png)<br><br>
  > Ejecuto `nslookup -query=NS bitbucket.com` y obtengo cuatro resultados en todas las consultas, siendo estos quienes resuelven el DNS: **ns-1476.awsdns-56.org.** | **ns-76.awsdns-09.com.** | **ns-1827.awsdns-36.co.uk.** | **ns-657.awsdns.18.net.**<br><br>
  >![Texto alternativo](./imagenes/nslookup-query.png)<br><br>

  >**PROCESO CONSULTA DNS**<br>
  > En base a las respuestas, en especial la última se ve que mi sistema está utilizando un resolutor local de DNS, **127.0.0.53**, para enviar consultas DNS.
  > Este resolutor local no conoce directamente las respuestas para todos los dominios, así que actúa como un intermediario. 
  > Si la respuesta no está en mi caché, el resolutor local reenviará la consulta a servidores DNS externos configurados en mi sistema.
  > Un externo podría ser 8.8.8.8, Los resolutores externos se comunican con los servidores autoritativos de bitbucket.com (ns-1476.awsdns-56.org, etc.)
  > los que me salen en la respuestas de las consultas anteriores, y así obtener la respuesta final. 
  > Una vez obtenida la respuesta esta viaja de los servidores autoritativos → resolutor externo → resolutor local → mi terminal.<br><br>

  ## - Cuál es el servidor de correo electrónico. Si hay varios, determina cual es primero por su prioridad.
  > Ejecuto el siguiente comando de **host** `host -t MX bitbucket.com` y la respuesta que recibo es que no hay no hay record de MX para ese dominio.<br><br>
  >![Texto alternativo](./imagenes/MX.png)<br><br>
  > Con **dig** ejecuto el comando `dig bitbucket.com MX` y lo mismo.<br><br>
  >![Texto alternativo](./imagenes/dig-MX.png)<br><br>
  >![Texto alternativo](./imagenes/diga-bucket.png)<br><br>
  > Recibo answer 0 lo que indica que no hay registros MX para el dominio bitbucket.com, no hay configurado o establecido un servidor para los correos
  > que se envien a bitbucket.com. Al no tener los registros MX en este caso, el servidor de correo intentará usar las IP asociadas al registro A de bitbucket,
  > Por tanto los correos dirigidos o enviado a bitbucket.com se intentarían entregar a uno de esos servidores en las direcciones IP A, los de la segunda foto de dig.<br><br>
  > Con **nslookup** ejecuto el comando `nslookup -query=mx bitbucket.com`<br><br>
  >![Texto alternativo](./imagenes/nslookup-MX.png)<br><br>

  ## - Haz la búsqueda de forma autorizada, es decir, que el servidor que contesta sea uno de los registos NS del dominio.
  > He de usar alguno de los obtenidos en el paso 2, como puede ser **ns-1476.awsdns-56.org.**
  > Ejecuto el siguiente comando de **dig** con esa direccion `dig <servidorDNS> dominio A` por tanto `dig ns-1476.awsdns-56.org. bitbucket.com A` <br><br>
  >![Texto alternativo](./imagenes/lookupautorizado1.png)<br><br>
   >![Texto alternativo](./imagenes/lookupautorizado2.png)<br><br>
  > Con **lookup** ejecuto el comando `nslookup bitbucket.com ns-1476.awsdns-56.org`<br><br>
  >![Texto alternativo](./imagenes/lookupautorizado3.png)<br><br>

## Suplantar servicio DNS localmente.

> El archivo **/etc/hosts** es una forma de configuración local del sistema que me permite asignar nombres de dominio a direcciones IP 
> sin utilizar servidores DNS externos.<br><br>
> 

## - Edita el fichero `/etc/hosts` para que resuelva un nombre de dominio falso con el siguiente esquema: 
  - `miapellido.local` asociado a la dirección 127.0.0.1
  - `miapellido.es` asociado a una dirección pública inventada.
  - `miapellido.com` asociado a la dirección del dominio de la primera parte de la práctica. Si eres el nº 1 sería `github.com`
  > ![Texto alternativo](./imagenes/suplantar_dns.png)<br><br>
## - Comprueba la resolución de los tres registros con alguna de las herramientas de diagnóstico.
> **host**<br><br>
> ![Texto alternativo](./imagenes/suplantar_dns2.png)<br><br>
> **nslookup**<br><br>
> ![Texto alternativo](./imagenes/suplantar_dns3.png)<br><br>
> **dig**<br><br>
> ![Texto alternativo](./imagenes/suplantar_dns5.png)<br><br>
## - Comprueba la misma resolución pero haciendo que el servidor consultado sea el 8.8.8.8<br><br>
> **nslookup**<br><br>
> ![Texto alternativo](./imagenes/suplantar_dns4.png)<br><br>
> **dig**<br><br>
> ![Texto alternativo](./imagenes/diges.png)<br><br>
> **dig**<br><br>
> ![Texto alternativo](./imagenes/diglocal.png)<br><br>

> A considerar, en .es y .local aparece **status: NXDOMAIN** lo cual indica que el dominio no existe en los servidores DNS de Google,
> ni en ningún DNS público,y es que el servidor DNS de Google no tiene acceso a mi archivo local **/etc/hosts**, por lo que no sabe nada 
> sobre las configuraciones locales que he hecho en mi máquina.<br><br>
> Además aparece **ANSWER: 0:** lo que indica que no hay registros A, ya que el dominio no existe, Como esos dominios no están registrados 
> en Internet los servidores DNS de Google no tienen información sobre ellos.<br><br>
> Por último **AUTHORITY SECTION:** Muestra información del servidor autoritativo para el TLD (.es). Aquí el servidor DNS me está indicando quién 
> es el encargado de administrar los dominios .es, el cual confirma que el dominio no está registrado.
> **La razón es que el comando dig @8.8.8.8 está consultando un servidor DNS externo que no tiene información sobre mi configuración local en /etc/hosts**<br><br>
> ![Texto alternativo](./imagenes/digcom.png)<br><br>
> Aqui en cambio **status: NOERROR** Indica que el dominio si existe,***borra.com tiene la IP de bitbucket.com*** y que el servidor DNS tiene registros asociados para él.
> De hecho en **ANSWER SECTION:** Ahora si muestra los registros de tipo A (direcciones IPv4) asociados al dominio, tiene dos registros A.
> Por tanto la diferencia a probar con este ejericio fue:<br><br>
> **Consultas con dig o nslookup a servidores DNS externos**: Estos servidores resolverán los dominios basándose en registros públicos de Internet. Si el dominio no está registrado públicamente, me devolverán un error como NXDOMAIN.<br><br>
> **Consultas localmente (sin @8.8.8.8)**: Si hago la consulta sin especificar un servidor DNS (solo dig borra.local o dig borra.com), mi sistema utilizará el archivo /etc/hosts, por lo que resolverá las direcciones según lo que configuraste localmente.<br><br>

## Configuración del servidor  DNS con BIND9

## - Configura el dominio `miapellido.edu` en bind9. Debes hacerlo:
  - En el servidor linux
  - Conectándote al mismo mediante SSH
  - Utiliza las direcciones IP de tu red privada (192.168.1xx.0/24)
  > Compruebo que **bind9** este activo.<br><br>
  >![Texto alternativo](./imagenes/status_dib9.png)<br><br>
  > He de configurar BIND9 desde **/etc/bind/** en concreto `sudo nano /etc/bind/named.conf.local` y lo configuro con mis datos.
  > El archivo **/etc/bind/named.conf.local** es donde puedo definir zonas de DNS locales y es el servidor BIND quien las gestiona.
  > Cuando creo una **zone** como ***borra.edu*** en ese archivo, BIND gestionará  la zona para el dominio ***borra.edu*** y como además
  > establezo en la zone **file "/etc/bind/db.borra.edu"** con esto le indico a BIND cómo resolver las consultas para ese dominio y qué 
  > archivo de zona debe usar para obtener la información.
  > Por tanto, El archivo **named.conf.local** simplemente le dice a BIND qué archivo de zona utilizar para gestionar las consultas de **borra.edu**. 
  > Luego, BIND consulta **/etc/bind/db.borra.edu** para obtener la información.

  >Proceso de resolución de consultas:
  > - 1 - Cuando quiero acceder a borra.edu, el servidor de nombres BIND consultará su archivo de configuración (el que contiene la zone borra.edu).
  > - 2 - Como he configurado la zona para borra.edu en **named.conf.local**, BIND va a buscar la información correspondiente en el archivo de zona **/etc/bind/db.borra.edu**.
  > - 3 - Si el archivo de zona contiene un registro A para www.borra.edu, así lo he hecho, BIND responderá con la dirección IP configurada en ese archivo.<br><br>
  >![Texto alternativo](./imagenes/named.conf.png)<br><br>
  - Deberás usar un fichero con un nombre similar a este: *db.miapellido.edu*
  >![Texto alternativo](./imagenes/fichodns.png)<br><br>
  
## - Se debe definir:
  - Número de versión 1
  - Un correo de administrador dentro del dominio.
  - Dos registros MX.
  - Dos registros NS con máquinas del propio dominio (por ejemplo *dns1* y *dns2*).
  - Varios registros A, al menos: *www, dns1, dns2*
  - Un registro CNAME asociado a la misma dirección que *www*
   >Para ello accedo al archivo con `sudo nano /etc/bind/db.borra.edu` y lo edito con el resultado de la segunda foto:<br><br>
   >![Texto alternativo](./imagenes/fichodns.png)<br><br>
   >![Texto alternativo](./imagenes/db.borra.edu.png)<br><br>
   > La primera línea de **SOA** establece el registro SOA el cual indica que este archivo de zona es la autoridad para el dominio borra.edu, es la cabecera
   > que declara quien controla la configuración de ese dominio de zona. He de establecer un servidor DNS principal y el correo 
   > electronico del administrador del dominio.
   > Ojo he declarado cual sera el servidor NDS principal, pero he de configurarlo si o si luego en el archivo, por tanto si he declarado **dns1.borra.edu.** Como
   > principal implica que debo decalrar un dns llamado **dns1** (mismo nombre quitando el dominio de zona). Le asigno obviamente una IP que será ese servidor
   > el que gestionará las peticiones **dns1    IN  A   192.168.2.1** el servidor ubuntu, es decir la IP 192.168.2.1 es una máquina configurada como servidor DNS y así
   > cualquier cliente o servidor que intente contactar con **dns1.borra.edu.** sabrá a qué IP conectarse, lo que hará que las consultas no fallen y responderá el servidor
   > las peticiones.
   > El símbolo **@** en el archivo de zona es un alias que se utiliza en los archivos de zona de BIND para hacer referencia al dominio principal de la zona, 
   > que en mi caso es **borra.edu**. Es una forma abreviada de referirse a la raíz del dominio, sin tener que escribir borra.edu.
   >**A**: Apunta directamente un nombre de dominio a una dirección IP, asocia un nombre de dominio a una dirección IP en formato IPv4. Por tanto, cuando se
   > acceda a **borra.edu*+ el dominio tal cual definido para la zona se resolverá a 10.1.1.1, que es la IP que has asignado en el registro A.
   >**www**: En cambio aqui defino que cuando el dominio de la zona, vaya precedido de **www** es decir, **www.borra.edu** al establecer **www  IN  A   10.1.1.1**
   > configuro que www.borra.edu debe resolver a la dirección IP 10.1.1.1, pero podría poner otra distinta.
   >**CNAME**:Apunta un nombre de dominio a otro dominio. Lo utilizo para que los subdominios  que defina apunten al mismo lugar, con CNAME esos subdominios "apuntan" al dominio principal.
   > He configurado 2 subdominio, uno es **info** y lo he configurado en un registro CNAME así **info   IN  CNAME   www.borra.edu.**, logrando ahora que apunte a **www.borra.edu** , por tanto,
   > cuando se acceda a **info.borra.edu** el servidor DNS va a buscar **www.borra.edu**, porque es lo que se ha definido en el registro CNAME. A su vez,  www.borra.edu tiene un registro A apuntando a 10.1.1.1, 
   > y por tanto la resolución de info.borra.edu también devolverá **10.1.1.1**.
   > En definitiva apunta un nombre de dominio a otro dominio y ese segundo dominio puede tener un registro A que apunte a una IP como este caso.
## - Comprobar que la configuración es correcta :
  - Usar el comando de verificación sintáctica.
  > Ejecuto estos 2 comandos `sudo named-checkconf` y `sudo named-checkzone borra.edu /etc/bind/db.borra.edu`.<br><br>
  >![Texto alternativo](./imagenes/verificacion-sintactica.png)<br><br>
  - Reiniciar el servicio
  > Lo reinicio y lo vuelvo a comprobar otra vez.<br><br>
  >![Texto alternativo](./imagenes/verificacion-db.png)<br><br>
  - Probar la resolución usando nslookup y dig. Ojo, la resolución de nombres debe correr a cargo del servidor Ubuntu.
  > Dig: www.borra.edu<br><br>
  >![Texto alternativo](./imagenes/verificacion-dig1.png)<br><br>
  > Dig: info.borra.edu<br><br>
  >![Texto alternativo](./imagenes/verificacion-dig2.png)<br><br>
  > Dig: borra.edu<br><br>
  >![Texto alternativo](./imagenes/verificacion-dig3.png)<br><br>

  > Nslookup: www.borra.edu tanto local como desde el server.<br><br>
  >![Texto alternativo](./imagenes/verificacion-ns1.png)<br><br>
  >![Texto alternativo](./imagenes/ns-server1.png)<br><br>
  > Nslookup: info.borra.edu tanto local como desde el server.<br><br>
  >![Texto alternativo](./imagenes/verificacion-ns2.png)<br><br>
  >![Texto alternativo](./imagenes/ns-server2.png)<br><br>
  > Nslookup: borra.edu tanto local como desde el server.
  >![Texto alternativo](./imagenes/verificacion-ns3.png)<br><br>
  >![Texto alternativo](./imagenes/ns-server3.png)<br><br>
## - Trabajo extra. Para mejorar tu calificación se te propone:
  - Añadir una zona de resolución inversa
  > La red que he usado en el ejercico es la red 192.168.2.0/24, la ya he configurado en la práctica 1, soy el número 2 de la lista.
  > Por tanto, el dominio y en consecuencia la zone será **2.168.192.in-addr.arpa**
  > Para ello edito en `sudo nano /etc/bind/named.conf.local`<br><br>
  >![Texto alternativo](./imagenes/zona-inversa.png)<br><br>
  > Después creo el archivo de zona de para esta resolución inversa.<br><br>
  >![Texto alternativo](./imagenes/cp-inversa.png)<br><br>
  > Después procedo a editar el archivo.<br><br>
  >![Texto alternativo](./imagenes/cp-inversa.png)<br><br>
  >![Texto alternativo](./imagenes/inversa-db.png)<br><br>
  > Hago las comprobaciones y reinicio y compruebo<br><br98>
  >![Texto alternativo](./imagenes/prueba-inversa.png)<br><br>
  >![Texto alternativo](./imagenes/dig-inverso1.png)<br><br>
  >![Texto alternativo](./imagenes/dig-inverso2.png)<br><br>

  - Instalar Bind9 en el cliente Ubuntu y definirlo como DNS esclavo del primero.
  > Primero ejecuto los comandos `sudo apt update` y luego `sudo apt isntall bind9 -y`<br><br>
  > Accedo al archivo de **named.conf.local** del cliente ubuntu, esclavo, con el comando
  > `sudo nano /etc/bind/named.conf.local` y lo configuro para la misma zona, pero el
  > rol ahora es de esclavo, **type slave** además indico la IP del maestro y el fichero donde guardar la zona.<br><br>
  >![Texto alternativo](./imagenes/edicion-esclavo.png)<br><br>
  > Después accedo a mi ubuntu server, el servidor maestro, de nuevo a `sudo nano /etc/bind/named.conf.local` y lo configure añadiendo estas dos lineas  **allow-transfer { 192.168.2.2; };** y  
  **also-notify { 192.168.2.2; };** a explicar: ***La direccion IP que se encuentra en los corchetes es la del esclavo*** <br><br>
  >![Texto alternativo](./imagenes/maestro-edicion.png)<br><br>
  >Después reinicio en ambos con `sudo systemctl restart bind9` y desde el esclavo hago una consulta a  la zona esclava `dig @127.0.0.1 -x 192.168.2.1` <br><br>
  >![Texto alternativo](./imagenes/consulta-esclava.png)<br><br>
  > Finalmente en mi cliente edito **sudo nano /etc/resolv.conf** y establezco **nameserver 127.0.0.1** y hago las prueba de resolución inversa desde el esclavo<br><br>.
  >![Texto alternativo](./imagenes/prueba-inversa1.png)<br><br>
  >![Texto alternativo](./imagenes/prueba-inversa2.png)<br><br>

  IMPORTANTE: HE USADO LA RED INTERNA EN LA TODA PRÁCTICA2  QUE CONFIGURAMOS EN LA PRACTICA 1, SIENDO LA IP DEL SERVIDOR 192.168.2.1 Y LA DEL CLIENTE UBUNTU 192.168.2.2


