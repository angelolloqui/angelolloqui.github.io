---
layout: post
title:  "Acceptance testing with Calabash and CocoaPods"
date:   2013-01-13 11:30:34
categories: 
    - ios
    - android
    - testing
    - cocoapods
    - tools
    - calabash
    - libraries    
permalink: /blog/:title
notice: "I am receiving feedback from readers that this solution is not always working. Please, have in mind that this post is old and the content might be outdated."
---

This week I have been delighted with a very attractive (and quite new) acceptance testing framework called “[Calabash](http://calaba.sh/)”. My experience in acceptance testing is limited, but Calabash quickly took my attention for a few reasons:

*   **It uses Cucumber** for defining tests. Cucumber is a “language” that is very close to real English, making tests very easy to write and read. Lot better than other alternatives, especially if you have to work with non-technical QA.
*   **It is multiplatform**. The same tests can be run both in iOS and Android! How cool is that? write tests once and run in multiple platforms.
*   **It runs on real devices**. Nothing else to say, tests should always be run in simulator and real devices. Even more, there are some interesting services in the Internet that allow you to run your Calabash app tests in hundreds of real devices, which can be very helpful for Android apps.

However, integrating the framework in a project with [CocoaPods](http://cocoapods.org/) was not as straightforward as I thought. In this article, we are going to see why and how to fix it.

#### Why not use the automatic integration tool from Calabash

Calabash SDK (0.9.126) comes with a very straightforward integration tool that will automatically modify your XCode project to use Calabash. All you need to do is to run the command “calabash-ios setup” and a new target for calabash with proper configuration will be added. However, this procedure has the following problems:

1.  If you use the automatic installation tool, **CocoaPods** (as is in the current version 0.16.0)  **does not work anymore**:
    
        % calabash-ios setup
        % pod update
        ....
        ### Error
        
        ```
        [!] Xcodeproj doesn't know about the following attributes {"FileEncoding"=>"4"} for the 'PBXFileReference' isa.
    
    It looks like CocoaPods have a problem reading the project file due to the virtual links that the library contains. It will be eventually solved, but for now the issue is there.
2.  The framework is stored inside the project into a calabash.framework folder **not managed by CocoaPods**. Despite the fact that the framework can be moved, this is still not convenient because it will require you to make a few changes in your project settings and you will still have the framework somewhere else. Besides, when using CocoaPods, you will probably expect that a command like “pod update” will update all your stuff, including Calabash.
3.  Additionally, I personally **prefer to avoid automatic tools** that hide the integration process. It is always good to know and follow the manual installation process at least once to understand what is required.

#### Manual installation with CocoaPods

To manually install Calabash, you need to follow the steps:

*   Duplicate main target![](/ckeditor_assets/pictures/9/content_calabash_1.png)
*   Rename your new target to use the “-cal” suffix instead of “copy”![](/ckeditor_assets/pictures/10/content_calabash_2.png?1357908722)
*   Rename your schemes to use the “-cal” suffix instead of “copy”![](/ckeditor_assets/pictures/11/content_calabash_3.png?1357908756)![](/ckeditor_assets/pictures/12/content_calabash_4.png?1357908781)
*   Rename your info plist file to use the “-cal” suffix instead of “copy”![](/ckeditor_assets/pictures/13/content_calabash_5.png?1357908795)
*   Change the buildsettings to use the new names (look for "copy" string to filter them quickly)![](/ckeditor_assets/pictures/14/content_calabash_6.png?1357908820)
*   Add Calabash to Podfile inside the proper target.
    
        target 'SupermarketDirect-cal', :exclusive => false do
          pod 'Calabash', '>= 0.9.126'
        end
    
    As you see, we are importing at least the version 0.9.126. This is because the previous Spec did not work, and so I had to create this new 0.9.126 Spec to handle the installation from CocoaPods. By the moment you read this, it should be already merged into the cocoa specs so no problem should arise.
*   Run “pod install”
*   Run your app from the new target.  
    At this point, is highly probable that you get a “duplicate symbols” linking error. If so, is because you are linking twice the pods library. Just do the following and run again:
    *   Remove libPods.a from target![](/ckeditor_assets/pictures/15/content_calabash_8.png?1357908922)

**UPDATE:** Thanks to Aaron for pointing out that you may need to remove -lPods from "Other Linker Flags"

If everything goes OK then you should be able to see a log like:

    2013-01-10 14:05:31.901 SupermarketDirect-cal[18764:c07] Started LPHTTP server on port 37265

Congratulations! your project is ready for Calabash testing!

#### What’s next?

Play with the cucumber tool and write some tests. There are not many tutorials and references about calabash yet, but a few of intereseting resources are:

*   [Official homepage](http://calaba.sh/)
*   [Calabash W](http://github.com/calabash/calabash-ios/wiki)[ikipedia](https://github.com/calabash/calabash-ios/wiki)
*   [Calabash Google Group](https://groups.google.com/forum/?fromgroups#!forum/calabash-ios)
*   [iOS Automated Testing with Calabash, Cucumber and Ruby](http://www.moncefbelyamani.com/ios-automated-testing-with-calabash-cucumber-ruby/)