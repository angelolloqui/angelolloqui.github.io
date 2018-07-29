---
layout: post
title:  "iOS Performance tips (I): Drawing shadows"
date:   2013-09-06 11:30:34
categories: 
    - ios
    - performance
    - optimizations
    - shadows
permalink: /blog/:title
---

It is very common to see shadows in different apps. In many occasions shadows are just rendered by using an image with the shadow predrawn, which basically performs as any other UIImageView. However, sometimes you may need to draw the shadow from code. When this time comes you will face performance problems that in case of heavy use (especially in iPad in combination with table or collection views) it can slow down you UI. 

It might seem negligible, but if your view is a composition of multiple views applying shadows then you will actually experience a very big UI performance degradation. For example, in my latest project where I had a grid view with shaded cells like this:

![](/ckeditor_assets/pictures/18/content_ios_simulator_screen_shot_sep_6_2013_4_43_41_pm.png?1378478756)

the **frame rate plunged from the standard 60FPS to less than 15FPS** (a very poor performance) just by adding shadows. This of course happened because I was not doing it correctly, and with the techniques explained in this post everything came back to normal 60FPS even with the shadows.

#### Standard shadow drawing

If you have rendered shadows before, you probably have written something like the following:

    //Remember to #import <QuartzCore/QuartzCore.h>
    myView.layer.shadowOpacity = 0.3f;

Pretty simple right? But what is going on under the hood? Well, a very interesting point is that **shadows are applied based on the alpha channel of your layer**, pixel by pixel. This means that if you have a UIView with transparent areas (very common with UIImageView and UILabel), the shadow will adapt and be drawn following the exact same shapes. This allow you to do nice effects like this one:

![](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/CocoaDrawingGuide/Art/shadows_2x.png)

However, what if your shadow is lot more simple? in most cases all you want to draw is a simple shadow, maybe with a rectangular shape or something a little more complex, but still simple enough to be drawn following a path. If that is your case then you are probably wasting a lot of valuable GPU power to check the alpha channel with no sense. 

#### Shadow drawing based on path

If drawing shadows in a standard way is slow there has to be a way to define the shape of the shadow and save the computational power of computing the alpha channel of every pixel. Indeed, it exists and it is called “**shadowPath**”.

Defining a shadow path for a rectangle is as easy as this:

    myView.layer.shadowPath = [[UIBezierPath bezierPathWithRect:self.centerView.bounds] CGPath];

but of course, you could define [many other different paths with more complex shapes](http://nachbaur.com/blog/fun-shadow-effects-using-custom-calayer-shadowpaths), which should be enough for most cases.

When using shadow paths, bear in mind a few things:

*   **It is very fast!** The video card does not need to read pixels or even load external memory from images. All it has to do is fill the whole surface with the alpha color and just apply some gradients on the corners. In the example application, drawing with shadowpaths is perceived as fast as not having any shadow, at a constant frame rate of 60FPS.
*   **It does not resize** with the view, even if you are using autolayout or autoresizing masks. This means that if your view is resized then you will have to explicitly change the shape of the path as well. This can be easily achieved by subclassing your view and resetting the path in the  \- (void)layoutSubviews method. If you are setting the shadow from your view controller (you should not) then you could do something similar on the viewWillLayoutSubviews method.
*   **It is animatable**, so you could animate them if required in a pretty straightforward way by using CAKeyframeAnimation.

#### Shadow drawing with rasterization

If your view has a complex shadow then shadow paths are not an option. In that case, you can still improve your app performance by carefully choosing views for offscreen rendering. But what is offscreen rendering and rasterization? there is a brilliant article on [objc.io's issue 3](http://www.objc.io/issue-3/moving-pixels-onto-the-screen.html) that explains the whole process in detail, but let me give a very brief explanation

##### Offscreen rendering in a nutshell

When your app needs to draw something on the screen, the GPU takes your layer hierarchy (UIView is just a wrapper on top of CALayer, which in the end are OpenGL textures) and applies one by one on top of each other based on their x,y,z position. In regular rendering, the whole operation happens in special frame buffers that the display will directly read for rendering on the screen, repeating the process at a rate around 60 times per second.

If your view is composed of too many layers, the computational cost of compositing all the views so many times per second can be too high for your GPU to handle, which will result in some frames lost. Of course, if your views do not change much, you could save time by storing some of the intermediate compositions in additional slots of memory to reuse on next frames. This process of **caching the composited layer** is called Off-Screen rendering (the name already suggests that rendering is not done in the screen buffer but somewhere else, now you can see why), and the way to trigger it on CoreGraphics is by setting the layer’s ‘**shouldRasterize**’ property to YES like this:

```
    cell.layer.shouldRasterize = YES;
    cell.layer.rasterizationScale = [UIScreen mainScreen].scale;
```

_Please, note that the scale matters or you would have a non-retina rendered layer in a retina display, resulting in blurred views._

Of course the process have some drawbacks as well. The main one is that offscreen rendering **requires a context switch** (GPU has to change to a different memory area to perform the drawing) and then copying the resulting composited layer into the frame buffer. Every time any of the composited **layers change, the cache needs to be redrawn again**. This is why in many circumstances offscreen rendering is not a good idea, as it requires additional computation when need to be rerendered. Besides, the layer requires **extra video memory** which of course is limited, so use it with caution. 

However, if your view does not change much, then offscreen rendering of shaded views might be a good idea as the extra cost of making the offscreen rendering could pay off compared to the computational cost of redrawing the shadow every frame.

But how do you know if your rasterized view will be reused across frames? well, we know that when the rasterized view changes the cached composition needs to be updated, but what if it keeps the same? Check this part of the documentation header of the shouldRasterize property:

> As an implementation detail the rendering engine may attempt to cache and reuse the bitmap from one frame to the next. 

This statement means that even if the view does not change, **it is up to the rendering engine to decide whether to reuse the cache** from one frame to the next one or not. So actually the only way you can be sure about how your app is behaving is profiling your app. To help with that there is an option called “**Color Offscreen-Rendered**” both in Simulator and Instruments that will color offscreen rendered areas. Red color means that your view is rerendered and therefore rasterization is only slowing things down. Green color means that your composited view is being reused across frames, probably getting a performance gain (especially with expensive computations like shadows).

![](/ckeditor_assets/pictures/19/content_screen_shot_2013-09-06_at_4_53_45_pm.png?1378479252)

#### Conclusion

Shadow paths are the best way to go if your shadows are simple enough to be defined with a polygon. However, you will have the extra work of setting your path correctly when your view resizes. 

If you require complex shapes or per pixel shadows then you can not use the shadow paths, but you can still improve performance in most cases by rasterizing the view. However, remember to profile your application because if the rasterization misses too often then it could even perform worse than regular shadows.

As you see there is not a global solution, but with this options in mind you should be able to solve your performance issues in almost all situations.