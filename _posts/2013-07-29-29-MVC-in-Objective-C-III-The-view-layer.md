---
layout: post
title:  "MVC in Objective-C (III): The view layer"
date:   2013-07-29 11:30:34
categories: 
    - mvc
    - objective-c
    - patterns
    - view
permalink: /blog/:title
---

In [previous chapters](http://angelolloqui.com/blog/26-MVC-in-Objective-C-I-Introduction) we have exposed how the MVC pattern separate concerns in three layers and we have analyzed how to correctly implement the Model layer into your iOS project. In this post, I am going to continue with the next element in the MVC roles: the View role.

The View role is the responsible for handling all the UI layer. Ideally, this role should not know anything or very little about the associated [controller](http://angelolloqui.com/blog/34-MVC-in-Objective-C-IV-The-Controller-layer) and [model](http://angelolloqui.com/blog/27-MVC-in-Objective-C-II-Model) layer (at most how to use the model, but not how to perform direct interaction/manipulation on it). In an iOS project this layer is implemented by the UIView class (and subclasses) together with the Interface Builder files.

As we saw in the Model post, there are many ways and tricks to implement this role in a project, but I will try to summarize all the best practices and tools that I have found so far, not only regarding MVC but also as general View related suggestions. As usual, you might be already using many or not agree with some of them, but hopefully you will get some fresh air out of it. So lets start with my suggestions because the list long!

#### Use Interface Builder

There is a lot of controversy regarding this topic, and I could even write a full post only talking about it with more than 10 valid points in each direction. Some developers do not like to use it, while others do. I am not going to extend on every single one, but let me mention the two most important reasons for me:

*   **Separation of concerns**: IB enforces you to separate your views and controllers into two totally different components (your IB files and your UIViewControllers) that can only connect together by setting some explicit keywords (IBOutlet, IBAction, IBOutletCollection) and properties. The result of it is that without additional effort you will be implementing the MVC pattern in a more strict way that if you create your views from your controllers. Besides, having these decoupled components also helps reusability. For example, when dealing with universal apps, you could have one controller and two IB files (~iphone, ~ipad) for each component in your code, without requiring you to manually write if statements like “if (IPAD)” to distinguish between platforms at a source code level. Just use a generic controller with all properties you need in both platforms, and connect them accordingly unsetting those which do not apply to your platform. Clean and simple.
*   **Easy to use**: IB files are very easy to use thanks to the visual IDE that XCode provides. This will help you out creating nice UI screens with little effort, but it will also help any new developer that joins the project. He will know exactly where to look at when he needs to update the UI, and in an eye’s blink he will be able to modify it without having to read hundreds of equivalent Objective-C lines of code.

Of course, there are many problems derived from the use of it, but lets be fair and also consider the two that for me are the most annoying:

*   **Applying global styles**: In most apps there are some global styles that should be shared around your IB files (colors, fonts,...). This is really inconvenient because if you change anything on the future you will need to modify one by one all your IB files and components. In order to minimize the issue, I suggest to use a custom “user defined runtime attribute” that indicate the style to be applied. This way you will be able to change styles globally with very little effort, while keeping simplicity in your XIB files and controller code. This approach is used by some libraries like [NUI](https://github.com/tombenner/nui) with success.
*   **SCM Conflicts**: When working in big teams is common to suffer conflicts, specially if you use StoryBoards rather than simple XIB files. However, if you are using GIT, I would suggest to use the XCode integrated Git utility as it handles almost any conflict automatically. Besides, with XCode5 XIB files have changed to a more explicit XML, which should contribute resolving conflicts.

In summary, even if this is too long a topic to be discussed here with a lot of valid points in both directions, my advice is to use it as much as you can. It will pay off with cleaner and shorter code, and better MVC conventions adoption (which is all this post is about).

#### Use custom components, prefer composition pattern

I have never seen an app where the UI elements do not need to be customized somehow. Sometimes, by using UIAppearance methods is enough, but sometimes you need to make some deeper customizations.

You can usually just use standard UI elements (buttons, labels,...) and just set the properties you need to change manually from IB/code. However, while this looks OK when you only have to do it once, I think is a terrible idea. The main problem with it is that you will probably end up putting this code in your UIViewController, and then your controller will be handling UI appearance methods that are obviously not his responsibility. 

Another option is to create a category in the class you want to customize (for example a UIButton) that will apply this logic just by calling a method. While this approach is lot better than the previous one because the code is encapsulated, it can not be used from IB. As I mentioned before I am a true believer of IB, so it does not solve the problem.

A better approach is to create custom components. For example, if you want a button that has some visual enhancements then just make a new subclass of UIButton, and apply all your custom logic in it (do not forget to call it from awakeFromNib as well as standard init methods). This way you are separating concerns and at the same time allowing the component to be used from IB files.

One interesting aspect to mention is that when making custom components you can create them by composition of already existing components or by creating a fully scratch component.

While both approaches are fine, I advocate for the composition pattern against the custom drawing mainly because composition will take advantage of existing behaviours such as accessibility labels, animatable properties,... in a better way. Besides, resulting code will be lot simpler, cognitive load will be reduced and in many cases it can even result in performance gains.

#### Use Model objects as view input

Some people argue that passing a model objects to the view is not MVC compliant. I totally disagree, and in last term I think it makes things a lot more reusable and maintainable (which is the whole purpose of the MVC pattern).

Dealing with objects in views is especially common when using table view rows. In general, a table view row will contain at least 3 or 4 elements depending on your model data. For example, a simple Twitter app will have a row displaying for each tweet its text, user name and time. What I encourage to do in that case is to create a UITableViewCell subclass (custom component as explained above) with the 4 elements inside, and instead of exposing the elements to external components through properties in the .h file, expose a property to the associated model object. 

In other words, instead of:

    //AGTweetCell.h
    @interface AGTweetCell : UITableViewCell
    @property (strong, nonatomic) UILabel *usernameLabel;
    @property (strong, nonatomic) UILabel *textLabel;
    @property (strong, nonatomic) UILabel *timeLabel;
    @end

    
    //TableViewController
    - (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath {        
        static NSString *cellIdentifier = @"TweetCell";
        AGTweetCell *cell = [tableView dequeueReusableCellWithIdentifier:cellIdentifier];
        ….
        cell.usernameLabel.text = tweet.user.name; 
        cell.textLabel.text = tweet.text; 
        cell.timeLabel.text = tweet.timeString; 
        return cell;    
    }

Do something like:

    //AGTweetCell.h
    @interface AGTweetCell : UITableViewCell
    @property (strong, nonatomic) Tweet *tweet;
    @end

    //AGTweetCell.m
    @interface AGTweetCell ()
    @property (strong, nonatomic) UILabel *usernameLabel;
    @property (strong, nonatomic) UILabel *textLabel;
    @property (strong, nonatomic) UILabel *timeLabel;
    @end
    
    @implementation AGTweetCell
    - (void)setTweet:(Tweet *)tweet {   
        _tweet = tweet;
        cell.usernameLabel.text = tweet.user.name; 
        cell.textLabel.text = tweet.text; 
        cell.timeLabel.text = tweet.timeString; 
    }
    @end

    //TableViewController
    - (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath {        
        static NSString *cellIdentifier = @"TweetCell";
        AGTweetCell *cell = [tableView dequeueReusableCellWithIdentifier:cellIdentifier];
        ….
        cell.tweet = tweet;    
        return cell;    
    }

It is a little more code if used once, but the main advantage of it is that the rendering logic is encapsulated into the view object and thanks to that you can safely change your table cell in the future (maybe adding some new elements like the user image) without changing your controller. It is the duty of the cell to defined how it needs to be rendered, and the controller to pass the required information to cell, so it is OK regarding the MVC pattern -which by the way is especially true considering that, as explained in the Model post, a Tweet is nothing else that a container of information, just like NSArrays, NSDictionaries or NSStrings are -

#### Use categories instead of defines or constants for simple objects

When dealing with simple objects such as UIFonts or UIColors it is very common to see things like:

    #define kAntartidaMediudFontFamily @”AntartidaRounded-Medium”

    [UIFont fontWithName:kAntartidaMediudFontFamily size:12];

While this is OK for many developers, I prefer to use categories. So, the previous line could be changed by:

    [UIFont appFontOfSize:12];

By implementing a simple category:

    @implementation UIFont (SDAppearance)
    + (UIFont *)appFontOfSize:(CGFloat)fontSize {
        return [self fontWithName:@"AntartidaRounded-Medium" size:fontSize];
    }
    @end

There are not many advantages for this, but I like it because it just improves readability and it gives you more control on the logic behind the appFontOfSize methods. For example, you could maybe perform some check before using the provided fontSize, get the font name from a dynamic theme manager, add a breakpoint in the font creation or reuse the same UIFont element to get some memory improvements. As I said, not a lot of advantages, but whenever your start doing it this way you will like it better.

#### Use UIAppearance

Before iOS5, customizing and iOS app was quite tricky. Many components such as navbars did not have clear ways to set custom appearance to them. Hopefully, iOS5 came with the UIAppearance protocol and most native UI components now use it. Explaining in depth how to work with UIAppearance would take an entire new post, so I better redirect you to some interesting existing articles such as the one in [NSHipster](http://nshipster.com/uiappearance/) or [NSCookBook](http://nscookbook.com/2013/01/ios-programming-recipe-8-using-uiappearance-for-a-custom-look/). 

Anyway, keep in mind to use UIAppearance methods when possible. It helps out a lot decoupling styling from the rest of your code, specially from incorrect places such as UIViewControllers. Appearance should be self contained in the view role, and UIAppearance gives you a straightforward way to get it done.

#### Never use tags

Please, do not ever use tags! Never? No! Never! 

Tags are the typical hack that looks cool the first time you see it (wow! it saves me time creating properties!) but it will definitely struggle you in the future. Why would you care about following software patterns like MVC but then use such an unstructured programming style? For every place you use a tag, a property could have been used instead, always! For example, components coming from IB can be linked by using IBOutlet (or IBOutletColletion if many), and custom components can easily add properties to the class pointing to the UI element that had the tag before. Indeed, even if it sometimes requires a little more effort, the advantages you get worth it:

*   **Compiling time checks**: When using tags, if you change a UILabel for a UITextView, your code could break and you will not be notified by the compiler because it does not know the type of component that you are setting. Properties are typed, so a compiler error will be thrown.
*   **No collisions**: Using tags can result in collisions where more than one component uses the same tag. It might seem irrelevant when you use constants with random values, but if you are reusing components in many different places then you could end up with one because looking for a tag will traverse down all your view hierarchy. There is nothing worse than expect a view to have one specific tag and then find a totally different one because the tag was used somewhere else.
*   **Easier to access**: Having to look for a specific tag in the view hierarchy is more complicated (and slow) than just accessing a property. Besides, if you move your component out of this hierarchy then you will not be able to retrieve it again, while properties are always accessible and clearly defined.

#### Make flexible views

Views with resizing elements are everywhere. Even if your app is iPhone only and you do not support multiple orientations, you still have to deal with resized views for iPhone5 vs 4S and lower. Even more, your app window could be smaller than expected because the system might be taking some space out of your app window for other purposes (for example, when you are in tethering). 

In order to solve multiple size problems, Apple introduced Springs&Struts (AutoresizingMasks), and more recently Autolayout. In general, AutoresizingMasks should be enough for most views, and therefore I recommend to stick to it when possible. On the other hand, Autolayout gives you finer control, but is way too complex for most cases, making your views more difficult to create and maintain. Anyway, no matter whether you use one or the other, if you use them correctly you should not need to set frames almost anywhere.

For example, if you have code like:

    self.pagingView.frame = CGRectMake(self.pagingView.frame.origin.x, screenBounds.size.height-100, self.pagingView.frame.size.width, self.pagingView.frame.size.height);
    

or even worst:

    CGRect screenBounds = [[UIScreen mainScreen] bounds];
    CGRect frameImageView = welcomeViewController.imageView.frame;
    frameImageView.size.height = kImageHeight;
    if (screenBounds.size.height == 568)
    {
        frameImageView.size.height = kImageHeight + 50;
    }

Then is highly probable that you have set your autoresizing properties wrongly. Review all of them because in 99% of the cases the previous code can be removed by setting proper autoresizing constraints which makes things cleaner, more reusable and future proof for other screen sizes. 

#### Understand the difference between frame and bounds

If you are confused with these two you are not the only one. Many experience developers have trouble distinguishing when to use one or the other, or even understanding the difference at all. To keep the explanation simple, the frame references the view position in relation to the parent view, while the bounds express the portion of the view that will be visible on the internal view’s coordinate. Checkout [Apple’s documentation](http://developer.apple.com/library/ios/#documentation/WindowsViews/Conceptual/ViewPG_iPhoneOS/WindowsandViews/WindowsandViews.html#//apple_ref/doc/uid/TP40009503-CH2-SW9) for more detail.

In general, you should use the view’s bounds when you are in a view trying to layout its subviews based on your size, but the frame when you are placing the views under a parent coordinate. Example:

    - (void)layoutSubviews {
    	[super layoutSubviews];
    	CGRect bounds = self.bounds;	
    	//Use of frame for setting subview’s position but bounds for internal sizing
    	subview1.frame = CGRectMake(10, 10, bounds.size.width-20, bounds.size.height-20);
    }

#### Set AccessibilityLabels

I admit, accessibility labels are my most common misused feature. The main problem is that many developers (and testers) do not know they exist or feel reluctant to set them because it requires some extra effort for apparently no gain. However, apart from the obvious benefits that it provides to handicapped people, it also allows UI acceptance testing tools to properly work with your app.

For example, recently we introduced [Calabash](http://calaba.sh/) in one of our projects that were not using accessibility labels, and even if we could still manage to test without them, the resulting code was so unmaintainable that I decided to add the accessibility labels afterwards to the whole app. Of course, I would have saved time and helped my users if I had done it in first instance.

Conclusion: Always try to remember setting them, you will help people and it will eventually save your time if you plan to do acceptance testing to your app (and you should).

#### Explore CALayer, CATransform, CIFilter and others

Most developers are not used to work with the lower level drawing layers. In particular, explore CALayer masks and CATransforms as they are the most useful ones. They provide excellent tools for making nice effects and animations. I have seen examples of hundreds of complex lines of code that could have been accomplished by just a few ones of CATransforms and masks. Some nice examples of what can be accomplish:

*   [http://evandavis.me/blog/2013/2/13/getting-creative-with-calayer-masks](http://evandavis.me/blog/2013/2/13/getting-creative-with-calayer-masks)
*   [http://nachbaur.com/blog/fun-shadow-effects-using-custom-calayer-shadowpaths](http://nachbaur.com/blog/fun-shadow-effects-using-custom-calayer-shadowpaths)

#### Research useful tools

When working with views, there are a plethora of alternatives, from open source to licensed ones. However, I like those which are free and open source. Some of the most useful ones:

*   [Reveal](http://revealapp.com/): Amazing utility for inspecting your app views. A must have if you need to understand how your views are laid out.
*   [DCIntrospect](https://github.com/domesticcatsoftware/DCIntrospect): Very simple utility to check the position of your views. It lets you inspect your view hierarchy and even change it on the fly. Not as advanced as the previous one but more simple.
*   [AGImageChecker](https://github.com/angelolloqui/AGImageChecker): Busted! this is my own tool. Nevertheless I think is a nice tool for detecting images that are being resized, modified or blurry. It also has a plugin for uploading/downloading images to Dropbox, which is very effective when working with designers because they can freely modify images in your app and see the result without requiring you to change them in the app.
*   [AGi18n](https://github.com/angelolloqui/AGi18n): Once again my own library, but it is worth to take a look if you are localizing iOS apps that uses IB files.
*   [NUI](https://github.com/tombenner/nui): A tool for extracting all your view styling into a CSS like file. Very useful if your app requires lots of customizations and you have CSS skills within your team.
*   **Simulator debug options**. Is always a good idea to activate this options and reduce the amount of problematic areas pointed out by the tool. Example: Simulator -> Color misaligned images.
*   **ftxdumperfuser**: It is common to use custom fonts in your app. Sometimes, these fonts do not come with proper alignment or margins. If that is the case, use ftxdumperfuser as [explained here](http://http://www.andyyardley.com/2012/04/24/custom-ios-fonts-and-how-to-fix-the-vertical-position-problem/).