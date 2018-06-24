---
layout: post
title:  "MVC in Objective-C (I): Introduction"
date:   2013-02-21 11:30:34
categories: 
    - mvc
    - objective-c
    - patterns    
permalink: /blog/:title
---

**MVC** stands for **Model** **View** **Controller**, and it is a pattern that allows developers to differentiate code depending on three different **roles**. It is extremely popular due to its simplicity and it is implemented in different ways in almost every single technology out there -in different ways though. MVC helps you making your code lot more **reusable**, **maintainable** and easier to **extend**. 

![MVC image](/ckeditor_assets/pictures/16/content_mvc.png?1362172304)

One important keyword in my definition that many people do not realize is that it is about roles, not about classes. This means that a single role can be implemented by many classes (obvious) but likewise a single class could implement multiple roles! Yeah, 1 class multiple roles, you read it right! it sounds weird because we are using this pattern to isolate code of different roles, and therefore mixing it in the same class looks like not applying the pattern at all. But wait a second, who said that we actually need to mix the code? Objective-C provides categories and a customizable runtime that can help you out dealing with a multirole class without having to mix code. 

In this series of posts I am going to explain one by one the purpose of each role, and show some of the **best practices, pitfalls and useful tools** that I have found so far. You do not have to fully agree with me, you are probably aware of many of them already, but I am pretty sure that some will help you better designing and organizing your code in your next projects! 

They will be long so have a little patient and keep reading :)

*   [MVC in Objective-C (II): Model](./27-MVC-in-Objective-C-II-Model)
*   [MVC in Objective-C (III): View](./29-MVC-in-Objective-C-III-The-view-layer)
*   [MVC in Objective-C (IV): Controller](./34-MVC-in-Objective-C-IV-The-Controller-layer)