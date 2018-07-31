---
layout: post
title:  "Playtomic's Shared Architecture using Swift and Kotlin"
---

Choosing the technology stack is one of the first and most important decisions when starting any project from scratch. At [Playtomic](https://playtomic.io), we knew that we wanted to pick a stack for the mobile apps that would allow us to deliver the best possible experience given our limited resources available.

Mobile stacks range over a plethora of alternatives:

#### Web technologies

Solutions like responsive web apps or progressive web apps allow you to leverage your web frontend experience while having a single project for all. Some native capabilities can not be used but for most apps it would be enough. Distribution is through web browsers and not inside an App Store which may be an advantage or disadvantage depending on your case

#### Hybrid

Next step on the road to native you can choose to use an hybrid framework like Phonegap/Cordoba, which also uses web technologies but get wrapped into an app bundle, offering extra capabilities and an improved UX over pure web.

#### Multiplatform Solutions

There are several multiplatform solutions like Xamarin, ReactNative or the newer Flutter. They all have their own selling points and disadvantages, but in general they offer a unified solution to build native apps by using a single language and, to some degree, a single UI layer with custom components, sharing most of the code across the platforms while delivering a good UX, very close (if not the same) than the one delivered by native apps.

#### Native

The industry standard, and the one that brings the best UX is also the one that requires the most resources. Building native means building an app per platform, each one with its own language, set of tools, libraries and frameworks.



## Our dilemma

Without getting into too much detail about each one and our analysis, we knew that we wanted Playtomic to be a leader in the sports area, and being a mobile first project we wanted it to bring the best possible experience to our end users.

We also knew that once picked, that technology stack would be our one and only one stack for long since we do not have the resource power to maintain a “Frankenstein app” built in parts with different stacks or Big Bang refactors to completely migrate from one to another. We wanted “the stack” to be production ready and with enough community and maturity to have the “certainty” that it will be supported for our project’s life.

That basically left us with 3 candidates: **Xamarin**, **ReactNative** and **Native**; and from those first two we were much more appealed by React than Xamarin because of its programming model, the possibilities to share code with a future web frontend and the amazing community and tooling.

On the other hand, when selecting a solution, you also have to consider the team you have. In our case, we were expert native developers, with experience in both platforms (Android and iOS) and with little to no experience in React or other multiplatform stacks. Besides, at that moment, there was no web frontend developer or anyone within the company with enough React experience to coach the mobile team during the learning curve.

Having all that in mind, **our best fit was native**. It delivers almost everything we wanted (best UX, industry standard, maturity, long term vision, great tools, great community,...) except for one important aspect: **cost**

As explained before, going native would mean to build 2 apps (iOS/Android), which has an extra cost compared to the single app of multiplatform solutions. But how much? Let’s try to put very rough numbers with an example (*note that these are not real numbers, they are just an estimate based on our own previous experience, don’t take them too seriously but just as an illustration*):

* **Native**: Let’s say you are building a feature that takes you 100h on iOS. Then, porting it to Android would take around 80h extra (not 100 because there is always knowledge that can be “ported” from the first platform). A total of 180h

* **Multiplatform**: The same feature would take you around 120h, depending on how much you can reuse and the technology used. It is not write once run everywhere but close enough to add only a small percentage of extra work over a single platform.

So, roughly 180h vs 120h, or in other words **around 50% extra time to go native**. That is quite a lot! Especially for a small team like ours.

So, our next question was: **can we find a way of building a native app maximizing reusability across platforms and keeping the cost down**, close to the one delivered by multiplatform solutions? And if so, will it take us less than 1-2 months work of setup? (That was the time we had until the product and design teams would start delivering well defined features to build)

I had participated in the past of some very small projects (Proof Of Concepts and a minor library) using this approach with excellent results. But building a full app is a completely different challenge, especially when the application grows.


## Shared foundations

So, we started building with one objective in mind: **reusability across native platforms**

What we did was to split the app in 3 main parts for both platforms:

—-diagram——

*  **Anemone SDK**: framework used to connect to our backend and provide persistence. It offers Model, Service and some utilities.
*  **Playtomic UI**: framework with visual components like custom textfields, input validators, generic datasources/adapters, animations, ...
*  **App code**: the application code, where our modules and  features are built. It includes and uses the former two.

We made sure that both frameworks offered the same public API (different implementations) and we also built a few facades over concepts that would be used across the app and that were provided differently on each platform, keeping the same API again. To name a few:

*   `Promise`
*   `HttpClient`
*   `JSONObject`
*   `NavigationManager`
*   `LocationManager`
*   ...

We also picked the combination of Swift/Kotlin because of their enormous similarities and we used [SwiftKotlin](https://github.com/angelolloqui/SwiftKotlin) to minimize the time needed to transpile code from iOS to Android.

Finally, we added a few extensions over foundation objects to provide some of the missing methods on one or the other language (ex: `compactMap`, `flatMap`, `let`,...)

## Internals

### AnemoneSDK

Our SDK basically offers Models and Services. It makes heavy use of networking and  promises (with a facade on top of [JDeferred](https://github.com/jdeferred/jdeferred) and [PromiseKit](https://github.com/mxcl/PromiseKit)) to deal with asynchronous calls. We chose Promises over RX alternatives because of simplicity, since we do not need very complex operations on this end. A few examples:

#### IHttpClient

A common interface to deal with networking. On iOS, it is implemented with `NSURLSession` while Android uses `OKHttp` library

```
—-code—-
```

#### Tenant

Model object that contains information of a club. Note how thanks to the common JSON interface code looks almost identical and behaves equally for example when we get a Number where a String was expected.

```
—-code—-
```

#### XxxService

See how there is a minor difference due to the language difference when using generics, but for the rest are almost identical.

```
—— code ——
```

### PlaytomicUI

This module contains UI components and utilities such as custom textfields, tag views, sliders, generic table cells, generic datasources/adapters, input validators, animations, view stylers,... they are packed into a framework/library and used by the application. An example:

```
—-Email validator—-
```

### Application

Our application code is divided into modules and managers.

Each module can have its own internal architecture and communicates with the others through the use of Coordinators. For most of our modules we use an MVP pattern, as the presenter manipulation required is pretty small (remember our Anemone SDK deals with our network and persistence, so that would be the Entity and Repository in a VIPER architecture, and our Coordinators correspond to the Routers). In some cases, where there is business logic or complex data manipulation involved in clients, we also use Interactors. Nevertheless, modules could be implemented in MVVM or other patterns as long as they publish a Coordinator for the rest to consume.

The benefit of splitting code this way is that our Presenters and Interactors have no platform dependencies so they can be transpiled with almost no work. Coordinators are also very similar and quick to transpile, leaving the Managers and the Views as the only parts that require specific work per platform.

\_\_\_\_gif transpiling\_\_\_\_

Let’s see a few examples of each of this:

#### XxxPresenter

```
—\- code —-
```

#### XxxCoordinator

```
—\- code —-
```

#### XxxView

```
—\- code —-
```

As you can see, by using PlaytomicUI components and some of our extensions, code in both platforms is also very similar even on the View layer. The main work on this part corresponds to laying out elements in Interface Builder (iOS) or in layout XMLs (Android).

An interesting note to make here is that we could  have decided to write the Views in code or with tools like [Layout](https://github.com/schibsted/layout). That would make possible to reuse much more here as well, but we intentionally chose not to because we wanted to keep this part (the one the user actually sees and experiences) as standard as possible. This also allows us to use and follow platform components and conventions when desired and the de facto developer tools available, hence keeping a moderate learning curve and taking advantage to a full extent of our native development expertise.

#### ILocationManager

A common interface to get user’s location from GPS. Platform implementations use `LocationManager` and `GoogleServices`.

```
ILocationManager
```

## The good, the bad and the ugly

After 1.5 years working with the explained Shared Architecture, conventions and tools, we have a pretty solid view of what’s working for us and what is not working that well. Let me try to make an introspection:

### The good

* **Team unification**: there is no Android/iOS team distinction because the same developer always transpiles his work to the other platform. This results in extreme union, platform parity and less disputes/blockages
* **Team performance**: developing app code is much faster than writing 2 platforms independently. It typically takes just a few minutes to transpile Presenters, Interactors, Coordinators, Models and Services. XML and Xib files takes the rest of the time, and every now and then some code in managers. In average, we take about 30% extra time to convert from one to the other platform, depending on the amount and complexity of the views involved, pretty close to multiplatform solutions.
* **Fully native UX**: Visual components and app performance is the same than any other native app. Besides, there is no extra penalty on app size nor app launch time like in multiplatform solutions.
* **Long term vision**: we use the facto tools, frameworks and languages on each platform, and we have no important dependencies. We can have the certainty that code will be valid for many years, even if at some point team grows and we stop sharing code they will still be valid standard native projects independently.
* **Good abstractions and code quality**: The fact that we want to reuse as much code as possible forces developers to think very carefully the abstractions they want to build. It encourages for proper separation of concerns, single responsibility classes, more testing (to verify the transpilation), etc. In fact I would even say that code reviews are also more effective as you can compare the PR side by side with the counterpart and detect issues on a higher level. Quality is not just desirable but it is also perceived as an actual productivity boost from day 1.
* **Reduced project’s cognitive load**: Having 1 code base makes understanding the project and remembering the internal details much easier.

### The bad

* **Extra architecture work**: it is no secret that building these shared abstractions and extensions take time. In our case we dedicated about 1 month to architectural foundations, and since then we have had to make some changes and additions every now and then. The total overhead is difficult to calculate, but it is noticeable especially at the beginning.
* **Hidden bugs from language differences**: transpilation works great, most of the time 💥. However, during these 18 months working on it, we have encountered 3 or 4 times bugs derived from language differences that were unexpected. Especially important is the Value type (`struct`) in Swift that has no counterpart in Kotlin or the sort in place of arrays in Kotlin. These differences impose restrictions and are a source of bugs if not considered properly.
* **Maximum common factor of language features**: in parallel to the previous bullet, having to share code imposes restrictions on the way you use a language (or more transpilation work is required). As a result, we tend to write code using the maximum common factor of language features. A few examples of limitations on Swift are value types and protocol extensions, while in Kotlin the usage of decorators or default parameters in interfaces.
* **View layer needs work per platform**: writing views require specific work per platform. That has an impact on development time, testing and bug fixing that would not ocurr that much with multiplatform solutions with shared UI.
* **Learning curve**: all the architectural components and code conventions that we use are specific from this project and therefore require some learning. Nevertheless, to be fair all projects have their own internal conventions and architecture design, so at least having the same across both platforms means that there is only 1 curve to pass and not 2.

### The ugly

* **Hybrid conventions**: Kotlin and Swift are very similar but they use different naming conventions. For example, in Kotlin/Java constants are written in upper case while in Swift they aren’t. Or the classical `I` prefix so common in Java does not exist in Swift (the closer would be to suffix `Protocol` to the name). As a result, when sharing code you have to either come with a mix of conventions or penalize the transpilation process with more manual edition to adapt from one to the other. We started with conventions per platform and we are gradually moving into a single convention that feels to us like the best of both worlds and which is becoming our de facto mobile team convention (but it would look “ugly” to external developers)
* **Replicate changes manually**: transpilation works great when building new features because you can copy&paste the full transpiled code. However, when maintaining code, it is up to the developer to remember to apply the same change made on platform A into platform B. As a result, we have sometimes forgotten to replicate, resulting in some inconsistencies on app behavior. We are controlling that through PR, forcing both platforms to have the same kind of changes and reviewing them in parallel, but there is still the case for human error.
* **Team scaling**: having such a small team helps when using this approach since it requires lots of communication between members. We are not sure how this would scale with more developers, but we suspect it won’t the day we have 6+ people or so. Besides, we are “forced” to hire (or teach) multiplatform experts as long as we want to keep sharing code efficiently.

Overall, when looking back, we feel the decision has been the correct one for our team. That does not mean that using React would have been a mistake (probably not), but we are very satisfied with the results we are getting. Sure, we run into some issues every now and then, and we have had to invest a couple of months on making the abstractions (this time would have gone to learning React anyway), but we have now the best UX possible with a very decent development speed. Moreover, we are not dependent on some third party framework or tool (even SwiftKotlin could be removed and just transpile code manually, which is not that bad anyway) what gives us long term confidence, and we are free to chose the application architecture we prefer per module (MVP, MVVM, VIPER, REDUX,...). We can also leverage all of the native goodies the instant they are announced and we can use the team knowledge to full extent.