# Understanding WebViews

# 理解webviews

When it comes to accessing internet content, we typically use a browser like Chrome, Firefox, Safari, Internet Explorer, and Edge. You are probably using one of those browsers right now to read this article! While browsers are pretty popular for the whole task of accessing internet content, they have some serious competition that we probably never paid much attention to. This competition comes in the form of something known as a WebView. This article will explain all about what this mysterious WebView is and why it is sorta kinda cool.

我们经常使用浏览器要接触网上的内容，像 Chrome, Firefox, Safari, Internet Explorer, 以及 Edge。你很可能使用这里的一种来阅读这个文章！尽管浏览器在范文整个互联网的嘻嘻的时候非常受欢迎，但是他们也受到我们可能从没关注的严重的竞争。竞争来自于我们称之为WebView的形式。本文将会解释神秘的WebView是什么以及为什么它有些酷。

[competition]
n. the activity or condition of competing.
竞争，

Onwards!

向前！

***

WebView 101

WebView 概述

Let's get the boring definition out of the way. A WebView is an embeddable browser that a native application can use to display web content. There are a two sets of words to highlight here:

让我们把枯燥的定义放在一边。WebView 是一种嵌入式的浏览器，原生APP可以用来展示网页内容。这里有两组要强调的词：

[out of the way] 
放在一边 (of a place) remote; secluded.
[highlight]
突出 an outstanding part of an event or period of time.

The first set of words is native application (aka app). A native app is one written in the language and UI framework designed specifically for a particular platform:

第一组词是原生APP。原生APP是指使用使用针对特定平台设计的的语言和UI框架编写的应用程序：


In other words, the app isn't a cross-platform web app running in the browser. Instead, think of your apps as primarily being written in a language like Swift, Objective-C, Java, C++, C#, etc. that works more closely with the system. To put this into context, most apps you use in your mobile device are going to be native apps. Many popular apps like Microsoft Office on your desktop/laptops are as well.

话句话说，（~~APP在浏览器方面不是跨平台的网络应用~~）该应用不是在浏览器中运行的夸平台应用程序。相反，（~~想象你的app~~）将你的应用视为使用比如 Swift, Objective-C, Java, C++, C#这些更接近于系统的语言来编写。为了将其置入上下文中，你在移动端使用的大多数应用都是原生应用。许多流行的应用比如 Microsoft Office 在你的手机、电脑里是同样的。

The second words to highlight are embeddable browser. We all know what a browser is. It is a standalone app that we can use to surf the internet:

第二个要强调的词是嵌入式浏览器。我们都知道它是什么。它是一个独立的应用，可以用来上网：

[standalone]
独立 (of computer hardware or software) able to operate independently of other hardware or software.

If you think of the browser as two parts, one part is the UI (address bar, navigation buttons, etc.), and the other part is the engine that turns markup and code into the pixels we can see and interact with:

如果你把浏览器看做两部分组成，一部分是UI（像地址栏、导航按钮等），另一部分是引擎，用来把标记和代码转化成我们可以看的的像素，并可交互：

[markup]
标记 a set of tags assigned to elements of a text to indicate their relation to the rest of the text or dictate how they should be displayed.

A WebView is just the browser engine part that you can insert sort of like an iframe into your native app and programmatically tell it what web content to load.

一个 WebView 正是引擎部分，（~~你可以在原生应用里插入，可编程的来控制展现的内容~~）你可以将其插入到本机应用程序中，并以编程的方式告诉它要加载哪些Web内容。

Putting all of this together and connecting some dots, a WebView is just a visual component/control/widget/etc. that we would use as part of composing the visuals of our native app. When you are using a native app, a WebView might just be hiding in plain sight next to other native UI elements without you even realizing it:

综合上述并在联系到一些点，WebView 是一个 可视 组件/控件/小部件等。我们可以将它视为原生应用视觉效果的一部分。当你在使用原生应用的时候，WebView 可能隐藏在其他原生UI元素的旁边你可能都没注意到：

[dot]
点 a small round mark or spot.
[widget]
a small gadget or mechanical device, especially one whose name is unknown or unspecified.
[control]
控件
[plain]
朴素 not decorated or elaborate; simple or ordinary in character.

Your WebView is almost like a web-friendly island inside a large ocean of nativeness. The contents of this island don't have to be local to your app. Your WebView will commonly load web content remotely from a http:// or https:// location. This means you can take parts (or all) of your web app that lives on your server and rely on the WebView to display it inside your native app:

你的 WebView 像是一个在原生的海洋里的一个网页友好的小岛。所加载的内容不必放在本地，WebView通常会通过http:// 或 https:// 远程加载内容。这意味着你可以APP里的一部分内容放在服务端依赖 WebView 来在原生app里展现：

This flexibility opens up a whole world of code reuse between your browser-focused web app and the parts of your web app that you want to display inside a native app. If all of this doesn't sound crazy awesome...

这样的灵活性是的在聚焦浏览器的web应用和原生应用之间的代码复用成为可能。如果这一切请起来不是很棒。。。

[flexibility]
灵活性 the quality of bending easily without breaking.

...your JavaScript running inside your WebView has the ability to call native system APIs. This means you aren't limited by the traditional browser security sandbox that your web code normally has to adhere by. The following diagram explains the architectural differences that make this possible:

... WebView 里运行的 JavaScript 有能力访问原生系统的API。这意味着你的代码不必遵循传统浏览器的安全沙盒限制。下图解释了使他成为可能的结构性差异所在：

[sandbox]
sand 沙子
[adhere]
固守 stick fast to (a surface or substance).

By default, any web code running inside a WebView or web browser is kept isolated from the rest of the app. This is done for a host of security reasons that revolve around minimizing the extent of damage some malicious JavaScript might be able to do. If the browser or WebView go down, that's unfortunate but OK. If the entire system goes down, that is unfortunate...but NOT OK. For arbitrary web content, this level of security makes a lot of sense. You can never fully trust the web content that gets loaded. That isn't the case with WebViews. For WebView scenarios, the developer typically has full control over the content that gets loaded. The chance of malicious code getting in there and causing mayhem on your device is pretty low.

默认情况下，网页代码运行在WebView中或浏览器中是和应用的其他部分隔离的。这是出于乙烯类安全方面的考虑，尽量减少恶意JavaScript代码带来的破坏程度。如果浏览器或WebView出现故障，不幸但是没大问题。如果整个系统出现故障，不幸而且不OK。对于网络上任何内容，这样做是有意义的。你不能全部信任网页上加载的内容。WebView的情况并非如此。对于WebView的方案，开发者通常可以完全控制加载的内容。恶意代码进入你的设备并在成混乱的可能性非常低。
[minimize]
极少化 reduce (something, especially something unwanted or unpleasant) to the smallest possible amount or degree.
[extent]
程度 范围 the area covered by something.
[malicious]
恶毒 characterized by malice; intending or intended to do harm.
[arbitraay]
随意、肆意 based on random choice or personal whim,rather than any reason or system
[scenario]
脚本 a written outline of a moive,novel, or state work giving details of the plot and individual scenes.

【cm】**我认为还是手写英文会提升的更快。2019.06.12**

That is why, for WebViews, developers have a variety of supported ways to override the default security behavior and have web code and native app code communicate with each other. This communication commonly happens via something known as a bridge. You can see this bridge visualized as part of the Native Bridge and JavaScript Bridge in the earlier diagram. Going into detail on what these bridges look like goes beyond the scope of this article, but the key point to take away is the following: the same JavaScript you write for the web will not only work inside your WebView, it can also call into native APIs and help your app more deeply integrate with cool system-level functionality like sensors, storage, calendar/contacts, and more.

WebView Use Cases
Now that we've gotten an overview of what WebViews are and some of the powerful tricks they have up their sleeve, let's take a step back and look at some of the popular situations we'll see WebViews in our native apps.

In-App Browsers
One of the most common uses for a WebView is to display the contents of a link. This is especially true on mobile devices where launching a browser, switching the user from one app to another, and hoping they find their way back to the app is an exercise in disappointment. WebViews solve this nicely by loading the contents of the link fully inside the app itself.

Take a look a the following video of what happens when we click on a link in the Twitter or Facebook apps:



Neither Twitter nor Facebook load the linked content in the default browser. They instead use a WebView to fake an in-app browser and render the content as part of the app experience itself. Twitter's in-app browser looks really plain, but Facebook goes one step further with a fancy looking address bar and even a nifty menu:



 

I just happened to pick on Twitter and Facebook since I had those two apps installed and could readily record a video to share with all of you. There are bunch of apps that open links in a similar way by relying on the WebView to behave as an in-app browser.

Advertisements
Advertising still remains one of the most popular ways native apps make money. How are most of these ads served? As web content served through a WebView:



While native ads do exist, most native solutions utilize a WebView behind the scenes and serve the ads from a centralized ad server similar to what you would see in your browser.

Full Screen Hybrid Apps
So far, we've been looking at WebViews as minor supporting actors in a stage fully dominated by native apps and other native UI elements. WebViews have the depth and range to be the stars, and there is a large class of apps where the web content loaded inside the WebView forms the entire app user experience:



These apps are known as hybrid apps. From a technical point of view, these are still native apps. It just happens that the only native thing these apps do is host a WebView that, in turn, loads the web content and all the UI that users will interact with.	Hybrid apps are popular for several reasons. The biggest one is developer productivity. If you have a responsive web app that works in the browser, having the same app work as a hybrid app on a variety of devices is fairly simple:



When you make an update to your web app, the change will be instantly available to all devices that use it since the content is coming from one centralized location, your server:



If you had to deal with a pure native app, not only would have you have to update the project for each platform you built the app for, you may have to go through the time-consuming app certification process in order to make your update available via all of the app stores. From a deployment and updating point of view, hybrid apps are very convenient. Combine this convenience with native device access that gives your web apps superpowers, you have a winning technical solution. The WebView makes it all possible.

Native App Extensions
The last big category you'll see WebViews used in has to do with extensibility. Many native apps, especially on the desktop, provide a way for you to extend their functionality by installing an add-in or extension. Because of how easy and powerful web technologies are, these add-ins and extensions are often built in HTML, CSS, and JavaScript instead of C++, C#, or whatever. One popular example of this is Microsoft Office. The various apps that make up Microsoft Office are as native and old-school as you can get, but one of the ways you can build extensions for them involves web technologies. For example, a popular such extension is the the Wikipedia app:



The way these web-based extensions like the Wikipedia one are surfaced inside an Office app like Word is through...yep, a WebView:


The actual content shown inside the WebView comes from this URL. When you visit that page in the browser, you don't really see a whole lot. It is the intersection between the native app functionality and web code functionality (exposed via the WebView) that makes the full experience work. As a user of the Wikipedia extension inside the Word app, you may never question what is going on under the covers because the functionality is nicely integrated and just works.

WebViews (Usually) Aren't Special
WebViews are pretty awesome. While it may look like they are entirely special and unique beasts, remember, they are nothing more than just a browser positioned and sized inside your app without any of their fancy UI thrown in there. There is more to it, but that's the gist of it. For most purposes, you don't have to specially test your web app inside a WebView unless you are calling native APIs. Otherwise, the functionality between what you see inside a WebView is the same as what you would see in the browser, especially if you match the rendering engines:

On iOS the web rendering engine is always WebKit, the same one that powers Safari...and Chrome. Yes, you read that correctly. Chrome on iOS actually uses WebKit under the covers.
On Android, the rendering engine under the covers is usually Blink, the same one that powers Chrome.
On Windows, Linux, and macOS, since these are the more permissive desktop platforms, there is a lot of flexibility in choosing the WebView flavor and rendering engine used under the covers. The popular rendering engines you see will be Blink (Chrome) and Trident (Internet Explorer), but there is no one engine that you can rely on. It all depends on the app and what WebView implementation it is using.
We can spend more time looking at WebViews and going even deeper into some of the specialized behavior they provide, but that gets us a little too into the weeds. For what we are trying to do here, staying on the roads and having a broad view of what WebViews are is just right...for now.

If you have a question about this or any other topic, the easiest thing is to drop by our forums where a bunch of the friendliest people you'll ever run into will be happy to help you out!