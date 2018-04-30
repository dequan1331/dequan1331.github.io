---
layout: HybridPageKit-EN
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
>_View on GitHub : **Easy integration framework for Content pages of News App**_ 
>
>_[HybridPageKit](https://github.com/dequan1331/HybridPageKit) ：A high-performance、high-extensibility、easy integration framework for Hybrid content page. Support most content page types of News App._
>
>_Base on [ReusableNestingScrollview](https://github.com/dequan1331/ReusableNestingScrollview)、[WKWebViewExtension](https://github.com/dequan1331/WKWebViewExtension)、and the details metioned in this article。_
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

<center><img width="80%" height="80%" src="https://raw.githubusercontent.com/dequan1331/dequan1331.github.io/master/assets/img/index-en.png"></center>

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

To reduce the cost of development and maintenance of complex UI and complex interaction module, reduce the logic between Web and Native module communication, improve the module display speed in Web. [HybridPageKit](https://github.com/dequan1331/HybridPageKit) change all non-Text components to Native.
	
<center><img width="70%" height="70%" src="https://raw.githubusercontent.com/dequan1331/dequan1331.github.io/master/assets/img/div.png"></center>

-	The page template uses the empty div occupancy:

	Combined with the template and data from server, all the non Text class components in templates are mapped to the unified Class Div, which is combined with many unique ID data binding. For synchronous data, the Size of the component is set at the same time, and the asynchronous data is set to 0 first. The template is rendered by WebView after replacement. 

-	Get frame through JS When rendering completes:

	When webView render successfully callback, it then gets all specificy Div class frame and Id by JS.
-	Add NativeView with frame:

	At the same time, it will async download image data, create adn init Native view, asynchronous data fetching and so on. When JS returns all the frames, add the Native component to scrollview by frame and ID.

-	Change the font size & asynchronous request for component data:It will be explained in followings.


	 
## 4. Reuse of all components when scrolling

After change all non-Text components to Native, the number of pictures, rich media and the components in Native extension area are increased, the content page without components reuse is facing challenges from two aspects: scroll performance and App memory. At the same time, in order to improve the user experience, it is necessary to calculate the position of each component when scrolling, so as to distinguish different regions, such as prepare, delay release and other logics. 

### 1. Mainstream scrolling reuse framework

-	Inherit special ScrollView:

	Like [LazyScrollView](https://github.com/alibaba/LazyScrollView),for implementing the subViews reuse when scrolling, it is necessary to inherit the special ScrollView, which is not feasible for WKWebView.

-	Inherit special Model:

	
	Because the scrolling reuse needs to save the data information of View, most open source frameworks need to inherit special data Model to generate the necessary parameters or methods. For a high-extensibility framework supporting many types of components, the implementation method of inheritance is not easy to extend and maintain.
	
-	Less scrolling state:

	The simplest way to calculate the position of the scrolling time is to calculate whether or not the component is on the screen. For the preloading requirements, most open source frameworks also add buffer to the screen area, and still cannot distinguish the specific state, such as entering the buffer, entering the screen, leaving the screen or buffer and so on, which cannot satisfy the complexity business logic.

### 2. Reuse of components in WebView when scrolling

<center><img width="60%" height="60%" src="https://raw.githubusercontent.com/dequan1331/dequan1331.github.io/master/assets/img/scrollData.png"></center>

-	No need to inherit:

	In order to support subviews reuse logic on all types of scrollview, such as WebView, ScrollView, and so on, [ReusableNestingScrollview](https://github.com/dequan1331/ReusableNestingScrollview) extend the scrollView delegate to a dispatcher and extend a handler separately to handle the subViews reuse, so that the reuse and recovery of all the subviews when scrolling can be supported without inheritance.

-	Data driven:
	
	Because View needs to reuse and recycle, the data、state、frame and corresponding View types are stored in the model, which not only easy to expand, but also optimizes the logic of reuse, and also caches the key information such as frame to optimize the rendering layout logic.
	
-  	Protocol Oriented Programming:

	As the types of view and Model to scrolling reuse are numerous, the general logic cannot be achieved without dynamic expansion of NSObject and UIView. So in order to better support extensions and more flexible implementations, [ReusableNestingScrollview](https://github.com/dequan1331/ReusableNestingScrollview) use POP, easy for any Model to reuse and extend by add protocol.
		
-  	More scrolling state:

	In order to support more complex needs, such as video preloading & auto play, Gif preloading & auto play, [ReusableNestingScrollview](https://github.com/dequan1331/ReusableNestingScrollview) extend the state of the component during the scrolling process, increase the custom workRange, and make the state of the component in the scrolling process to 3 regions: None, prepare region and Visible region, for more comprehensive and accurate compute state switching, more flexible support for business scenarios. At the same time, it expands to two level caching through 3 states, and sets up different strategies for View at different levels of caching.
	
In summary, in [ReusableNestingScrollview](https://github.com/dequan1331/ReusableNestingScrollview), only need to add protocol to model, and dispatch the scrollview delegate, so that it can realize the recovery and reuse function of any scroll view View.

### 3. Reuse all components in content page when scrolling

After changing all non-Text component native, and solving the reuse problem in  WebView, we apply the implementation to the overall architecture of the content page, which includes the Native extension area. If looking at the dimension of the content page, the content WebView can be used as a component. It is a sub View of Container with the various components of the extended area, and can also be implemented and managed with the above mentioned [ReusableNestingScrollview](https://github.com/dequan1331/ReusableNestingScrollview).
	
<center><img width="30%" height="30%" src="https://raw.githubusercontent.com/dequan1331/dequan1331.github.io/master/assets/img/Rns2.png"></center>

So the whole content page is the two realization by [ReusableNestingScrollview](https://github.com/dequan1331/ReusableNestingScrollview) of scrolling reuse, data drive, component self management and component state switching from two dimensions. 
	
## 5.	Asynchronous fetching and dynamic adjustment of components

Faced with complex needs, on-demand loading, asynchronous fetch and other optimization strategy, [HybridPageKit](https://github.com/dequan1331/HybridPageKit) also optimize for special scene. 

### 1. Font size change in WebView

When changing the font size in WebView, all Native components need to be adjusted at the same time. We observer the contenSize change of WebView, and when the change occurs, it will re execute a JS that gets the new frames of all components. Based on scrolling reuse,it requires only the adjustment of the in screen components frame, and the rest only needs to be assigned to the Frame of the component corresponding to the Model, which greatly improves the efficiency. On this basis, it is necessary to dynamically detect whether the webview contenSize is smaller than the screen height, and when the height is less than one screen, the frame of webview and Native extension component should be adjusted at the same time.

### 2. Async fetch component data in WebView

For the component that fetch data asynchronously, because the height of the occupying Div is 0 at initialization, when the data is obtained and the component view is rendered, the JS dynamic modification needs to be first executed for the size of the occupying Div, and then the Native component frame shall be reassigned according to the above logic.

### 3. Async fetch component data in native extension area

The components in Native extension area are different from the components in WebView and do not rely on WebView rendering. So when the dynamic adjustment is occurs, it is just to change the frame stored in the Model and change the frame of the component in the screen.

<br>

> ***
>_View on GitHub : **Easy integration framework for Content pages of News App**_ 
>
>_[HybridPageKit](https://github.com/dequan1331/HybridPageKit) ：A high-performance、high-extensibility、easy integration framework for Hybrid content page. Support most content page types of News App._
>
>_Base on [ReusableNestingScrollview](https://github.com/dequan1331/ReusableNestingScrollview)、[WKWebViewExtension](https://github.com/dequan1331/WKWebViewExtension)、and the details metioned in this article。_
>
>***

<br>

## <center>- Architecture of Content Page -</center>
***

On the basis of the key points above, how to design a general architecture、quickly respond to various requirements、 easy to expand、 easy to maintain, and have high performance and small memory becomes the key to the realization of the whole content page architecture. [HybridPageKit](https://github.com/dequan1331/HybridPageKit) focus on the three key directions of flexible reuse、 high cohesion、 low coupling and easy implementation, and design and implement a component-based content page architecture.

## 1.	Component decoupling and communication

In order to meet the relative independence of the content page, support fast response iteration and component reuse, the overall structure of the content page should meet the characteristics of generality, easy extension, and high cohesion and low coupling, base on [ReusableNestingScrollview](https://github.com/dequan1331/ReusableNestingScrollview), use component-based solution to implement all content page business modules.

### 1. Component-based decoupling

In order to achieve the high cohesion of the component and low coupling with the content page, [HybridPageKit](https://github.com/dequan1331/HybridPageKit) split business logic is an independent component processing unit, and each processing unit is implemented through the MVC mode. Model as component data only needs to implement parsing logic and implement delegate. Controller only needs to implement delegate for inter component communications, and selective implementations such as the controller lifecycle, the WebView key callback, and the scrolling reuse related methods. Through self - management and reuse of components, components can integrate unified reporting logic, business logic into their own Controller, and be reused flexibly on different types of pages. 

### 2. Component communication

To better implement the component-based structure, the Controller of the component needs to be registered when the content page initializes. In each key life cycle or business event, the content page adopts the centralization communication, broadcasting the method, and the delegate methods is implemented on demand in component Controller.  For new or deleted functions, we only need to expand the methods in delegate, trigger methods in content pages, and implement methods in components.

<center><img width="60%" height="60%" src="https://raw.githubusercontent.com/dequan1331/dequan1331.github.io/master/assets/img/componentComm.png"></center>


## 2.	Reuse management of components and WebView

### 1. Reuse of components and WebView

In order to improve the rendering speed of WKWebView, a global WKWebView reuse recovery pool is used to reuse WKWebView. In addition to basic thread safety and reuse state management, loading of special Url is needed before entering the recovery pool to maintain the webview backFowardList. The View of the component is also managed through a global reuse pool, so that the same component View can be flexibly appeared in the pages of the content page, list page and other App pages, which can greatly reduce the cost of development and improve the efficiency of operation.

### 2. Automatic recovery & memory management

WebView and component View implement automatic recovery logic. Each time dequeue for a new View, detect whether all components superView is nil. It is automatically recovered to prevent memory leak, and support the maximum number of View thresholds and memory warning automatic release logic.

## 3.	Architecture of Content page

### 1. High-extensibility components

To increase the number of key business event for component, we only need to extend the methods in delegate and implement them in related components.At the content page Controller, the broadcast agent method can be triggered by a unified function. For adding components, it is only necessary to create a fully independent MVC code for components, so as to implement data parsing Model and implement scrolling reuse delegate, to implement the methods needed in delegate in component Controller, and to register in the content page when initialization. Deleting components completely does not need to concern content pages, only to delete independent MVC structures and stop registering.

### 2. High-extensibility Content Page

In order to realize the flexible reuse of native extension area, in [HybridPageKit](https://github.com/dequan1331/HybridPageKit), the non WebView type content page is expanded. As mentioned earlier in the article, if WebView is regarded as a component, based on [ReusableNestingScrollview](https://github.com/dequan1331/ReusableNestingScrollview), it can be completely replaced by a common View (similar to a Banner video content page), or an extensible View (question answer page) or even tableView. Therefore, all types of content pages in App can be implemented and reused components only by simple configuration.

<center><img width="80%" height="80%" src="https://raw.githubusercontent.com/dequan1331/dequan1331.github.io/master/assets/img/pageType.png"></center>

### 3. Content page architecture

Base on [ReusableNestingScrollview](https://github.com/dequan1331/ReusableNestingScrollview)、[WKWebViewExtension](https://github.com/dequan1331/WKWebViewExtension) and component-based，the architecture of [HybridPageKit](https://github.com/dequan1331/HybridPageKit) is below：

<center><img width="70%" height="70%" src="https://raw.githubusercontent.com/dequan1331/dequan1331.github.io/master/assets/img/hybrid.png"></center>

By inheriting the special content page Controller and simply configuring it, we can generate different types of content page. Integrates basic Mustache parsing and rendering、combined with background data, it only needs to implement component MVC logic in corresponding pages. Model only implement the protocol, Controller registers in the content page, and implement the protocol.

<br>

> ***
>_View on GitHub : **Easy integration framework for Content pages of News App**_ 
>
>_[HybridPageKit](https://github.com/dequan1331/HybridPageKit) ：A high-performance、high-extensibility、easy integration framework for Hybrid content page. Support most content page types of News App._
>
>_Base on [ReusableNestingScrollview](https://github.com/dequan1331/ReusableNestingScrollview)、[WKWebViewExtension](https://github.com/dequan1331/WKWebViewExtension)、and the details metioned in this article。_
>
>***

<br>


## <center>- Optimization of Loading speed -</center>
***

The news App content page is loaded and rendered based on WKWebView under the framework of Native page. Therefore, from the perspective of optimization, we extend two dimensions, that is, the optimization of Web dimension and the optimization of Native dimension.

### 1. Web dimension

-	Reuse of WKWebView : 

	The reuse of WKWebView can greatly shorten the time from WebView creation to the end of rendering.
	
- 	Using HTTP caching : 

	For the necessary CSS and JS in WebView, and the necessary foundation Icon, we can increase the efficiency by setting HTTP cache and relying on browser's own cache. At the same time, the resource MD5 is checked to ensure refreshing resources.
	
-  Reduction of resource request concurrency : 

	By converting all non-Text components into Native, Web pages load only the latest Html string, thus it can reduce the resource requests and concurrency of business logic.
	
- 	Decrease Dom & Javascript complexity : 

	By converting all non-Text components into Native, it greatly reduce the complexity of Dom, the complexity of CSS, and the excessive JS business logic.
	
-  Other general methods for Web optimization : 

	Streamline Javascript, use of iconFont, CSS & Javascript file compression, etc.. 


### 2. Native dimension

-	Data template separation & Parallel loading :

	Based on server data and Native components, the page template and component data are separated from the content page Html so that all the resources, such as picture 、video, can be loaded asynchronously at the appropriate time by Native. It is not dependent on the rendering of Web.

-  Preload data & delayed load component:

	Most of the App put the content page's key content requset on the list page. When entering the content page, extract the content template directly from Cache and give it to WebView rendering directly. [ReusableNestingScrollview](https://github.com/dequan1331/ReusableNestingScrollview) expand the rich state and the two level cache, each component can also on-demand loading, preloading and delay release.

-	Native non-Text components & Priority :

	Changing the non-Text components to Native in WebView can greatly reduce the process required for display, reduce IPC communication, reduce I/O and picture repect decode logic, and improve the UI display speed.
	
	The decoupling and self management of components, as well as the implementation of broadcast delegate, provide a basis for on-demand loading of components and priority loading components. Most of the components of the content page do not need initialization、fetching data and rendering before the content page is rendered. Component-based components can implement business logic in different key lifecycle callbacks based on business priorities to mitigate the pressure of content page creation, template splicing, and WebView rendering. Taking a simple example, as the content WebView is almost all larger than a screen, all the components in the extended area can achieve View creation, network fetching after the end of the WebView rendering, which does not affect the user's use, at the same time, it releases the pressure of the network, IPC and CPU before the end of the rendering, and improves the display speed of the webview.
	
	<center><img width="80%" height="80%" src="https://raw.githubusercontent.com/dequan1331/dequan1331.github.io/master/assets/img/need-en.png"></center>
<br>

- 	Reuse when scrolling & Reuse between pages & Model cache frame:

	The extension of data Model base on [ReusableNestingScrollview](https://github.com/dequan1331/ReusableNestingScrollview), caching View frame information, combined with view's reuse when scrolling, can greatly reduce the logic and computation of UI layout. The reuse of components when scrolling and the reuse of components between pages can also reduce the initialization time of component View.

-	Other general optimization:

	The technology implementation and business logic optimization based on App, such as asynchronous execution I/O, image encoding and decoding optimization and resource cache, DNS cache and so on. 

### 3. Overall optimization method

To sum up, from the click of a cell on the list, to the end of the WebView rendering, and finally to the user's scroll operation, the whole optimization strategy is as follows based on the order of time:

<center><img width="90%" height="90%" src="https://raw.githubusercontent.com/dequan1331/dequan1331.github.io/master/assets/img/opt-en.png"></center>


<br>

> ***
>_View on GitHub : **Easy integration framework for Content pages of News App**_ 
>
>_[HybridPageKit](https://github.com/dequan1331/HybridPageKit) ：A high-performance、high-extensibility、easy integration framework for Hybrid content page. Support most content page types of News App._
>
>_Base on [ReusableNestingScrollview](https://github.com/dequan1331/ReusableNestingScrollview)、[WKWebViewExtension](https://github.com/dequan1331/WKWebViewExtension)、and the details metioned in this article。_
>
>***

<br>

## <center>- Tips -</center>
***

For the complete solution of the news App content page, there are also some basic technical points, such as template engine and template splicing module, JSApi injection and management module and so on. Due to space constraints, there is no details for further development on this. 

-	For content page of news App, excluding the basic rendering of HTML string, it also needs temporary H5 pages to support Game、activities and promotion. In order to interact with Native, these are some important parts of the entire implementation such as the custom JSApi injection, the choice of JSBridge, the black-and-white list of domain in the server, and the related security considerations. Meanwhile, Since WKWebView supports reuse, the WebView which loading local Html string should be separately managed from the different reuse pools of WebView which loading url. 

-	For the management of the content page images, the majority of App are incorporated into the unified picture management system of App. No matter which open source image library is used, the image caching strategy of the content page should be distinguished from others, or the cache strategy of `LRU + FIFO` is used to avoid the entry of a large number of pictures in the content page to take up the cache space, resulting in the release of the pictures in list .

- 	Because of the different server interface and technology selection of each App, in [HybridPageKit](https://github.com/dequan1331/HybridPageKit) the template splicing based on Mustache is simply realized, the reason is mainly because of its logic-less, multi terminal integration convenience and open source community's activity. For this part of the logic, it is required to customize the extension according to the format of the server data and business requirements.

The overall optimization of content pages depends on the technical implementation and structure of the entire App. In the process of implementation and optimization, there are many trade-offs and compromises, as well as many general and detailed optimization, which are not detailed here. 

<br>

## <center>- Ending -</center>
***

The implementation of all the analysis of the paper is achieved into three frameworks except business logic:[HybridPageKit](https://github.com/dequan1331/HybridPageKit)、[ReusableNestingScrollview](https://github.com/dequan1331/ReusableNestingScrollview)以及[WKWebViewExtension](https://github.com/dequan1331/WKWebViewExtension) Finally, dozens of lines of code can be used to complete various types、 high-performance 、high-extensibility、easy integration hybrid content page of News App.  

<br>

> ***
>_View on GitHub : **Easy integration framework for Content pages of News App**_ 
>
>_[HybridPageKit](https://github.com/dequan1331/HybridPageKit) ：A high-performance、high-extensibility、easy integration framework for Hybrid content page. Support most content page types of News App._
>
>_Base on [ReusableNestingScrollview](https://github.com/dequan1331/ReusableNestingScrollview)、[WKWebViewExtension](https://github.com/dequan1331/WKWebViewExtension)、and the details metioned in this article。_
>
>***