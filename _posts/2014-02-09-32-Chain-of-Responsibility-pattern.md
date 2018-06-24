---
layout: post
title:  "Chain of Responsibility pattern"
date:   2014-02-09 11:30:34
categories: 
    - cocoa
    - ios
    - objective-c
    - patterns
permalink: /blog/:title
---

Today I am going to write about a pretty uncommon but very useful pattern in iOS apps, called **Chain of Responsibility**, exploring how we can take advantage of some Cocoa methods to implement it easily and how/when to use it in your iOS apps to reduce dependencies in modular apps. 

But before starting, I want to acknowledge [Saul Mora](https://twitter.com/casademora) because it was on one of his presentations (about a year ago) where I first saw this pattern in action. He applied it for the Model layer, while I do it for the Controller flow mostly, but the main idea behind the whole post is basically the same and you could apply it for either case. I suggest you spend some time checking [his presentation](http://vimeopro.com/360conferences/360idev-2013/video/74745264) about this pattern, very inline with this post.

So that been said, let’s start with some basic concepts.

Intro to the CoR pattern
------------------------

From [Wikipedia](http://en.wikipedia.org/wiki/Chain-of-responsibility_pattern)

In object-oriented design, the chain-of-responsibility pattern is a design pattern consisting of a source of command objects and a series of processing objects. Each processing object contains logic that defines the types of command objects that it can handle; the rest are passed to the next processing object in the chain. A mechanism also exists for adding new processing objects to the end of this chain.  
This pattern promotes the idea of loose coupling, which is considered a programming best practice.

As we can see from the previous definition, we have two entities in this pattern: 

*   **Commands:** This entity contains the information about the actions to perform and data associated to it.
*   **Processing objects:** This entity receives a command to handle and performs some logic on it. It also contains some kind of reference to the next node in the chain.

The idea is that a command is dispatched on some element of the chain, and it navigates up until one of the processing objects handles it. This way we can make very loosely coupled and reusable components because they can fire commands without knowing who will handle it (if anyone), but we will see some example of this later.

CoR in Cocoa
------------

In iOS/Mac apps, this pattern is extensively used by the UI. It is implemented as part of the [UIResponder class](https://developer.apple.com/library/ios/documentation/uikit/reference/UIResponder_Class/Reference/Reference.html), which is the base class for all UI elements such as UIView, UIViewController, UIWindow and even the UIApplicationDelegate (similarly on Cocoa with the NS counterpart). UIResponder contains references to the parent element in the chain (named nextResponder), forming a tree with the UIApplicationDelegate as the root node and the deepest UIView pieces as the leafs. So, a typical tree branch would go from a button to the superview, and from there to the view controller, the navigation controller, the window and then the AppDelegate. Extended documentation [can be found here](https://developer.apple.com/library/mac/documentation/cocoa/conceptual/eventoverview/EventArchitecture/EventArchitecture.html).

Besides the nextResponder attribute, UIKit also provides a few methods for sending an action up in the chain, which is commonly used for touches and similar UI events. The most interesting ones are:

*   \[UIApplication sendAction:to:from:forEvent:\]: This method sends an specific action to the current First Responder in your app.
*   \[UIResponder canPerformAction:withSender:\]: Returns a BOOL indicating if the receiver can handle an specific action.
*   \[UIResponder targetForAction:withSender:\]: Returns the first element in the responder chain that can handle the action. It calls canPerformAction:withSender: on each next responder to determine whether it can invoke the action. iOS7+ only.

Lastly, another interesting note to make is that the responder chain is also exposed in Interface Builder, which could help as connecting actions in our views with the corresponding handler somewhere else in the responder chain.

Implementing CoR for modular apps
---------------------------------

So we already have processing objects (the UIResponders in the responder chain), we can send commands down in the responder chain by using \[UIApplication sendAction:to:from:forEvent:\] or find the handler up in the chain by the use of \[UIResponder targetForAction:withSender:\]. 

However, there are a few things that are not solved yet:

1.  In order for  \[UIApplication sendAction:to:from:forEvent:\] to work we need to set the firstResponder on the firing object, which is not always possible (by default UIResponder return NO to canBecomeFirstResponder, except for some specific cases like UITextFields with side effects).
2.  \[UIResponder targetForAction:withSender:\] is iOS7+ only, so we need to figure out something similar if we need support for iOS6 and down.
3.  Sending actions is most of the times not enough. You normally will want some extra contextual information together with the action.
4.  UIKit Responder chain does not allow changes.

All the points above could be solved with some hacking. For example, we could swizzle the \[UIResponder canBecomeFirstResponder\] to return YES by default, so we could set the first responder to any responder in the chain and allow the sendAction:to:from:forEvent: to pass the command to any point in the chain, and we could implement our own version of \[UIResponder targetForAction:withSender:\] for iOS6 and lower. However, all these hacks look like an abuse to me.

Instead we  can easily **implement our own dispatcher methods**, similar to the ones provided by Cocoa, that with just a few lines of code will look for the next handler in the responder chain and pass it our specific command. This approach gives us freedom to define the commands as they best fit in our project, containing any **contextual information**,  and we do not need to mess with the system’s methods. With a little extra effort we could “remember” the last firing object and command to allow a similar behaviour to the \[UIApplication sendAction:to:from:forEvent:\] but without hacking in the \[UIResponder canBecomeFirstResponder\] method.

[Here you can see a very basic implementation](https://gist.github.com/angelolloqui/8807330) that allows you to send a custom made command up in the chain by only using the nextResponder compatible with old iOS versions and following similar conventions to those provided by Cocoa. 

However, this simple construction does not allow us to modify the responder chain, which can be very useful for dynamic behaviours or adding extra nodes at the end of the chain. If you only want to add a single responder at the end you can easily override the nextResponder method in your UIApplicationDelegate like:

    //MyAppDelegate.m
    - (UIResponder*)nextResponder {
    	return _myCustomNextResponder;
    }

Still, deeper changes in the chain will be difficult (and dangerous) to implement. For that reason, I implemented [a different version that “creates” a **second chain**](https://gist.github.com/angelolloqui/8807692), accepting any NSObject as part of the chain, and providing setters to **change the chain on runtime**. Of course, this second chain relies on nextResponder by default, so the chain will be set up automatically as the UIKit nextResponder chain does for any UIResponder object. 

How to use it
-------------

So now we have the chain in place. How can we use it?

**Sending** a command with the previous code is as simple as:

    //In your command firing object. Example: VC as a response to an event
    - (IBAction)myEventFireAction {
    	[self sendCommand:[MyConcreteCommand command]];
    }

While **handling** the command could be:

    //In your command consumer. Example: AppDelegate
    - (void)performMyConcreteCommand:(MyConcreteCommand *)command {
    	//Handle the command
    }

For this example, I am using a specific **command subclass** defined as:

    @interface MyConcreteCommand : AGCommand
    @end
    
    @implementation MyConcreteCommand
    - (SEL)action {
        return @selector(performMyConcreteCommand:);
    }
    @end

We could also define commands to return (IBAction) so we can use them from Interface Builder, but then we need to realize that in that case the firing object will be the UI element that fired the action (UIButton normally) and not the command itself, making everything a little more complex because you need to check the sender type and you can not pass contextual information. Moreover, in case you use the “parallel” chain commented in the previous section you will be using to different chains depending if the action is fired from the IB or from code. Hence, **I personally prefer to avoid IBAction commands handlers** and fire them all from source code as shown above, it adds a little more code to your controllers but keeps things explicit and simple.

Of course, you could create one command “per action” or a generic command that stores information for performing multiple actions. My personal preference is to **create multiple commands**, because it helps separating concerns, and I **support it with a protocol** where I define all command actions to improve autocompletion and refactoring in the IDE.

For example:

    @protocol MyCommandConsumer 
    @optional
    - (void)performMyConcreteCommand:(MyConcreteCommand *)command;
    @end

And finally, you can even provide a **factory method** in your command class, that contains the logic for command creation and use the command similarly to other class clusters in Objective-C. Factory methods are very useful for generic components as it allows the component to fire multiple actions without actually knowing anything about them.

Advantages
----------

The whole pattern is very interesting in multiple ways. Lets explore some of the benefits:

*   **Reduced amount of dependencies** (the only dependency is to the CoR pattern itself), which results in very clean and reusable code. Your components or controllers do not depend on others, and completion blocks and delegates can be replaced by commands easily.
*   **Better Separation of Concerns**: By using a pattern like this one you could very easily create components that only do whatever they are supposed to do, and fire commands for actions that they are not ready to handle or might result on a side effect. Network access, navigation controller push/pop, global events,... they can all be self contained in nodes at the end of the chain letting our components with no dependency on them.
*   **Pluggable architecture and dynamic behaviours**: The use of commands helps when you have to design an application composed by multiple modules that can be plugged in or out (even on runtime). You could easily place your modules behind the UIApplication delegate responder and you could  adapt the chain to your configuration on compile time or runtime to create a fully customizable application. We will dig into this idea with a real example later on this post.
*   **Allows multiple handlers for same request**: By using commands you can add functionality to your apps without changing the original code, just by adding new responders in the chain in the proper places and let the fire the command up. This could be very useful for things such as analytics, where you do not want to modify the behaviour but add an extra handler for the tracking.

Disadvantages
-------------

*   **Extra complexity**: The most clear disadvantage of the pattern is the complexity added to the project. It requires new developers to get acquainted with the pattern before being able to work with it, so the **learning curve** is also deeper than in a regular project. In cases where the project is simple the advantages you will get from the pattern will not pay off for the extra complexity added.
*   **Difficult debugging**: Even when you are familiar with the pattern, debugging commands is tricky. In regular projects an action is normally consumed within the limits of the view/viewcontroller associated to it (or its delelgate/blocks handlers), but with the commands the action could be consumed in a completely unrelated class. It is comparable to debugging  Observers or Notifications, although the use of methods like targetForCommand can help a lot.

A real life example
-------------------

OK, I exposed a few use cases already but you might be wondering if I use this pattern in real apps. The answer is yes, but let me give you an example.

For my last project I had to build an app that could be resold as a white label app for multiple customers. This **app should be able to add, remove or modify** some of the features and navigation flows **depending on an external app configuration file**. So for example, for app A, we could have lists of products that when tapped go to a product detail, while for app B the same action would add the product to a shopping basket and for C no action is performed. 

The whole app has been designed with the **CoR** pattern using the code explained above. So I have generic components like DataSources, that encapsulate the logic behind a single concern (managing the table with a giving array of data objects for example), and that **fires commands when interactions occur**. When the app is started, I load some **extra responders** (modules) at the end of the chain based on the app configuration (some modules for app A will be different than for app B or C while others are the same) and let the chain of responsibility to handle the cell tap. 

So the whole flow goes like this:

1.  App is started, app A, B and C load different modules in the chain
2.  A generic datasource draws all cells for the products
3.  User taps on a product cell
4.  A command is created based on the object associated to the cell (using a generic factory method using the cell object, so no dependencies here)
5.  The command is fired and goes up in the chain looking for a responder that can handle it
6.  For app A, a module with a product detail push behaviour will be found, while for B it will be one with the model logic for adding to the shopping cart and for C no responder or a non-op. 

What is even better, taps on some cells might behave differently in some specific situations (inside some special view controllers). In this case, the controller can easily consume the commands and so adapting the behaviour of your app easily.

Then, at some point we had to add **analytics tracking**, which resulted in just **an extra module** on the chain getting all desired commands, tracking the event and sending it up in the chain to let them be consumed by another module. A very elegant analytics solution with all **code contained within the same file** instead of tens of lines spilled around all controllers for tracking user actions, and no “if” statements checking whether we are in app configuration A, B or C to do this or that action instead.

The result is source code that has **very little dependencies**, that can be **plugged in/out easily**, **reused** across multiple modules, **flexible** to change behaviour when needed and with very **clear and defined concerns** for each component .

None of these things can be easily done only by the use of blocks, delegates or notifications.

Conclusion
----------

CoR, as any other pattern, has his **benefits and drawbacks**. In this case, this pattern is clearly **not ideal for simple projects** or those with a very monolithic structure where modules and plugins are not considered and code reuse is scarce. 

However, when used properly it can give you multiple benefits. Especially it **helps with a more modular and dynamic** app architecture, promote **code reuse** and keep **separation of concerns** in a clean and elegant way. Moreover, Cocoa already implements it internally for their UIResponder objects, so with very little effort we can take advantage of it for our own purposes.

Experiment with it and feel free to get in touch with me with your experiences/opinions! I will be happy to see more people using it :)