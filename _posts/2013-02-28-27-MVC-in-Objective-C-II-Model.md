---
layout: post
title:  "MVC in Objective-C (II): Model"
date:   2013-02-28 11:30:34
categories: 
    - mvc
    - objective-c
    - patterns
    - model
permalink: /blog/:title
---

Today, as part of the [MVC in Objective-C series](26-MVC-in-Objective-C-I-Introduction) that I am writing, I am going to introduce the best practices that I have found so far when dealing with the Model role in your iOS app. But lets first give a quick introduction about the Model role:

The Model role in MVC is the one responsible for dealing with the state of the app. It encapsulates the code responsible for managing data, applying business rules, etc. This layer is usually the most decouple part of your app as it does not communicate directly with the controllers or views (only indirectly when the other layers request its information).

When implementing the Model in an iOS app you typically have to deal with different kind of problems:

#### Formatting (modeling) the information

Data can be formatted in many different ways. I have seen many developers to often use generic data structures such as dictionaries and arrays to store model information. While this could work for a very simple model, it is a huge issue when the model grows. Instead, use a simple class with just a few properties to access the info, nothing else. It provides developers compilation type checks, it allows you to clearly see what the structure of the stored information is and will help you easily maintaining and extending it in the future. 

For example, if you have user data, use a User class with firstName and lastName properties among others. You could also create a getter for fullName that append both strings together, but a registration method should not be part of it. For God’s sake, do not use a dictionary with keys such as “name”, “email”,... it will become a mess sooner than you think!

    NSLog(@"name: %@", user.name); //YES
    NSLog(@"name: %@", [user objectForKey:@"name"]); //NEVER!

What looks better? So please help your teammates!

#### Managing local data

Local data can be stored in many different ways. From very simple formats such as csv to more complex ones such as plists or even databases. My personal preference is to use plists for very simple things (configuration files generally) and use SQLite databases for the rest. In particular, even if I was quite sceptical about it at first, I would recommend using CoreData for your data model unless you really need to get something very particular from your datasource. It is true that adding CoreData means dealing with many new problems and even performance issues, but you also get some very nice features such as [NSFetchedResultsController](http://developer.apple.com/library/ios/#documentation/CoreData/Reference/NSFetchedResultsController_Class/Reference/Reference.html), lazy load relations, data modeling tools,... that worths the price. 

Anyway, leaving the data store aside, an important thing is how you access/modify that data. I have seen many projects making queries or having to populate records from the database outside the Model layer, which is clearly not a good option. You should always provide data accessors for your model, as well as a clear and straightforward way to store or update new objects. How? well, you could have a global factory class, but my personal preference is to create a set of categories for your data model classes providing methods like “fetchAllUsingPredicate”, “createInstance“, “save”,... This categories could be placed inside a different folder, and imported in your .pch file to be available for all classes. Then, of course, generic implementations could be abstracted into a superclass to avoid duplication of similar code.

For example, continuing with the previous User class, the data model could be defined as:

    //  User.h
    @interface User : NSObject
    
    @property (nonatomic, strong) NSNumber *identifier;
    @property (nonatomic, strong) NSString *email;
    @property (nonatomic, strong) NSString *firstName;
    @property (nonatomic, strong) NSString *lastName;
    
    @end

And a simple access model could be:

    
    //  User+LocalAccessors.h
    @interface User (LocalAccessors)
    
    + (NSArray *)fetchAll;
    + (NSArray *)fetchAllUsingPredicate:(NSPredicate *)predicate;
    + (User *)createInstance;
    - (void)save;
    - (void)delete;
    
    @end

Then use it like this from any other part of the code:

    NSArray *users = [User fetchAll];
    NSLog(@"users: %@", users);

As you see, there are no other methods in these files (only the corresponding .m implementation) and all the logic behind the data management is hidden from the rest of the app. Even more, you do not need to know the store that the fetch is using underneath! It could be a CoreData model, but it could also be loaded from a simple plist file or even from an in memory cache with no persistent backup. It doesn’t matter, the rest of your code will work anyway as long as you implement the defined accessor methods.

Moreover, all this accessor methods could  be defined in a protocol and use it in every single model entity to provide uniform accessor methods for all your model. 

Being said that, if you are working with CoreData (as I do), I would recommend you the use of [MagicalRecord](https://github.com/magicalpanda/MagicalRecord). It defines a very good set of accessor methods and other useful utilities to make the use of CoreData much better. And it is implemented following similar conventions to the ones just explained!

#### Managing remote data

If managing local data can easily mess up your code, managing remote data makes things even worse because networking is essentially the same problem but in an asynchronous and failure prone environment. Besides, most apps provide a caching mechanism that will store the remote information locally to retrieve it faster next times or without Internet connection. Lot of complex stuff to handle, so how should we approach such a thing to do it properly?

Well, the most important problem here is the asynchronism. In an hypothetical synchronous networking environment we could manage errors as in any other place, and we could easily cache responses to allow the offline connections. What is more, we could use the same pattern explained above (in local data management) to provide an abstract way to fetch, update or delete objects. As we explained before, a “fetchAll” request could be accessing a plist file, as well as an external REST API without any difference. Unfortunately, networking is asynchronous, so we need to solve this issue in an elegant way.

Objective-C uses many different patterns for handling async calls. Three of them are delegates, notifications and [KVO](http://developer.apple.com/library/ios/#documentation/cocoa/conceptual/KeyValueCoding/Articles/KeyValueCoding.html), which are not particularly bad (well, KVO is), but not really clean to implement. They require you to write custom methods on your classes to receive the notifications/callbacks and they will quickly get complex if you handle multiple networking operations. A better solution though are blocks. By using blocks you can define the exact code that you want to execute when the operation is finished, very much like a synchronous operation. Of course, this freedom can also mean chaos, so we need to create very specific rules on how to design such a solution.

In my case, what I like the most is to follow the exact same approach than the one used in the local (synchronous) data accessors, but this time expose the results within a passed block. Example:

    // User+RemoteAccessors.h
    typedef void(^UserObjectBlock)(User *user);
    typedef void(^UserArrayBlock)(NSArray *results);
    typedef void(^UserErrorBlock)(NSError *error);
    
    @interface User (RemoteAccessors)
    
    + (void)loadAllOnResult:(UserArrayBlock)resultBlock onError:(UserErrorBlock)errorBlock;
    + (void)createInstanceOnResult:(UserObjectBlock)resultBlock onError:(UserErrorBlock)errorBlock;
    - (void)saveOnResult:(UserObjectBlock)resultBlock onError:(UserErrorBlock)errorBlock;
    - (void)deleteOnResult:(UserObjectBlock)resultBlock onError:(UserErrorBlock)errorBlock;
    
    @end

As you can see, the exact same idea applies here, but you need to handle it asynchronously:

    [User loadAllOnResult:^(NSArray *users) {
    NSLog(@"users: %@", users);
    } onError:nil];

But, what if we want to show cached results while the networking operation is running? then, all we need to do is to define the same remote accessors but with a synchronous response as follows:

    //  User+RemoteAccessors.h
    @interface User (RemoteAccessors)
    
    + (NSArray *)loadAllOnResult:(UserArrayBlock)resultBlock onError:(UserErrorBlock)errorBlock;
    
    @end

Which will return an array of cached users immediately and the server response after a while (within the block). Simple and straightforward!

Of course, implementing the RemoteAccessors category this way can be complex, but fortunately there are a plethora of quite good solutions out there to help us with it. To name my two personal favorites:

*   [**AFNetworking**](http://afnetworking.com/): It is a general purpose library to perform network operations. It supports blocks and it has a lot of contributions with different features. If you want to implement the mentioned accesors this is probably the best way to go. 
*   [**RestKit**](http://restkit.org/): This library is lot more complex than AFNetworking. It does not only handle networking operations, but it also takes care of the data mapping, relations and many other issues (including CoreData objects). It is a very good solution if your remote API follows a standard REST and if it is not too complex. However, after working with it in a couple of projects, I found it very hard to make advance stuff such as conditional caching and dynamic mappers, but of course it can be done. Anyway, RESTKit introduces the idea of encapsulating the mapper in a different class than the model that it corresponds to, which I think is a very good idea. For example, this is a mapper that I use in one of my projects that will set all the information needed to transform a simple networking call into an object model:

```
    @implementation SDUser (APIConnectorMapping)
    
    + (NSDictionary *)mappingObjectsByPattern {
        NSMutableDictionary *mappingDict = [NSMutableDictionary dictionary];
        
        RKObjectMapping *mapping = [self attributeMapping];
        [mappingDict setObject:mapping forKey:@"/auth/login"];
        [mappingDict setObject:mapping forKey:@"/auth/register"];
        [mappingDict setObject:mapping forKey:@"/auth/user"];
        return mappingDict;
    }
    
    + (RKObjectMapping *)attributeMapping {
        RKObjectMapping *mapping = [RKObjectMapping mappingForClass:[self class]];
        [mapping mapAttributes:@"customerId", @"lastName", @"firstName", @"email", @"mobilePhone", @"street", @"city", @"birthday", nil];
        [mapping mapKeyPathsToAttributes:
         @"id", @"identifier",
         @"key", @"sessionKey",
         @"created", @"sessionDate",
         @"postalCode", @"postCode",
         @"number", @"houseNumber",
         nil];
         [mapping mapKeyPath:@"defaultDelivery" toRelationship:@"defaultPickupPoint" withMapping:[SDPickupPoint attributeMapping]];
    
        return mapping;
    }
    
    @end
```

In brief, use categories to hide networking logic (or an external data manager if the calls are too complex), provide blocks for handling the asynchronous operations, and encapsulate the mappers in a different piece of code to be able to modify them easily when needed.

#### Global events

So far we have seen how to access information on demand, when we need it. For example, the users example above could be used from a screen displaying a table of users. But what if we need to manage something more global? App events like a login action can be difficult to implement, specially when it can change at any point depending on server conditions or background threads.

The essence of the problem is very similar to the remote data accessing explained above, but the differences are that we might not be the “owners” of the request, we might not have any control over it and it could affect to many different subscribers. 

For example, following the User example, what if we want to provide a login call? we could do it following the same convention, adding to the category with the remote calls a method to make login and a block to handle the result. However, if the rest of the app needs to get notified, how can we do it? Once again the async handling options we have are delegates, notifications, KVO and blocks. However, this time blocks do not seem to fit so nicely because we should provide a way to register and unregister multiple blocks for the same event (ouch! I love blocks!). A similar problem applies to delegates, and KVO is a messy solution. So, for this kind of application events, I prefer to use notifications. This way you could declare this global events inside you model, your login method will just need to post the notification when is done, while we keep the elegance of the block handling solution expressed above. But use them with caution though! they grow exponentially when you start implementing them! believe me :)

Login example:

    //  User+RemoteOperations.h
    static NSString *kUserLoginNotification = @”kUserLoginNotification”;
    @interface User (RemoteOperations)
    
    // This method will fire kUserLoginNotification on success
    - (void)performLoginOnResult:(UserObjectBlock)resultBlock onError:(UserErrorBlock)errorBlock;
    
    @end

Your calling class will perform login as expected:

    [user performLoginOnResult:^(User *user) {
    NSLog(@"logged in user: %@", user);
    } onError:nil];

while other parts of your app could subscribe to changes by:

    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(loginChanged) name:kUserLoginNotification object:nil];

#### Business rules, flows and others

Not always your model is about accessing and writing information. Sometimes it could require to apply complex business rules or flows. For this cases I like to create a new class (or set of classes) that will perform those activities -as everybody, I guess. I usually suffix it with the word “Manager” to denote its different nature.

Anyway, what I wanted to comment here is that for this kind of classes I would recommend always to abstract your calls as much as possible and implement [IOC (Inversion Of Control)](http://en.wikipedia.org/wiki/Inversion_of_control) patterns when appropriate to decouple your code from this specific piece of code -if you do not know much about IOC, [dependency injection](http://en.wikipedia.org/wiki/Dependency_injection), [service locators](http://en.wikipedia.org/wiki/Service_locator_pattern) or [factory methods](http://en.wikipedia.org/wiki/Factory_pattern) you should look for them now, they are very helpful-. Otherwise, changing your logic in the future will be difficult and tricky. Do things properly from the beginning and it will pay off.

#### Conclusion

iOS apps are usually simple regarding the functionality contained in the model layer. However, the special nature of the apps (distributed and with error prone connections) make the problem quite complex. To avoid having problems I suggest you follow strict conventions inside your model layer, especially isolating the code that handle the data itself (in plain Objective-C classes), the access to a local data source, the access to a remote data source, and the global events. No matter what conventions you use, the implementation details should always be completely hidden from the caller and generic enough to be able to change your implementation details if needed. I posted a few simple examples of interfaces that could help you abstracting the implementation details, and at the same time I mentioned some libraries that could be used to implement their details. 

After all, it would not be strange if after some time of development you need to switch to a different networking library, database manager or change your business rules. For example, I am right in the middle of changing the networking layer in one project and is amazing to see that I do not have to change a single line of code in the controllers or views, only in the implementation details of the networking accessors, and everything just works as it was working before. Remeber, you can implement your model layer in many different ways, but only by following good design principles you can do such a change without compromising the rest of your code -and keeping it clean at the same time-.

Keep tunned for my next post about MVC, the View role.