---
layout: post
title:  "Gestión de memoria en iPhone SDK"
date:   2010-07-07 11:30:34
categories: 
    - ios
    - memory management
    - objective-c
permalink: /blog/:title
---

El desarrollo de aplicaciones para iPhone o iPad, a diferencia de Android y otras plataformas, tiene un tema bastante espinoso y que a la gente le suele parecer complicado al principio: la gestión de memoria. Debido a que la aplicación es compilada a código nativo, sin máquinas virtuales de por medio, no contamos con un recolector de basura que nos haga la limpieza de memoria de las variables en desuso. En esta entrada trataré de exponer las convenciones y consideraciones que debes tener en cuenta al enfrentarte a este tipo de desarrollos.  
  
Como decía anteriormente, lo más importante es saber que aunque en Objective-C puede existir un recolector de basura cuando programamos para MacOs, no existe en iPhone o iPad, por lo que toda la gestión es manual. ¿Qué significa manual? pues que toda memoria que quieras utilizar tienes que crearla (normalmente mediante alloc), retenerla si es necesario (lo veremos después), y liberarla cuando dejes de necesitarla (con release). Esta forma de trabajar es chocante si vienes de lenguajes donde no sea necesario, tales como los interpretados (Java, .Net,...) o los de scripting (ruby, php,...).  
  
Si no hacemos la gestión de memoria de forma correcta pueden ocurrir dos cosas:  
  
1.- **Se queda memoria sin liberar**: el iPhone emepezará a quedarse sin memoria con cada uso de la aplicación, y será necesario reiniciar el dispositivo pasados unos cuantos usos según la cantidad de memoria dejada. A este tipo de lagunas de memoria los llamamos "leaks".  
2.- **Se libera la memoria antes de tiempo**: cuando el código trate de acceder a un área de memoria que ya ha sido liberada, la aplicación lanzará una excepción y se cerrará.  
  
Como veis, ambas cosas con muy preocupantes y hay que evitarlas por todos los medios.  
  
Para facilitarnos la vida, Apple nos ha aportado un mecanismo de "reference counting" en el objeto NSObject y ha creado dos convenciones que deberás seguir si quieres hacer esta gestión de memoria de forma correcta, y que te ayudarán a determinar qué clase debe hacer que retain/release y en que momento.  
  
Pero expliquemos por partes:  
  
#### Reference counting.  
  
El reference counting es un mecanismo por el cuál los objetos llevan algo parecido a un contador de punteros que le apuntan. De esta forma, si un objeto está apuntado por varios otros, no podemos liberarlo hasta que deje de ser apuntado por el resto. La diferencia con otros entornos con Garbage Collector es que en Objective-C este conteo se realiza de forma manual, realizándose más o menos de esta forma:  
R.1.- Cuando reservamos un objeto mediante su método "alloc" o "copy", el nuevo objeto (llamémosle MiObjeto) pasa a tener un contador de +1.  
R.2.- A partir de este momento, cuando otro objeto apunta a MiObjeto debe indicarselo mediante la invocación del método "retain". El método "retain" sumará un +1 al contador de referencias de MiObjeto.  
R.3.- Cuando otro objeto deja de apuntar a MiObjeto, se invoca el método "release". El método "release" decrementará en 1 el contador de referencias. Si MiObjeto pasa a tener un contador de 0 se invoca el método "dealloc", que es donde se realiza la liberación.  
  
Como podeis ver, de esta forma no invocamos directamente los métodos de liberación de memoria (dealloc), sino que lo que hacemos es invocar siempre el método "release" para permitir que, si otros objetos tienen retenido el que queremos liberar, se destruya la memoria y se produzca el fallo 2.- del primer párrafo. Esta liberación ocurrirá cuando los otros objetos realicen a su vez el release de MiObjeto (ojo, como queda claro, cada retain y alloc debe ejecutar en algún momento un release, dado que de otra forma tendremos un leak como se indica en 1.-)  
  
Ejemplo de reference counting:

    
    for (int i=0; i < 1000; i++) {
    	//Creamos memoria
    	id objeto = [[ClaseCreadora alloc] init];
    	//Pasamos el objeto a otra clase
    	[ClaseB nuevoObjeto:objeto];
    	//Liberamos la memoria
    	[objeto release];
    }
    

En este ejemplo vemos cómo se ha hecho un release por cada alloc, y se puede suponer que ClaseB estará haciendo un retain sobre objeto si lo necesita utilizar más tarde.  
  
#### Convención ownership  
  
El reference counting nos ayuda a evitar gran cantidad de problemas surgidos al compartir memoria entre diferentes clases y objetos, pero plantea problemas a la hora de decidir dónde y quién debe hacer los releases. Por ejemplo, sin un método tiene que devolver un objeto nuevo, y no guarda ningún puntero al mismo, ¿cómo libera la memoria de este nuevo objeto? como no guarda el puntero no puede hacer el release más tarde, tampoco lo puede ejecutar inmediatamente porque si lo hace se liberará la memoria en este mismo momento y producirá el fallo 2.- al intentar usarla, y tampoco es buena idea devolverlo sin el release porque en ese caso se producirá un leak si la otra clase no lo libera. Aquí entra en juego la convención Ownership.  
  
Según esta convención, todo objeto es responsable de liberar la memoria que crea o retiene, pero aporta un par de reglas extras para solucionar el problema anterior:  
  
O.1.- Cuando un objeto va a utilizar memoria creada por otro método debe hacer un retain para evitar su liberación prematura.  
O.2.- Todo objeto debe liberar la memoria creada (mediante métodos alloc, copy o new) y retenida (mediante método retain), invocando el método release en algún momento en su ciclo de vida. Por cada alloc/retain ejecutado debe hacerse un release.  
O.3.- Si un método devuelve un objeto que no va a liberar, el método deberá llamarse allocXXX, copyXXX o newXXX (donde XXX se sustituye por el nombre que elijas). De esta forma, el que invoca el método puede conocer esta característica y, aplicando la regla O.2.- podrá liberarla. Esta misma regla podría aplicarse N veces si cada método devuelve a su vez el objeto nuevo a su padre.  
  
Ejemplo convención Ownership:

    
    - (Clase *) newObjectWithData:(NSData *)data {
    	//Creamos el objeto
    	Clase *object = [[Clase alloc] init];
    	//Ejecutamos lo que sea
    	[object setData:data];
    	//Devolvemos el objeto
    	return object;
    }
    

En este ejemplo podemos hacer la devolución gracias a haber nombrado al método newXXX, que indicará al que lo invoque que el objeto de tipo Clase * devuelto está sin liberar.  
  
#### Autorelease  
  
Por último, tenemos un último recurso que nos ayudará a gestionar la memoria. Lo he dejado al final porque es el poco óptimo y propenso a erores, por lo que deberías tratar de evitarlo donde sea posible. Se trata del método autorelease, que nos permite indicar que una variable debe ejecutar un release pero más tarde. ¿Cuándo? pues no lo sabes a ciencia cierta porque dependerá de la pila de llamadas, pero el autorelease funciona así:  
  
A.1.- Un método ejecuta un autorelease sobre un objeto (MiObjeto) que tenía retenido (retain) o que había creado previamente (alloc/copy).  
A.2.- El método autorelease busca la NSAutoreleasePool en la pila de llamadas más cercana y programa la ejecución del release sobre este objeto  
A.3.- La ejecución de la pila de llamadas termina, se ejecuta el NSAutoreleasePool release y se realizan todos los releases programados (se realiza el release sobre MiObjeto).  
  
Así, durante todo el espacio entre A.1 y A.3 la variable seguirá retenida, pero no se tratará de un leak aunque se pierda el puntero ya que este release está programado en la AutoreleasePool. El espacio entre A.1 y A.3 normalmente será la pila de llamdas del sistema (por ejemplo, si se pulsa un botón en el interfaz gráfico se crea una pila y se destruye cuando se termina de procesar el evento de pulsación), pero pueden crearse pilas dentro de otras para hacer las liberaciones de una forma más frecuente.  
  
Ejemplo devolución con autorelease:

    
    - (Clase *) objectWithData:(NSData *)data {
    	//Creamos el objeto
    	Clase *object = [[Clase alloc] init];
    	//Ejecutamos lo que sea
    	[object setData:data];
    	//Devolvemos el objeto
    	return [object autorelease];
    }
    

En este ejemplo a diferencia del utilizado en el patrón ownership vemos que hemos ejecutado un autorelease sobre el objet, por lo que el método no deberá llamarse newXXX porque el padre no tiene que preocuparse de la gestión de memoria de dicho object (el release no se ha ejecutado todavía pero lo hará más tarde).  
  
Ejemplo de uso de NSAutoreleasePool:

    
    //Creamos una nueva pool
    NSAutoreleasePool *pool = [[NSAutoreleasePool alloc] ini];
    for (int i=0; i < 1000; i++) {
    	//Creamos memoria
    	id objeto = [[ClaseCreadora alloc] init];
    	//Pasamos el objeto a otra clase
    	[ClaseB nuevoObjeto:objeto];
    	//Programamos el autorelease para hacerlo mas tarde
    	[objeto autorelease];
    }
    //Liberamos toda la memoria con autorelease desde la creacion de la pool
    [pool release];
    

  

### Resumen  
  
La gestión de memoria es una tarea que al principio parece compleja, pero una vez comprendidas las normas y el funcionamiento acaba siendo un tema bastante más sencillo de lo que es en otros lenguajes sin reference counting o patrones ownership. Al final, normalmente se resumen en:  
  
a. Toda memoria creada en tu clase con métodos alloc, copy o new debe llevar un release o autorelease en su ciclo de vida  
  
b. Toda memoria que uses creada por otra clase deberías protegerla con retain y liberarla cuando no sea necesaria (idem a punto a.)  
  
c. Si un método devuelve memoria sin liberar debe llamarse newXXX, allocXXX o copyXXX  
  
d. Puedes ejecutar autorelease si necesitas posponer la ejecución de un release, pero no puedes saber el instante exacto en el que se realizará a priori (por ello, debes seguir b. si se usa esa memoria también a posteriori desde otro punto)  
  
e. Los métodos y clases de Apple cumplen los apartados anteriores.