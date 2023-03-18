# Dns-Linux

DNS LINUX  Daniel Otero Ferreira 2º ASIR


Instalaré bind9 con el comando ‘sudo apt install bind9’ y la herramienta dnsutils con ‘sudo apt install dnsutils’.

Primero crearé una carpeta llamada ‘dns’ donde trabajar y la abriré con Virtual Studio Code.
 
Dentro de la carpeta ‘dns’ también crearé una carpeta llamada ‘bind’, y dentro de esta crearé dos carpetas llamadas ‘etc’ y ‘var’.

Luego creare un archivo ‘docker-compose.yml’ dentro de esta carpeta y agregaré la siguiente configuración:

‘version: ‘3’

services:
  server:
     image: debian:buster-slim
     container_name: dns_server
    	volumes:
  	    - ./bind/etc:/etc/bind
   	    - ./bind/var:/var/cache/bind

client:
   image: alpine
   container_name: dns_client
   networks:
     dns_net:’


En este archivo he creado dos servicios, uno ‘server’ y otro ‘client’. El servicio ‘server’ lo configuraré con un volumen separado para  la configuración del servidos DNS en la carpeta ‘bind’, dentro de una red interna para los contenedores y con la ip fija ‘172.28.0.2’.

El servicio ‘client’ utiliza una imagen Alpine y se unirá a la misma red interna que el servicio ‘server’.

Para crear una red interna para los contenedores agregaré las siguientes líneas al fichero docker-compose.yml:

‘networks:
   my_network:
      driver: bridge’

Y en el apartado del servidor agregaré lo siguiente:



‘networks:
   dns_net:
      ipv4_address: 172.28.0.2’


En la carpeta ‘/bind/etc’ crearé un archivo de configuración llamado ‘named.conf.options’ en la que establecer la ip fija del servidor añadiendo la siguiente línea:

‘listen-on { 172.28.0.2 }’

Y para configurar los forwarders añadiré en el mismo archivo lo siguiente:

forwarders {
   8.8.8.8;
   8.8.4.4;
};

Para que los cambios se apliquen en el servidor debemos ejecutar el comando ‘sudo systemctl restart bind9.service’.

A continuación crearé un archivo de zona llamado ‘tierramedia.com’ en la carpeta ‘/bind/etc’ y otro llamado ‘named.conf.local’ en el que añadiré lo siguiente con el fin de crear una zona para el dominio ‘tierramedia.com’ y crear un archivo para la zona en la carpeta ‘/bind/var’:

‘zone "tierramedia.com" {
   type master;
   file "/var/cache/bind/tierramedia.com.";
};’

Para configurar los registros  NS, A, CNAME, TXT y SOA agregaré lo siguiente en mi archivo de zona:

A:

‘www       IN      A      172.28.1.2’

NS:

‘@         IN      NS     ns1.tierramedia.com.’

CNAME:

‘ftp       IN      CNAME  www.tierramedia.com.’
