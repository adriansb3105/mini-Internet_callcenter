# Configuración de Asterisk

Primero se va a empezar por crear el backup del folder astersik

`sudo cp -R /etc/asterisk /etc/asterisk_backup`

Ahora vamos a modificar solamente tres archivos de configuración de la herramienta *asterisk*

El primero va a ser el archivo */etc/asterisk/sip.conf* con la siguiente información.
El contexto será [guia_asterisk], el cual agrupa los clientes para una configuración única dentro de
la red.


	[general]
	context=guia_asterisk
	allowguest=no
	allowoverlap=no
	bindport=5060
	bindaddr=0.0.0.0
	srvlookup=no
	disallow=all
	allow=ulaw
	alwaysauthreject=yes
	canreinvite=no
	nat=yes
	session-timers=refuse
	localnet=172.16.16.4


	[101]
	context=guia_asterisk
	type=friend
	callerid=”usuario1”
	username=user1
	secret=passuser1
	nat=no
	canreinvite=yes
	host= dynamic
	qualify=yes

	[102]
	context=guia_asterisk
	type=friend
	callerid=”usuario2”
	username=user2
	secret=passuser2
	nat=no
	canreinvite=yes
	host= dynamic
	qualify=yes

El segundo archivo a modificar es */etc/asterisk/extensions.conf*

	exten => extensión, prioridad, parámetros
La extensión, indica el numero marcado.
La prioridad el orden en que se ejecutan las acciones (1 mayor prioridad) y parámetros la acción que se ejecuta.
Para este caso la línea indica que si llaman al número 101 se ejecuta el comando Dial (destino, timeout, opciones).

El comando Dial nos indica:

- Destino: Protocolo y numero de marcación del usuario.

- Timeout: Segundos para contestar la llamada.

- Opciones:
“T” Permite al usuario que realiza la llamada transferirla pulsando #
“t” Permite al usuario que recibe la llamada transferirla pulsando #
“m” Indica que mientras se espera la contestación se escuche una música especial.

Así que al final del archivo se agrega las siguientes líneas

	[guia_asterisk]
	exten =>101,1,Dial(SIP/101,30,Ttm)
	exten =>101,2,Hangup

	exten =>102,1,Dial(SIP/102,30,Ttm)
	exten =>102,2,Hangup

El tercer archivo a modificar es */etc/asterisk/voicemail.conf*
	
	[main]
	101 => passuser1
	102 => passuser2

Revisar el archivo */etc/asterisk/manager.conf*


# Automate Calls

Lo primero es asegurarnos que el módulo **pbx_spool.so** esté habilitado, sin este módulo, entonces este proceso simplemente no funcionará.
Primero ingresamos a la consola de *asterisk*
	sudo asterisk -r

Ingresamos el comando
	module show

Y nos aseguramos que exista la entrada
	pbx_spool.so   Outgoing Spool Support

También debemos comprobar el atributo **autoload=yes** en el archivo */etc/asterisk/modules.conf*




sudo vim test.call

	Channel: SIP/102
	MaxRetries: 1
	RetryTime: 10
	WaitTime: 30
	Context: guia_asterisk
	Extension: 101

sudo su - root

sudo cp test.call /var/spool/asterisk/outgoing/




Multiple Channels
https://issues.asterisk.org/jira/browse/ASTERISK-175