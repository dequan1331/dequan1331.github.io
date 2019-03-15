---
layout: web_in_iOS
---


## 目录

## iOS中Web容器与加载

### 1. iOS中的Web容器

<center>
	<img width="15%" height="15%" src="https://raw.githubusercontent.com/dequan1331/dequan1331.github.io/master/assets/img/2/1.png">
	<img width="15%" height="15%" src="https://raw.githubusercontent.com/dequan1331/dequan1331.github.io/master/assets/img/2/2.png">
	<img width="13%" height="13%" src="https://raw.githubusercontent.com/dequan1331/dequan1331.github.io/master/assets/img/2/13.png">
</center>


目前iOS系统为开发者提供三种方式来展示Web内容：

- #### UIWebView
	
	UIWebView从iOS2 开始就作为App内展示Web内容的容器，但是长久以来一直遭受开发者的诟病；系统级的内存泄露、极高内存峰值、较差的稳定性、Touch Delay以及Javascript的运行性能及通信限制等等。在iOS12以后已经标记为Deprecated不再维护。

- #### WKWebView

	在iOS8中，Apple引入了新一代的WebKit framework，同时提供了WKWebView用来替代传统的UIWebView。它更加的稳定、拥有60fps滚动刷新率、丰富的手势、KVO、高效的Web和Native通信，默认进度条等等功能，而最重要的，是使用了和safari相同的Nitro引擎极大提升了Javascript的运行速度，以及独立的进程管理，降低了内存占用及Crash对主App的影响。
	
	
-  #### SFSafariViewController

	在iOS9中，Apple引入了SFSafariViewController。其特点就是在App内可以打开一个高度标准化的、和Safari一样展示和特性的页面。同时SFSafariViewController可以和Safari共享Cookie和表单数据等等。

-  #### Web容器选型
	
	对于SFSafariViewController：由于其标准化程度之高，使之界面和交互逻辑无法和App统一，基于App的整体体验一般都使用在相对独立的功能和模块中，最常见的就是在App内打开App Store或者广告、游戏推广的页面。

	对于UIWebView/WKWebView：如果说之前由于NSURLProtocol的问题，好多App都在继续使用UIWebView，那么随着App放弃维护UIWebView（iOS12），全部的App应该会陆续的切换到WKWebView中来。当然，WKWebView也为开发者们带来了一些难题。最初都整理在，随着系统的不断升级也修改和修复了一些问题，简单列举几个：
	


### 2. WebKit框架与使用

- #### WebKit.framework

<center>
	<img width="70%" height="70%" src="https://raw.githubusercontent.com/dequan1331/dequan1331.github.io/master/assets/img/2/4.png">
</center>

[官方文档](https://developer.apple.com/documentation/webkit)

- #### Web容器使用流程与关键节点

	对于大部分日常使用来说，创建与配置、加载、接收回调

	对于Web开发者，业务逻辑一般通过基于Web页面和Dom渲染的关键节点来处理。而对于iOS开发者，WKWebView提供的的注册、加载和回调时机，没有明确的与Web加载的关键节点相关联。准确的理解和处理两个维度的加载顺序，选择合理的业务逻辑处理时机，才可以实现准确而高效的应用。
	
	<center>
	<img width="70%" height="70%" src="https://raw.githubusercontent.com/dequan1331/dequan1331.github.io/master/assets/img/2/5.png">
	</center>
	
- #### 

	使用WKWebView带来的另外一个好处，就是我们可以通过源码理解部分加载逻辑，为Crash提供一些思路，或者使用一些私有方法。
		1. `NSURLProtocol支持`

		- WKWebView最为显著的改变，就是不支持NSURLProtocol。为了兼容旧的业务逻辑，一部分App通过[WKBrowsingContextController]()中的非公开方法实现了NSURLProtocol。

			```objc
			+ (void)registerSchemeForCustomProtocol:(NSString *)scheme WK_API_DEPRECATED_WITH_REPLACEMENT("WKURLSchemeHandler", macos(10.10, WK_MAC_TBA), ios(8.0, WK_IOS_TBA));
			```
	
		- 在iOS11中，系统增加了 `setURLSchemeHandler`函数用来拦截自定义的Scheme。但是不同于UIWebView，新的函数只能拦截自定义的Scheme[(SchemeRegistry.cpp)](https://github.com/WebKit/webkit/blob/master/Source/WebCore/platform/SchemeRegistry.cpp)，对使用最多的HTTP/HTTPS依然不能有效的拦截。
		
			```objc
		    static const StringVectorFunction functions[] {
		        builtinSecureSchemes,                // about;data...
		        builtinSchemesWithUniqueOrigins,     // javascript...
		        builtinEmptyDocumentSchemes,
		        builtinCanDisplayOnlyIfCanRequestSchemes,
		        builtinCORSEnabledSchemes,           //http;https
		    };
		```
	
	
	2. 白屏的原因主要分两种，一种是由于Web的进程Crash（多见于内部进程通信）；一种就是WebView渲染时的错误（Debug一切正常只是没有对应的内容）。对于白屏的检测，前者在iOS9之后系统提供了对应Crash的回调函数，同时业界也有通过判断URL/Title是否为空的方式作为辅助；后者通过对比业界也有判断SubView是否包含WKCompsitingView，以及通过随机点截图等方式作为白屏判断的依据。
	
	
	3. 其他WKWebView的系统级问题如Cookie、POST参数、异步Javascript等等一系列的问题，可以通过业务逻辑的调整重新适配
	
	4. 由于WebKit源码等开放性，我们也可以利用私有方法来简化代码逻辑、实现复杂的产品需求。例如在[WKWebViewPrivate](https://github.com/WebKit/webkit/blob/master/Source/WebKit/UIProcess/API/Cocoa/WKWebViewPrivate)中可以获得各种页面信息、直接取到UserAgent、 在[WKBackForwardListPrivate]()中可以清理掉全部的跳转历史、以及在[WKContentViewInteraction]()中替换方法实现自定义的MenuItem等等。

		```objc
		@interface WKWebView (WKPrivate)
		@property (nonatomic, readonly) NSString *_userAgent WK_API_AVAILABLE(macosx(10.11), ios(9.0));
		...
		
		@interface WKBackForwardList (WKPrivate)
		- (void)_removeAllItems;
		...
		
		@interface WKContentView (WKInteraction)
		- (BOOL)canPerformActionForWebView:(SEL)action withSender:(id)sender;
		```



### 3. App中的应用场景

由于WebView系统级提供两种类型的加载方式，加载URL & 加载HTML\Data，所以基于此延伸出两种不同的业务场景：加载URL的页面直出类和加载数据的模板渲染类，同时各自也有不同的优化重点及方向。

- 页面直出

通常各类App中的Web页面加载都是通过加载URL的方式，比如嵌入的运营活动页面、广告页面等等。

- 模板渲染

需要WebView加载，且交互逻辑较多的页面，最常见的就是新闻类App的内容展示页。


## iOS中的Web与Native的通信 - Javascript

单纯的使用Web容器加载页面已经不能满足复杂的功能，开发者希望数据可以在Native和Web之间通信传递来实现复杂的功能，而Javascript就是通信的媒介。对于有WebView的情况，虽然WKWebView提供了系统级的方法，但是大部分App仍然使用基于URLScheme的WebViewBridge用以兼容。而脱离了WebView容器，系统提供了JavaScriptCore的framework，它也为之后蓬勃发展的跨平台和热修复技术提供了可能。

### 1. 基于WebView的通信

- UIWebView & WKWebView

	虽然UIWebView可以使用私有方法、
	
	web 和 app 通讯机制也通过 message handler 有很大提升。这个 API 真正神奇的地方在于 JavaScript 对象可以_自动转换_为 Objective-C 或 Swift 对象

```
window.webkit.messageHandlers.{NAME}.postMessage()
```

```

```

- WebViewJavascriptBridge

<center>
	<img width="70%" height="70%" src="https://raw.githubusercontent.com/dequan1331/dequan1331.github.io/master/assets/img/2/6.png">
</center>

### 2. 脱离WebView的通信 JavaScriptCore

- JavascriptCore原理浅析

JavascriptCore一直作为WebKit中内置的JS引擎使用，在iOS7之后，Apple对原有的C/C++代码进行了OC的封装，成系统级的framework供开发者使用。作为一个引擎来讲，JavascriptCore的词法、语法分析，以及多层次的JIT编译技术都是值得深入挖掘和学习的方向，由于篇幅的限制暂且不做深入的讨论。

词法分析 语法分析 字节码生成

[图](*******)

- JavascriptCore.framework

	提供了脱离WebView执行Javascript的环境和能力。

	1. JSVirtualMachine：提供了JS执行的底层资源及内存。虽然Java与Javascript没有一点关系，但是同样作为虚拟机，JSVM和JVM做了一部分类似的事情。每个JSVirtualMachine独占线程，拥有独立的空间和管理，但是可以包含多个JSContext。应用就是多线程？
	2. JSContext：提供了JS运行的上下文环境和接口。可以不准确的理解为，就是创建了一个Javascript中的Window对象。
	3. JSValue/JSManagedValue：提供了OC和JS间数据类型的封装和转换[Type Conversions](https://developer.apple.com/documentation/javascriptcore/jsvalue)。除了基本的数据类型，OC中的Block转换为JS中的function，Class转换为Constructor。
	4. JSExport：提供了类、属性和实例方法的调用接口。ProtoType & Constructor

	<center>
	<img width="40%" height="40%" src="https://raw.githubusercontent.com/dequan1331/dequan1331.github.io/master/assets/img/2/7.png">
</center>

- 内存
	Javascript使用GC机制管理内存，而OC采用引用计数的方式管理内存。
	由于 block 可以保有变量引用，而且 JSContext 也强引用它所有的变量，为了避免强引用循环需要特别小心。避免保有你的 JSContext 或一个 block 里的任何 JSValue。相反，使用 [JSContext currentContext] 得到当前上下文，并把你需要的任何值用参数传递。
	
	1. 循环引用
	2. 提前释放

- 基本使用

	1. 创建JSContext

	2. call js

		```objc
		JSValue *value = [self.jsContext evaluateScript:@"document.cookie"];
		```
	3. 赋值Block回调
	
		```objc
		//Native
		self.jsContext[@"addMethod"] = ^ NSInteger(NSInteger a, NSInteger b) {
	      return a + b;
		};
		
		//JS
		console.log(addMethod(1, 2));    //3
		```
	
	4. JSExport协议

		```objc
		//Native
		@protocol testJSExportProtocol <JSExport>
		@property (readonly) NSString *string;
		...
		@interface OCClass : NSObject <testJSExportProtocol>
		
		//JS
		var OCClass = new OCClass();
		console.log(OCClass.string);
		```
		

	无论通过以上两种方式进行通信，其核心都是将 保存到一个全局的Object中，例如window。
	
	对于OC中的属性和实例方法，JavaScriptCore在prototype中创建对应的属性和方法，而类方法则在
	
	对于每一个导出的实例方法，JavaScriptCore都会在prototype中创建一个对应的方法；
	对于么一个导出的实例属性，JavaScriptCore都会在prototype中创建一个对应一个存取器属性；
	对于每一个导出的类方法，JavaScriptCore会在constructor对象中创建一个对应的JavaScript function.


### 3. App中的应用场景

-  对于基于WebView的通信，主要用于App向H5页面中注入的Javascript Open Api，如提供Native的拍照、音视频、定位；以及App内的登录、分享等等功能。
-  对于JavaScriptCore，则催生了动态化、跨平台以及热修复等一系列技术的蓬勃发展。

## 动态化、跨平台与热修复

整体上来说，就是国外开发者痴迷于跨平台的开发，国内开发者执着于热更新与修复。

积极探索的方向

随着Google开源了基于Dart语言的Flutter，跨平台的技术又进入了一个新的发展阶段。

庞大的技术分支，但是作为最重要的Native和Web的通信桥梁，


在此基础之上，围绕着DSL的解析、方法表的注册、参数传递的设计以及OC Runtime的运用等不同方向，封装成了一个又一个跨平台的项目。

<center>
	<img width="70%" height="70%" src="https://raw.githubusercontent.com/dequan1331/dequan1331.github.io/master/assets/img/2/9.png">
</center>

### 2. 基于Web的跨平台技术



以Javascript作为DSL的跨平台技术方案。

目前，React Native已经开始了新一轮的重构，在线程模式、渲染方式、Native侧架构以及Api方向都会有较大的变化，相信未来在性能和使用上都会有更好的体验。

<center>
	<img width="70%" height="70%" src="https://raw.githubusercontent.com/dequan1331/dequan1331.github.io/master/assets/img/2/8.png">
</center>


```objc
    JSValue* (^callNativeBlock)(JSValue *, JSValue *, JSValue *) = ^JSValue*(JSValue *instance, JSValue *tasks, JSValue *callback){
        NSString *instanceId = [instance toString];
        NSArray *tasksArray = [tasks toArray];
        NSString *callbackId = [callback toString];
        WXLogDebug(@"Calling native... instance:%@, tasks:%@, callback:%@", instanceId, tasksArray, callbackId);
        return [JSValue valueWithInt32:(int32_t)callNative(instanceId, tasksArray, callbackId) inContext:[JSContext currentContext]];
    };
    
    _jsContext[@"callNative"] = callNativeBlock;
    
    
    //解决iOS7内存问题，设置了WXPolyfillSet
    context[@"nativeSet"] = [WXPolyfillSet class];
    
```

### 3. 基于Web的热修复技术

对于国内的iOS开发者来说，审核周期、敏感业务、支付分成以及bug修复都催生了热修复方向的不断探索。在苹果加强审核之前，几乎所有大型的App都把热修复当成了iOS开发的基础能力，最近[移动开发还有救么](https://mp.weixin.qq.com/s/6znQEtQ9L2zptohSTx9-Ew)也详细的介绍了相关黑科技的前世今生。在所有iOS热修复的方案中，基于Javascript、同时也是影响最大的就是JSPatch。

- 通信采用Block回调的方式
- 而同样基于JavascriptCore，热修复技术没有注册的机制。
- 那么JS是如何运行OC方法不报错
- require 就是在全局作用域上创建一个Object


```objc
    context[@"_OC_callI"] = ^id(JSValue *obj, NSString *selectorName, JSValue *arguments, BOOL isSuper) {
        return callSelector(nil, selectorName, arguments, obj, isSuper);
    };
    context[@"_OC_callC"] = ^id(NSString *className, NSString *selectorName, JSValue *arguments) {
        return callSelector(className, selectorName, arguments, nil, NO);
    };


	static NSString *_replaceStr = @".__c(\"$1\")(";

	__c: function(methodName) {
		...
		return function(){
			...
        	var ret = instance ? _OC_callI(instance, selectorName, args, isSuper):
                         		 _OC_callC(clsName, selectorName, args)
    		return _formatOCToJS(ret)
      }
		
    	
```


## iOS中Web相关优化策略

随着Web技术的不断升级以及App动态性业务需求的增多，越来越多的Web页面加入到了iOS App当中。与之对应的，首屏展示速度——这个对于移动客户端Web的最重要体验优化，也成为了移动客户端中Web业务最重要的优化方向。

用户体验、增长与转化率上都至关重要



### 1. 不同业务场景的优化策略

对于单纯的Web页面来说，业界早已有了合理的优化方向以及成熟的优化方案，而对于移动客户端中的Web来说，开发者在进行单一的Web优化同时，还可以通过优化Web容器以及Web页面中数据加载方式等多个途径做出优化。

关键渲染路径。

更为激进的优化，实现成本和对项目的侵入性比较大，LocalServer
通过特殊的格式规范实现了分段缓存+增量更新，，业务形态

### 2. Web维度的优化
- HTTP缓存
- 离线包

### 3. Native维度的优化

- Native接管资源请求，替代Web内核的资源加载，可以做到并行加载

### 4. 优化整体流程

## iOS中Web相关延伸业务

### 1. Javascript Open Api

随着App业务的不断发展，单纯的Web加载与渲染无法满足复杂的交互逻辑如拍照、音视频、蓝牙、定位等，同时App内需要统一的登录态，统一的分享逻辑以及支付逻辑等。

当然对于Api的设计和文档规范，[微信JS-SDK说明文档](https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421141115) 就是一个很好的例子。

- 1. 基于域名的安全控制
- 2. 由于使用客户端注入，可以根据业务逻辑选择注入的等级
- 3
- 4

<center>
	<img width="40%" height="40%" src="https://raw.githubusercontent.com/dequan1331/dequan1331.github.io/master/assets/img/2/12.png">
</center>

### 2. 模板引擎

对于新闻资讯类App，为了达到并行加载数据以及处理复杂的展示逻辑，绝大部分App都采用数据和模板分离下发的方式。而模板引擎相关技术的使用会使这种逻辑和表现分离的业务场景实现的更加简洁和优雅。

同时也提供了组件化管理业务模块的

<center>
	<img width="70%" height="70%" src="https://raw.githubusercontent.com/dequan1331/dequan1331.github.io/master/assets/img/2/10.png">
</center>

本质就是字符串的解析和替换拼接。其实模板引擎都大同小异，Logic-less的宗旨也导致Mustache

- GRMustache是基于[mustache](http://mustache.github.io/)

### 3. 资源动态更新和管理

对于Web网络请求的资源来说，通过HTTP的缓存策略可以减少通信，提升加载速度。而对于本地的样式文件、JS注入文件、默认图片等资源，频繁的读取磁盘也在一定程度上影响了资源加载速度。（离线包）

上文提到在WKWebView中虽然可以使用私有函数实现NSURLProtocol（或者iOS11+提供系统函数），但是仍然有许多问题。

- WebServer
	1. 可以在App中内置WebServer，将读取本地资源文件变成本地服务器的请求，这样就能扩展资源数据为Response，通过HTTP缓存技术实现层次化的缓存结构。（直接在file的url上加）
	2. 业界流行的Objc版开源WebServer，不外乎年久失修的[CocoaHTTPServer](https://github.com/robbiehanson/CocoaHTTPServer)以及[GCDWebServer](https://github.com/swisspol/GCDWebServer)，当然以上两个都是基于HTTP类型的，基于Socket有[]()。
	3. 通过[ports used by Apple](https://support.apple.com/en-us/HT202944)，我们可以猜测有很多端口默认已经被占用。端口的存储与占用，前后台切换重启
	4. 由于代码suspendInBackground的设置，在前后台切换的时候需要根据业务需求重启Server。
	4. url重定向，加时间
	5. Server / Connection / Request / Reponse
	6. RFC的各种设置
	7. GCDWebServerProcessBlock或GCDWebServerAsyncProcessBlock 
	
	socket建立
	```objc
	 if (bind(listeningSocket, address, length) == 0) 
    if (listen(listeningSocket, (int)maxPendingConnections) == 0) 
    
    //绑定Socket端口并接收数据源
      dispatch_source_t source = dispatch_source_create(DISPATCH_SOURCE_TYPE_READ, listeningSocket, 0, dispatch_get_global_queue(_dispatchQueuePriority, 0));

	//接收数据并创建Connection
	dispatch_source_set_event_handler(source, ^{
		...
       GCDWebServerConnection* connection = [(GCDWebServerConnection*)[self->_connectionClass alloc] initWithServer:self localAddress:localAddress remoteAddress:remoteAddress socket:socket]; 
		...

	//   dispatch_read(_socket, length, dispatch_get_global_queue(_server.dispatchQueuePriority, 0), ^(dispatch_data_t buffer, int error) {

	//GCDWebServerMatchBlock处理
    self->_request = self->_handler.matchBlock(requestMethod, requestURL, requestHeaders, requestPath, requestQuery);
      _handler.asyncProcessBlock(request, [completion copy]);

    //ProcessBlock & GCDWebServerAsyncProcessBlock
    
	```

	

- 动态更新
	1. 在资源内容发生变化时更改其Url，强制用户下载新资源。通常情况下，可以通过在文件名中嵌入文件的修改时间 & 版本号来实现。

	<center>
	<img width="40%" height="40%" src="https://raw.githubusercontent.com/dequan1331/dequan1331.github.io/master/assets/img/2/11.png">
</center>

