---
layout: post
title:  "Localization in iOS apps made simple"
date:   2013-03-17 11:30:34
categories: 
    - ios
    - libraries
    - agi18n
    - interface builder
    - method swizzling
    - i18n
permalink: /blog/:title
---

Localizing iOS apps with the standard tools is tedious, especially when you use Interface Builder files. To resolve that, I have created a new tool called [AGi18n](https://github.com/angelolloqui/AGi18n) that makes it extremely easy. But let’s first start by analyzing the existing approaches for it together with their problems, to later introduce the library and all the goodies that you get from it.

#### Localizing your code (existing solutions)

When setting a string in a component from code you should use the NSLocalizedString macro. By using it, texts are automatically translated into the user’s language on runtime by looking in the Localizable.strings file.

Example:

    myLabel.text = NSLocalizedString(@"my text", @"a tip for translator");

And the corresponding es.lproj/Localized.strings (Spanish translation):

    /* a tip for translator */
    "my text" = "mi texto";

This makes localizing strings fairly simple, but you still need to create a proper Localizable.strings file for each language. For that, Apple provides a command line utility called “genstrings” that will look through your code extracting all the NSLocalizedStrings and add them into the Localizable.strings file. 

While this approach works quite well, genstrings utility lacks of some control to merge different versions of the Localizable.strings file, which can make maintenance difficult.

#### Localizing your XIB files (existing solutions)

If localizing your source code is easy, localizing your Interface Builder content is not. There is no direct counterpart for the NSLocalizedString macro when you work with IB. Therefore, the available options are:

##### Localize elements from code

Link all your IB elements to your code through IBOutlets and set the text manually using NSLocalizedStrings. While is a quite simple solution, there is no need to say that this approach results in an enormous waste of time and a very dirty solution. You should always try to avoid it as much as possible.

##### Manually Localize XIB files

Localize your IB files manually by using the localization option provided within XCode. This approach gives you much more control because it lets you resize your IB elements accordingly to the language (for example, resize elements in verbose languages). However, having to manually localize every single XIB file is tedious and it is a terrible way to do it if you need to export the texts for giving it to a third party translator (you need to manually introduce the translations into the localized version of the XIB file). Besides, updating your XIB files with new elements require you to replicate the change in every single language! Ouch! Not a solution at all!

![](/ckeditor_assets/pictures/17/content_screen_shot_2013-03-17_at_5_31_53_pm.png?1363598445)

##### Use ibtool

ibtool is a command line utility that extracts texts and injects them back into your XIB files. You can use it to get a strings file with all your XIB texts in the following format:

    /* Class = "IBUILabel"; text = "Age"; ObjectID = "M3f-at-Qtm"; */
    "M3f-at-Qtm.text" = "Edad";

Then, when you have your files translated, you can inject this texts back into the XIB file using the same tool again.

However, even though this approach is quite better than the previous two, it also has some drawbacks:

*   **The output file depends on XIB elements, not on content**: The keys used in the resulting strings file are totally dependent on the IB element instead of the content. This means that if you change a label by a textview then you will need to translate the item again, even if the text is exactly the same. The same problem applies to texts that are repeated multiple times (multiple instances will need to be translated) or if you accidentally remove an element and create it again.
*   **The format used is different to Localizable.strings**: It is not a major problem, but having to deal with multiple Localizable.strings formats is annoying and complicates things when working with third party translators.
*   **It is time consuming**: Going through every single XIB file in your project, and having to extract/inject content into their localizable versions is too much work. This forces you to use an automated scripting or similar, which turns out to add more complexity to the whole process.

#### A new approach

Summing up, the existing solutions give you ways to translate your content, but they are very different depending if you are localizing elements in your code or if they are contained in IB files. Besides, they are complex and maintaining an app updated with new strings introduces even more problems.

Can’t we design a way to just localize all our app in the same way, no matter where the text come from (code vs XIB), and that merges the new texts in the existing Localizable strings automatically? Yes! let me introduce you [AGi18n](https://github.com/angelolloqui/AGi18n) :)

[AGi18n](https://github.com/angelolloqui/AGi18n) is the last library that I have been working on, and its focus is to provide **a tool to automatically extract and use the localized strings transparently**, both in your NIB files as well as your code.

All you need to do is run the utility “agi18n” from the command line (inside your project), and it will go through all your existing languages (folders in the form of language.lproj). For every language, it will extract the code strings running Apple's “genstrings”, extract the IB strings by running “genxibstrings” (built-in utility that outputs the result of ibtool into the same format that the genstrings utility) and merge together the results, removing duplicates and preserving the oldests ones. Isn’t that nice?

But of course, extracting the strings from the XIB files is only part of the problem. We still need to inject the translated content back into the XIB files. For that, [AGi18n](https://github.com/angelolloqui/AGi18n) uses a runtime injector, that will automatically look for every string used in your XIB file and try to translate it seamlessly when the view loads. 

In summary, the working flow with [AGi18n](https://github.com/angelolloqui/AGi18n) would be:

1.  Install agi18n if you haven’t yet
2.  Use NSLocalizedString for code strings and just regular IB files (without Localized versions) in your project.
3.  Run agi18n command
4.  Give the Localized.strings files to translators and import them back when done

And that’s all! 5 minutes, no need to spend more time on it. You will have a **Localizable.strings file for each language** with all the existing strings in the **same format**, **no duplicates**, and with the **keys sorted** in alphabetical order to easily find them.

And what if we add new XIB files? or new elements in our existing files? nothing, just run again agi18n and translate the **new strings that will be added** in your existing Localizable.strings files.

Take a look to this demo video if you want to see how it works:

<iframe allowfullscreen="" frameborder="0" height="500" src="http://www.youtube.com/embed/4Cxv24W2MqA" width="100%"></iframe>