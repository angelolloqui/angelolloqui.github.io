---
layout: post
title:  "Oddities of UIStackView"
date:   2016-03-06 11:30:34
categories:  
    - ios
    - uistackview
    - autolayout
    - uikit
permalink: /blog/:title
---

With iOS9, Apple introduced a new container view called UIStackView. Stack views can be seen as the iOS equivalent for the LinearLayout in Android. Likewise, the usage is pretty straightforward, especially if you from Interface Builder, and when used properly can save the developers lots of time.

However, this special view comes with **its own set of gotchas** that are hard to find at first, and that can complicate your life hugely. With this post I want to cover the main issues and tricks that I have found while using stack views for a few months. Note that I am not going to cover the basic usage of UIStackView, there are plenty of references on Internet for that (check references below). Let’s get started!

#### Where is the drawing phase?

One of my first issues with Stack Views was **setting up some basic properties like background color or borders**. An interesting aspect of this views is that they do not have a normal render phase. Instead, they are container views, and any code you place in the drawRect method drawing into the current context will actually output nothing. This is what the documentation says:

The UIStackView is a nonrendering subclass of UIView. It does not provide any user interface of its own. Instead, it just manages the position and size of its arranged views. As a result, some properties (like backgroundColor) have no affect on the stack view. Similarly, you cannot override layerClass, drawRect:, or drawLayer:inContext:

If you want to put a background color to the view, set a border or any other similar drawing operation, you will need to **add an extra view/layer behind** (or inside), and set your properties and rendering code there. Do not waste your time subclassing the stack view to add a drawRect implementation, your current context will still not be rendered.

#### The arrangedSubviews asymmetry

UIStackView requires you to add your views to them by using the addArrangedSubview or insertArrangedSubview:atIndex: if you want them to be “laid out”. As stated in the documentation:

This method automatically adds the provided view as a subview of the stack view, if it is not already.

Which basically means that when you add an arranged subview, **it will automatically become a normal subview** of the stack (both in subviews and arrangedSubviews). However, it is interesting that the removeArrangedSubview: **does not actually remove the view from subviews** as you could think. Instead, calling removeArrangedSubview: will only remove the view from the arrangedSubviews, but the view will still remain as part of the subviews in the view hierarchy. This asymmetry may cause memory issues if you do not remember to call the removeFromSuperview in the arranged subviews, especially if your stack content is changed frequently removing/adding views

#### The hidden property

When dealing with UI, it is pretty common that you want to hide a view depending on some internal state, and use the “hidden” space for the rest of the views. With autolayout, a view of certain intrinsic size will still occupy the space defined, no matter if the view is hidden or not, resulting in empty areas in the screen. You would normally link one of the key constraints (like positioning) to change the priority or constant to get the other views on top of your hidden view.

With UIStackView, your layout will be automatically recomputed when you change a the hidden property, **using the empty space** for the other views. 

This is especially interesting if you change the hidden property **along with an animation**. Normally, a bool property like hidden would not be animated. However, with stack views modifying hidden results in a frame update, and as a result the views will **animate to their new positions**. Nevertheless, remember that if you want the animation to be shown properly, you will need to make sure that your layout is recomputed, like you would do with Autolayout and the layoutIfNeeded method. Example:

    UIView.animateWithDuration(0.25) {
    	arrangedView.hidden = true
    	stackView.layoutIfNeeded()
    }

#### Unable to simultaneously satisfy constraints

One of the most difficult issues to resolve when playing with UIStackView are constraints conflicts on runtime. As you are not creating the constraints yourself (the stack does it), it is pretty hard to figure out what the problem is. 

I have found **a very common issue** when marking an arrangedSubview as hidden, which will output something like:

    Unable to simultaneously satisfy constraints.
        Probably at least one of the constraints in the following list is one you don't want. Try this: 
    (1) look at each constraint and try to figure out which you don't expect; 
    (2) find the code that added the unwanted constraint or constraints and fix it. 
    (Note: If you're seeing NSAutoresizingMaskLayoutConstraints that you don't understand, refer to the documentation for the UIView property translatesAutoresizingMaskIntoConstraints) 
    (
        "<NSLayoutConstraint:0x7f7f5be18c80 V:[UISegmentedControl:0x7f7f5bec4180]-(8)-|   (Names: '|':UIView:0x7f7f5be69d30 )>",
        "<NSLayoutConstraint:0x7f7f5be508d0 V:|-(8)-[UISegmentedControl:0x7f7f5bec4180]   (Names: '|':UIView:0x7f7f5be69d30 )>",
        "<NSLayoutConstraint:0x7f7f5bdfbda0 'UISV-hiding' V:[UIView:0x7f7f5be69d30(0)]>"
    )
    
    Will attempt to recover by breaking constraint 
    
    
    Make a symbolic breakpoint at UIViewAlertForUnsatisfiableConstraints to catch this in the debugger.
    The methods in the UIConstraintBasedLayoutDebugging category on UIView listed in  may also be helpful.
    

Example from [StackOverflow](http://stackoverflow.com/a/32885955/378433)

Basically, what happens here is that the stack view **will automatically set your height to 0** (if vertical layout, or width if horizontal) when you **make the view hidden**. In the specific example attached above, the arranged view contains a segment control that has some extra padding of 8px by using required autolayout constraints. As a result, when the subview tries to resize to 0, the constraints with the padding can not be satisfied (requires a minimum of 16px size, but the stack wants to set it to 0), resulting in the above error. A way to solve it is to make sure that you have no size constraints like that, or if you need them, **lower their priority so the AutoLayout engine will know which one to break** (always yours). Have in mind that all the stackview managed constraints are required (priority 1000)

#### Conclusion

**Stack views are a great** way to improve your projects. They are performant, and provide a very easy way to manage views, making development and maintenance easier. However, they also come with their own oddities that you need to be aware if you do not want to spend hours dealing with strange bugs. I hope that this post can save developers time and at the same time encourage for its use.

#### References

*   [Apple UIStackView reference ](https://developer.apple.com/library/prerelease/ios/documentation/UIKit/Reference/UIStackView_Class_Reference/)
*   [UIStackView introduction tutorial](https://www.raywenderlich.com/114552/uistackview-tutorial-introducing-stack-views)
*   [Ambigous layout issue on Stack Overflow](http://stackoverflow.com/a/32885955/378433)