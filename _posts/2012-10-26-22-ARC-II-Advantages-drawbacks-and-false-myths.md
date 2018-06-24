---
layout: post
title:  "ARC (II) - Advantages, drawbacks and false myths"
date:   2012-10-26 11:30:34
categories: 
    - arc
    - memory management
    - objective-c
    - xcode
    - optimizations
permalink: /blog/:title
---

[In my previous post I analyzed the fundamentals of ARC](./21-ARC-I-Introduction-to-ARC-and-how-it-works-internally). Today, I am going to write a quick post about some general advantages, problems and myths around it. I warn you that I will not enter in details, but it will be useful as an introduction to the next (and probably last) post of the series. 

#### Advantages

There are many advantages of using it, but I would summarize them on the following:

*   **It Keeps it Simple Stupid (KISS)**! Working with ARC makes your code easier to write, read and maintain. Therefore, your code will be less error prone and you could actually save some time writing (specially in dealloc methods).
*   **Safer**: Do you think your code is secure without ARC? Yes, you know how to retain your instance variables, but, let me ask you if you always retain your temporal variables. Probably not always, right? In the end, if it is a temporal variable there is little point on retaining a variable and releasing it a few lines later right? Well, then check out the last example exposed in my previous post! You can never be absolutely sure if an autoreleased returned value will be available the full variable life time, so you should retain always autoreleased variables to be 100% sure it will work. ARC variables by default are strong, which means that they will be retained, and the compiler never forgets :). Besides that, the introduction of weak references can also help a lot in solving problems with released pointed objects (usually delegates).
*   **Less leaks**: If you are a master of iOS development you probably have few memory leaks or even not at all. However, even if that is the case is common to work with different people who might not be as experienced as you are. The use of ARC ensures that there is no leak due to an incorrect use of retain/release. However, be careful with the memory retain cycles! they are still there. 
*   **Reduces autorelease pools**: This one is pretty interesting because I thought that ARC would make heavy use of autoreleasing pools. However, that’s part of a false myth and we will take a look to it later. For now, all you need to know is that it actually reduces the amount of autoreleases in your code (if everything is ARC compatible though), which could make your apps to actually execute faster if your code produces heavy memory pools.

#### Problems

Even if the advantages of using ARC are enough to encourage any developer to move to it, there are some issues when working with ARC that you still need to consider:

*   **Less experienced programmers**: Because ARC is easy, it can make developers to not care about memory at all. This could eventually create problems such as memory retain cycles. Remember, ARC helps you writing better code, but you still need to know the basics of memory management.
*   **Working with Foundation objects, C returned data, structs, and other low level data**: ARC knows how to handle memory if the object is an Objective-C class (an instance of NSObject), but what about other allocated memory? things such as C structs or arrays can not be managed by ARC, so this force you to take care by yourself. It’s not that ARC introduces overhead if you use such kind of data types, but it certainly make it a littler more difficult because, against the general idea of ARC, this time you have to handle it yourself. For dealing with this issue, ARC introduces the concept of bidges, which allows developers to tell ARC how to move objects in and out of ARC control. A quickly explanation of existing bridges:

    *   `__bridge` simply transfers a pointer between ARC and non-ARC with no transfer of ownership.
    *   `__bridge_transfer` moves a non-Objective-C pointer to Objective-C and also transfers ownership, such that ARC will release the value for you.
    *   `__bridge_retained` moves an Objective-C pointer to a non-Objective-C pointer and also transfers ownership, such that you, the programmer, are responsible for later calling CFRelease or otherwise releasing ownership of the object.

In general, I would say that if your project uses a lot of non-ARC compatible libraries which can not be ported (third party libraries, C/C++ based or Foundation) then I would not move to ARC. It will make your code dirtier with bridges and it could also lead to bigger memory pools. Otherwise (most projects), ARC will give you enough advantages to give it a try.

#### False myths

Now that we have a general idea of how everything works, I think is the perfect moment to introduce some false myths that I have seen so far:

*   **It is a garbage collector**: Completely false. A garbage collector is some kind of external entity that runs while your app does and checks for memory that is no longer valid. ARC on the other hand is a compile time tool, which introduces memory management calls in the resulting binary of your app. There is no process profiling your memory, and that is one of the reasons why it can be so fast. The pitfall, of course, is that it can not provide some of the advanced features that some garbage collectors give, like detection of memory retain cycles.
*   **You don’t have to worry about any aspect of memory management**: As stated in the previous bullet, ARC is not a garbage collector, and therefore it can not detect memory retain cycles. You still have to worry to set your strong variables carefully or you could end up with two objects that retains each other. Besides that, manual memory management has to be done for non-ARC objects as we showed previously.
*   **ARC only works on iOS5 and later**: I am not sure why this myth is so common, but I guess that is due to weak references. Weak references only work on iOS5+, but the rest of ARC is fully iOS4 compatible, which nowadays is enough for most projects. If you need to support iOS4 just do not use weak variables and you are done! Afterall, weak references were not available before ARC so you should not miss anything in your old projects when moving to ARC.
*   **It is slower than manual memory management**: This one is my favorite because I indeed believed it when I started with ARC, but it is also false. You might think that in order to make secure code, ARC needs to make a lot of unuseful retain/release calls and probably abuse of autorelease. After All, we have seen in the previous post all the calls that ARC does to retain objects and how it autoreleases the returned objects of your methods. This should definitely be slower right? Well, on the hand hand ARC memory calls are pure C functions, which are slightly faster than standard Objective-C calls. However, the real reason for ARC being so fast has little to do with that. Do you want to know why? Then keep reading the [next post of the serie, where I explain some of the nice magic going on under the hood](./23-ARC-III-ARC-Optimizations)!