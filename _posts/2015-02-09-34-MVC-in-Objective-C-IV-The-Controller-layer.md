---
layout: post
title:  "MVC in Objective-C (IV): The Controller layer"
date:   2015-02-09 11:30:34
categories:  
    - mvc
    - objective-c
    - patterns
permalink: /blog/:title
---

#### Disclaimer

It has been a long time since I started with the [MVC post series](./26-MVC-in-Objective-C-I-Introduction). It is not that I didn’t want to write about it, but by the time I was supposed to start with this post [Objc.io](http://www.objc.io/issue-1/) came with a very good issue on [View Controllers](http://www.objc.io/issue-1/), which made my post almost irrelevant as most of the ideas were already mentioned there. Anyway, after receiving quite a lot of messages asking for this post, I finally decided to spend some time and try to address some new points and explain those that are important. That being said, if you haven’t read objc.io issue do not hesitate because it is worth the time and an excellent complement for this post.

The controller role
-------------------

In MVC, the Controller is the part of your software that **communicates** your [**Model**](./27-MVC-in-Objective-C-II-Model) layer with your [**View**](./29-MVC-in-Objective-C-III-The-view-layer) layer, and it is by far the most abused role in iOS project. But lets first define how a good controller should look like:

A good controller should know how to request data to the model but not its internal details or how to fetch it; it should know what view to use but not how to draw it; it should be the one receiving events from one layer and passing messages to the other layer but not the one creating the events. In brief, a Controller should be the glue needed to connect the Model and the View, but ideally it should not do anything else than that.

However, in iOS projects we often find controllers that make lot more duties that the ones they should. This is not a coincidence: delegation patterns used in most of Apple’s components can be easily implemented on controllers, and the UIViewController API exposes methods like viewWillLayoutSubviews that should concern the View role only. Even the UIViewController name denotes this lack of real distinction between views and controllers!

Nevertheless, writing good controllers make your code lot more **reusable** (not only the controllers but the other layers also), **easier to test**, more **maintainable** and more **flexible**. So, lets review some important guidelines:

### Define controllers by functionality and not by UI elements

When defining a controller it is important to think about the specific **behaviour/features** that we are trying to build rather than the specific UI elements that we need to use. 

For example, if you are implementing a controller to show a table of results, do not call it MyTableViewController and build it to be used with a table only. Instead, try defining it as a controller that could potentially be used to present data inside a table, a collection view or any other component that can handle multiple elements. You could put in your controller the required logic to request data to the Model and the code to send that data to the View, but the View API should be generic enough to not expose any implementation details of it. If you find yourself changing the controller code when you move from a table to a collection view then you are probably not doing enough in your View layer.

As a tip to start, I suggest you move out of your controllers the UITableViewDataSource, UITableViewDelegate and all the protocols/methods alike. What I normally do is define a **generic datasource** that is instantiated inside the xib file and contains all references to the table/collection view that it will handle. From the controller perspective there are no views linked to it, just a datasource object that provides generic methods like reloadWithData:(NSArray *)data, and it is the datasource the one that will transform the Model objects in the array into concrete UITableViewCells if it is used with a table or UICollectionViewCells if it is used with a collection view, with no changes on the controller. The datasource is generic, so the same one can be reused across the whole project, reducing bugs and development time.

A real and fully functional example of such view controller is here:

    //MOACartViewController.h
    @interface MOACartViewController ()
    @property (strong, nonatomic) IBOutlet MOADataSource *dataSource;
    @end
    
    //MOACartViewController.m
    @implementation MOACartViewController
    
    - (void)viewWillAppear:(BOOL)animated {
        [super viewWillAppear:animated];
        
        __block typeof(self) weakSelf = self;
        [MOFCartDAO loadCartOnResult:^(MOFOrder *result) {
            [weakSelf.dataSource reloadWithData:result.orderItems];
        } onError:^(NSError *error) {
            [weakSelf.dataSource reloadWithData:@[error]];
        }];
    }
    
    @end

(You can find an implementation of how I use such a DataSource in other projects combined with some other techniques [here](https://gist.github.com/angelolloqui/88647fdfaae19f9d6c89))

As you can see, this controller is used to fetch a cart from server and present the list of items, but **it does not know nor expose any implementation/UI details**. In fact, this same controller can be used for example in iPhone together with a table and in iPad in a collection view without any single change on the controller. Ultra slim controller!

The same idea could be used to define other view components that, through **generic APIs**, move the view details away from the view controller.

### Do not write any view related code

Linked to the previous point, view related code should not be placed inside view controllers. And by view related code I mean any code used for creating views and subviews, setting their colors, changing layouts, animating things up and down,... Of course your controller will need to have an associated View object and it might need to do very basic layout to put it in place, but you should keep it to the minimum extent possible. If you are setting colors, fonts, texts or changing layouts/frames from your view controller then you are mixing responsibilities. **Keep the view management in the view file** (either a xib file or a UIView subclass), provide **generic APIs** that are not dependent on the UI elements used and you will be able to reuse your controllers with different views easily. 

### Do not write any model related code

By model I mean anything that handles data or state. So for example, if you need to fetch data from a third party service, write a proper Model layer that gives you the data ready to be used by the View, as we explained in first post or as shown in the previous example with the cart items. Your controller will be the one that **knows what kind of data to request** and how to pass it to the view, but it is **not the one actually fetching** or drawing. 

### Do not write platform dependent code

If your controller does only think about functionality and it does not contain view related code (as explained before), then it should not need any platform dependent code. There are basically two forms of **platform dependent code**:

**1\. Different view layouts:**

    if (IPAD) {
        _myCustomView.frame = CGRectMake(0, 0, 200, 200);
        _title.text = data.veryLongDescription;
    }
    else {
        _myCustomView.frame = CGRectMake(0, 0, 50, 50);
        _title.text = data.shortDescription;
    }

This code is **wrong** in many ways. First, because as we saw before, **view manipulation** should be done in the View layer, not in the controller. Second, **hardcoding** values like the frame diminishes our controller reusability, they should always be relative to the other components of the view (again, part of the view logic). Instead, try to set proper **autoresizing** mask or **autolayout** properties in your View level. If that is not enough, then use **different xib** files (~iPhone and ~iPad, or Size classes) or move this logic to a **UIView subclass** if you are not using Interface Builder. You can also create **multiple IBOutlets**, one for the long title and another for short title, and set the text to both no matter the device (~iPhone XIB could set one, while the ~iPad could set the other).

**2\. Different event handling:**

    - (IBAction)myCustomAction:(id)sender {
        if (IPAD) {
            //Do action A
        }
        else {
            //Do action B
        }
    }

This code denotes a **failure on splitting responsibilities**. If you need to perform Action A in one case and Action B in another, then just create 2 methods that clearly state their responsibility in the method name. Then, from your View just **connect the proper method for each case**. It will simplify things because you will clearly see what does the method do just by seeing the name and it will not result on some different behaviour depending on platform that is only known when checking the controller’s method source code.

### Use composition with child controllers

Before iOS5 a controller was supposed to deal with a full screen view. However, that changed when Apple introduced child controllers. Nowadays there is no point on keeping a controller “per screen” and you should in fact decompose your UI in many different sections controlled by their own controller and view. 

You can **add a child** view controller by doing:

    - (void) displayContentController: (UIViewController *)contentVC {
       [self addChildViewController:contentVC];
       contentVC.view.frame = [self frameForContentController];
       [self.view addSubview:self.contentVC.view];
       [content didMoveToParentViewController:self];
    }

And **remove** it with:

    - (void)hideContentController:(UIViewController *)contentVC {
       [contentVC willMoveToParentViewController:nil];
       [contentVC.view removeFromSuperview];
       [contentVC removeFromParentViewController];
    }

[More information here](https://developer.apple.com/library/ios/featuredarticles/ViewControllerPGforiPhoneOS/CreatingCustomContainerViewControllers/CreatingCustomContainerViewControllers.html)

The composite pattern is one of the best ones for building reusable components, and it is preferable over others like inheritance. 

Why? well, when you build with **composition** in mind you normally create a lot of **small** components with very **simple** and **clear** responsibilities. Because they are small and simple, they rarely require changes, they are easy to reuse and they can be tested with little effort. Then, when it is time to make use of them, you can easily add them in your xib file and connect them appropriately. If you need to replace some functionality for some specific screen or platform you can easily modify the composition to include other components or connect them differently.

On the other hand, If you use **inheritance**, you would create a base controller with some of the generic functionality, and then many different subclasses overriding existing behaviour or extending them. The main problem of this approach is that it requires a lot of **knowledge about the base class** (or classes because is common to have multiple levels of inheritance for complex views) and they need to expose a lot of **internal details** to make them customizable. Exposing internal details is exactly what you want to avoid to create maintainable code and it is a pain when you need to reuse it. Besides, all your subclasses will now depend on the base class, which is normally quite **complex**, and any change in there can potentially affect all subclasses in many different ways. Resulting code is less reusable and it is prone to errors when modified.

So, if you want to make your code reusable, **divide** your UI in small pieces, make them **generic**, **independent** from each other (using delegates or blocks in their API) and **connect** them creating a composition from a **parent controller that knows how the “orchestra” works** together.

### Understand and respect the life cycle

UIViewControllers have a well defined life cycle that every developer should respect and understand. A typical life sequence could be:

\- init → viewDidLoad → viewWillAppear → viewDidAppear →.... → viewWillDisappear → viewDidDisappear → dealloc

Of course there are many other methods that will/could be called like loadView, viewWillLayoutSubviews, willRotateToInterfaceOrientation or didReceiveMemoryWarning among many others. You can check the official Apple [documentation here](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIViewController_Class/index.html).

However, I often see in many projects (even in popular open source) that they do not follow the life cycle appropriately making code to behave in an unexpected way. For example, some **common errors** are:

*   **Set view properties in init** method (data, frame, colors, …) → view is not created yet, and should not be. It is lazy loaded.
*   **Assume viewDidLoad will be called only once** → the view of a VC can be destroyed on low memory conditions and recreated later, calling this more than once.
*   **Assume frames are properly set during viewDidLoad** → actual frames are not set yet but a default placeholder one is used, they could be incorrect.
*   **Do not revert operations** started on viewWillAppear on their viewWillDisappear and the alike. → code should be symmetric when possible
*   **Manually call to life-cycle methods** like viewWillAppear. → life-cycle methods are called by UIKit, never by the developer or unexpected results might happen.
*   **Do not call \[super\] methods**, especially common in viewWillAppear. → Always call super or unexpected effects might take place.

### Unidirectional dependencies

Dependencies are one of the causes why code is not reusable. When writing code, you must always have a very clear picture of how code relates to each other and implement a clean design where dependencies are **structured as a tree** rather than a graph, from **top to bottom** with **linear** and **unidirectional** dependencies.

For example, if controller A uses B, you should think of A as the owner and B as slave, where the slave (B) should never communicate directly to the owner (A) but through a generic API (delegates or blocks are most common approaches).

Another very interesting principle to take in mind is the [Law of Demeter](http://en.wikipedia.org/wiki/Law_of_Demeter)

Basically, what this principle says is that your components are shy, so shy that they can **only communicate** with their **direct friends**, but never to friends of friends. 

It is common to see code that pushes a view controller, and then access certain properties in that controller and operate on them. Things like:

    [self.detailViewController.selectedItem markUnread];

Are breaking the Law of Demeter and making use of multiple levels of abstraction. Your controller should only access the first level of the public API, and not anything else contained inside it. 

### Never modify parent contexts

Another very common mistake is to assume that your view controller will be always used in some particular way, and therefore make changes on a context that it does not own. 

For example, if controller A presents a modal controller B to select a city from a list of cities, you might be tempted to dismiss the controller when a city is selected from B itself. Instead, you better call a delegate (or block) when the city is selected and let A do the inverse operation that it did to present B (dismiss or pop). This way, your class B can be reused in other contexts and your push/pop operations are placed in the same context. 

Conclusions
-----------

**MVC is not a silver bullet**, and some Cocoa decisions make it sometimes even harder to implement it properly. However, if you are careful you can write apps that properly follows the MVC principles, resulting in more **reusable** and **maintainable** code. 

Lastly, if you are not happy with MVC, then explore other alternatives. A very interesting one worth mention is the [MVVM](http://en.wikipedia.org/wiki/Model_View_ViewModel), together with Reactive programming on the hands of [Reactive Cocoa](https://github.com/ReactiveCocoa/ReactiveCocoa). More on this in a future article.