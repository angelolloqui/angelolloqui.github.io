---
layout: post
title:  "ARC (I) - Introduction to ARC and how it works internally"
date:   2012-10-11 11:30:34
categories: 
    - arc
    - memory management
    - objective-c
    - xcode
    - optimizations
permalink: /blog/:title
---

It has been about a year since Apple released [ARC](http://developer.apple.com/library/ios/#releasenotes/ObjectiveC/RN-TransitioningToARC/Introduction/Introduction.html), and even if there is a lot of good information out there I still see some misconceptions, false myths and reluctances to adopt it . Because of that, I am going to write a set of posts about it, but focusing in explaining how it works under the hood and why some of the myths out there are not really true.

So today, lets start by presenting ARC.

#### What is ARC and where does it come from?

Since the very beginning, Objective-C defined a few good [rules regarding memory management](http://developer.apple.com/library/ios/#documentation/Cocoa/Conceptual/MemoryMgmt/Articles/MemoryMgmt.html#//apple_ref/doc/uid/10000011i). One of them is called **Ownership** convention, which is there to define who should retain and release what. With it, if you are the owner of an object, you are responsible of releasing it, and you can pass ownership to the calling method by using some special words on your method names (allocXXX, newXXX, copyXXX). Thanks to this convention, the developer doesn’t need to check the internals of a method looking for whether it returns an autoreleased or retained object. Everything he needs to know is defined in the method name.

Of course, the developer is not the only one who could check such a thing, and XCode introduced a tool to analyze the code and look for incorrect uses of the ownership pattern (named Analyzer).

At that point, the XCode Analyzer was a good tool, but many of us wondered how difficult would it be to improve it to the next level and let it fix the code. Of course, fixing code is not only adding or removing calls to retain/release. Analyzer should be able to know a few more contextual information and things like accessing ivars could easily break it down if they are not managed somehow by it. 

Of course, Apple was already working on such a tool, and it was released with the name of ARC (Automatic Reference Counting) about a year ago.

#### Basic differences of ARC compatible code

With ARC, developers are banned from using retain/release/autorelease, because the compiler will take care of that, inserting the missing memory statements wherever they are needed. ARC also adds some new qualifiers in variables and properties, which are somehow equivalent to the non-ARC counterparts, but with some exceptions. Let’s take a look:

*   `__autoreleasing`: This qualifiers indicates that the variable will be released with the next pool drain. Internally it retains and autoreleases the object. Note that this qualifier has no sense in a property, and therefore it is not allowed.
*   `__strong`: Equivalent to retain qualifier in non-ARC.
*   `__unsafe_unretained`: Implies the object is not retained. Equivalent to assign qualifier in non-ARC.
*   `__weak`: Similar to assign qualifier in the way that it does not retain the object, but weak variables are also nullified when the object that it points to is released. It works atomically  to guarantee the correctness of releasing and nullifying.

Besides defining new qualifiers, there are a few more changes introduced by ARC when comparing to non-ARC code:

*   Every variable is declared as **strong by default**. This is especially important in ivars as they used to be defaulted to assign when no qualifier present. This means that with ARC you can safely assign an object to an ivar with using the property, it will be retained.
*   `__block` qualifiers in ARC **retains** the object, while it doesn’t in non-ARC environments. Be especially cautious when dealing with blocks, otherwise it could lead you to retain cycles (the compiler will probably warn you anyway).
*   deallocs are **no longer needed**. ARC will release any variable that loses the scope, which also includes the ivars. Dealloc is still available in case you need to free other objects such as structs or make operations such as close a file or interrupt a communication. However, you should not call \[super dealloc\] explicitly. No more nullifying nightmare in deallocs.

Lastly, there are also a few more things to take in mind when working with ARC, such as bridges, but will treat them later.

#### ARC internals

OK, we have seen a very basic introduction, but we want to know more. How does it work internally? well, let’s take a look to some code examples:

A simple assignment in non-ARC:

    Foo *foo = [[Foo alloc] init];
    [foo something];
    [foo release];

ARC counterpart:

    Foo *foo = [[Foo alloc] init];
    [foo something];

What ARC compiles:

    Foo *foo = [[Foo alloc] init];
    [foo something];
    objc_release(foo);

Quite similar right? except for the fact that obj_release is a pure C function, they are completely equivalent.

An assignement with an autoreleased method call  (both ARC and non-ARC codes are equivalent):

    Foo *foo = [self foo];
    [foo something];

    - (Foo *)foo {
    	return foo_;
    }

What ARC compiles:

    Foo *foo = objc_retainAutoreleasedReturnValue([self foo]);
    [foo something];
    objc_release(foo);

    - (Foo *)foo {
    		return objc_retainAutoreleaseReturnValue(foo_);
    }

In this case, we can see a small difference between both versions. In ARC, the compiler is **extremely paranoid** and will retain the autoreleased object received from \[self foo\] and release it afterwards, when it is out of scope (remember that variables are strong by default). This may seem an unuseful overhead, but we will see later that **ARC does some magic in there to solve the overhead** and will indeed have a better performance that the non-ARC version. Besides, the extra retain/release is not completely unuseful. Check out the following piece:

    Foo *foo = [self foo];
    [foo bar];
    [self setFoo: newFoo];
    [foo bar]; // bang! non-ARC code crashes!

In this example, assigning a new value to \[self foo\] will release its memory and the second call to \[foo bar\] will make it **crash in a non-ARC** environment. This snippet may seem very obvious, but the point is that the call to \[self setFoo\] could be performed by a different method or thread, even from inside of bar, which may not look so obvious but will equally crash. On the other hand, the ARC counterpart will efficiently solve the problem because the object will still be retained until the variable is out of scope.

#### What’s next?

[In my next post](./22-ARC-II-Advantages-drawbacks-and-false-myths) I’ll analyze some false myths about ARC as well as some of the advantages and drawbacks that you could encounter when working with it.