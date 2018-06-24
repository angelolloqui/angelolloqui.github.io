---
layout: post
title:  "DRM for mobile devices - Introduction and alternatives"
date:   2012-11-29 11:30:34
categories: 
    - drm
    - security
    - streaming
    - video
    - mobile  
permalink: /blog/:title
---

**DRM** ([Digital Rights Management](http://en.wikipedia.org/wiki/Digital_rights_management)), is a set of **access control techniques** created to avoid illegitimate use of **premium content**. Any proper DRM solution should deny the ability to view, copy or share the protected content to any other hardware/software that the one that is has been designed for.

Audio/video DRM solutions use **asymmetric encryption** together with other **obfuscation techniques** to protect the content. They are mainly composed by two parts:

*   **Server side**: Delivers the streaming content (Live or On Demand media), with some encryption applied to it, only to allowed users. They usually require the users to register somehow (for example with a login) to recognise valid users from the rest.
*   **Client side**: Downloads the stream and decrypts it. Because the client side is executed in an unsecured environment (the client), prior to start decrypting they have to implement environment checks to ensure that security has not been compromised. Because of it, DRM clients requiere closed environments where the sourcecode is as protected as possible and where the private key can not be stolen.

### Why should I care about DRM?

This is probably off-topic, but I can not write a post about DRM without asking myself -- and sharing it with you --  this question. Especially when you have in mind these two points:

1.  **Completely secured DRM is an illusion**: It is both technically and theoretically impossible to create a security mechanism that decrypts content on the client but at the same time hide it from itself. Whenever you deliver content to a client, this client could potentially modify/crack his environment to just bypass all security checks. For example, if I am a trusted user that can play secured content in my iOS app, how can DRM prevent me from, instead of playing it, modifying the app to save it to disk and upload it to Torrent? Sure, DRM solutions will check that I have not altered the app in anyway, that I do not have a jailbroken device and that I am not debugging it (among other things), but absolutely everything can be bypassed or spoofed. So, DRM is not about avoiding unauthorized access, but about making it difficult to an extent that it will not be worthy to crack it down.
2.  **Moral concerns**: I am not going to deep inside this one because it is completely subjective. However, I have to say that I have some reservations regarding it. I understand why so many people think that the content has to be protected in some way, but at the same time I think that the actual business models are completely outdated and that DRM is just a way to try to perpetuate something that does no longer work. Besides, is leading legitimate users to problems with credentials, slowness and unsupported platforms.

Nevertheless, there are a lot of conflicting interests between big media companies and copyright holders that enforce publishers to use DRM even if they would not like to do it, and believe me if I say that it is big business.

For example, I am working for a client who does only have the rights for showing premium content to a reduced set of his app users (his customers). They know that there is no 100% secure solution, but they need to prove that they are using “industry standards” mechanisms to prevent unauthorized users from viewing the content - and avoid copyright holders from suing them-. They are spending hundreds of thousands of euros just to accomplish that, probably more than in the app itself. An interesting business opportunity.

### Different DRM technologies

So, as explained before, providing DRM is more about using accepted standards than really creating a secured way to deliver your content. Therefore, even if you could probably build a pretty secured In-house solution, you will probably be requested to implement an industry known solution. Lets mention some of the existing alternatives, so you have an idea of where you should investigate next to find out what best suits your needs:

*   **[PlayReady](http://www.microsoft.com/playready/)**: PlayReady is a proprietary solution from Microsoft that has been used widely in the industry since 2007. It is considered as one of the most trusted technologies, with client support for almost every existing platform out there and integrated in many DRM-ready servers ([Verimatrix](http://www.verimatrix.com/), [Authentec](http://www.authentec.com/), [Irdeto](http://irdeto.com/),...). It is currently in use by industry leaders such as Netflix, BBC, HBO or Samsung. In order to display in a regular browser it makes use of the Silverlight plugin.
*   **[Widevine](http://www.widevine.com/)**: Widevine is a company and  DRM solution recently acquired by Google, and since then increasing popularity. This is the DRM solution adopted by many of the streaming service providers out there ([BrightCove](http://www.brightcove.com/), [Ooyala](http://www.ooyala.com/), [Kaltura](http://corp.kaltura.com/),...), as well as some big companies such as BlockBuster, BestBuy or Samsung. Most platforms are supported, including mobile devices and smart TVs. In order to watch it in a regular browser, you might need to install a custom plugin.
*   **[Marlin](http://www.marlin-community.com/)**: Marlin is a DRM solution created using open standards and coordinated by the MDC (Marlin Developer Community). It is not as adopted as the previous two options, but still has a share of the DRM market. To name a few, it is used as the IPTV standard in Japan and companies such as Sony, Philips or Panasonic are behind the MDC community. An interesting point of this DRM is that any company could potentially become a partner and take part in Marlin development.
*   **[Adobe Access](http://www.adobe.com/products/adobe-access.html)**: Adobe Access is a very common solution due the popularity of flash based videos on the web. For iOS devices, Adobe provides a native SDK to integrate within your apps that handles all the DRM calls for you, whereas regular browsers will use the broadly available FlashPlayer (99% of PCs have it installed)

### Next steps in your decision making

Ok, so hopefully you already have an idea of the DRM technology you want to use. However, that is just the beginning. There are a few more things to consider before starting to work on your solution.

#### Streaming technology

How are you going to deliver the content to your mobile client? Traditionally there have been a lot of different streaming technologies that used custom protocols such as [RTMP](http://en.wikipedia.org/wiki/Real_Time_Messaging_Protocol), but nowadays the industry is moving towards **[HTTP adaptive streaming](http://en.wikipedia.org/wiki/Adaptive_bitrate_streaming)** techniques. With HTTP adaptive streaming the video is sliced into chunks and deliver using regular HTTP file downloads. This techniques scale and bypass firewalls better, allowing segment caching, reducing server costs and in general providing a better user experience. Let’s enumerate a few of the most common options:

*   **[HLS](http://en.wikipedia.org/wiki/HTTP_Live_Streaming)** (HTTP Live Streaming): This technology was introduced by Apple, but has also been adopted by other mobile OS like Android. It is an HTTP adaptive streaming protocol that works by slicing the original video into MPEG2-TS segments of about 10 seconds and listing all of them (including different bitrates and qualities) into a M3U8 file. In general, HLS streams have big buffers which provide a very stable streaming but with latencies of about 30 seconds. This is the default streaming protocol for iOS.
*   **[Smooth Stream](http://www.iis.net/downloads/microsoft/smooth-streaming)**: This is a streaming technology property of Microsoft announced in October 2008. It is also an HTTP adaptive streaming solution that works in many senses like HLS. However, due to the use of MPEG4 files and some other internal differences, the Smooth Stream fragment size is typically of 2 seconds, what introduces in general a smaller latency and faster quality switches.
*   **[HDS](http://www.adobe.com/products/hds-dynamic-streaming.html)** (HTTP Dynamic Streaming): HDS is the Adobe’s alternative for using Adaptive HTTP Streaming with their products. It was announced in 2009 and first deliver in June 2010. Using MPEG4 files of 2-5 seconds, is a very similar solution to Microsoft’s Smooth Stream, and adapts the stream by checking bandwidth and CPU use of the device.
*   **[DASH](http://en.wikipedia.org/wiki/Dynamic_Adaptive_Streaming_over_HTTP)** (Dynamic Adaptive Streaming over HTTP): DASH is a streaming technology that was first announced in 2010. The inception of the idea behind this protocol came when  observing a bunch of vendor specific HTTP streaming solutions which work in very similar ways but which are totally incompatible between them. DASH tries to gather the best features of all existing HTTP adaptive streaming solutions and create an open standard out of all of them. The specification is still in development but has already received support from many big companies including Apple, Microsoft and Adobe. A promising future, but too early to work with it though.

Of course, these are only a few of the most popular existing options, but there is a plethora of  other solutions based and not based on HTTP streaming.

#### Client side Library

What’s next? After choosing your prefered DRM and streaming technique, then you have to look for the client side library that supports your desired configuration. Note that it may be hard or even impossible to use all possible configurations, and they will be dependent on the mobile OS to support. For example, PlayStream it is usually adopted together with Smooth Stream, and even if it can also be delivered on top of an HLS streaming you will need to use third party libraries on your iOS device, which will probably make things harder and more expensive.

#### Streaming server

As well as the client side library, you need to look for a streaming server that supports the desired configuration. There are many streaming servers out there that support multiple DRM and streaming techniques. Some examples are [Adobe Media Server](http://www.adobe.com/products/adobe-media-server-family.html), [Wowza](http://www.wowza.com/), [Red5](http://www.red5.org/), [IIS Media Service](http://www.iis.net/media),...  all of them with many different features, plugins and license models. You will have to look for the one that bests suits your needs.

#### Anything else?

Of course, there are many things that we have not commented - license models and costs, designing your server infrastructure, choosing a CDN, supported video encodings,... - but there is one that might be of special importance: Whenever you are working directly with a DRM solution (and not via an intermediate partner) you will likely need to make a partnership with the organization behind it, and it might get complicated depending on the requested requirements. For example, if you choose the Google’s Widevine you will need to pass the CWIP program before getting the iOS SDK. This is mainly because anyone with access to the client decoders could theoretically reverse engineer it, compromising the security of the whole DRM technique, and for that reason they want to control its access. 

### Easier alternatives

As we have seen so far, implementing a DRM solution end-to-end is quite complex, especially if you want to support multiple client platforms and mobile devices. There are a lot of things to consider, decisions to take and work to do. Because of that, there are other companies that offer simpler alternatives if you do not mind to pay extra money. Let’s see a couple of them:

*   **Packaged DRM Solutions**: Some companies offer closed packages that include a predefined set of DRM technologies - streaming servers and client libraries- already compiled, protected and ready to use, which will be probably easier to setup and use if you do not mind to pay a good amount of money. A good example of such a closed package is [Verimatrix](http://www.verimatrix.com/), which provides an end-to-end service with Marlin and PlayReady DRM that you can install in your own infrastructure and use in your mobile devices with their SDK.
*   **Streaming services on the Cloud**: Some companies provide the full streaming service on the cloud to help you setting up everything as fast as possible with proven scalability and quality. This approach frees you from installing the server software, managing the infrastructure to scale on consumption peaks, they provide many different streaming protocols, simple web CMS, and even sometimes they also provide DRM and client side libraries ready to use out of the box. Of course, this companies sell their services as a SaaS, and it will probably cost you more money if you make a heavy use of it, but can be a very good solution if your users and resources are limited. Some notable examples are [BrightCove](http://www.brightcove.com/), [Ooyala](http://www.ooyala.com/), [Kaltura](http://corp.kaltura.com/), [Azure Media Service](https://www.windowsazure.com/en-us/home/features/media-services/) or [Vmix](http://www.vmix.com/), but as usual many other options exist.

### What’s next

In the next post we will have a look to the Secure HLS technology proposed by Apple, and analyze why this can not be considered a full DRM solution. We will also take a look to DRM from a Webapp perspective. Keep tuned.