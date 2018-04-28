---
layout: default
---

Shown by relevant data by the end of 2017, China's mobile phone news App scale has reached 636 million people, and mobile App has become one of the most important ways of news and content dissemination. With the competition and development of the industry, the **content page in App** plays a more important role in improving the quality of App, enhancing the Time on App and increasing the viscosity of the user. Meanwhile, it also faces more challenges.

1.	**Content pages are becoming more and more rich in presentation.**As the main part of the content page, news data has gradually increased more text style, content form, rich media, and more abundant elements such as advertising, voting and so on.

2. **Content pages need more extended areas to increase user time and viscosity of user.**In addition to the information part, each App has gradually created more and more extended reading part, such as concern module, recommended reading module, comment module, App operations module and so on.
3. **The competition for short video and live broadcast is becoming more and more intense.**More and more news App display videos as an independent module and an independent content page.
4.	**App similarity**The homogenized products are competing fiercely. It requires faster update speed, better user experience and smaller implementation cost.

As a result, the architecture design and technology optimization of the news App content page should also cooperate with the development of product form, and have the ability to extend and a stable and high quality experience under the increasingly complex demand challenge. 

Based on the analysis of the current news App content page technology options, such as `今日头条、腾讯新闻、天天快报、一点资讯等` this paper will explore the technical implementation and optimization of the news App content page.

<br>

> ***
>_插播广告 —— 几十行代码完成新闻类App多种形式内容页_ 
>
>_[HybridPageKit](https://github.com/dequan1331/HybridPageKit) ：一个针对新闻类App高性能、易扩展、组件化的通用内容页实现框架。_
>
>_基于[ReusableNestingScrollview](https://github.com/dequan1331/ReusableNestingScrollview)、[WKWebViewExtension](https://github.com/dequan1331/WKWebViewExtension)、以及本文中关于内容页架构和性能的探索。_
>
>***

<br>

## 1. Definitions

Combined with the current mainstream content page implementation, we divide the content pages into two parts. And in order to facilitate subsequent reading, we simply define the key nouns.

1.	The upper part is divided into common WebView.It usually includes title + author (follow) + content, which we call **`WebView content area`**.
2. 	The next half is mainly a variety of extensions that are parallel to the WebView.  It usually includes praise and award, advertising, related recommendations, popular comments, and so on. We call it the **`Native extension area`**.
3. Each independent module , complex UI presentation in WebView and extension area is called a **`module`** or **`component`**.

As a whole, the right page of the entire content page is generally the comment page. Whether slip scrollview to the right or push a new page controller, these two ways are simpler and more independent, so this paper will temporarily leave away the right comment section.

## 2. Contents

<center><img width="80%" height="80%" src="https://raw.githubusercontent.com/dequan1331/dequan1331.github.io/master/assets/img/index.png"></center>

## <center>- Technical solutions -</center>
***

## 1.WebView Type
	
Unlike Weibo, the content of news App is mainly composed of paragraphs, with pictures and rich media among it. At the same time, in order to the same presentation of cross platform, the reproduce of PC web pages、different website & App articles, and the emphasis on reading instead of interaction,  using **WebView** to load and render local HTML string data has become a universal solution for the news App.

### 1. UIWebView ~~VS~~ WKWebView

-	Stability:
	
	UIWebView has many WebCore, JavaScriptCore Crash, and systematic memory leak may cause OOM, which is a great danger to the stability of the App. In contrast, WKWebView, based on the independent process, does not occupy App memory computation, and does not lead to the main App Crash. So in terms of system level stability, WKWebView has great advantages.

- 	Speed:
	
	WKWebView greatly optimizes the speed of JS through JIT, but for the scene of the news App content page, simple entry&exit page, and simply load and render HTML string, WKWebView is much slower than UIWebView.（[Benchmark](https://github.com/dequan1331/WebViewBenchMark)）。
	
-  	Compatibility:

	NSURLProtocol、longpress MenuItems Bug（before iOS11）、clear cache in iOS8、config Cookies and UA、POST paras、async evalute JS...This series of problems has become the biggest challenge to replace WKWebView.
	
-  	Extensibility:

	WKWebView has more interfaces, more HTML and CSS support, and more friendly JS interaction. At the same time, the continuous updating of Api and the active community are of great advantages in terms of long-term use.

### 2. Extension of WKWebView

Through the above analysis, WkWebView has great advantages from system level stability, performance and subsequent extensibility. By extending the native WKWebView [WKWebViewExtension](https://github.com/dequan1331/WKWebViewExtension) and reuse of the WKWebView [HybridPageKit](https://github.com/dequan1331/HybridPageKit), the problem of native WKWebView has been solved to a great extent, and it has played a very good effect. 

-	Extensions:

	Through the analysis of time consuming by stages, under the usage scenario of content pages, WKWebView has great optimization space from alloc to start rendering. In [HybridPageKit](https://github.com/dequan1331/HybridPageKit), the reuse of WKWebView and use HTTP cache greatly reduces the time for WKWebView to load and render HTML, making the time consuming lower than the native UIWebView. 

	Through the private methods and code optimization, [WKWebViewExtension](https://github.com/dequan1331/WKWebViewExtension) supports NSURLProtocol, fix the MenuItems bug under iOS11, and supports cleaning the cache in iOS8, the security JS method, and the secondary NavigationDelegate for JSBridge logic of old code.

- 	Problems no need to be solved:

	For the usage scenarios of news App content pages, some WKWebView problems do not necessarily to a universal solution. For example, requests cannot be carried with POST parameters, and Javascript can be executed asynchronously. All these problems can be solved by refactoring the code. In particular, it is not recommended that holding Runloop to synchronized with JS.

-  	Remaining problems:

	At present, in the process of using WKWebView, the only unsolved problem is a reliable and comprehensive white screen detection solution, which supports the reloading of the WKWebView in any case of Crash.  Methods such as system Crash callback, observing WebView Title,  contentSize , and even random color pick on screen cannot satisfy all of the white screen scenarios.


## 2. 	Scroll With WebView area and Native extension area

For current mainstream App, simple WebView can no longer satisfy complex presentation and logic. How to handle multiple View in WebView and extended area when scrolling, extend flexibly, and support pull down refresh and so on, different news App also have different technical solutions.

### 1. Use TableView

<center><img width="50%" height="50%" src="https://raw.githubusercontent.com/dequan1331/dequan1331.github.io/master/assets/img/tableView.png"></center>
	
-	Principle:

	Because of many modules are list type in the extended area (such as related articles, hot comments, etc.), the simplest implementation is that the module of the Native extension area is broken down to the granularity of the cell, and the whole page is implemented by TableView. For the connection between the extended area and the WebView, there are two solutions as shown in the above figure: TableView is inserted into WebView based on WebView's Inset (or Div occupancy) & WebView is the Header of TableView.

-	Advantages:

	This solution is relatively simple and easy to realize the layout of each module. At the same time, based on the TableView, it can easily dynamically handle the updates, inserts and deletions of each module, and support loading more. The combination of WebView is also fluent. 

-	Disadvantages:

	This solution distinguishes the module of the Native extension area to the level of the cell, which can only be managed by cell or section mode, and cannot reuse UI and business logic of whole module. The layout of UI depends on tableView mode and its flexibility is poor. With the increase of component types, different types of view does not make full use of the reuse of tableView cell.
	
	At the same time, no matter which solution, it will affect the independent rendering of WebView or TableView, and increase the difficulty of maintenance. Moreover, Header and Inset are difficult to implement for head area extension, such as drop-down refresh, etc.

### 2. ScrollView Nesting

<center><img width="70%" height="70%" src="https://raw.githubusercontent.com/dequan1331/dequan1331.github.io/master/assets/img/Scroll.png"></center>

-	Principle:

	This solution use a ScrollView as Container, and the components such as WebView and extension area modules are SubView respectively. All SubView is not allowed to scroll, and all scrolling of content page occurs on Container.  For the scroll view in SubView, if the ContentSize is less than the screen height, it will be used as a common View, otherwise it will be set to the screen height, and through the calculation of offset and Frame, it will dynamically adjust the view relative to Container's Frame and its own ContentOffset to achieve the scroll effect.

-	Advantages:
	
	This solution is completely independent of the implementation of each module, making UI and business logic one-to-one correspondence. The rendering of WebView is independent. The modules are loaded and layout flexibly, easily managed and reused, and the business logic of the module is high cohesion and low coupling. It is easy to add and delete modules. The possibility of flexibility and reuse are hence greatly improved.

-	Disadvantages:

	Because this solution needs to calculate the scroll offset, change module frame and offset, and all modules`frame should be refreshed manually when the module is dynamically updated, the complexity of the implementation is greatly improved.
	
	In [ReusableNestingScrollview](https://github.com/dequan1331/ReusableNestingScrollview)，在[HybridPageKit](https://github.com/dequan1331/HybridPageKit),the above scrollView nesting logic is encapsulated. It hides complex implementation logic and boundary conditions, and fully retains the characteristics of flexibility. At the same time, extended pull load more and drop-down refresh logic, so that the framework is simple and flexible to expand.


## 3. 	Display of complex UI and complex interactive modules in WebView

As the WebView content areas gradually support complex presentation, simple H5 base rendering cannot support the existing requirements, such as video interaction, music continuation, and maps, voting and other components. At the same time, complex UI and logic in Web also greatly reduce the rendering speed of WebView, and increase the cost of development and maintenance.

### 1. Difficulties in Complex UI and Interactive

-	In order to the better interactive experience, the content of rich media in the information content is increasing, such as video continuous play, small window play, music continuous play, map, voting and so on. At the same time, with the App update, such as the simple module of picture, it also increases the interaction of click to full screen, long press to save, QR code detect, double click to expansion, etc.. These complex UI and logic results in increased CSS and JS, increased communication between Native and Web, and a large amount of use of LocalStorage and other HTTP cache to increase the cost of development and maintenance.

### 2. Simple picture display time

-	The simplest way to pictures in WebView is to send Img tags directly from the server, depending on the download and rendering of WebView itself. However, the flexibility of this way is relatively low, the App cannot reasonably control the download time, nor make custom caching and clipping.
-	For the upgrading of simple Img tags, that is, the server data sends pictures data separately, the client chooses the download time and cache strategy according to the needs. The Html template use Div to occupy first. After the Native is successfully downloaded, replace the src value to display. Although this method solves the problem of flexibility, it also brings the complexity of the whole process and the communication delay between IPC.
-	In order to concurrently with the flexibility and shorten the Loading time of the picture, we replace all the pictures in the content WebView as Native ImageView, which can reduce the unnecessary process and communication, greatly improve the speed of the loading.

### 3. Native all components without Text

为了减少实现复杂UI、复杂交互模块的开发、维护成本、减少模块在Web和Native间的逻辑流程，提高Web中模块的加载展示速度，在[HybridPageKit](https://github.com/dequan1331/HybridPageKit)中将Web中全部非文字类模块全部Native化。
	
<center><img width="70%" height="70%" src="https://raw.githubusercontent.com/dequan1331/dequan1331.github.io/master/assets/img/div.png"></center>

-	The page template uses the empty div occupancy
:

	Combined with the template and data from server, all the non Text class components in templates are mapped to the unified Class Div, which is combined with many unique ID data binding. For synchronous data, the Size of the component is set at the same time, and the asynchronous data is set to 0 first. The template is rendered by WebView after replacement. 

-	Get frame through JS When rendering completes:

	When webView render successfully callback, it then gets all specificy Div class frame and Id by JS.
-	Add NativeView with frame:

	在进行以上两个步骤的同时，进行下载图片数据、NativeView创建、初始化、异步数据拉取等工作。在JS回调全部位置时，根据位置及ID，粘贴Native组件。

-	调整字体大小，组件异步数据拉取：对于异步的变化，在复用逻辑之后，下文将结合一并说明。


	 
## 4. 内容页全部组件的滚动复用

在Native化全部非文字类组件之后，面对文章中图片、富媒体数量的增多，以及Native扩展区元素的增加，没有复用回收的内容页从滚动性能及内存两个两个方面都面临着挑战。同时，为了更好的提升用户体验，需要对各个组件滚动时的位置进行计算，从而区分不同的区域进行诸如预处理、延迟释放等逻辑。

### 1. Mainstream scrolling reuse framework

-	Inherit special ScrollView:

	Like [LazyScrollView](https://github.com/alibaba/LazyScrollView),for implementing the subViews reuse when scrolling, it is necessary to inherit the special ScrollView, which is not feasible for WKWebView.

-	Inherit special Model:

	由于滚动复用需要保存View对应的数据信息，大部分开源框架需要继承特殊数据Model，生成对应必要的参数或方法，对于支持多种类型组件的通用框架来说，继承的实现方式不易于扩展和维护。
	
-	View滚动状态简单:

	滚动时位置的计算，最简单的方式就是根据屏幕的高度计算是否进入屏幕，对于预加载的需求，绝大部分开源框架也是只是在屏幕区域的上下增加了Buffer，仍然不能区分具体的状态，如进入buffer、进入屏幕等，无法满足复杂的业务逻辑。

### 2. WebView中组件的滚动复用

<center><img width="60%" height="60%" src="https://raw.githubusercontent.com/dequan1331/dequan1331.github.io/master/assets/img/scrollData.png"></center>

-	无需继承:

	在[ReusableNestingScrollview](https://github.com/dequan1331/ReusableNestingScrollview)中，为了兼容WebView、ScrollView等一切滚动视图中子View的复用回收，我们通过scrollView delegate的扩展分发，扩展handler单独处理子View的复用回收，这样就在无需继承的前提下，支持所有滚动视图中子View的复用回收。

-	数据驱动:
	
	由于View需要不断的复用回收，所以数据、状态、位置、对应的View类型都存储在对应的Model中，不但实现了数据驱动易于动态扩展，同时优化了复用的逻辑，也缓存住了Frame等关键信息优化了渲染布局逻辑。
	
-  	面向协议:

	由于滚动复用的模块对应的View及数据Model种类众多，在不动态扩展NSObject、UIView的情况下，无法做到通用的逻辑公用。所以为了更好的支持扩展、更灵活的实现方式，[ReusableNestingScrollview](https://github.com/dequan1331/ReusableNestingScrollview)中面向通过扩展数据Protocol，使得任何Model轻松实现复用回收对应逻辑。
	
-  	更加丰富的状态:

	在[ReusableNestingScrollview](https://github.com/dequan1331/ReusableNestingScrollview)中，为了满足更复杂的需求，如视频预加载及自动播放、Gif预加载及自动播放等，我们扩展了组件在滚动过程中的状态，增加自定义workRange，使组件在滚动过程中的状态变为3种，即None、prepare区域及Visible区域，更加全面准确的记录状态切换，更加灵活的支持业务场景。同时通过3种状态扩展为二级缓存，对View在不同级别的缓存设置不同的策略。
	
综上，通过[ReusableNestingScrollview](https://github.com/dequan1331/ReusableNestingScrollview)只需将模块对应Model扩展增加协议，滚动视图扩展Delegate，就可实现任何滚动视图中子View的回收复用功能。

### 3. 内容页中全部组件的滚动复用

在解决了内容WebView中非文字类组件的Native化、滚动复用之后，我们将实现思想运用到包含Native扩展区的，内容页整体架构中。如果从内容页的维度去看，内容WebView也可以算作一个组件，它和扩展区的各种组件一起作为Container的子View，也可以运用上面提到的[ReusableNestingScrollview](https://github.com/dequan1331/ReusableNestingScrollview)进行实现和管理。
	
<center><img width="30%" height="30%" src="https://raw.githubusercontent.com/dequan1331/dequan1331.github.io/master/assets/img/Rns2.png"></center>

所以整个内容页就是从两个维度、运用[ReusableNestingScrollview](https://github.com/dequan1331/ReusableNestingScrollview)中的实现方法两次实现滚动复用回收、数据驱动、组件自管理以及组件状态切换逻辑。
	
## 5.	组件异步拉取与动态调整

面对复杂的需求、以及按需加载、异步拉取等优化体验的策略，在[HybridPageKit](https://github.com/dequan1331/HybridPageKit)中也针对响应的场景做了高效的处理。

### 1. WebView字体大小调整

当WebView中字体大小调整时，需要同时调整全部Native组件的位置。我们监听WebView的ContenSize变化，当变化发生时，重新执行获取组件位置的JS语句获得全部组件的新位置。基于滚动复用的逻辑，只需要对在屏幕中的组件View的位置进行调整，其余只需要重新对组件对应Model的Frame进行赋值，极大提升了效率。在此基础上，要动态的检测ContenSize是否小于屏幕高度，高度小于一屏幕时，要同时调整Native扩展区组件的位置。

### 2. WebView中组件异步拉取数据渲染

对于异步拉取数据的组件，由于初始化时占位Div的高度为0，当数据获取成功，并渲染好组件后，需要首先执行JS动态修改对应占位Div的大小，之后按照以上的逻辑，重新赋值Native组件位置。

### 3. Native扩展区组件异步拉取数据渲染

Native扩展区中的组件不同于WebView中的组件，不依赖WebView自身渲染。所以当动态调整大小时，之需调整全部Native扩展区组件数据Model中保存的Frame信息，同时调整在屏幕中的组件位置即可。

<br>

> ***
>_插播广告 —— 几十行代码完成新闻类App多种形式内容页_ 
>
>_[HybridPageKit](https://github.com/dequan1331/HybridPageKit) ：一个针对新闻类App高性能、易扩展、组件化的通用内容页实现框架。_
>
>_基于[ReusableNestingScrollview](https://github.com/dequan1331/ReusableNestingScrollview)、[WKWebViewExtension](https://github.com/dequan1331/WKWebViewExtension)、以及本文中关于内容页架构和性能的探索。_
>
>***

<br>

## <center>- 内容页组件化架构 -</center>
***

在实现了以上技术关键点的基础上，如何合理的设计内容页通用的架构，快速响应内容页的各种需求调整，使整体架构易扩展、易维护，同时有较高的性能及较小的内存占用，成为了整个内容页架构实现的重点。在[HybridPageKit](https://github.com/dequan1331/HybridPageKit)中，我们围绕灵活复用、高内聚低耦合、易于实现扩展三个重点的方向，设计实现了基于组件化的内容页整体架构。

## 1.	组件化解耦及组件通信

为了满足内容页业务的相对独立，支持快速响应迭代及组件整体复用，内容页整体的结构应满足通用性、易于扩展、以及高内聚低耦合的特点。所以在[ReusableNestingScrollview](https://github.com/dequan1331/ReusableNestingScrollview)的支持下，采用组件化的方式实现全部内容页业务模块。

### 1. 组件化解耦

为了达到组件的高内聚、与内容页的低耦合，在[HybridPageKit](https://github.com/dequan1331/HybridPageKit)中拆分业务逻辑为独立的组件化的处理单元，每个处理单元通过MVC模式实现。其中Model作为组件的数据，只需要在实现解析逻辑同时，实现对应delegate即可。Controller只需要实现组件间通信的delegate，选择性的实现例如controller生命周期、webview关键回调、以及滚动复用相关的方法即可。通过组件的自管理及复用，组件可以集成统一的上报逻辑、业务逻辑到自己的Controller中，并且在不同类型的页面灵活复用。

### 2. 组件通信

为了更好的实现组件化的结构，组件的Controller需要在内容页初始化时进行注册。内容页在每个关键的生命周期或业务节点，采用中心化通信，广播执行响应的方法，组件的Controller按需实现处理即可。对于新增、删除功能，只需扩展delegate中的方法，内容页中触发方法、组件中实现方法即可。

<center><img width="60%" height="60%" src="https://raw.githubusercontent.com/dequan1331/dequan1331.github.io/master/assets/img/componentComm.png"></center>


## 2.	组件及WebView的复用管理

### 1. WebView & 组件View全局复用

为了提高WKWebView渲染速度，通过建立全局WKWebView复用回收池来复用WKWebView。除了基本的线程安全、复用状态管理等，在进入回收池前要load特殊Url以维护整个backFowardList。组件的View也是通过全局的复用回收池进行管理，使得相同的组件View可以灵活的出现在内容页、列表页等App内各个页面，极大的减少了开发成本，提高运行效率。

### 2. 自动回收 & 内存管理

WebView及组件View实现自动回收逻辑，每次在申请新View时检测活动队列中View的SuperView是否为nil，是则自动回收防止内存泄露，同时增加View最大数量阈值、内存告警自动释放逻辑等。

## 3.	内容页整体架构

### 1. 易于扩展业务节点 & 组件类型

对于增加关键的业务节点用于组件业务处理，我们只需扩展delegate中的方法，在相关组件中实现。内容页Controller中在相应位置，通过统一函数触发广播代理方法即可。对于增加组件来说，只需创建组件完全独立的MVC代码，实现数据解析Model并实现滚动复用delegate，在组件Controller中实现delegate中需要的方法等待调用，以及初始化时在内容页注册即可。删除组件完全无需操作内容页，删除独立的MVC结构并停止注册即可。

### 2. 易于扩展内容页类型

为了实现内容页扩展区的灵活复用，在[HybridPageKit](https://github.com/dequan1331/HybridPageKit)中也扩展了非WebView类型的内容页。就像文中之前提到的，如果将WebView看做一个整体作为一个组件，基于[ReusableNestingScrollview](https://github.com/dequan1331/ReusableNestingScrollview)的位置动态管理，完全可以替换成普通的View（类似Banner视频内容页），或者可扩展收起的View（问题回答页面）甚至tableView等。所以整个App内各种类型的内容页只需要简单的配置，便可进行实现和组件复用。

<center><img width="80%" height="80%" src="https://raw.githubusercontent.com/dequan1331/dequan1331.github.io/master/assets/img/pageType.png"></center>

### 3. 内容页架构

结合[ReusableNestingScrollview](https://github.com/dequan1331/ReusableNestingScrollview)、[WKWebViewExtension](https://github.com/dequan1331/WKWebViewExtension)以及组件化的设计思路，[HybridPageKit](https://github.com/dequan1331/HybridPageKit)整体的架构如下：

<center><img width="70%" height="70%" src="https://raw.githubusercontent.com/dequan1331/dequan1331.github.io/master/assets/img/hybrid.png"></center>

通过继承特殊的内容页Controller并进行简单的配置，即可生成不同类型的内容页整体架构。框架内集成基本的Mustache解析和渲染。结合后台数据，只需实现对应页面中组件MVC逻辑即可。其中Model只需继承对应Protocol，Controller在内容页中注册，继承对应Protocol即可。

<br>

> ***
>_插播广告 —— 几十行代码完成新闻类App多种形式内容页_ 
>
>_[HybridPageKit](https://github.com/dequan1331/HybridPageKit) ：一个针对新闻类App高性能、易扩展、组件化的通用内容页实现框架。_
>
>_基于[ReusableNestingScrollview](https://github.com/dequan1331/ReusableNestingScrollview)、[WKWebViewExtension](https://github.com/dequan1331/WKWebViewExtension)、以及本文中关于内容页架构和性能的探索。_
>
>***

<br>


## <center>- 首屏加载速度优化 -</center>
***

新闻类App内容页，在Native的页面框架下，基于WebView进行加载和渲染。所以，从优化的角度就延伸出两个维度，即从Web的维度优化，以及从Native的维度优化。

### 1. Web维度的优化

-	WKWebView的复用 : 

	通过WKWebView的复用，极大的缩短了WebView从创建到渲染结束的时间。
	
- 	利用HTTP缓存 : 

	对于内容WebView中必要的CSS以及JS，以及必要的基础Icon，可以通过设置HTTP缓存，依靠浏览器自身缓存提高效率。同时通过资源md5校验以保证刷新资源。
	
-  减少资源请求并发 : 

	通过Native化全部非文字类的内容，Web页面只加载最近本的Html内容，减少了业务逻辑的资源请求和并发。
	
- 	减少Dom & Javascript复杂度 : 

	通过Native化全部非文字类的内容，极大的减少了Dom的复杂度、CSS的复杂度以及过多的JS业务逻辑。
	
-  其它Web优化通用方法 : 

	精简Javascript，使用iconFont，CSS & Javascript文件压缩等

### 2. Native维度的优化

-	数据模板分离，资源并行加载 :

	基于后台数据以及Native化组件，内容页Html中模板与数据分离，使得全部资源如图片视频等都可以通过Native在合适的时机异步并行加载。不依赖与Web的渲染。

-  预加载数据,延迟加载组件:

	对于内容页关键内容（Webview）的拉取，大部分App都放到了列表页中进行。进入内容页时直接从Cache中取出内容模板，直接交给WebView渲染。基于[ReusableNestingScrollview](https://github.com/dequan1331/ReusableNestingScrollview)扩展丰富的状态及二级缓存，在页面滚动的过程中各个组件也可以精确的实现按需加载、预加载等逻辑。

-	Native化非文字UI，及组件化实现负载均衡 :

	WebView中非文字类UI Native化，极大的缩短了展示所需的流程，减少了进程间通信，减少了I/O及图片编解码逻辑，提高了类似图片类的UI展示速度。
	
	组件的解耦与自管理，以及广播delegate的实现，为组件的按需加载、按优先级加载提供了基础。对于内容页的各个组件来说，在内容页展示之前大部分是不需要初始化、数据拉取以及渲染的。组件化之后的组件可以根据业务优先级，在不同的关键生命周期回调中实现业务逻辑，以减轻内容页创建、模板拼接以及WebView渲染的压力。简单的举例，由于内容WebView几乎都大于一屏，扩展区中的全部组件都可以在WebView渲染结束后进行View创建、网络拉取和渲染等，这样即不影响用户的使用，同时极大的释放了渲染结束前的网络、XPC及CPU压力，提高首屏展示速度。
	
	<center><img width="80%" height="80%" src="https://raw.githubusercontent.com/dequan1331/dequan1331.github.io/master/assets/img/need.png"></center>
<br>
- 	Reuse when scrolling & Reuse between pages & Model cache frame:

	The extension of data Model base on [ReusableNestingScrollview](https://github.com/dequan1331/ReusableNestingScrollview), caching View frame information, combined with view's reuse when scrolling, can greatly reduce the logic and computation of UI layout. The reuse of components when scrolling and the reuse of components between pages can also reduce the initialization time of component View.

-	Other general optimization:

	The technology implementation and business logic optimization based on App, such as asynchronous execution I/O, image encoding and decoding optimization and resource cache, DNS cache and so on. 

### 3. Overall optimization method

To sum up, from the click of a cell on the list, to the end of the WebView rendering, and finally to the user's scroll operation, the whole optimization strategy is as follows based on the order of time:

<center><img width="90%" height="90%" src="https://raw.githubusercontent.com/dequan1331/dequan1331.github.io/master/assets/img/opt.png"></center>


<br>

> ***
>_插播广告 —— 几十行代码完成新闻类App多种形式内容页_ 
>
>_[HybridPageKit](https://github.com/dequan1331/HybridPageKit) ：一个针对新闻类App高性能、易扩展、组件化的通用内容页实现框架。_
>
>_基于[ReusableNestingScrollview](https://github.com/dequan1331/ReusableNestingScrollview)、[WKWebViewExtension](https://github.com/dequan1331/WKWebViewExtension)、以及本文中关于内容页架构和性能的探索。_
>
>***

<br>

## <center>- Tips -</center>
***

对于新闻类App内容页的完整的解决方案，还有一些基本的技术点，比如模板引擎及模板拼接的模块、JSApi注入及管理的模块等等，由于篇幅所限，暂且不做深入的展开。

-	新闻类App的内容页，除去基本的渲染HTML数据外，同时也需要支持服务于活动、运营的临时H5页面。这些页面为了和Native进行交互，在自定义JSApi注入、JSBridge的选择、后台下发domain黑白名单、以及相关的安全性考虑也是整个实现中重要的一环。同时由于WKWebView支持复用回收，加载本地Html类型的WebView应该与加载H5的WebView在不同的回收复用池分开管理。

-	对于底层页图片的管理，绝大多数App都将之纳入了App统一的图片管理体系中。无论使用哪个开源图片库，在缓存策略上，尽量将底层页图片的缓存策略与其他的有所区分，或者使用`LRU + FIFO`的缓存策略，避免进入底层页大量图片占用缓存空间，导致列表图片释放。同时从使用的角度来说，重复进入同一篇文章的场景也不会频繁的出现。

-	由于各个App的数据接口和技术选型不同，在[HybridPageKit](https://github.com/dequan1331/HybridPageKit)中只简单的实现了基于Mustache的模板拼接，主要是由于它的logic-less、多终端集成的方便以及开源社区的活跃。对于这部分逻辑，需要根据后台数据的格式及业务需求自定义的扩展。

The overall optimization of content pages depends on the technical implementation and structure of the entire App. In the process of implementation and optimization, there are many trade-offs and compromises, as well as many general and detailed optimization, which are not detailed here. 

<br>

## <center>- Ending -</center>
***

The implementation of all the analysis of the paper is achieved into three frameworks except business logic:[HybridPageKit](https://github.com/dequan1331/HybridPageKit)、[ReusableNestingScrollview](https://github.com/dequan1331/ReusableNestingScrollview)以及[WKWebViewExtension](https://github.com/dequan1331/WKWebViewExtension) Finally, dozens of lines of code can be used to complete various types、 high-performance 、high-extensibility、easy integration hybrid content page of News App.  

有任何疑问，欢迎提交 issue， 或者直接修改提交 PR!

<br>

> ***
>_插播广告 —— 几十行代码完成新闻类App多种形式内容页_ 
>
>_[HybridPageKit](https://github.com/dequan1331/HybridPageKit) ：一个针对新闻类App高性能、易扩展、组件化的通用内容页实现框架。_
>
>_基于[ReusableNestingScrollview](https://github.com/dequan1331/ReusableNestingScrollview)、[WKWebViewExtension](https://github.com/dequan1331/WKWebViewExtension)、以及本文中关于内容页架构和性能的探索。_
>
>***