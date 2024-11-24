# APUNTES DNS

## 1 - Proceso DNS

### 1.1 - El navegador revisa su caché local:

Cuando escribo un nombre de dominio como google.com, para acceder a el desde el navegador, este, el navegador primero revisa si ya tiene la dirección IP correspondiente en su caché. Esto ocurrirá claro si ya visité ese sitio recientemente.

### 1.2 - El sistema operativo revisa el archivo /etc/hosts o el equivalente en Windows:
Antes de recurrir al DNS, el sistema revisa configuraciones locales como /etc/hosts. Si ahí hay una entrada para el dominio (google.com), se usa la dirección configurada, sin consultar servidores externos.

### 1.3 - Consulta al servidor DNS configurado, mi caso (Movistar):
Si el navegador y el sistema operativo no tienen una respuesta, el sistema operativo envía una consulta al servidor DNS configurado en tu conexión a Internet. Generalmente, este servidor DNS es proporcionado por mi ISP (en mi caso, Movistar), a menos que lo se cambie manualmente (por ejemplo, usando 8.8.8.8 de Google DNS).

Servidor DNS de Movistar: Este es un resolutor DNS local que mantiene una caché de respuestas previas para acelerar la resolución de nombres.

### 1.4 -El servidor DNS de Movistar revisa su caché:
Si Movistar ya tiene la dirección IP correspondiente al dominio en su caché (porque otro usuario de Movistar consultó ese dominio recientemente), me devuelve la respuesta inmediatamente.

### 1.5 - Recursión si no tiene la respuesta:
Si el servidor DNS de Movistar no tiene la respuesta en su caché, comienza un proceso recursivo para encontrar la dirección IP:

Consulta a un servidor raíz (Root Server): El servidor raíz no tiene la IP del dominio, pero redirige la consulta al servidor de nombres autoritativo correspondiente al TLD (como .com, .es).
Consulta al servidor TLD: Por ejemplo, para google.com, el servidor TLD del .com te redirige al servidor autoritativo específico de google.com.
Consulta al servidor autoritativo: Este servidor finalmente responde con la dirección IP correspondiente al dominio.

### 1.6 - Respuesta al cliente:
Una vez que el servidor DNS de Movistar obtiene la dirección IP, la guarda temporalmente en su caché para responder más rápido en futuras consultas similares y me la devuelve. Ahora mi navegador puede usar esa IP para conectarse al servidor web correspondiente.


## 2 - ROOT SERVER - ¿Qué es un servidor raiz?

Los servidores raíz son el nivel más alto en la jerarquía del sistema DNS, y su función principal es actuar como un punto de referencia para todas las consultas DNS. Los servidores raíz no dependen de la extensión del dominio (.com, .org, .info, etc.), sino que tienen información sobre dónde encontrar los servidores TLD (Top-Level Domain) que gestionan las distintas extensiones.

Hay 13 servidores raíz principales en el mundo, identificados por letras (A a M). No son "13 máquinas individuales", sino redes globales distribuidas de servidores que comparten estas funciones. Por ejemplo:

Todos los servidores raíz tienen la misma información básica sobre cómo localizar los servidores TLD para las extensiones de dominio (.com, .es, .info, etc.).

## 3 - Explicación detallada del apartado  1.5 -> Cuando un servidor DNS (como el de Movistar) no tiene la respuesta en su caché.

### 3.1 - Consulta al servidor raíz (Root Server), Un gran índice o directorio:

El servidor raíz actúa como un índice o directorio global. Su función principal es redirigir las consultas al conjunto de servidores TLD correspondientes a la extensión del dominio (como .com, .es, .info, etc.). El servidor raíz no sabe directamente la dirección IP del dominio (por ejemplo, google.com).
Sin embargo, sabe dónde están los servidores TLD (Top-Level Domain) que gestionan cada extensión de dominio:

Para .com, el servidor raíz no sabe la dirección IP de google.com, pero te redirige al servidor TLD del .com (como a.gtld-servers.net).
Para .es, te redirigen a los servidores TLD del .es.
Para .info, te redirigen a los servidores TLD del .info.

La dirección de los servidores TLD está codificada en los servidores raíz y es independiente del dominio específico que estás consultando.

### 3.2 - Consulta al servidor TLD, Una carpeta de dominios con la misma extensión.

Los servidores TLD funcionan como una carpeta que contiene y  gestiona información sobre todos los dominios con una extensión específica.  Por ejemplo:

Los servidores TLD del .com (como a.gtld-servers.net) tienen un listado de todos los dominios .com y los gestionan.
Los servidores TLD del .es (como a.nic.es) gestionan todos los dominios .es.
Los servidores TLD del .info (como a.info-servers.net) gestionan todos los dominios .info.

El servidor TLD tampoco sabe la dirección IP del dominio específico (por ejemplo, google.com), pero sabe quién es el servidor de nombres autoritativo (Authoritative DNS Server) que gestiona ese dominio.

### 3.3 Consulta al servidor autoritativo:

El servidor autoritativo es el que finalmente conoce la dirección IP asociada al dominio específico (por ejemplo, google.com o bitbucket.com), el servidor autoritativo es el responsable de gestionar la información del dominio específico.

Este servidor es gestionado por la entidad que administra el dominio (por ejemplo, para google.com el servidor autoritativo pertenece a Google o Atlassian para bitbucket.com).

El servidor autoritativo tiene la dirección IP del dominio y responde con la dirección IP correspondiente (por ejemplo, 172.217.0.46 para google.com) al resolver la consulta.

## 4 Jerarquía DNS y grado de importancia

**Tipos de TLD en igualdad jerárquica**

**TLD generales (gTLD):**

Son los dominios más comunes y conocidos: .com, .org, .net, etc.
También incluyen nuevos gTLD como .tech, .xyz, .blog, etc.
No están asociados a una región geográfica.

**TLD regionales (ccTLD):**

Son los dominios asociados a países o regiones específicas: .es (España), .uk (Reino Unido), .fr (Francia), etc.
Tienen siempre dos caracteres y están relacionados con un territorio.

Ambos tipos de TLD están en ***igualdad jerárquica desde la perspectiva de los servidores raíz***. Los servidores raíz no priorizan entre .com, .es, .blog o cualquier otro TLD. Simplemente dirigen la consulta al servidor TLD correspondiente, según la extensión.

**Servidores TLD**

Dentro vamos a tener Dominios de segundo nivel (SLD) y subdominios o niveles inferiores.
Los niveles inferiores (SLD y subdominios) están subordinados al TLD y son gestionados por los servidores autoritativos correspondientes.

**Servidores TLD**

Cada servidor TLD se ocupa exclusivamente de los dominios bajo su extensión. Por ejemplo:
El TLD .com está gestionado por los servidores TLD que manejan todos los dominios .com.
El TLD .es está gestionado por los servidores TLD que manejan todos los dominios .es.

Los TLD son independientes unos de otros, pero su "importancia" depende de su uso (por ejemplo, .com es más utilizado globalmente que .museum o .coop).

**Dominios de segundo nivel (SLD)**

Son los nombres que las personas o empresas contratan dentro de un TLD. Por ejemplo:
En google.com, el SLD es google, registrado dentro del TLD .com.
En miweb.es, el SLD es miweb, registrado dentro del TLD .es.

**Subdominios o niveles inferiores**

Estos se definen dentro de un SLD. Por ejemplo:
En www.google.com, www es un subdominio de google.com.
En ventas.miweb.es, ventas es un subdominio de miweb.es.



