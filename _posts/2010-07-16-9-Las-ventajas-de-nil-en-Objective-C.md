---
layout: post
title:  "Las ventajas de nil en Objective-C"
date:   2010-07-16 11:30:34
categories: 
    - deployment target
    - ios
    - ipad
    - iphone    
permalink: /blog/:title
---

Para los que venimos de lenguajes como Java o C++ donde un objeto null es un peligro, cuando llegamos a Objective-C descubrimos la gran ventaja que tenemos en este lenguaje.  
  
En objective-C, un puntero apuntando a nil es un objeto sobre el que se pueden invocar métodos.  
  
Es decir, cualquiera de las siguientes líneas son perfectamente válidas:

    
    id objeto = nil;
    [objeto metodo];
    [nil metodo];

Quizá te preguntes que pasa si se ejecuta un método sobre nil? pues nada, cualquier ejecución sobre nil simplemente devuelve a su vez un nil. Por ejemplo, en este código:

    
    id objeto1= [nil metodo];
    id objeto2 = [objeto1 metodo];
    

En ambos casos, objeto1 y objeto2 serán nil a su vez, pero ningún error habrá ocurrido (en vez de los "Null pointer exception" que tendríamos en Java).  
  
  
Quizá pienses que esto tampoco aporta nada, ya que simplemente con hacer una comprobación de si objeto1 es nil antes de invocar nada sobre él sería suficiente, pero lo cierto es que se convierte en algo muy práctico, cómodo y sobre todo robusto frente a errores. No significa que nunca tengas que comprobar si algo es nil, ya que es probable que en muchos casos tengas que hacer una ejecución diferente en un caso o en el otro, pero sí ahorra muchas comprobaciones innecesarias y por tanto código, además de reducir los errores por un "descuido de comprobación de null".  
  
Un ejemplo muy claro de este buen uso es en las properties y en los deallocs. Por ejemplo, un dealloc típico podría ser algo como:  
 

    
    - (void)dealloc {
    	[titleLabel_ release]; titleLabel_ = nil;
    	[subtitleLabel_ release]; subtitleLabel_ = nil;
    	[introLabel_ release]; introLabel_ = nil;
    	[authorLabel_ release]; authorLabel_ = nil;
    	[dateLabel_ release]; dateLabel_ = nil;
    	[commentsLabel_ release]; commentsLabel_ = nil;
    	[textLabels_ release]; textLabels_ = nil;
    	[contentScroll_ release]; contentScroll_ = nil;
    	[multimediaView_ release]; multimediaView_ = nil;
    	[remoteImageView_ release]; remoteImageView_ = nil;
    	[multimediaFooterLabel_ release]; multimediaFooterLabel_ = nil;
    	[twitterAlertViewController_ release]; twitterAlertViewController_ = nil;
    	[bannerView_ release]; bannerView_ = nil;
    	[super dealloc];
    }
    

Imaginate la cantidad de líneas extras que llevaría el dealloc anterior si tenemos que comprobar cada variable antes de liberarla para evitar provocar un "null pointer exception" sobre variables que no estén creadas!  
Por cierto, puedes ver que después de cada release hago una igualación a nil. Esto no es obligatorio pero sí es una muy buena práctica porque si este release se hace en otro punto es bueno que el puntero lo apuntemos a nil para que no de errores en un hipotético uso posterior.  
  
  
Consideraciones  
  
A pesar de que la ejecución de métodos sobre nil sea correcta, no significa que lo sea por fuerza en los parámetros de un método. Por ejemplo, en un array no podemos agregar un nil como objeto ya que lanza un error:

    
    [array addObject:nil]; // Error
    

Por esto, aunque podemos relajarnos con las comprobaciones a nil al invocar métodos, hay que tener cuidado cuando se usa como parámetro de un método de otro objeto. El nil es mucho mejor que el null de Java, pero no perfecto ;)