---
layout: post
title:  "Manteniendo compatibilidad con versiones anteriores de iOS"
date:   2011-03-08 11:30:34
categories: 
    - deployment target
    - ios
    - ipad
    - iphone    
permalink: /blog/:title
---

Es muy habitual encontrarnos con la necesidad de dar soporte a usuarios que no estén actualizados con el último iOS en nuestras aplicaciones iPhone o iPad, pero los SDKs nuevos dan soporte únicamente para un conjunto muy reducido de versiones anteriores, lo que normalmente no es suficiente.

Ante esto, muchos developers no tienen muy claro como conseguirlo, y recurren a soluciones muy poco prácticas (y con varios problemas). La más habitual es tener diferentes SDKs instalados en su máquina, para compilar usando uno u otro dependiendo del iOS al que quieran dar compatibilidad. Esta solución, además de ser poco elegante, ocupar mucho espacio en disco y dar problemas a la hora de subir la app al AppStore, tiene otros dos problemas muy grandes:

1.  La arquitectura de los simuladores cambia cada cierto tiempo. Esto hace que una librería compilada para un simulador en la 4.3 por ejemplo, no pueda ejecutarse en un simulador en la 3.0. Si usamos librerías de terceros (como Flurry por ejemplo) podemos vernos en un problema para conseguir ejecutar nuestra aplicación en un simulador. Seguimos pudiendo ejecutar en dispositivo, pero la falta de simulador puede hacer el desarrollo mucho más lento.
2.  No podemos utilizar funcionalidad agregada en SDKs posteriores. Esto parece de perogrullo, pero si compilamos con un SDK3.0 no podremos hacer uso de cosas como FastApp Switching y similares.

  

Todo esto tiene una solución mucho más sencilla y elegante, que es utilizando el llamado "Deployment Target".  
  
Pero antes de nada, voy a explicar la diferencia entre algunos conceptos que no siempre están claros y pueden mezclarse:

*   **Versión de iOS**: iOS es el sistema operativo que corre en el dispositivo del usuario. Aquí caben muchísimas opciones, y nuestros usuarios estarán en una u otra. Así pues, si nuestra aplicación da soporte para versiones iOS3.0 o posterior, cualquier usuario con una versión iOS3.0 o posterior podrá ejecutar nuestra app. Según la versión de su iOS, el usuario podrá tener unas funcionalidades u otras (por ejemplo, los mapas se introdujeron en la 3.0 o el FastAppSwitch en la 4.0).
*   **Versión de SDK instalada**: Este concepto se refiere a la versión del SDK de desarrollo que te descargaste e instalaste en tu máquina. Si te lo has bajado de la página de Apple, será la última versión disponible. (Por ejemplo, ahora mismo estamos en la 4.3)
*   **Base SDK**: Establece con que versión de SDK se va a compilar un proyecto. El BaseSDK siempre tiene que ser igual o inferior a tu versión de SDK instalada. Normalmente, un SDK incluye varios Base SDKs anteriores, por si el usuario desea compilar con alguno previo.
*   **Deployment Target**: Establece a partir de qué versión de iOS podrá ejecutarse tu aplicación. El deployment target por defecto se establece igual al BaseSDK utilizado, pero puede cambiarse a uno anterior.

  

Una vez explicados estos conceptos, que en muchos casos parecen iguales o pueden resultar confusos, voy a tratar de explicar como solucionar el problema de dar soporte a usuarios con versiones previas de forma elegante , sin perder la compatibilidad con librerías compiladas para simulador y pudiendo dar a usuarios con versiones avanzadas de iOS una funcionalidad extra de los nuevos SDKs.  
El truco está en configurar correctamente el DeploymentTarget. Para ello, basta con ir a las propiedades del proyecto (abre XCode, pincha en la raiz del proyecto con el botón derecho, "Get Info", Pestaña "Build") y busca el campo "iOS Deployment Target". Verás que te da un listado de versiones de iOS mucho más extenso del que tienes en las opciones del BaseSDK. Seleccionas tu deployment target deseado y listo.  
  
La dificultad de esta solución viene cuando tu app usa alguna funcionalidad que se introdujo en alguna versión de SDK posterior a la que has seleccionado en Deployment Target.  
Digamos que pretendes usar Notificaciones locales (introducidas en el 4.2), pero quieres dar soporte a usuarios que sigan en la 3.0. En este caso, puedes compilar con un BaseSDK 4.2 y configurar el Deployment Target a la 3.0. Hasta aquí todo bien (el compilador se lo traga y todo parece funcionar bien en simulador y en un dispositivo con iOS 4.2 o posterior). El problema está en que en el momento en que tu app trate de ejecutar el código asociado a las notificaciones locales, la app se cerrará si el iOS del usuario no es el 4.2 o posterior, ya que no será capaz de ejecutar ese código.  
¿Cómo nos protegemos? la solución pasa por estar muy atento a qué métodos utilizas que no estuviesen presentes en el SDK 3.0, para comprobar si existen antes de usarlos en tiempo de ejecución. Para ello, en caso de ser una clase, debes comprobar si la clase existe antes de usarla mediante la ejecución de "NSClassFromString", mientras que deberás usar el "respondsToSelector" en caso de tratarse de un método.  
  
Ej. 1 - Comprobación de clases - ¿Dispositivo soporta Local Notifications?

    
    if (NSClassFromString(@"UILocalNotification")!=nil) {
    	// Soporta LocalNotifications
    }
    else {
    	// No soporta Local Notifications
    }
    

  

Ej. 2 - Comprobación de métodos - ¿Dispositivo soporta multitasking:?

    
    if ([[UIDevice currentDevice] respondsToSelector:@selector(isMultitaskingSupported)]) {
    	// Soporta multitasking
    }
    else {
    	// No soporta multitasking
    }
    

Si el usuario no soporta la funcionalidad, dependerá de tí decidir que hacer (le muestras un mensaje, no haces nada, le das otra alternativa,...), pero por lo menos garantizas que la app se ejecuta correctamente en todos los iOS y que los usuarios con versiones avanzadas podrán disponer de toda la funcionalidad. Y ya de paso, permites compilar para simulador aunque estés usando librerías de terceros y te ahorras los muchos GBs que ocupa cada SDK :)