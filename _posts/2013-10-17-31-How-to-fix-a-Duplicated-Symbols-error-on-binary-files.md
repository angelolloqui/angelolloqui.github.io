---
layout: post
title:  "How to fix a \"Duplicated Symbols\" error on binary files"
date:   2013-10-17 11:30:34
categories: 
    - libraries
    - tools
    - errors
    - linking
    - building
    - fix
permalink: /blog/:title
---

When including third party libraries into your project, you can run into a “**Duplicated Symbols**” error on the linking process. This annoying error is due to a name collision between one or more of your classes, usually caused by either:

*   **You do not use a prefix as namespace** and you use a generic name such as Session, User or similar. This has an easy solution, as all you need to do is rename your classes to use a prefix. For example, User could be renamed into AGUser. Using prefixes is a good practice that you should always follow in programming languages without namespaces like C or Objective-C.
*   **One or more of your libraries are including the same third party library**. This is quite common on static frameworks and libraries built with little care. Usually, the creator of the library includes generic utilities such as SBJON, Reachability and similars inside the compiled binary. Then, your project or some other library also making use of it tries to include it again, resulting in the duplicated symbols error. If you have access to the source code, it could be solved easily by leaving the duplicated one out of your target. Unfortunately, when this problem occurs, many times comes from compiled libraries or frameworks that do not give us control on the source code files but just a binary instead. Solving this issue may not seem easy or even possible, but as we are going to see in this post it is not so difficult as it may look.

#### Special consideration

The whole post assumes that both duplicated libraries are the same version, or at least **fully backwards compatible**. Besides, we also assume that both duplicated libraries **have not been modified**, as any modification could break the commented compatibility. If the libraries are not compatible then you will need to analyze the differences and see if you can come with a “mixed” library that provides the desired compatibility or move one of your components to use the other incompatible version. In any case, the solution is very bound to your application and exceeds the purpose of this post.

#### Example

For improving readability we are going to fix a real project.  In this example project we have a static framework called Serenity that contains the SBJSON library inside. The duplicated symbols appears when using CocoaPods with the “unoffical-twitter-sdk”, which also has a dependency to SBJON. In this case, the duplicated symbols are therefore contained in Serenity and Pods.a binaries.

![](/ckeditor_assets/pictures/20/content_duplicatedsymbols.png?1382024975)

We could fix it by playing with the Podspecs to leave out the SBJSON from the “unoffical-twitter-sdk”, but we have decided to remove the SBJSON from Serenity instead as it should not have been added in the first instance and anyway it contains an older version of SBJSON that the one in CocoaPods.

You should be able to follow the same process on your conflicting libraries in your project. Just check and decide which version you want to keep and which one you want to remove (usually keep the newer one).

#### Slimming down the fat binary

Most of the libraries are actually **fat binaries**. This means that you can find the compiled binary code for more than one architecture within the same file. For example, in our example project, the Serenity framework contains three architectures: i386 (Simulator), armv7 (iPhone 3GS+ compatible) and armv7s (iPhone 5). Note that armv7s at the moment of writing this post is not recognized by the lipo tool as a valid architecture (it is displayed as “cputype (12) cpusubtype (11)”) so we will need to make a small trick to support it. Hopefully Apple will update the tools soon to support it. You can check the architectures of your library by running:

```
    $ lipo Serenity -info
    Architectures in the fat file: Serenity are: armv7 (cputype (12) cpusubtype (11)) i386
```

In order to proceed with the whole process, we need to extract each architecture on its own file. We can do it by running:

```
    $ lipo Serenity -thin armv7 -output Serenity.armv7
    $ lipo Serenity -thin i386 -output Serenity.i386
    $ xcrun -sdk iphoneos lipo Serenity -thin armv7s -output Serenity.armv7s
```

Note the use of the xcrun in the last case due to the lack of support for armv7s. The rest of the steps explained in the post are equally valid for armv7, armv7s and i386.

#### Removing duplicated symbols

Now that we have the two different architectures in thin files, we can proceed to remove the duplicated symbols on each file. Lets first take a look to the symbols included in Serenity:

```
    $ ar -t Serenity.armv7
    __.SYMDEF
    MainView.o
    PushRequest.o
    PaymentViewController.o
    SBJsonBase.o
    SBJsonParser.o
    SBJsonWriter.o
    NSObject+SBJSON.o
    NSString+SBJSON.o
    FileManager.o
    Barcode.o
    qr_draw_png.o
    QR_Encode.o
    png.o
    pngerror.o
    pngget.o
    pngmem.o
    pngpread.o
    pngread.o
    pngrio.o
    pngrtran.o
    pngrutil.o
    pngset.o
    pngtrans.o
    pngwio.o
    pngwrite.o
    pngwtran.o
    pngwutil.o
```

We picked the armv7 version, but the i386 and armv7s should be identical.

As we suspected, the library contains five symbols that are duplicated:

*   SBJsonBase.o
*   SBJsonParser.o
*   SBJsonWriter.o
*   NSObject+SBJSON.o
*   NSString+SBJSON.o

We just have to remove them by running:

```
    $ ar -d -sv Serenity.armv7 SBJsonBase.o
    d - SBJsonBase.o
```

Run the command with the other 4 symbols also. Then, repeat everything with the other architectures (i386 and armv7s in our case). If you check the binaries again with the `ar -t` option you should see that the symbols are not longer there.

#### Fattening the library

Ok, we are almost done. We have the three architectures without the duplicated symbols. All we need to do now is **combine them back again into a fat library** that will be used by our project. So first lets move the old version and keep a copy just in case something goes wrong.

```
    $ mv Serenity Serenity_original
```

And now lets build the fat library:

```
    $ lipo Serenity.armv7 Serenity.armv7s Serenity.i386 -create -output Serenity
```

And there you have! you should have a new binary file without the duplicated classes. You can now remove the intermediate Serenity.armv7, Serenity.armv7s and Serenity.i386 files as they are not longer needed.

#### Finishing

If you now build your project again, you should be able to link without the duplicated symbols error presented before (at least without this one, maybe you have another one though).

It is extremely **important that you test your app again**, because if any of the special considerations mentioned above are not met then your app could malfunction or even crash. Test, test and test!

Finally remember, you just modified a third party library so do not forget that any update on it will probably include the duplicated symbols again. And please, do not blame me if something does not work, just ask your third party developer to do his job properly and leave dependencies out of the binary :D.