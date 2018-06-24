---
layout: post
title:  "Oddities working with images in iOS"
date:   2012-09-14 11:30:34
categories: 
    - ios
    - interface builder
    - images
    - agimagechecker
    - debugging
    - runtime
permalink: /blog/:title
---

For the last 10 days I have been working in a [new library to check iOS images](https://github.com/angelolloqui/AGImageChecker) on runtime. I will make another post to explain it in a future, but today I wanted to remark some oddities that I found when working so tightly with the UIImageView class (it is funny that I haven’t realized on most these oddities after almost 3 years working with them)

#### UIImageView

UIImageView, as any other view in iOS, extends UIView. This characteristic would make everyone think that UIImageView have all the functionality and behave as any other view. However, there are some interesting considerations when working with this component:

##### drawRect is not called

One interesting point contained in the [official documentation](http://developer.apple.com/library/ios/#DOCUMENTATION/UIKit/Reference/UIImageView_Class/Reference/Reference.html):

> The UIImageView class is optimized to draw its images to the display. UIImageView will not call drawRect: a subclass. If your subclass needs custom drawing code, it is recommended you use UIView as the base class.

This is pretty strange, but it also means that you can not override or swizzle the drawRect to perform your custom drawing in UIImageViews because it will never be invoked. 

##### userInteractionEnabled

This field is used to configure whether the view should receive interaction and it is defined in UIView as a boolean with a default value of YES. However, in UIImageView, the default value is set to NO, which means that by default images will not receive interaction such as touches. This is important for example if you plan to add a gesture to an image or subclass it. You need to remember to set this property to YES or play with the parent view instead.

##### contentModeCenter

UIImageView allows many different content modes to set the image within it. One of them is UIViewContentModeCenter, which allows you to place the image in the exact center of the view (vertically and horizontally) as expected. Which might be not expected is that this exact center can be a floating point number. This may cause blurry images, as aligning a view/image in a .5 position causes the antialias to run. For example, if you have a view of 15x15 and an image of 10x10, setting the mode center will position the image in (2.5, 2.5), rendering it blurry (on non-retina devices). This is especially important when dealing with external images, as it is common to receive different image sizes. 

#### Interface Builder

Besides the oddities related to the UIImageView class, Interface Builder also adds some interesting behaviour when working with them.

##### accessibilityLabel

When you add an UIImageView in a IB file and set an image in it, ~~IB will automatically set its accessibilityLabel property to the imageName~~. This is indeed very useful, as this property helps improving accessibility and may be useful to debug it later, but remember that this will only work if you set the image from IB. You still need to do it manually when setting images programmatically.

**EDIT**: This is indeed false. To accomplish such a functionality you need to activate your accesibility. Check out an example here: [https://gist.github.com/3872250](https://gist.github.com/3872250)

##### Missing image

Have you ever seen a log message like this?

Could not load the "MissingImageName" image referenced from a nib in the bundle with identifier "your.bundle.id"

It is the default message that is logged when you try to load an image from an IB file that doesn’t exists. The message itself is useful, but there are some derived consequences not so pleasant if you dig into it.

First of all, apart from displaying such a message, IB also adds a transparent image to your imageview. WTF?! Yes! that is right. Not only it doesn’t set the image to nil, but also it adds a non-empty (but transparent) 1x1px image to it! This is of course a tremendous pain if you need to somehow check whether all images are loaded. I still can not understand the reason for making a 1x1 image instead of nullifying it or setting it at least with a 0px image, but it is how it is. If you want to make such a check, then you will have to find if the 1x1 image is totally transparent, and then assume that it comes from the IB. I haven’t found any other property so far that can be used to detect missing images loaded from IB. Please, tell me if you know any other way.

Secondly, the presence of such a message could make you think that it will also be displayed when trying to load a missing image programmatically, but this will not happen. Only IB images show it and behave as explained above. Keep that in mind.

#### Others

##### UIButtons use UIImageView

When you set an image or background in a UIButton, the internals of this class uses UIImageView to display it. It may not surprise you, but I was expecting UIButtons and other similar components to render the images directly by using draw primitives, but they don’t. They use a composite pattern, at least on iOS5 (we’ll see if something changes in future iOS versions).

This is actually not really interesting for most developers, but it is a blessing for my library that by swizzling UIImageView methods work also in buttons.