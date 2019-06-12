# Web Architecture 101

# Web 主题综述
cm:101 是一个对初学者来说进入某个领域的大纲 的意思，参见[维基百科](https://en.wikipedia.org/wiki/101_(topic))

The basic architecture concepts I wish I knew when I was getting started as a web developer

副标题：我希望当时作为一名初学者就该掌握的关键概念

![架构图](http://prsfbfi6e.bkt.clouddn.com/trans/translate-001.png “架构图”)

The above diagram is fairly good representation of our architecture at Storyblocks. If you're not experienced web developer, you'll likely find it complicated. The walk through below should make it more approachable before we dive into the details of each component.

上图描述了我在 Storyblocks 用到的知识结构，如果你是初学者，可能会认为它复杂。在我们深入每个概念的细节之前先举个例子会更好理解些。

> A user searches on Google for “Strong Beautiful Fog And Sunbeams In The Forest”. The first result happens to be from Storyblocks, our leading stock photo and vectors site. The user clicks the result which redirects their browser to the image details page. Underneath the hood the user’s browser sends a request to a DNS server to lookup how to contact Storyblocks, and then sends the request.

> 某用户在google上搜索“Strong Beautiful Fog And Sunbeams In The Forest”，第一个结果恰巧来自于Storyblocks，我们领先的 [re]stock photo 和适量图网站。用户点击了链接并跳转到了详情页面。这个过程中浏览器请求了DNS的服务来找寻如何连接 Storyblocks, 之后发送请求。

> The request hits our load balancer, which randomly chooses one of the 10 or so web servers we have running the site at the time to process the request. The web server looks up some information about the image from our caching service and fetches the remaining data about it from the database. We notice that the color profile for the image has not been computed yet, so we send a “color profile” job to our job queue, which our job servers will process asynchronously, updating the database appropriately with the results.

> 这个请求打到了负载均衡，由它随机地转发到我们部署的10台web服务器里的一台来处理。web服务器从缓存服务中找寻这个照片的相关信息并且从数据库中获取相关数据。此时我们注意到这个照片的颜色属性还没有被计算出来，所以发起一次计算颜色的任务添加到队列，当这个任务异步处理完成之后，会用计算结果适当地更新数据库。

> Next, we attempt to find similar photos by sending a request to our full text search service using the title of the photo as input. The user happens to be a logged into Storyblocks as a member so we look up his account information from our account service. Finally, we fire off a page view event to our data firehose to be recorded on our cloud storage system and eventually loaded into our data warehouse, which analysts use to help answer questions about the business.

> 接下来，我们尝试使用图片的名称在全文搜索引擎里找相关的图片。用户恰好也是一个注册用户的话我们也会调取账户服务获取他的相关信息。最终，我们还会发送页面要被浏览的事件到云存储系统并最终收集到我们的warehouse,提供给分析师来帮助分析我们的商业问题。

> The server now renders the view as HTML and sends it back to the user’s browser, passing first through the load balancer. The page contains Javascript and CSS assets that we load into our cloud storage system, which is connected to our CDN, so the user’s browser contacts the CDN to retrieve the content. Lastly, the browser visibly renders the page for the user to see.

> 想在，服务器返回一个HTML的文本并且再次经过负载均衡发送给用户的浏览器。页面包含的javascript脚本和CSS文件也是存储在云静态资源，当然也是解析到我们的CDN。用户的浏览器会查找CDN来找到连接地址。最终浏览器渲染出页面用户就可以看了。

Next I'll walk you through each component, providing a "101" introduction to each that should give you a good mental model for thinking through web architecture going forward. I'll follow up with another series of articles providing sepecific implementation recommendations based onwaht I've learn in my time at Storyblocks.

接下来我会介绍每一个概念，罗列每个概念里的要点，以在你的脑海中构建一个完整的web架构。我将跟进另一个系列的文章，并根据我在 Storyblocks 学到的提供具体的实施建议。

[implementation]  
履行 performance, implementation, fulfillment

1. DNS
DNS stands for “Domain Name System” and it’s a backbone technology that makes the world wide web possible. At the most basic level DNS provides a key/value lookup from a domain name (e.g., google.com) to an IP address (e.g., 85.129.83.120), which is required in order for your computer to route a request to the appropriate server. Analogizing to phone numbers, the difference between a domain name and IP address is the difference between “call John Doe” and “call 201-867–5309.” Just like you needed a phone book to look up John’s number in the old days, you need DNS to look up the IP address for a domain. So you can think of DNS as the phone book for the internet.

DNS 即 域名系统，它是使得全世界的网络连接起来的一个关键技术。基本的DNS服务提供一种‘key/value’的查找，例如从域名(google.com)到一个IP(5.129.83.120),以便你的设备通过IP路由找到对应的服务器。举个手机号码的例子，域名到IP的对应就像你操作”呼叫 小明“和“拨号 201-867–5309”，就像以前需要一个电话本来查找小明的电话号码，你需要DNS来通过域名找到IP。你可以认为DNS就是互联网上的电话本。

[domain]  
域，域名 domain, region, territory
领域 field, area, domain, realm, sphere, territory
[appropriate]  
适当 appropriate, suitable, adequate, advisable, apposite
适合 appropriate, proper, fit, fitting, acceptable, decent

There’s a lot more detail we could go into here but we’ll skip over it because it’s not critical for our 101-level intro.

为了单纯做概要的介绍，我们选择忽略一些本可以介绍的细节。

[critical]  
临界 critical, crossover, crucial, extreme, climacteric, terminate


2. Load Balancer
Before diving into details on load balancing, we need to take a step back to discuss horizontal vs. vertical application scaling. What are they and what’s the difference? Very simply put in this StackOverflow post, horizontal scaling means that you scale by adding more machines into your pool of resources whereas “vertical” scaling means that you scale by adding more power (e.g., CPU, RAM) to an existing machine.

在说负载均衡之前，我们先说下应用部署的和横向和纵向的扩展。它们分别是什么以及它们的区别？[re]就像这个[StackOverflow](https://stackoverflow.com/questions/11707879/difference-between-scaling-horizontally-and-vertically-for-databases)所描述的，横向扩展意味着你通过给资源池增加机器来实现而纵向扩展是指通过提高现有机器的性能（如CPU，RAM）来实现。

[vertical]  
垂直 vertical, perpendicular
[horizontal]  
横 horizontal, transverse, traverse, unexpected, perverse, unruly

In web development, you (almost) always want to scale horizontally because, to keep it simple, stuff breaks. Servers crash randomly. Networks degrade. Entire data centers occasionally go offline. Having more than one server allows you to plan for outages so that your application continues running. In other words, your app is “fault tolerant.” Secondly, horizontal scaling allows you to minimally couple different parts of your application backend (web server, database, service X, etc.) by having each of them run on different servers. Lastly, you may reach a scale where it’s not possible to vertically scale any more. There is no computer in the world big enough to do all your app’s computations. Think Google’s search platform as a quintessential example though this applies to companies at much smaller scales. Storyblocks, for example, runs 150 to 400 AWS EC2 instances at any given point in time. It would be challenging to provide that entire compute power via vertical scaling.

在实际工作中，你大多数（几乎全部）想要去横向扩展，保持简单，[re]内容会中断，服务器会挂掉、网络会变慢、数据中心偶尔会连接不上。拥有多台机器可以使你的应用在意外中断的情况下也能运转。换句话说，你的应用是具有容错性的。其次，横向扩展可以把后端服务（业务服务、数据库、X服务）分成几块，分别部署在自己的机器上。最后，你可能会遇到不能纵向扩展的情况，没有一个机器性能足够强悍来完成应用的所有计算。作为一个典型的例子，想想谷歌搜索平台，尽管这适用于规模小的多的公司。拿 Storyblocks 来说，常年运行着 150-400台的 AWS EC2 实例来提供服务。如果通过纵向扩展来提供计算资源是一项挑战。

[stuff]  
东西 thing, stuff, east and west
材料 material, stuff, data, makings
[degrade]  
降级 degrade, demote, reduce to a lower rank
[occasionally]  
偶尔 occasionally, once in a while, haphazard, haply
[outages]
a period when a power supply or other service is not available or when equipment is closed down.
[tolerant]  
宽容 tolerant, lenient
[minimally]  
微创
[quintessential]  
典型 representing the most perfect or typical example of a quality or class.

Ok, back to load balancers. They’re the magic sauce that makes scaling horizontally possible. They route incoming requests to one of many application servers that are typically clones / mirror images of each other and send the response from the app server back to the client. Any one of them should process the request the same way so it’s just a matter of distributing the requests across the set of servers so none of them are overloaded.

我们继续说负载均衡，它像神奇的工具一样可以实现横向扩展。它转发进来的请求到部署的服务器中的一个，这些机器是各自的一个副本或镜像，服务器发送响应到客户端。每一台服务器都以相同的方式处理请求，这样负载均衡只需要通过配置文件分配请求而我们不同担心某台机器过载。

[sauce]  
酱汁
[distribute]  
分发 distribute, issue, hand out  
分配 assign, allocate, distribute, allot, apportion, assort

That’s it. Conceptually load balancers are fairly straight forward. Under the hood there are certainly complications but no need to dive in for our 101 version.

就是这样，概念上来说负载均衡是很简单的。当然其中有一些复杂的细节我们在这里略过。

[complication]  
并发症 complication  
复杂 complexity, complication, intricacy, complicacy

3. Web Application Servers

3. Web服务器

At a high level web application servers are relatively simple to describe. They execute the core business logic that handles a user’s request and sends back HTML to the user’s browser. To do their job, they typically communicate with a variety of backend infrastructure such as databases, caching layers, job queues, search services, other microservices, data/logging queues, and more. As mentioned above, you typically have at least two and often times many more, plugged into a load balancer in order to process user requests.

抽象的来说，Web服务器比较容易描述。它们响应用户请求执行核心业务计算并返回HTML给客户端。为了完成作业，它们经常需要和更后端的一些结构进行通信，像数据库、缓存层、任务队列、搜索服务、其他微服务、[re]日志等。以上提及的，[re]你需要至少两个经常更多，插入负载均衡来处理用户的请求。

[infrastructure]  
基础设施 base installation, base station, infrastructure  
[plug]  
扑落 shake off, plug, scatter, shake out of  
这里取插入的意思  

You should know that app server implementations require choosing a specific language (Node.js, Ruby, PHP, Scala, Java, C# .NET, etc.) and a web MVC framework for that language (Express for Node.js, Ruby on Rails, Play for Scala, Laravel for PHP, etc.). However, diving into the details of these languages and frameworks is beyond the scope of this article.

你需要知道的是，web服务器处理请求需要选择一门后端语言(Node.js, Ruby, PHP, Scala, Java, C# .NET, 等.)和一个对应的MVC框架（Node.js 的 Express、Ruby on Rails, Scala 的 Play, PHP de Laravel 等）。但是，详细述说这些语言以及框架已经超出了本文的范围。

[scope]  
范围 range, scope, extent, area, limits, reach  

4. Database Servers

4. 数据库服务

Every modern web application leverages one or more databases to store information. Databases provide ways of defining your data structures, inserting new data, finding existing data, updating or deleting existing data, performing computations across the data, and more. In most cases the web app servers talk directly to one, as will the job servers. Additionally, each backend service may have it’s own database that’s isolated from the rest of the application.

每一个现代的网络应用都使用了一个或多个数据库来存储信息。数据库提供了一种方式来定义数据结构、插入新的数据、查找存在的数据、更新删除存在的数据、[re]通过这些数据来做一些计算，等等。多数情况下Web服务器会和一个数据库通信，[re]业务服务也会。另外，每一个后端服务也可能会有自己独立的服务器。

[leverage]  
杠杆作用 grip purchase hold support anchorage force strength  
[isolated]  
孤立的 far away from other places, buildings, or people; remote.  
[Additionally]  
另外 additionally, in addition, furthermore, moreover, besides  
[perform]  
履行 grip purchase hold support anchorage force strength  

While I’m avoiding a deep dive on particular technologies for each architecture component, I’d be doing you a disservice not to mention the next level of detail for databases: SQL and NoSQL.

尽管我尽量避免过多的设计一个核心概念的细节，然而我要是不说下这两个数据库：SQL 和 NOSQL 就有点对不住读者了。

[disservice]  
a harmful action.  
坏处 harm, hurt, disadvantage, damage, detriment, disservice


SQL stands for “Structured Query Language” and was invented in the 1970s to provide a standard way of querying relational data sets that was accessible to a wide audience. SQL databases store data in tables that are linked together via common IDs, typically integers. Let’s walk through a simple example of storing historical address information for users. You might have two tables, users and user_addresses, linked together by the user’s id. See the image below for a simplistic version. The tables are linked because the user_id column in user_addresses is a “foreign key” to the id column in the users table.

SQL,即结构性查询语言，是1970年代发明的，提供一种查询关系的数据的标准方式，有广泛的用户。SQL数据库使用表来存储数据，表之间使用共同的ID、通常是数字来关联。我们来看一个简单的例子：存储用户的历史地址信息。你可能有两张表，用户表和用户地址信息表，通过用户ID关联。下图是简单的示例。表是关联的因为地址表里的user_id列是用户表里的id的外键。

[audience]  
读者 reader, audience, readership, constituency  
听众 audience, listeners  
这里是受众更好 

![数据库示例](http://prsfbfi6e.bkt.clouddn.com/trans/translate-002.png “数据库示例”)

If you don’t know much about SQL, I highly recommend walking through a tutorial like you can find on Khan Academy here. It’s ubiquitous in web development so you’ll at least want to know the basics in order to properly architect an application.

如果你不知道SQL,我强烈推荐你在 [Khan Academy](https://www.khanacademy.org/computing/computer-programming/sql) 上找一些教程看。在你构建一个应用的过程中它是到处存在的，你至少需要知道一些基础额东西。

[tutorial]  
个别指导 a period of instruction given by a university or college tutor to an individual or very small group.  
[ubiquitous]  
普及 present, appearing, or found everywhere.  
[architect]  
design and make.  

NoSQL, which stands for “Non-SQL”, is a newer [re]set of database technologies that has emerged to handle the massive amounts of data that can be produced by large scale web applications (most variants of SQL don’t scale horizontally very well and can only scale vertically to a certain point). If you don’t know anything about NoSQL, I recommend starting with some high level introductions like these:

NoSQL, 即‘非-SQL’,是一种新的数据库技术，它的出现解决了在扩展应用时出现的大量的数据（许多不同的SQL不能很好地水平扩展只能在某一点上纵向扩展）。如果你不知道NoSQL,我建议从以下高级介绍开始：

[emerge]  
出现 move out of or away from something and come into view.  
[variants]  
变种 a form or version of something that differs in some respect from other forms of the same thing or from a standard.  

https://www.w3resource.com/mongodb/nosql.php
http://www.kdnuggets.com/2016/07/seven-steps-understanding-nosql-databases.html
https://resources.mongodb.com/getting-started-with-mongodb/back-to-basics-1-introduction-to-nosql


I would also keep in mind that, by and large, the industry is aligning on SQL as an interface even for NoSQL databases so you really should learn SQL if you don’t know it. There’s almost no way to avoid it these days.

我同时也会记住，大体上，业界正在将SQL作为接口同时NoSQL也是，你要是不知道SQL还是需要学习下的。工作中几户避免不了会接触它。

[by-and-large]  
大体上  
[align]  
place or arrange (things) in a straight line.  
行程一条直线；取这里取共识意思吧  

5. Caching Service

5. 缓存服务

A caching service provides a simple key/value data store that makes it possible to save and lookup information in close to O(1) time. Applications typically leverage caching services to save the results of expensive computations so that it’s possible to retrieve the results from the cache instead of recomputing them the next time they’re needed. An application might cache results from a database query, calls to external services, HTML for a given URL, and many more. Here are some examples from real world applications:

缓存服务提供一类简单的 key/value 数据结构，以接近 O(1) 的时间来找寻信息使得可能。应用通常使用缓存服务来保存开销大的计算后结果。使得下次需要的时候可以从缓存中拿到数据而不是重新计算。一个应用可能会缓存数据库的查询结果，外部服务的调用结果，一个URL返回的HTML，等等许多。这里有一些真实世界的例子：

[retrieve]  
取回，恢复 get or bring (something) back; regain possession of.  

* Google caches search results for common search queries like “dog” or “Taylor Swift” rather than re-computing them each time

* 谷歌缓存一些简单的搜索词的页面，比如输入'狗'或'泰勒斯威夫特'，而不是每次都重新计算

* Facebook caches much of the data you see when you log in, such as post data, friends, etc. Read a detailed article on Facebook’s caching tech here.

* Facebool 缓存你登录后的尽可能多的数据，比如发过的信息，朋友们，等等。[这里](https://medium.com/@shagun/scaling-memcache-at-facebook-1ba77d71c082)可以读到 Facebook 使用的缓存技术的文章


* Storyblocks caches the HTML output from server-side React rendering, search results, typeahead results, and more.

* Storyblocks 缓存从服务端出来的 React 页面，搜索结果，预输入结果，以及更多的。

[typeahead]  
预输入

The two most widespread caching server technologies are Redis and Memcache. I’ll go into more detail here in another post.

两个使用最广泛的缓存技术是 Redis 和 Memcache 。 我将在本栏目的另一篇文章里细说。

6. Job Queue & Servers

6. 任务队列 和 服务

Most web applications need to do some work asynchronously behind the scenes that’s not directly associated with responding to a user’s request. For instance, Google needs to crawl and index the entire internet in order to return search results. It does not do this every time you search. Instead, it crawls the web asynchronously, updating the search indexes along the way.

大多数的Web 应用需要后台做一些与用户请求不直接相关的异步的任务。比如，Google需要为了返回结果需要找寻整个互联网。但不是你每次搜索都会这样。相反，它异步的爬寻互联网，并同时更新搜索结果。

[crawl]  
爬 an act of moving on one's hands and knees or dragging one's body along the ground.

While there are different architectures that enable asynchronous work to be done, the most ubiquitous is what I’ll call the “job queue” architecture. It consists of two components: a queue of “jobs” that need to be run and one or more job servers (often called “workers”) that run the jobs in the queue.

尽管有不同的方案可以使得异步的任务完成，但是最普遍的额方式是我称之为‘任务队列’的方式。它由两部分组成：一个等待被执行的充满任务的队列 以及 以及一个或多个 任务服务（通常称为works）来执行队列里的任务。

Job queues store a list of jobs that need to be run asynchronously. The simplest are first-in-first-out (FIFO) queues though most applications end up needing some sort of priority queuing system. Whenever the app needs a job to be run, either on some sort of regular schedule or as determined by user actions, it simply adds the appropriate job to the queue.

任务队列存放着一堆等待被异步执行的任务。应用大多数都使用先进先出队列，但是最终还是需要某种优先队列。当应用程序需要运行一个任务时候，无论是常规计划里的一个还是由用处触发的操作，都是只需要将任务添加到相应的队列里。

[priority]  
优先权 a thing that is regarded as more important than another.

Storyblocks, for instance, leverages a job queue to power a lot of the behind-the-scenes work required to support our marketplaces. We run jobs to encode videos and photos, process CSVs for metadata tagging, aggregate user statistics, send password reset emails, and more. We started with a simple FIFO queue though we upgraded to a priority queue to ensure that time-sensitive operations like sending password reset emails were completed ASAP.

拿 Storyblocks 来说，会有个任务队列来支持后台的为了市场做的任务。我们在 编码对视和图片，处理CSV文件的原信息[re]标签，聚合用户数据，发送密码充值邮件等都会做任务队列。 我们从一个简单的先进先出的队列开始，后来更新成一个优先队列，来保证对时间敏感度高的操作先执行，比如发送密码重置邮件。
[aggregate]  
合计 formed or calculated by the combination of many separate units or items; total.  
[statistics]  
统计 the practice or science of collecting and analyzing numerical data in large quantities, especially for the purpose of inferring proportions in a whole from those in a representative sample.

Job servers process jobs. They poll the job queue to determine if there’s work to do and if there is, they pop a job off the queue and execute it. The underlying languages and frameworks choices are as numerous as for web servers so I won’t dive into detail in this article.

任务服务处理任务。它们轮训任务队列来决定是否有任务要执行，如果有则拿出一个任务并执行。众多的后台语言以及框架可供选择，我在这里就不必深入细节了。
[poll]  
轮询 poll, vote  
投票 vote, poll, cast a vote  
[underly]  
下面的，隐含的 (especially of a layer of rock or soil) lie or be situated under (something).  


7. Full-text Search Service

7. 全文检索服务

Many if not most web apps support some sort of search feature where a user provides a text input (often called a “query”) and the app returns the most “relevant” results. The technology powering this functionality is typically referred to as “full-text search”, which leverages an inverted index to quickly look up documents that contain the query keywords.

很多应用（如果不是大多数）提供某种检索服务，用户提供一个检索词，应用返回最‘关联’的结果。支持这样功能的技术通常叫做‘全文搜索’，它利用反向索引快速查找文档中包含的检索词。

[inverted]  
倒，颠 put upside down or in the opposite position, order, or arrangement.

![检索示意图](http://prsfbfi6e.bkt.clouddn.com/trans/translate-003.png “检索示意图”)  

Example showing how three document titles are converted into an inverted index to facilitate fast lookup from a specific keyword to the documents with that keyword in the title. Note, common words such as “in”, “the”, “with”, etc. (called stop words), are typically not included in an inverted index.

图示展示了三个文档的标题是如果转为反向序列，使得使用文档标题里的关键词搜索的话很容易命中。还是得说下，一般的词比如‘是’‘和’‘里’等（称为终止符），不会称为反向索引。

[facilitat]  
促进 make (an action or process) easy or easier.  


While it’s possible to do full-text search directly from some databases (e.g., MySQL supports full-text search), it’s typical to run a separate “search service” that computes and stores the inverted index and provides a query interface. The most popular full-text search platform today is Elasticsearch though there are other options such as Sphinx or Apache Solr.

尽管某些数据库提供了全文检索功能（比如MySql），我们经常还是会起一个检索服务用来计算和存储反响索引并提供查询接口。现在最流行的全文检索服务是 Elasticsearch ，同时也还有一些别的可选，比如  Sphinx 或 Apache Solr.

8. Services

8. 服务

Once an app reaches a certain scale, there will likely be certain “services” that are carved out to run as separate applications. They’re not exposed to the external world but the app and other services interact with them. Storyblocks, for example, has several operational and planned services:

一旦应用达到了一定程度的规模，他们可能作为一个独立的服务分离出来。他们没有暴露于外部世界，但是应用和其他服务可以与之通信。比如 Storyblocks , 有几项[re]运营和计划服务：

[carve]  
雕刻 cut (a hard material) in order to produce an  aesthetically pleasing object or design.  
[scale]取规模比较好   
规模 scale, scope, extent, proportion  
比例 proportion, scale  
尺度 scale, dimension, measure, yardstick, criterion  
[expose]  
暴露 make (something) visible, typically by uncovering it.  
[interact]  
相互作用 act in such a way as to have an effect on another; act reciprocally.  

* Account service stores user data across all our sites, which allows us to easily offer cross-sell opportunities and create a more unified user experience

* 账户服务存储所有网站上的用户信息，允许我们更容易实现交叉销售并有统一的用户体验
[unified] 
统一的 make or become united, uniform, or whole.

* Content service stores metadata for all of our video, audio, and image content. It also provides interfaces for downloading the content and viewing download history.

* 内容服务存储着我们所有的视频、音频、图像的元信息。同时还提供下载和查看下载历史的接口。

* Payment service provides an interface for billing customer credit cards.

* **支付服务**提供用户信用卡支付的接口。

* HTML → PDF service provides a simple interface that accepts HTML and returns a corresponding PDF document.

* HTML转PDF 服务提供了简单的接口，实现了HTML转相应的PDF文件。

[correspond]
相应地 similar in character, form, or function.


9. Data

9. 数据

Today, companies live and die based on how well they harness data. Almost every app these days, once it reaches a certain scale, leverages a data pipeline to ensure that data can be collected, stored, and analyzed. A typical pipeline has three main stages:

今天，公司的生死依赖于他们挖掘他们数据的能力。几户所有的应用，当他们达到一定规模的时候，都会有[re]数据通道来保证数据可以被手机、存储和分析。一个典型的统统有三个主要的阶段：
[harness]
英文释意：control and make use of (natural resources), especially to produce energy.

1. The app sends data, typically events about user interactions, to the data “firehose” which provides a streaming interface to ingest and process the data. Often times the raw data is transformed or augmented and passed to another firehose. AWS Kinesis and Kafka are the two most common technologies for this purpose.

1. 客户端发送数据（通常是用户交互操作）到数据 "firehose" ，它提供流接口来摄取和加工数据。经常数据被转换并扩厂值另一个firehose。AWS Kinesis 和 Kafka 是用于此目的使用最多的技术。

[firehose]
消防水管，流数据捕获
[ingest]
en: take (food, drink, or another substance) into the body by swallowing or absorbing it.
[augmented]
adj. having been made greater in size or value.
v. make (something) greater by adding to it; increase.

2. The raw data as well as the final transformed/augmented data are saved to cloud storage. AWS Kinesis provides a setting called “firehose” that makes saving the raw data to it’s cloud storage (S3) extremely easy to configure.

2. 原始数据以及最终转换并扩充的数据将保存在云存储上。 AWS Kinesis 提供了被称为 “firehose” 的设置，可以将原始数据保存在云上，并且很容易配置。
[raw]
adj. raw, green, living, crude, unripe, uncooked；original, primary, former, raw, unsorted, one-time
生的，不熟的，原始的，

3. The transformed/augmented data is often loaded into a data warehouse for analysis. We use AWS Redshift, as does a large and growing portion of the startup world, though larger companies will often use Oracle or other proprietary warehouse technologies. If the data sets are large enough, a Hadoop-like NoSQL MapReduce technology may be required for analysis.

3. 转移和扩充的数据经常被加载到数据仓库进行分析。我们使用 AWS Redshift，[re]创业恭送的大部分和不断增长的部分也是，尽管大公司常用 Oracle 或者其他准有仓库技术。如果数据集足够大，可能需要类似 Hadoop 的 NoSQL MapReduce 来进行分析。

[warehouse] 仓库
n. warehouse, depot, storehouse, depository, depositary, wareroom
[portion] 部分
n. part, portion, copy, share
[proprietary]所有权
n.ownership, proprietary, property, possession, proprietorship, property right
[set]集合
n. set, assembly, assemblage, muster, congregation

Another step that’s not pictured in the architecture diagram: loading data from the app and services’ operational databases into the data warehouse. For example at Storyblocks we load our VideoBlocks, AudioBlocks, Storyblocks, account service, and contributor portal databases into Redshift every night. This provides our analysts a holistic dataset by co-locating the core business data alongside our user interaction event data.

架构图上没有描述的一步是：将数据从应用和服务的操作数据库中加载到数据仓库中。比如在 Storyblocks ，每晚我们都会讲 VideoBlocks, AudioBlocks, Storyblocks，账户服务和通闲着门户的数据加载到Redshift。这给了我们的额分析师一个身体的数据集。通过将核心数据和用户的交互数据放在一起对比。

[diagram] 图表；图解
n. chart, graph, diagram;diagram, graph, graphic solution, figure
[portal]正门；阳台
n. balcony, terrace, veranda, porch, verandah, portal
[holistic]整体的
[colocate]放在一起
v. share a location or facility with someone (or something) else.

10. Cloud storage

10. 云存储

“Cloud storage is a simple and scalable way to store, access, and share data over the Internet” according to AWS. You can use it to store and access more or less anything you’d store on a local file system with the benefits of being able to interact with it via a RESTful API over HTTP. Amazon’s S3 offering is by far the most popular cloud storage available today and the one we rely on extensively here at Storyblocks to store our video, photo, and audio assets, our CSS and Javascript, our user event data and much more.

据AWS称，“云存储是通过互联网进行存储、访问、共享数据的一种简单可扩展的方式”。你几乎可以把能存在本地文件系统的任何文件放到云上，使用RESTful API通过HTTP协议来进行快捷的交互。亚马逊的S3产品是目前市面上最流行的云存储产品，也是我们在 Storyblocks 广泛依赖的产品，来存储视频、图片以及音频信息、CSS和JavaScript文件、用户事件数据等等。
[access]访问；通路
v. access, interview, call, pay a visit;n.path, access, passageway, route, thoroughfare, chain
[extensively]广泛地

11. CDN

11. CDN

CDN stands for “Content Delivery Network” and the technology provides a way of serving assets such as static HTML, CSS, Javascript, and images over the web much faster than serving them from a single origin server. It works by distributing the content across many “edge” servers around the world so that users end up downloading assets from the “edge” servers instead of the origin server. For instance in the image below, a user in Spain requests a web page from a site with origin servers in NYC, but the static assets for the page are loaded from a CDN “edge” server in England, preventing many slow cross-Atlantic HTTP requests.

CDN 即 “Content Delivery Network”，提供了一种服务于静态资源例如 静态HTML、CSS、JavaScript和图片比源服务器快得多的访问方式。它通过把内容分发到世界各地的边缘节点使得用户可以从边缘节点下载资源，而不是源服务器。如下图中例子，西班牙的用户请求源服务器在NYC的页面，但是静态资源从英国额边缘节点获取，阻止了许多缓慢的夸大西洋的HTTP请求。

[prevent]防止；预防；阻止
prevent, avoid, guard against, forestall;
prevent, protect, safeguard;
prevent, stop, block, deter, impede, arrest

Source

![CDN示意图](http://prsfbfi6e.bkt.clouddn.com/trans/translate-004.png “CDN”)

Check out this article for a more thorough introduction. In general a web app should always use a CDN to serve CSS, Javascript, images, videos and any other assets. Some apps might also be able to leverage a CDN to serve static HTML pages.

[点击这里](https://www.creative-artworks.eu/why-use-a-content-delivery-network-cdn/)可以看到更透彻的介绍。通常Web网络应用应该始终使用CDN来提供CSS、Javascript、图片、视频一节其他资源。有一些应用可能可会使用CDN存储静态的HTML页面。

[thorough]
彻底 thorough, complete
透彻 thorough, penetrating

Parting thoughts



And that’s a wrap on Web Architecture 101. I hope you found this useful. I’ll hopefully post a series of 201 articles that provide deep dives into some of these components over the course of the next year or two.

这是Web架构101的一个简介，我希望你能发现这有用。我希望发布一系列的201文章来对这些概念做些耿深入的研究。






出处: https://engineering.videoblocks.com/web-architecture-101-a3224e126947