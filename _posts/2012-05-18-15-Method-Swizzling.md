---
layout: post
title:  "Method Swizzling"
date:   2012-05-18 11:30:34
categories: 
    - objective-c
    - runtime
    - ios
    - method swizzling
permalink: /blog/:title
---

Hoy voy a hablar sobre MethodSwizzling, para mí una de las técnicas más apasionantes de Objective-C. Eso sí, aviso para navegantes, estas técnicas son complejas y, según el uso que les des, pueden también ser peligrosas. Vamos a verlo!

#### Qué es MethodSwizzling?

Probablemente nunca hayas oído hablar de MethodSwizzling antes, o si lo habías hecho, no sepas exactamente qué es o para qué puede servir. Pues bien, ese nombre se usa para referirse simplemente a la técnica de **intercambiar métodos en runtime.**

Es decir, se trata de que métodos que ya existan sean cambiados por otros nuevos durante ejecución, incluso en clases que no has prgramado tú (del sistema por ejemplo). A nivel conceptual puedes entenderlo como una category o una herencia que reemplaza métodos del padre, pero el MethodSwzzling, al ser en runtime, va mucho mas allá. Ahora veremos por qué....

#### Cómo funciona?

Me encantaría deternerme a explicar como funciona esto por dentro, pero habría que analizar muchos aspectos de Objective-C a bajo nivel, así que daré una explicación muy rápida y me apunto el tema para otro post donde tratarlo como merece.

La idea importante a alto nivel es que Objective-C es un superconjunto de C. En él, las clases no son más que estructuras donde parte de su información contiene un listado de los métodos de la clase con el nombre, parámetros,… mientras que los métodos son funciones C con un par de parámetros extra sobre los declarados en Obj-C (el objeto y el nombre del método ejecutado).

Cuando tu código hace una llamada tipo:

    [myVar myMethod];

esto se traduce en tiempo de compilación por algo similar a:

    objc_msgSend(myVar, @selector(myMethod));

Optimizaciones aparte, lo que hará el código de objc_msgSend es buscar e invocar la función que corresponde a la clase y selector pasados por parámetros.

Y aquí viene la gracia! Ya que esta información reside en memoria, es posible cambiarla. De hecho, no solo es posible mediante hacking, sino que el propio **Objective-C viene con herramientas** que te lo ponen muy fácil: [ObjCRuntime](https://developer.apple.com/library/ios/#documentation/Cocoa/Reference/ObjCRuntimeRef/Reference/reference.html)

Pero veamos cómo se puede hacer en un ejemplo sencillo:

    @implementation UIPopoverController (MyCustomPopover)
    + (void)load  
    {  
        Class thisClass = [UIPopoverController class];
        
        Method presentPopoverFromRectCustom = class_getInstanceMethod(thisClass, @selector(presentPopoverFromRectCustom:inView:permittedArrowDirections:animated:));
        Method presentPopoverFromRect = class_getInstanceMethod(thisClass, @selector(presentPopoverFromRect:inView:permittedArrowDirections:animated:));
        method_exchangeImplementations(presentPopoverFromRect, presentPopoverFromRectCustom);    
    }  
    - (void)presentPopoverFromRectCustom:(CGRect)rect inView:(UIView *)view permittedArrowDirections:(UIPopoverArrowDirection)arrowDirections animated:(BOOL)animated {
        NSLog(@"Custom popover called");
        [self presentPopoverFromRectCustom:rect inView:view permittedArrowDirections:arrowDirections animated:animated];
    }
    
    @end

El código anterior hace uso del método load, que se ejecuta automáticamente durante el arranque de tu app (incluso siendo una category), para hacer lo siguiente:

1.  Obtiene el puntero al método presentPopoverFromRectCustom:::: (línea 6)
2.  Obtiene el puntero al método presentPopoverFromRect:::: (línea 7)
3.  Los intercambia (línea 8)

De esta forma, si incluyes el código anterior en tu proyecto, cada vez que abras un popover verás que se te invoca tu método presentPopoverFromRectCustom, y éste al invocar presentPopoverFromRectCustom desde dentro **estará invocando el presentPopoverFromRect original** (línea 12: recuerda que los punteros se han cambiado, pero en tiempo de ejecución, por lo que la llamada a Custom será la llamada al original y viceversa). Todo esto sin tener que cambiar absolutamente nada en tu código, y sirviendo también para los popovers que se abren desde librerías de terceros.

#### Diferencia con una Category y Herencia

*   **La diferencia con la herencia** es evidente: el cambio se aplica en la clase original, mientras que con herencia tendrías que crear una subclase y hacer que el código use tu subclase (donde se sobrescribirán los métodos). Esto requiere **muchísimos cambios a nivel de código** y en muchos casos **ni siquiera es posible** ya que puede invocarse desde **librerías de terceros** o desde el propio sistema.
*   **La diferencia con las categories** es mas sutil, pero igualmente clara. Si pretendes hacer lo anterior (popover) con una category sobrescribiendo el método presentPopoverFromRect, **pierdes la referencia al método original**, y por tanto, no puedes reproducir el comportamiento base. Esto es tremendamente peligros ya que la clase puede depender de cosas hechas en ese método y está lógicamente **desaconsejado por el propio Apple**.

  

En cambio, el **Method Swizzling** te permite tener una funcionalidad similar a la category pero sin perder el método original, y por tanto, permitiendo al mismo tiempo **inyectar tu código**, hacer la **llamada al código base** para que nada se rompa y **no cambiar nada** en tu proyecto ni librerías externas.

#### Ejemplos basados en hechos reales

Si hay una forma de ver el poder de esto es con ejemplos reales. Voy a suponer y analizar dos escenarios basados en situaciones vividas por mí mismo, y que se pueden presentar a cualquier developer sin demasiada dificultad:

##### Caso 1: La navbar customizada

Hasta iOS5 no había una forma sencilla de personalizar cosas tan simples como el background de la navbar. Sin embargo, a pesar de los nuevos métodos de UIAppearance, es frecuente que lo que quieras hacer no este implementado en el SDK o que tengas que dar soporte a iOS antiguos. En este caso, supondremos que queremos una navBar donde simplemente el título esté rotado 20º cuando se cumplen ciertos requisitos.

##### Caso 2: Feedback de red

En este caso, supondremos que la app debe dar al usuario feedback mediante un spinner en el centro de la ventana sobre el estado de la conexión de red, animándolo cada vez que se hace uso de la red con cualquier motivo (descarga de imágenes, sincronización de datos,…)

En cualquiera de los 3 casos anteriores, un developer que no conozca MethodSwizzling optaría por alguna de las siguientes alternativas (o soluciones derivadas de estas) según su experiencia:

##### Alternativa 1 - Hacer las llamadas a mano donde sea necesario

*   Solución al **Caso 1**: Modificar los controladores de tu app para que en el viewWillAppear se rote o enderece el title de la navBar.
*   Solución al **Caso 2**: Buscar las peticiones a Internet para que se aumente algún tipo de contador mediante la llamada a una función y se decremente cuando acabe.

  

Este enfoque es el más junior. El código estará muy lejos de ser mantenible, requiere muchos cambios, esta sujeto a error humano (qué fácil es olvidarse de hacer alguna llamada) y totalmente acoplado a la solución customizada que tenemos. Tampoco podremos llegar a código de terceros por lo que no será totalmente universal. Es decir, **una auténtica chapuza**.

##### Alternativa 2: Herencia

*   Solución al **Caso 1**: Crear tu propia navBar que evalúe las condiciones y rote el title si debe hacerlo. Modificar el código para que se cree tu nueva navbar en vez de la que aporta el sistema por defecto.
*   Solución al **Caso 2**: Crear una subclase de NSURLConnection que lleve el conteo de conexiones y activación del spinner. Modificar el código para que todos aquellos allocs de NSURLConnection pasen a utilizar tu subclase.

  

Este enfoque, aunque mejora sensiblemente el anterior en cuanto a mantenibilidad, **sigue siendo bastante malo**. El principal problema es que igualmente está sujeto a error humano y requiere modificar bastantes líneas de código en tu proyecto. Además, al igual que el anterior, tampoco podemos llegar a código de terceros, por lo que por ejemplo la comunicación desde librerías no activará nuestro spinner.

##### Alternativa 3: Categories

*   Solución al **Caso 1**: sobrescribir el método drawRect o similar de la UINavigationBar para que pinte el title girado
*   Solución al **Caso 2**: sobrescribir el método de conexión, o incluso ir más alla y combinado con la alternativa 2 sobrescribir el método de alloc para que éste genere un objeto de nuestra subclase en vez de NSUrlConnection.

  

Este enfoque sería **ideal si funcionase**: Permite llegar a toda la app y librerías, sin modificar código, sin error humano, contenido en uno o dos ficheros que simplemente agregando y quitando darán/quitarán la funcionalidad a todo el proyecto. El problema es que **no funciona**. Por qué? bueno, como hemos dicho, con la category estamos sobrescribiendo el método original. Esto significa que en el caso de la navbar no podremos pintar el título por defecto y en el caso 2 no podremos invocar un alloc por defecto. Podríamos tratar de simular el comportamiento del método estándar, pero será muy complejo conseguir el mismo comportamiento y será tremendamente peligroso con las nuevas actualizaciones del iOS (las cosas pueden funcionar de forma distinta).

Y entonces, que nos queda??? el **MethodSwizzling**!!! hemos dicho que esta técnica es parecida a las categories, pero con la ventaja de que pueden llamar al método original (problema en alternativa 3); y a la herencia, pero con la ventaja de que cambiamos la clase original (problema de la alternativa 2).

La idea, tal y como ya he mostrado en el ejemplo del Popover del principio, pasa por generar una category donde implementamos un nuevo método para pintar el título, conectar a Internet,… Luego, en el load hacemos que el método original y el nuestro se intercambien, y desde el nuestro nos aseguramos invocar el método original antes de hacer nuestra lógica. 

Con esto conseguiremos la funcionalidad que queríamos para los dos casos, todo ello contenido en un **único fichero** y con muy pocas líneas de código. Lo podremos **exportar a otros proyectos o eliminar** de este si no lo queremos más, **sin cambios en nuestro código**, y afectará a todas las clases, incluidas las de terceros o incluso el sistema. Y lo que es más importante, si lo has hecho correctamente y con métodos públicos, el código no debería romperse en nuevas versiones de iOS ya que, como digo, siempre se llama al método original.

#### Un gran poder conlleva una gran responsabilidad

Si después de leer esto estás pensando en usarlo, primero debes saber que, precisamente debido a su potencia, también **puede ser un instrumento muy peligroso**. Es como un bisturí: en manos de un aprendiz el resultado puede ser terrible, pero en las entrenadas manos de un buen cirujano puede salvar vidas. Por este motivo, si buscas en Internet sobre este tema verás a muchísima gente promulgando que el MethodSwizzling es muy peligroso o que incluso no debería utilizarse nunca en proyectos reales. Mi consejo, como siempre, es que lo **uses con cuidado**, pero que le des una oportunidad si eres un develper con experiencia. Debes estar seguro de lo que estás haciendo, tomar medidas necesarias y nunca abusar de ello. Si te interesa saber más sobre los peligros y cómo evitarlos, hay un muy buen post sobre esto en [http://stackoverflow.com/questions/5339276/what-are-the-dangers-of-method-swizzling-in-objective-c](http://stackoverflow.com/questions/5339276/what-are-the-dangers-of-method-swizzling-in-objective-c)