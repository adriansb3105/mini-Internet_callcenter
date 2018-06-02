# Call ​ ​center

Cada estación cuenta con un softphone conectado a una central PBX que está dentro de la organización y puede hacer llamadas IP a lo interno y a otras organizaciones.

Este call center realiza llamadas de forma automatizada. Es decir, que con base en una lista de “números”telefónicos se realizan llamadas a las demás organizaciones. Estas llamadas caen a cualquiera de los agentes inscritos​ ​en ​ ​la ​ ​lista ​ ​de ​ ​“clerk's”.



## Infraestructura ​ ​física
Se utilizará el siguiente ​ ​esquema:

1. Los conmutadores(​switch's​) se partirán lógicamente en cuantas ​VLAN sean necesarias​ ​(redes​ ​de ​ ​1º ​ ​y​ ​2º ​ ​nivel).
2. El enrutamiento debe ser estático tanto en los enrutadores del laboratorio como los​ ​hosts​ ​GNU/Linux.
3. Todos los servidores serán creados en máquinas​ ​virtuales​ ​sobre ​ ​Virtual ​ ​Box.
4. Las máquinas físicas del laboratorio #7 serán usadas como clientes de las redes​ ​de ​ ​2º ​ ​nivel.
5. Es necesaria alta cooperación entre las distintas organizaciones, por lo que deben establecer un mecanismo decomunicación entre todos (se
recomienda el foro disponible en el sitio de ​ ​mediación ​ ​para ​ ​el ​ ​curso).
6. Se propone un esquema lógico como el diagrama ​ ​al ​ ​final ​ ​de ​ ​este ​ ​documento.