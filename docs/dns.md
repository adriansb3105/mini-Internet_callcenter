# Servicio DNS
Como buena práctica se recomienda hacer un backup de todos los archivos del directorio *bind*

`sudo cp -R /etc/bind /etc/bind_orig`

## Configuración

La configuración básica para el **Caching Nameserver**, simplemente se agrega la dirección IP del servidor del ISP, tan solo se descomenta y edita lo siguiente en el archivo */etc/bind/named.conf.options*

	forwarders {
		172.16.16.4;
	};

## Servidor Primario

Para la configuración del **Primary Master** vamos a utilizar el FQDN (Fully Qualified Domain Name), el cual se puede obtener con el comando `hostname --fqdn`

#### Zona de búsqueda directa
El primer paso es editar el archivo */etc/bind/named.conf.local* y se agrega lo siguiente al final
	
	zone "callcenter.com" {
		type master;
		file "/etc/bind/db.callcenter.com";
	};

Ahora usando un template existente, se crea en archivo del paso anterior

`sudo cp /etc/bind/db.local /etc/bind/db.callcenter`

Lo siguiente es editar el archivo creado tal y como se muestra a continuación. (**Nota:** el número Serial debe ser incrementado cada vez que este archivo de modifique)

	;
	; BIND data file for callcenter.com
	;
	$TTL	604800
	@		IN	SOA	callcenter.com.	root.callcenter.com.	(
			2         	; Serial
			604800		; Refresh
			86400		; Retry
			2419200		; Expire
			604800 )	; Negative Cache TTL
	;
	@	IN	NS      callcenter.com.
	@	IN	A       172.16.16.4
	@	IN	AAAA    ::1
	ns	IN	A       172.16.16.4

#### Zona de búsqueda inversa
Ahora que la zona Directa está configurada y resuelve nombres a direcciones IP, también la zona Inversa es requerida, para resolver direcciones IP a nombres.
Al final del archivo */etc/bind/named.conf.local* se debe agregar lo siguiente (se deben tener en cuenta los primeros tres octetos de la red con la que estemos trabajando)
	
	zone "16.16.172.in-addr.arpa" {
		type master;
		file "/etc/bind/db.172";
	};

Se crea el archivo al cuál hace referencia el paso anterior, basado en un template existente

`sudo cp /etc/bind/db.127 /etc/bind/db.172`

Y se edita con la siguiente información (no olvidar aumentar el valor del Serial con cada modificación)
**Nota:** El número asociado al puntero PTR es el último octeto de la IP de nuestro Servidor

	;
	; BIND reverse data file for local 172.16.16.0 net
	;
	$TTL	604800
	@		IN	SOA		callcenter.com.	root.callcenter.com.	(
						2			; Serial
						604800		; Refresh
						86400		; Retry
						2419200		; Expire
						604800 )	; Negative Cache TTL
	;
	@		IN	NS		callcenter.com
	4	IN	PTR		callcenter.com.


## Servidor Redundante

Una vez finalizada la configuración de la **Primary Master**, se necesitará configurar el **Secundary Master** para mantener la disponibilidad del dominio en caso de que el Primario llegue a estar no disponible.
Primero en el Primary Server se debe permitir la transferencia de zona, se agrega la opción *allow-transfer* tanto en la zona directa como la inversa en el archivo */etc/bind/named.conf.local*

**Nota:** La dirección IP del parámetro *allow-transfer* es la dirección del Servidor Secundario (el redundante)

	zone "callcenter.com" {
		type master;
		file "/etc/bind/db.callcenter.com";
		allow-transfer { 172.16.16.9; };
	};

	zone "16.16.172.in-addr.arpa" {
		type master;
		file "/etc/bind/db.172";
		allow-transfer { 172.16.16.9; };
	};


Lo siguiente será cambiarse al servidor secundario, como ya se tiene instalado la herramienta *bind9*, entonces se edita el archivo */etc/bind/named.conf.local* con las siguientes declaraciones para las zona directa e inversa (La dirección IP debe ser la del Primary Server)

	zone "callcenter.com" {
		type slave;
		file "db.callcenter.com";
		masters { 192.168.1.4; };
	};

	zone "16.16.172.in-addr.arpa" {
		type slave;
		file "db.172";
		masters { 172.16.16.4; };
	};

**Nota:** Una zona solo será transferida si el Número Serial en el Primario es más grande que en el Secundario. Si se quiere mantener el Servidor Primario DNS notificando al Servidor Secundario de los cambios de zona, se debe agregar *also-notify { ipaddress; };* en el archivo */etc/bind/named.conf.local*

	zone "callcenter.com" {
		type master;
		file "/etc/bind/db.callcenter.com";
		allow-transfer { 172.16.16.9; };
		also-notify { 172.16.16.9; }; 
	};
	
	zone "16.16.172.in-addr.arpa" {
		type master;
		file "/etc/bind/db.172";
		allow-transfer { 172.16.16.9; };
		also-notify { 172.16.16.9; }; 
	};

***

### Reiniciar Servicios
Se debe reiniciar el demonio de *bind* con el siguiente comando

`sudo /etc/init.d/bind9 restart`

***

Para comprobar si existe algún error de sintaxis o algún otro problema que impida el correcto funcionamiento de los servicios, se recomienda constantemente revisar el archivo *syslog* y verificar que las entradas sean correctas, y no contengan errores, esto se puede llevar a cabo con el comando

`sudo tail /var/log/syslog`
