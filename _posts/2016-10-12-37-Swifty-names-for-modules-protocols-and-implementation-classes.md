---
layout: post
title:  "Swifty names for modules, protocols and implementation classes"
date:   2016-10-12 11:30:34
categories:  
    - swift
    - clean code 
    - mvvm
permalink: /blog/:title
---

Naming conventions are very important in software development for many reasons, but especially **for readability**. [According to Wikipedia](https://en.wikipedia.org/wiki/Naming_convention_(programming)): 

> Reasons for using a naming convention (as opposed to allowing programmers to choose any character sequence) include the following:
>
> *   to reduce the effort needed to read and understand source code
>*   to enable code reviews to focus on more important issues than arguing over syntax and naming standards.
> *   to enable code quality review tools to focus their reporting mainly on significant issues other than syntax and style preferences.

So, what is a good name? This issue is controversial because there are multiple dogmas and styles out there. In Swift, despite being a young language, naming conventions are quite established already (with Swift 3), and includes guidelines about casing, naming parameters, methods,... However, there is **no “defacto” way to name your implementation classes** when you use protocols as a public abstract API and **how to group code inside namespaces/modules**. 

But let’s just explore an example.

Let’s imagine we have an app that shows some Restaurants around you. Now, let’s also assume that for this app we use an MVVM architecture, although the same concepts explained in here would apply to any other architecture chosen (MVC, MVP, VIPER,...).

With that in mind, we could have for example a RestaurantListViewModel and a RestaurantListViewController. If we want to have protocols for defining APIs and abstracting the current implementation (especially important for Tests), we would need to either rename the implementations to something like RestaurantListViewModelImpl and RestaurantListViewControllerImpl or rename the interfaces to something like IRestaurantListViewModel and IRestaurantListViewController. 

The problem with this approach is that either way, **you are fabricating a name by adding a suffix/prefix that does not improve the naming** but requires you to think about conventions and whether to use the interface or implementation class. Besides, the full bloated name **does not tell you much about module composition**. For example, imagine you have a class named SearchRestaurantsResultsView: what can you tell about the class? is it an interface or a concrete class? Is it a custom UIView or a UIViewController? Is it part of a Search module or part of a Restaurant module? Is there an associated ViewModel for that class or is it a standalone class?

To improve these issues, Paul Blundell suggests the usage of [nested interfaces in Java](https://www.novoda.com/blog/better-class-naming/). Despite being a very interesting approach, Swift does not allow to nest protocols inside other declarations, so things like:

    protocol A {
        protocol B {}
    }
    
    enum A {
        protocol B {}
    }
    
    struct A {
        protocol B {}
    }
    
    
    class A {
        protocol B {}
    }
    

Do not compile.

### Typealiases to the rescue

So, nested protocols are not an option, but can we somehow get the same result? Yes, **by using typealiases**. Let’s see a few examples:

    //Declaration
    enum Restaurants {
        enum List {
            typealias ViewModel = RestaurantsListViewModel
        }
    }
    
    class RestaurantsListViewModel {}
    

    //Usage
    let viewModel = Restaurants.List.ViewModel()
    

In this simple example, we are not using protocols yet, but a nested enum that can help us defining modules. Note that I have opted by an enum rather than a struct/class, because **enums can not be instantiated**, so it suits our problem better.

You can now see how instantiation is **much cleaner**, clearly showing module composition Restaurants → List → ViewModel. 

An added benefit is that autocompletion will just **show you results that apply to the current level of namespace**, so for example typing Restaurants. will only show submodules of restaurants (no implementation classes yet) and same applies to Restaurants.List, that will only display things like Restaurant.List.View or Restaurant.List.ViewModel, but no other classes starting with the word “Restaurant” will be shown like it normally happens without the namespace trick.

![](/ckeditor_assets/pictures/23/content_screen_shot_2016-10-12_at_11_04_28.png?1476281860)

Great! but, how would it look with an actual interface/implementation approach? Well, likewise, we can make use of **typealisases**:

    //Using protocols
    extension Restaurants.List {
        typealias View = RestaurantsListView
    }
    
    protocol RestaurantsListView {
        init(viewModel: Restaurants.List.ViewModel)
    }
    
    class RestaurantsListViewController: UIViewController, Restaurants.List.View {
        required init(viewModel: Restaurants.List.ViewModel){
            super.init(nibName: nil, bundle: nil)
        }
        required init?(coder aDecoder: NSCoder) {
            fatalError("init(coder:) has not been implemented")
        }
    }

And use it like:

    let vm = Restaurants.List.ViewModel()
    let vc = RestaurantsListViewController(viewModel: vm) 
    navigationController?.pushViewController(vc, animated: true)
    

Here, we can see how a simple View interface can be implemented. Note that I chose to use a different name for the interface (ending in View) than for the implementation class (ending in ViewController). This is intentional because from the interface perspective we do not need to know if the implementation is a UIViewController, but for the concrete class implementation it is important to distinguish between ViewControllers and other artifacts. Anyway, you could also have chosen something like IRestaurantsListViewController for the interface name because this name will never be used directly but from the typealias (Restaurants.List.View), so the fact that you append an “I” or any other prefix/suffix in **the protocol name does not get exposed to the rest of your code**, it is just an implementation detail due to Swift limitations with nested protocols. Note also how I used the Restaurants.List.View in the controller class definition to conform to the protocol rather than the RestaurantsListView directly, because of the same reason.

### Structuring in modules

OK, so we are almost there! We have a mechanism to give proper names to our classes/protocols, exposing only readable names and grouping them in modules that allows us to better understand structure and get relevant suggestions while typing. 

Now, we can just create as many modules as we need, and we can nest them as deep as you want to clarify its structure. But wait, if we go deep in the nesting, do we need to add them all in a single file? Not really, because by **using extensions we can add the concrete typealiases inside the modules or submodules**, without having to create a massive file with all the enums and nested objects in it. Let’s see a very simple example.

Imagine a “Search” module, that contains some other submodules in it. You could have something as simple as:

    //File: search/search.swift
    enum Search {
        enum Filters {}
        enum Results {}
    }

Where it basically declares that the module is composed of 2 submodules, but without exposing any particular details about them (or even an empty enum with no content at this level).

Then, in subfolders inside search, you could define the concrete submodules with extensions like:

    //File: search/filters/filters.swift
    //Filters submodule
    extension Search.Filters {
        typealias View = ...
        typealias ViewModel = ...
        typealias Model = ...
    }

    //File: search/results/results.swift
    //Results submodule
    extension Search.Results {
        typealias View = ...
        typealias ViewModel = ...
        typealias Model = ...
    }

So now, from folder perspective, you have a main search.swift file defining the parts, and concrete <submodule>.swift files defining the concrete implementation. Very neat and very clear where to look for the specific implementation details when needed. 

![](/ckeditor_assets/pictures/24/content_screen_shot_2016-10-12_at_11_35_27.png?1476281920)

### Limitations

There are however a couple of caveats to point out.

*   **File names must be unique**: In Swift, file names are used to generate the final internal name for the classes. Even if you add them inside different folders, and use different names for the classes in it, the compiler will generate an error if 2 swift files are named the same. Because of this, you should not name your file like ViewModel.swift even if they are grouped under a module directory. Instead, just do it like RestaurantsListViewModel.swift. It would be nice to be able to apply short names on files, but Swift compiler is not there yet, maybe in future versions.
*   **Interface builder**: In current version of Xcode (8.0), IB can not follow typealiases properly (at least not in nested enums like the ones I showed). As a result, in your IB files you have to use the concrete implementation class name. So, instead of Restaurants.List.View, you have to use RestaurantsListViewController if you want your outlets to be properly displayed.

![](/ckeditor_assets/pictures/25/content_screen_shot_2016-10-12_at_11_36_53.png?1476281949)

### Conclusion

Naming is very important, proper naming can help to understand your code and module structure better, and at the same time it can improve the quality of suggestions while typing. 

We have introduced a way to do it in Java, but for Swift we need to take some workarounds due to nested protocol limitations. However, thanks to the usage of typealiases and nested enums we can get a very elegant and neat way of implement the desired behavior. 

If you want to check out any of the code used in this post, you can [do it here](https://gist.github.com/angelolloqui/63433ef83551780c4b1085c7a6df40b9)