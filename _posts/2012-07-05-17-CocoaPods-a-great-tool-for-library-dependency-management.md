---
layout: post
title:  "CocoaPods: a great tool for library dependency management"
date:   2012-07-05 11:30:34
categories: 
    - ios
    - cocoapods
    - libraries
    - tools
permalink: /blog/:title
---

Have you ever needed to use a third party library which needs other third party libraries and wondered why the hell they are so complicated to integrate within your iOS project? well, if that is your case then you will like this post and you will love [CocoaPods](http://cocoapods.org/).

But first, let me tell you that before writting this article I have worked with it for a couple of months in real projects. Thus, it is long enough to write some experiences that I had with it, but let’s start from the beginning.

#### What is CocoaPods?

CocoaPods is a tool written in Ruby that allows you to **manage library dependencies in Objective-C** projects. It is very simple to use, specially if you are used to Ruby Gems. The only thing you have to do is install it in your machine and then create a file called **Podfile** in your project, which will contain a list with the project dependencies to other libraries. For example:

    dependency 'DCIntrospect'
    dependency 'MGSplitViewController'
    dependency 'TouchXML'
    dependency 'Reachability'

After that, all you need to do is to run “pod install” (or “pod setup” if it is the first time).

CocoaPods will download the code associated to all the libraries into a Pods directory, and even the dependencies that these libraries could have with third party ones or Apple frameworks. All sources will be included and built in a new project called Pods, which will be imported in a workspace together with your project. Whenever you build your project, CocaPods will be built first and included as a static library to your project code.

Another important thing to know before continue reading this post is that the tool is just a bunch of installing routines, but the libraries are declared in simple txt files called **specs** where you define things such us the url, version number, other dependencies,... The specs are (almost) publicly available for editing.

#### Why to use it?

There are many reasons why you should try to keep your code dependencies as clean as possible: code maintainability, avoid conflicts with other team members or between libraries, save time,...

CocoaPods almost solves all of them, and does it with a **very little effort** from the developers, which is great!  With CocoaPods you will waste almost no time configuring frameworks and everything should work without problems! It is such a step forward in dependency management that you should at least give it a try.

#### Issues

However, not everything is as good as it seems. There are also some problems and concerns that you should keep in mind when using this library:

*   **Maturity**: The project is at this moment (July 2012) very new, but it is been built in such a fast way that it is almost impossible to keep the pace. Among the obvious problems related to immature software, one of the most annoying ones when working with CocoaPods is that you may find **specs using new features** that are not implemented in your installed version. This will give you more than a headache because it will crash your “pod install” command and will force you to update, which in fact may introduce new problems. This **will eventually be solved**, but right now it is a serious concern if you have a working project.  
     
*   **Quality of libraries**: Everyone can create a new spec for his library in CocoaPods, which is great, but it also introduces a lot of uncertainty and bugs. Right now there are at least 300 libraries (and increasing every day); many of them with more than one version, and **some with bugs or other issues**. This problem is not directly caused by CocoaPods (it is a problem of the libraries themselves), but CocoaPods makes it more dangerous. Why? well, because it is so easy to install libraries, it is also easy to add the unstable ones. Besides, CocoaPods hides the version of the library that you use and their dependencies, and you may find yourself using something that you did not want to use, **or an older or newer version than expected**. For example, just by installing RestKit you will find other 5 or 6 dependent libraries installed, which are supposed to work properly, but make you lose some control of your project.  
     
*   **Autoupdate**: Whenever you run “pod install” the tool **recreates the Pod project** and downloads any **updates related to your libraries**. In a perfect world this would be a bless! you could be updated at any time just by running one command! However, we do not live in such a perfect world, but in one where the libraries often **drop backward compatibility** when upgrading or where things that are compatible do not work in the same way from one version to the next one! And believe me, people will update the specs of your libraries to new versions! This problem could mean your complete failure if you don’t have a perfect unit testing (with in practice is impossible) or you do not **expend time checking** that everything works as it is supposed to work, not only in one platform but in all of them (any combination between device and iOS) whenever you run your updates. So, what can we do? well, there are **many solutions** for this problem, but you definitely have to think of something that works in your case. For example, you could add the Pods directory to your CMS to at least be aware when new changes come and test them thoroughly; you could create your own specs repository to control when a new library or upgrade should be included (explained later); or you could just prevent your developers from running the “pod install” command somehow! Choose whatever suits best for you, but do something or you will find lots of unexpected problems and crashes!  
     
*   **Code tracking**: Linked to the previous problem, if you do not keep the status of your Pods in your CMS you may face a problem when trying to recompile an old version of an app. You should always **keep a copy of all your project files** (in the exact state that they were) when you build a release version, or otherwise you could be unable to reproduce a bug if something goes wrong later. Once again, there are many solutions, which are similar to the ones exposed in the previous paragraph, but it is something else you have in mind.

#### A different approach

After facing many problems related to the issues explained before, in [Xaton](http://xaton.com/) **we are taking a different approach** to CocoaPods. We think that the tool is very useful, but we also think that it **fails often just due to incorrect specs**. So, what can we do with it? well, CocoaPods is the tool, but the specs live in a Git repository which can be forked and edited as you want. Therefore, what we are doing is **creating a completely new specs repository**, including only libraries that we know that are working fine. With this approach we can have full control of the libraries included in our projects, and we can **guarantee that everything works** just by having a proper control on the specs. Moreover, we can also **add our internal libraries** to the specs to be installed as any other public spec. 

This way we hope to get the best out of CocoaPods; we expect very easy and fast library integration, with conflict resolving mechanisms and with our internal libraries.

As you may see, I am talking in future because we are right in the middle of the switch, but I am really excited with this new idea because I think it will solve almost all of the issues we have found so far. I’ll keep you updated with any interesting experiences in future posts.

#### Conclusion

CocoaPods is not perfect, but it really **helps with a problem that is very hard to solve**. The **community** (which I have the pleasure to be part of) **is awesome** and extremely active. If you use it you may find problems at first, but I am sure that in a very short time it will be one of the “must have” tools for any iOS developer. 

**My suggestion**: give it a try with a new project which is non critical from a business point of view, and you will see how things get integrated nicely. Specially, use it if you are planning a POC or a quick project. Then, I am sure that you will want to use it in almost every project, but just remind all the things explained in the previous paragraphs or you could face very disgusting problems later. Use it wisely and help improving the quality and popularity of this amazing tool!

> Edit: Eloy Durán (@alloy) has replied to the post at: [https://gist.github.com/3059399](https://gist.github.com/3059399). Thanks Elloy for your time and for creating this awesome tool. Please, check out his comments.