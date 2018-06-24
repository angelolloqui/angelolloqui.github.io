---
layout: post
title:  "Servidores en Amazon con EC2 y EBS"
date:   2010-06-19 11:30:34
categories: 
    - amazonaws
    - ebs
    - ec2
permalink: /blog/:title
---

Llevo ya más de un año pegándome con Amazon y su servicio AWS. Concretamente, con servidores virtuales EC2, almacenamiento en su sistema S3 y su interconexión con EBS.  
  
Hoy me esta tocando hacer una limpieza de una instancia que tengo con un antiguo cliente que resulta que se cayó recientemente y me ha apetecido compartir una reflexión y así escribir la primera entrada técnica (muy light), aunque no tiene nada que ver con dispositivos móviles.  
  
Así pues, antes de exponer el grave problema al que me podría haber enfrentado si no hubiese hecho las cosas bien, os expongo un poco las partes de AWS que estoy utilizando (hay más, pero estas son las más comunes):

* **EC2 (Elastic Compute Cloud)**: Este es el servicio de Amazon que simula los servidores virtuales. Su funcionamiento es sencillo, seleccionas una imagen (AMI) de la que partir (puede ser una imagen genérica como una distribución Ubuntu o una imágen muy personalizada en la que tengas multitud de software ya configurado), seleccionas unas características de tu instancia en función del RAM, procesador, etc del que quieras disponer y la lanzas. En pocos segundos tendrás un servidor virtual con acceso vía SSH y con las características de tu AMI seleccionada. Sobre esta instancia, dado que es a todos los efectos un servidor virtual, puedes instalar el resto de software que desees, así como configurar cuentas, permisos,...  
  
* **S3 (Simple Storage Service)**: Este servicio proporciona un espacio persistente donde almacenar datos. A modo de simplificación, podemos entenderlo como un disco duro virtual y persistente.  
  
* **EBS (Elastic Block Storage)**: Con este servicio podremos interconectar los dos anteriores, simulando que el S3 es una unidad más de nuestra instancia EC2. Quizá te estás preguntando la necesidad de este servicio. Pues bien, la respuesta es que EC2 por sí mismo no tiene persistencia de datos! sí sí, como lo has oido, las instancias EC2 son instancias virtuales, que tienen su sistema de ficheros como cualquier otro servidor, pero a diferencia de un servidor estándar, este sistema de ficheros es volátil, lo que significa que si por alguna razón tienes que terminar con la instancia perderás los datos que has introducido desde que lanzaste la instancia a partir de un AMI.  
  
* **Elastic IP**: Este servicio nos permite mapear cualquier instancia con una IP externa y única. Es necesario si queremos que este servidor esté accesible externamente por ejemplo por un nombre de dominio. La gran ventaja de esto es que al arrancar nuevas instancias podemos seguir usando la misma IP y así no tenemos que esperar a que los DNS se refresquen para cambiar de máquina (tardan hasta 48h).  
  
  
Y ahora que he comentado las principales partes implicadas, os expongo lo ocurrido: Una instancia que llevaba meses funcionando (y en producción), resulta que ha dejado de funcionar. No sólo el servidor web está caido, sino que la instancia está completamente inaccesible, sin posibilidad de conectarte por SSH, FTP ni similares! Arggghhhh!!  
¿Qué hacemos? pues no nos queda otra que entrar en la consola de administración de Amazon AWS, arrancar una nueva y apuntar la IP pública a la nueva, pero ¿perderemos todos los datos desde el último backup? esto sería muy grave porque además no hacemos backups con toda la frecuencia que deberíamos. Por otra parte, ¿que pasa con el software que instalé para adaptar mi instancia? ¿tendré que volver a instalarlo?  
  
Bueno, calma, para resolver el primer problema (y más grave) tenemos el EBS. Gracias a este servicio, toda nuestra BD estará trabajando en una unidad persistente, que NO habrá perdido la información y que podremos enlazar con la nueva instancia. Os dejo un enlace donde se explica una forma sencilla y clara sobre como montar este tipo de sistemas, moviendo los datos de MySQL a la unidad de EBS:  
[http://lab.redmallorca.com/servidor-mysql-persistente-sobre-ec2-y-ebs/](http://lab.redmallorca.com/servidor-mysql-persistente-sobre-ec2-y-ebs/)  
  
¿Y el segundo problema? ¿tengo que volver a instalar todo el software? pues la respuesta es que tendrías que hacerlo, a no ser que hayas sido precavido y hayas generado una nueva AMI a partir de tu instancia una vez configurada. ¿Como hacerlo?, de nuevo, os enlazo a un post en donde lo explica muy clarito:  
[http://lab.redmallorca.com/crea-tu-propio-ami-basado-en-ubuntu-para-ec2/](http://lab.redmallorca.com/crea-tu-propio-ami-basado-en-ubuntu-para-ec2/)  
  
  
  
Así que, para todos los que trabajéis con Amazon EC2, ni se os ocurra hacerlo sin contemplar posibles cortes o caídas porque ocurren! Mi consejo, aunque tengo poca experiencia, es que una vez instalado todo el software en tu instancia a partir de la AMI pública generes un AMI privada con esta configuración, posteriormente configures el sistema para que la BD trabaje sobre la unidad EBS, y por último, opcionalmente me crearía otra AMI privada con todo esto montado. Ojo, esta última AMI es posible que os de problemas al arrancar, por estar enlazada al EBS (a mí me los ha dado), por eso os recomiendo que hagáis la AMI preEBS para poder recuperar todo si tenéis problemas con esta segunda AMI (solo tendríais que repetir los pasos de montaje de EBS, pero los datos seguirían estando ahí y el software funcionando).  
  
  
Así que ya sabéis, haced las cosas bien al principio aunque os lleve un par de horas extras de trabajo y luego no tendréis que lamentar las pérdidas!