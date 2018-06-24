---
layout: post
title:  "ARC (III) - ARC Optimizations"
date:   2012-11-02 11:30:34
categories: 
    - arc
    - memory management
    - objective-c
    - xcode
    - optimizations
permalink: /blog/:title
---

In my previous two posts about ARC ([post I](./21-ARC-I-Introduction-to-ARC-and-how-it-works-internally) and [post II](./22-ARC-II-Advantages-drawbacks-and-false-myths)) I have talked about the optimizations and how ARC can perform as fast (or even faster) as manual memory management. This is quite clear when working with simple alloc/retain/release examples because it is exactly equivalent to manual memory management but using C calls (which are faster). However, I haven’t explain **how it does it when dealing with methods that return autoreleased memory**. So, it is time to have some fun.

First, lets copy again the resulting ARC compiled code seen in [the first post](./21-ARC-I-Introduction-to-ARC-and-how-it-works-internally):

    - (Foo *)foo {
          return objc_retainAutoreleaseReturnValue(foo_);
    }

    Foo *foo = objc_retainAutoreleasedReturnValue([self foo]);
    [foo something];
    objc_release(foo);
    

As commented in [the previous post](http://angelolloqui.com/blog/22-ARC-II-Advantages-drawbacks-and-false-myths), at a first glance this seems to introduce some overhead. It calls objc_retainAutoreleaseReturnValue function for presumably retaining and autoreleasing a variable (to ensure the memory is returned autoreleased without modifying the retain count). Then, the receiver calls to  objc_retainAutoreleasedReturnValue (note is **not** the same method than objc_retainAutoreleaseReturnValue) which seems to retain the autoreleased object (and protect it from premature release due to a pool drain) and finally the memory is released (to keep the retain count balance). In brief, the performed memory calls seems to be:

```
  +1 retain      //objc\_retainAutoreleaseReturnValue(foo\_);
  -1 autorelease //objc\_retainAutoreleaseReturnValue(foo\_);
  +1 retain      //objc_retainAutoreleasedReturnValue(\[self foo\]);
  -1 release     //objc_release(foo);
```

Everything looks correct in this assumption, and by doing it this way we can ensure that memory will be valid on every single instruction shown, even if we release the original foo variable inside our code. However, it introduces some **extra memory calls that could potentially make things slower.**

Can we improve it? Yes, we can. Why not instead of doing the second and third call (autorelease + retain) we skip it completely? will this still guarantee memory safety? 

And the answer is YES. We can indeed remove these two calls and everything will continue working. Moreover, not only we remove 2 calls, but also one of them is an autorelease call, which in case of being executed will postpone the actual deallocation until the pool is drained (with the obvious impact in performance and memory use). 

That is an amazing improvement! **If we can remove these two instructions** then we can release the memory as soon as it is no longer needed, achieving a **better performance and decreasing memory** use by making less use of memory pools. Even more, code like this:

    
    NSArray *myArray = [[NSArray alloc] init];
    //Do something
    [myArray release];
    

Will indeed be equivalent to

    
    NSArray *myArray = [NSArray array];
    //Do something
    

Because the memory returned by the array method will no longer be allocated/autoreleased but allocated/released in both cases.

However, removing these lines is quite **difficult** for a reason: The **two calls are in different methods** (one in the caller and the other in the receiver method). Even more, it could potentially be in different libraries, or even in non-ARC compatible code! And of course, if we apply the optimization and one of the involved methods is not using ARC or is doing things differently for any reason, then our memory retain count balance is broken and we will end up with an app leaking or crashing. Not good!

#### How ARC “removes the two calls”

OK, it is not easy to remove the mentioned two calls, but let me advance you that ARC does something pretty equivalent. How? well, if we check the official documentation we can see that the call to objc_retainAutoreleaseReturnValue is doing a objc_retain (as expected) followed by an objc_autoreleaseReturnValue. There is where the magic starts! According to [the Clang documentation](http://clang.llvm.org/docs/AutomaticReferenceCounting.html#runtime.objc_retainAutoreleaseReturnValue):

>**8.4. id objc_autoreleaseReturnValue(id value);**  
>Precondition: value is null or a pointer to a valid object.  
>If value is null, this call has no effect. Otherwise, it makes a best effort to hand off ownership of a retain count on the object to a call toobjc_retainAutoreleasedReturnValue for the same object in an enclosing call frame. If this is not possible, the object is autoreleased as above.  
Always returns value.

>**8.15. id objc_retainAutoreleasedReturnValue(id value);**  
>Precondition: value is null or a pointer to a valid object.  
>If value is null, this call has no effect. Otherwise, it attempts to accept a hand off of a retain count from a call to objc\_autoreleaseReturnValue on value in a recently-called function or something it calls. If that fails, it performs a retain operation exactly like objc\_retain.  
>Always returns value.

Awesome!!! What this means is that instead of just calling the autorelease + retain, these functions do their “best attempt” to avoid it, and if it can not then it just does the autorelease + retain cycle! exactly the kind of improvement that we wanted! It will make things faster when possible and still will work when dealing with non-optimized or non-ARC code.

However, the documentation doesn’t tell much about how it can actually make such a thing. For that reason, [Mike Ash gives a really good explanation in his blog](http://www.mikeash.com/pyblog/friday-qa-2011-09-30-automatic-reference-counting.html) (which I am going to copy because I could not explain it better):

>“When objc\_retainAutoreleaseReturnValue runs, it looks on the stack and grabs the return address from its caller. This allows it to see exactly what will happen after it finishes. When compiler optimizations are turned on, the call to objc\_retainAutoreleaseReturnValue will be subject to tail-call optimization, and the return address will point to the call to objc_retainAutoreleasedReturnValue.  
>With this crazy return-address examination, the runtime is able to see that it's about to perform some redundant work. It therefore eliminates the autorelease, and sets a flag that tells the caller to eliminate its retain. The whole sequence ends up doing a single retain in the getter and a single release in the calling code, which is both completely safe and efficient.  
>Note that this optimization is fully compatible with non-ARC code. In the event that the getter doesn't use ARC, the flag won't be set and the caller will perform a full retain/release combination. In the event that the getter uses ARC but the caller does not, the getter will see that it's not returning to code that immediately calls the special runtime function, and will perform a full retain/autorelease combination. Some efficiency is lost, but correctness is preserved.”

There it is! so this memory calls are not simply doing a retain/autorelease operation, but **checking if the next instruction** to be executed is the opposite one. And it does that by **examining the return address of the function**!! 

Amazingly clever!!! Congratulations to the ARC team for this optimization!!!

#### Still not satisfied? Want more?

Then check out this great [post of Matt Galloway about objc_retainAutoreleasedReturnValue](http://www.galloway.me.uk/2012/02/how-does-objc_retainautoreleasedreturnvalue-work/) where he explains everything in full detail. Worth reading!.