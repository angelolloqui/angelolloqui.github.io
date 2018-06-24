---
layout: post
title:  "iPhone5 -568h image loading"
date:   2012-09-28 11:30:34
categories: 
    - images
    - iphone5
    - method swizzling
    - runtime
permalink: /blog/:title
---

With the introduction of the retina displays, Apple implemented a few changes when loading images that allowed the SDK to look for the corresponding retina image (named with a @2x convention) automatically. This was a relief for developers, as they do not needed to change the code in their apps. All an iOS developer needs to do is provide the corresponding @2x images of the non-retina ones.

Nowadays, with the introduction of iPhone5 and its bigger display, some images such as backgrounds should also be loaded from a different file or otherwise the image will be stretched. I expected that Apple would make a similar trick, this time with the -568h convention that they imposed for the Default screen. However, in this case, **they do not seem to do that**. WTF! Why? well, I don't know exactly why but if I try to guess I would say that it is because checking so many paths on runtime introduces a **considerable lag**, and in contrast with the retina version, the 568 counterparts are **used rarely**. They will only be applied in backgrounds and similar images, but not in small images such as icons. 

#### So, what can we do?

Well, thanks to the runtime libraries of Objective-C we can provide a clean solution based on **MethodSwizzling**  that actually makes the same trick without changing the code.

There are already a few implementations of such a utility:

*   [http://www.sourcedrop.net/FY53a14b0127f](http://www.sourcedrop.net/FY53a14b0127f)
*   [https://gist.github.com/3711077](https://gist.github.com/3711077)
*   [https://gist.github.com/3742463](https://gist.github.com/3742463)
*   [http://odoruinu.net/blog/2012/09/26/automatically-loading-iphone-5-sized-images-where-required/](http://odoruinu.net/blog/2012/09/26/automatically-loading-iphone-5-sized-images-where-required/)

However, they all seem to have some problems. For example:

*   They swizzle the imageNamed method without checking if it is an iPhone5, introducing an overhead that is not needed for iPads and iPhone4S (and previous versions).
*   They do not support loading files into bundles. For example, the image @"MyBundle.bundle/picture.png" will not load properly.
*   They have problems if the image name has the extension in it. An image called @"picture.png" will not load properly.
*   The scale property is not correctly set to 2.0.
*   They do not support XIB file loading

#### The solution

Because of that, I created a new category based on the previous code, but with a few improvements to solve the commented problems.

Here is the snippet:

[https://gist.github.com/3799648](https://gist.github.com/3799648)

UImage+H568.m

    //
    //  UIImage+H568.m
    //
    //  Created by Angel Garcia on 9/28/12.
    //  http://angelolloqui.com/blog/20-iPhone5-568h-image-loading
    //
    
    #import "UIImage+H568.h"
    #import <objc/runtime.h>
    
    @implementation UIImage (H568)
    
    
    + (void)load {
        if  ((UI_USER_INTERFACE_IDIOM() == UIUserInterfaceIdiomPhone) &&
            ([UIScreen mainScreen].bounds.size.height > 480.0f)) {
            
            //Exchange XIB loading implementation
            Method m1 = class_getInstanceMethod(NSClassFromString(@"UIImageNibPlaceholder"), @selector(initWithCoder:));
    		Method m2 = class_getInstanceMethod(self, @selector(initWithCoderH568:));
    		method_exchangeImplementations(m1, m2);
            
            //Exchange imageNamed: implementation
            method_exchangeImplementations(class_getClassMethod(self, @selector(imageNamed:)),
                                           class_getClassMethod(self, @selector(imageNamedH568:)));
        }
    }
    
    + (UIImage *)imageNamedH568:(NSString *)imageName {
        return [UIImage imageNamedH568:[self renameImageNameForH568:imageName]];
    }
    
    - (id)initWithCoderH568:(NSCoder *)aDecoder {
    	NSString *resourceName = [aDecoder decodeObjectForKey:@"UIResourceName"];
        NSString *resourceH568 = [UIImage renameImageNameForH568:resourceName];
        
        //If no 568h version, load as default
        if ([resourceName isEqualToString:resourceH568]) {
            return [self initWithCoderH568:aDecoder];
        }
        //If 568h exists, load with [UIImage imageNamed:]
        else {
            return [UIImage imageNamedH568:resourceH568];
        }    
    }
    
    + (NSString *)renameImageNameForH568:(NSString *)imageName {
        
        NSMutableString *imageNameMutable = [imageName mutableCopy];
        
        //Delete png extension
        NSRange extension = [imageName rangeOfString:@".png" options:NSBackwardsSearch | NSAnchoredSearch];
        if (extension.location != NSNotFound) {
            [imageNameMutable deleteCharactersInRange:extension];
        }
        
        //Look for @2x to introduce -568h string
        NSRange retinaAtSymbol = [imageName rangeOfString:@"@2x"];
        if (retinaAtSymbol.location != NSNotFound) {
            [imageNameMutable insertString:@"-568h" atIndex:retinaAtSymbol.location];
        } else {
            [imageNameMutable appendString:@"-568h@2x"];
        }
        
        //Check if the image exists and load the new 568 if so or the original name if not
        NSString *imagePath = [[NSBundle mainBundle] pathForResource:imageNameMutable ofType:@"png"];
        if (imagePath) {
            //Remove the @2x to load with the correct scale 2.0
            [imageNameMutable replaceOccurrencesOfString:@"@2x" withString:@"" options:NSBackwardsSearch range:NSMakeRange(0, [imageNameMutable length])];
            return imageNameMutable;
        } else {
            return imageName;
        }
    }
    
    @end
    

All you need to do is import the code into your project. It will automatically load the swizzled methods in iPhone5 and will look for an image named xxx-568h@2x.png before loading the standard retina image. If none is found, then it falls back to the standard loading mechanism. No changes in your code required.

#### Known issues

~~Despite the problems resolved, there is still one main issue that I do not know how to solve:~~

*  ~~**Interface Builder loaded images**. They do not use imageNamed or initWithContentsOfFile methods for loading images. If anyone knowns the method that IB  calls I will be happy to build a similar solution for IB loaded images. I have played a lot with this issue for my [AGImageChecker](https://github.com/angelolloqui/AGImageChecker) library and I finally managed to solve it by replacing the image when the UIImageView is loaded, but it is too complicated for something that should be simplier.~~

~~Besides the known issues, note that this trick only applies to images loaded with imageNamed method.~~

#### Update (25 of July 2013)

Last updated version of the code now includes support for automatic XIB file -568h image loading. Thanks to [Alex Zak](http://alexzak.me) for pointing out a solution and to [Artem Shimanski](https://github.com/mrdepth) for creating the trick for XIB files for his own utility.

#### Update (5 of May 2014)

Some people have asked me about the license. This piece of code is so small that it makes no sense to me to add any license. No need to request permission, name me or anything else. Just remember I am not responsible of any issue or malfuntion it might create, so use it under your own risk.