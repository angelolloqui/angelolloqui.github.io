---
layout: post
title:  "Liberación de memoria en IBOutlets"
date:   2010-07-06 11:30:34
categories: 
    - ios
    - memory management
    - objective-c
    - interface builder
permalink: /blog/:title
---

Hoy voy me he encontrado con un problema en el trabajo relacionado con la liberación de memoria y la convención Ownership. Bajo esta convención, las clases solo son responsables de liberar aquella memoria que reservan directamente (mediante alloc, retain o copy), pero resulta que no siempre es así.

El problema surge cuando nos enfrentamos a un proyecto iPhone/iPad donde tenemos parte del código creado de forma gráfica mediante Interface Builder. Al usar IB las variables se enlazan con el código mediante el etiquetado de las mismas con la palabra clave IBOutlet sin necesidad de reservar memoria.  
  
Es decir, si tenemos un UILabel que querecemos enlazar desde IB, lo que hacemos es declararla de forma:

    
    IBOutlet UILabel *label;

y desde Interface Builder conectarla con nuestro label creado gráficamente.  
  
Hasta aquí todo OK, el problema aparece cuando te enfrentas a la liberación de memoria. Bajo el patrón ownership, como no hemos hecho nosotros la reserva de memoria, no deberíamos hacer nosotros su liberación, ya que de hacerlo podemos estar decrementando en exceso su retain count y acabar liberando antes de tiempo este objeto (con el consiguiente BAD ACCESS). Pues bien, parece que con estas variables IBOutlet el patrón Ownership no se cumple, porque es el desarrollador el responsable de su liberación en el dealloc, y opcionalmente en el viewDidUnload.  
  
Podeis ver la documentación oficial aquí:  
[http://developer.apple.com/iphone/library/documentation/Cocoa/Conceptual/MemoryMgmt/Articles/mmNibObjects.html](http://developer.apple.com/iphone/library/documentation/Cocoa/Conceptual/MemoryMgmt/Articles/mmNibObjects.html)  
  
  
En resumen, las variables declaradas IBOutlet deben ser liberadas explícitamente por el desarrollador, y además Apple recomienda usar una property tipo retain en su declaración y liberarlas también en el viewDidUnload para mejorar la gestión de memoria en situaciones de Memory Warning.