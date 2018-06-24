---
layout: post
title:  "Restkit addition: Cache-Control"
date:   2012-08-15 11:30:34
categories: 
    - restkit
    - libraries
    - patch
    - ios
    - cache
permalink: /blog/:title
---

Today I am not going to explain anything new but to add a small patch to [RestKit](http://restkit.org/). If you still haven’t worked with RestKit before give it a try. It is a very useful library to manage connection to external APIs.

RestKit, between many other things, have a feature to set the cache policy that you want to use when interacting with the external WebServices. However, if you take a look to the defined policies, you may see (if nothing has changed since I wrote this post) that none of them give you the option to read the “Cache-Control” header of the HTTP response. And that is exactly what I have added in my pull request [https://github.com/RestKit/RestKit/pull/888](https://github.com/RestKit/RestKit/pull/888). Nevertheless, the pull request has not been merged yet, so here you have a temporary solution:

[https://gist.github.com/3360769](https://gist.github.com/3360769)

#### Installation

The code above uses [MethodSwizzling](http://angelolloqui.com/blog/15-Method-Swizzling) to add the specific “Cache-Control” behaviour on runtime, which makes it completely transparent for the developer. You don’t need to change your code, just import the two files in your project and you are done! the swizzling will do the rest. It will also work even if you are using subclasses of RKRequest.

#### Use

Whenever you want a request to use the new “Cache-Control” policy, just add the import to the RKRequest+CacheControl.h file and use it as any other standard policy. Example:

    RKObjectManager *manager = [RKObjectManager sharedManager];
    manager.client.cachePolicy = RKRequestCachePolicyControlMaxAge;

Or even add the cache policy to the default like this:

    manager.client.cachePolicy = RKRequestCachePolicyDefault | RKRequestCachePolicyControlMaxAge;

#### What about the "Expires" header?

The "Expires" header is an alternative way to control caching from the server side, but it is older and it lacks of the simplicity of the “Cache-Control” header. With "Expires", the code should convert the retrieved timestamp into a valid date, and adjust it based on the time difference between the date registered on the device and the one sent by the server (devices could have incorrect dates). Besides, “Cache-Control” has priority over "Expires", so it will only work if no “Cache-Control” is found. 

For all that, I don’t find "Expires" worth enough to spend time on it, but it could also be done. If you plan to add support for it to my current implementation of “Cache-Control” please contact me. I could help you and post the resulting code here for everyone.