---
layout: web-in-ios
---

移动开发领域近年来已经逐渐告别了野蛮生长的时期，进入了相对成熟的时代。而一直以来Native和Web的争论从未停止，通过开发者孜孜不倦的努力，Web的效率和Native的体验也一直在寻求着平衡。本文聚焦iOS开发和Web开发的交叉点，希望能通过简要的介绍，帮助开发者一窥Hybrid和大前端的构想。

<br>
> ***
>_插播广告 —— 几十行代码完成资讯类App多种形式内容页_ 
>
>_[HybridPageKit](https://github.com/dequan1331/HybridPageKit) ：一个针对资讯类App高性能、易扩展、组件化的通用内容页实现框架。_
>
> ***

# <center>- 目录-</center>
<br>
<center>
<img width="70%" height="70%" src="https://raw.githubusercontent.com/dequan1331/dequan1331.github.io/master/assets/img/2/19.png">
</center>
<br>
# <center>- iOS中Web容器与加载-</center>
---
## 1. iOS中的Web容器

目前iOS系统为开发者提供三种方式来展示Web内容：

<center>
	<img width="15%" height="15%" src="https://raw.githubusercontent.com/dequan1331/dequan1331.github.io/master/assets/img/2/1.png">
	<img width="15%" height="15%" src="https://raw.githubusercontent.com/dequan1331/dequan1331.github.io/master/assets/img/2/2.png">
	<img width="13%" height="13%" src="https://raw.githubusercontent.com/dequan1331/dequan1331.github.io/master/assets/img/2/13.png">
</center>

- #### UIWebView
	
	`UIWebView`从iOS2 开始就作为App内展示Web内容的容器，但是长久以来一直遭受开发者的诟病；系统级的内存泄露、极高内存峰值、较差的稳定性、Touch Delay以及Javascript的运行性能及通信限制等等。在iOS12以后已经标记为Deprecated不再维护。

- #### WKWebView

	在iOS8中，Apple引入了新一代的WebKit framework，同时提供了`WKWebView`用来替代传统的UIWebView。它更加的稳定、拥有60fps滚动刷新率、丰富的手势、KVO、高效的Web和Native通信，默认进度条等等功能，而最重要的，是使用了和safari相同的Nitro引擎极大提升了Javascript的运行速度。`WKWebView`独立的进程管理，也降低了内存占用及Crash对主App的影响。
	
	
-  #### SFSafariViewController

	在iOS9中，Apple引入了`SFSafariViewController`。其特点就是在App内可以打开一个高度标准化的、和Safari一样界面和特性的页面。同时`SFSafariViewController`支持和Safari共享Cookie和表单数据等等。

-  #### Web容器选型
	
	对于`SFSafariViewController`：由于其标准化程度之高，使之界面和交互逻辑无法和App统一，基于App的整体体验的考虑，一般都使用在相对独立的功能和模块中，最常见的就是在App内打开App Store或者广告、游戏推广的页面。

	对于`UIWebView/WKWebView`：如果说之前由于NSURLProtocol的问题，好多App都在继续使用`UIWebView`，那么随着App放弃维护`UIWebView`（iOS12），全部的App应该会陆续的切换到`WKWebView`中来。当然，最初`WKWebView`也为开发者们带来了一些难题，但是随着系统的升级与业务逻辑的适配也逐步的修复，后文会列举几个最为关注的技术点。
	
	`UIWebView/WKWebView`对主App内存的影响：
<center>
	<img width="35%" height="35%" src="https://raw.githubusercontent.com/dequan1331/dequan1331.github.io/master/assets/img/2/20.png">
	<img width="35%" height="35%" src="https://raw.githubusercontent.com/dequan1331/dequan1331.github.io/master/assets/img/2/21.png">
</center>
	

## 2. WebKit框架与使用

- #### WebKit.framework

	[WebKit](https://webkit.org/) 是一个开源的Web浏览器引擎。每当谈到WebKit，开发者常常迷惑于它和WebKit2、Safari、iOS中的framework、以及Chromium等浏览器的关系。

	广义的`WebKit`其实就是指`WebCore`，它主要包含了HTML和CSS的解析、布局和定位这类渲染HTML的功能逻辑。而狭义的`WebKit`就是在`WebCore`的基础上，不同平台封装Javascript引擎、网络层、GPU相关的技术（WebGL、视频）、绘制渲染技术以及各个平台对应的接口，形成我们可以用的`WebView`或浏览器，也就是所谓的`WebKit Ports`。
	
	<center>
	<img width="50%" height="50%" src="https://raw.githubusercontent.com/dequan1331/dequan1331.github.io/master/assets/img/2/23.png">
	</center>
	
	比如在`Safari`中 JS的引擎使用`JavascriptCore`，而`Chromium`中使用`V8`；渲染方面`Safari`使用`CoreGraphics`，而`Chromium`中使用`Skia`；网络方面`Safari`使用`CFNetwork`，而`Chromium`中使用`Chromium stack`等等。而`WebKit2`是相对于狭义上的`WebKit`架构而言，主要变化是在API层支持多进程，分离了UI和Web接口的进程，使之通过IPC来进行通讯。
	
	对于iOS中的WebKit.framework就是在WebCore、底层桥接、JSCore引擎等核心模块的基础上，针对iOS平台的项目封装。它基于新的`WKWebView`，提供了一系列浏览特性的设置，以及简单方便的加载回调。而具体类及使用，开发者可以查阅[官方文档](https://developer.apple.com/documentation/webkit)。

<center>
	<img width="50%" height="50%" src="https://raw.githubusercontent.com/dequan1331/dequan1331.github.io/master/assets/img/2/4.png">
</center>
<br>

- #### Web容器使用流程与关键节点

	对于大部分日常使用来说，开发者需要关注的就是`WKWebView`的创建、配置、加载、以及系统回调的接收。
	
	<center>
	<img width="50%" height="50%" src="https://raw.githubusercontent.com/dequan1331/dequan1331.github.io/master/assets/img/2/14.png">
	</center>

	对于Web开发者，业务逻辑一般通过基于Web页面中Dom渲染的关键节点来处理。而对于iOS开发者，`WKWebView`提供的的注册、加载和回调时机，没有明确的与Web加载的关键节点相关联。准确的理解和处理两个维度的加载顺序，选择合理的业务逻辑处理时机，才可以实现准确而高效的应用。
	
	<center>
	<img width="60%" height="60%" src="https://raw.githubusercontent.com/dequan1331/dequan1331.github.io/master/assets/img/2/5.png">
	</center>
	<br>
- #### WKWebView常见问题

	使用`WKWebView`带来的另外一个好处，就是我们可以通过源码理解部分加载逻辑，为Crash提供一些思路，或者使用一些私有方法处理复杂业务逻辑。
		
	1. NSURLProtocol

		`WKWebView`最为显著的改变，就是不支持NSURLProtocol。为了兼容旧的业务逻辑，一部分App通过[WKBrowsingContextController](https://opensource.apple.com/source/WebKit2/WebKit2-7601.1.46.9/UIProcess/API/Cocoa/WKBrowsingContextController.h.auto.html)中的非公开方法实现了NSURLProtocol。
		```objc
		// WKBrowsingContextController
		+ (void)registerSchemeForCustomProtocol:(NSString *)scheme WK_API_DEPRECATED_WITH_REPLACEMENT("WKURLSchemeHandler", macos(10.10, WK_MAC_TBA), ios(8.0, WK_IOS_TBA));
		```
	
		在iOS11中，系统增加了 `setURLSchemeHandler`函数用来拦截自定义的Scheme。但是不同于`UIWebView`，新的函数只能拦截自定义的Scheme[(SchemeRegistry.cpp)](https://github.com/WebKit/webkit/blob/master/Source/WebCore/platform/SchemeRegistry.cpp)，对使用最多的HTTP/HTTPS依然不能有效的拦截。
		```objc
		//SchemeRegistry
	    static const StringVectorFunction functions[] {
	        builtinSecureSchemes,                // about;data...
	        builtinSchemesWithUniqueOrigins,     // javascript...
	        builtinEmptyDocumentSchemes,
	        builtinCanDisplayOnlyIfCanRequestSchemes,
	        builtinCORSEnabledSchemes,           //http;https
	    };
		```
	
	
	2. 白屏

		通常`WKWebView`白屏的原因主要分两种，一种是由于Web的进程Crash（多见于内部进程通信）；一种就是WebView渲染时的错误（Debug一切正常只是没有对应的内容）。对于白屏的检测，前者在iOS9之后系统提供了对应Crash的回调函数，同时业界也有通过判断URL/Title是否为空的方式作为辅助；后者业界通过视图树对比，判断SubView是否包含`WKCompsitingView`，以及通过随机点截图等方式作为白屏判断的依据。
	
	
	3. 其他WKWebView的系统级问题如Cookie、POST参数、异步Javascript等等一系列的问题，可以通过业务逻辑的调整重新适配
	
	4. 由于WebKit源码等开放性，我们也可以利用私有方法来简化代码逻辑、实现复杂的产品需求。例如在[WKWebViewPrivate](https://github.com/WebKit/webkit/blob/master/Source/WebKit/UIProcess/API/Cocoa/WKWebViewPrivate)中可以获得各种页面信息、直接取到UserAgent、 在[WKBackForwardListPrivate](https://github.com/WebKit/webkit/blob/master/Source/WebKit/UIProcess/API/Cocoa/WKBackForwardListPrivate.h)中可以清理掉全部的跳转历史、以及在[WKContentViewInteraction](https://opensource.apple.com/source/WebKit2/WebKit2-7601.1.46.9/UIProcess/ios/WKContentViewInteraction.mm.auto.html)中替换方法实现自定义的MenuItem等等。

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


## 3. App中的应用场景

`WKWebView`系统提供了四个用于加载渲染Web的函数。这四个函数从加载的类型上可以分为两类：加载URL & 加载HTML\Data。所以基于此也延伸出两种不同的业务场景：加载URL的**页面直出**类和加载数据的**模板渲染**类，同时两种类型各自也有不同的优化重点及方向。

- 页面直出类

	```objc
	//根据URL直接展示Web页面
	- (nullable WKNavigation *)loadRequest:(NSURLRequest *)request;
	```
	通常各类App中的Web页面加载都是通过加载URL的方式，比如嵌入的运营活动页面、广告页面等等。

- 模板渲染类

	```objc
	//根据模板&数据渲染Web页面
	- (nullable WKNavigation *)loadHTMLString:(NSString *)string baseURL:(nullable NSURL *)baseURL;
	...
	```
	需要使用WebView展示，且交互逻辑较多的页面，最常见的就是资讯类App的内容展示页。
	
<br>
# <center>- iOS中Web与Native的通信 -</center>
---

单纯的使用Web容器加载页面已经不能满足复杂的功能，开发者希望数据可以在Native和Web之间通信传递来实现复杂的功能，而Javascript就是通信的媒介。对于有WebView的情况，虽然`WKWebView`提供了系统级的方法，但是大部分App仍然使用基于URLScheme的WebViewBridge用以兼容`UIWebView`。而脱离了WebView容器，系统提供了`JavaScriptCore`的framework，它也为之后蓬勃发展的跨平台和热修复技术提供了可能。

## 1. 基于WebView的通信

基于WebView的通信主要有 **两个** 途径，一个是通过系统或私有方法，获取WebView当中的 JSContext，使用系统封装的基于JSCore的函数通信。另一类就是通过创建自定义Scheme的iframe Dom，客户端在回调中进行拦截实现。

- #### UIWebView & WKWebView系统级

	在`UIWebView`时代没有提供系统级的函数进行Web与Native的交互，绝大部分App都是通过WebViewJavascriptBridge（下节介绍）来进行的通信。但是由于JavascriptCore的存在，对于`UIWebView`来说只要有效的获取到内部的JSContext，也可以达到目的。目前已知有效的几个私有方法获取Context的方法如下：
	
	```objc
	//通过系统废弃函数获取context
	- (void)webView:(WebView *)webView didCreateJavaScriptContext:(JSContext *)context forFrame:(WebFrame *)frame;
	
	//通过valueForKeyPath获取context
	self.jsContext = [_webView valueForKeyPath:@"documentView.webView.mainFrame.javaScriptContext"];
	```
	
	在`WKWebView`中提供了系统级的Web和Native通讯机制，通过Message Handler的封装使开发效率有了很大的提升。同时系统封装了JavaScript对象和Objective-C对象的转换逻辑，也进降低了使用的门槛。

	```objc
	// js端发送消息
	window.webkit.messageHandlers.{NAME}.postMessage()
	
	//Native在回调中接收
	- (void)userContentController:(WKUserContentController *)userContentController didReceiveScriptMessage:(WKScriptMessage *)message;
	```
<br>
- #### 拦截自定义Scheme请求 - WebViewJavascriptBridge

	由于私有方法的稳定性与审核风险，开发者不愿意使用上文提到的`UIWebView`获取 JSContext的方式进行通信，所以通常都采用基于iframe和自定义Scheme的 JavascriptBridge进行通信。虽然在之后的`WKWebView`提供了系统函数，但是大部分App都需要兼容`UIWebView`与`WKWebView`，所以目前的使用范围仍然十分广泛。
	
	在Github中类似的开源框架有很多，但是无外乎都是Web侧根据固定的格式创建包含通信信息的Request，之后创建隐式iFrame节点请求；Native侧在相应的WebView回调中解析Request的Scheme，之后按照格式解析数据并处理。
	
	而对于数据传递和回调处理的问题，在兼容两种WebView、持续的更新的[WebViewJavascriptBridge](https://github.com/marcuswestin/WebViewJavascriptBridge)中，iFrame request没有直接传递数据，而是Web和Native侧维护共同的参数或回调Queue，Native通过Request中Scheme的解析触发对Queue里数据的读取。
	
	<br>
	<center>
		<img width="70%" height="70%" src="https://raw.githubusercontent.com/dequan1331/dequan1331.github.io/master/assets/img/2/6.png">
	</center>

## 2. 脱离WebView的通信 JavaScriptCore

- #### JavascriptCore

	JavascriptCore一直作为WebKit中内置的JS引擎使用，在iOS7之后，Apple对原有的C/C++代码进行了OC的封装，成系统级的framework供开发者使用。作为一个引擎来讲，JavascriptCore的词法、语法分析，以及多层次的JIT编译技术都是值得深入挖掘和学习的方向，由于篇幅的限制暂且不做深入的讨论。

<center>
<img width="60%" height="60%" src="https://raw.githubusercontent.com/dequan1331/dequan1331.github.io/master/assets/img/2/24.png">
</center>
<br>

- #### JavascriptCore.framework

	虽然JavascriptCore.framework只暴露了较少的头文件和系统函数，但却提供了在App中脱离WebView执行Javascript的环境和能力。

	- `JSVirtualMachine`：提供了JS执行的底层资源及内存。虽然Java与Javascript没有一点关系，但是同样作为虚拟机，JSVM和JVM做了一部分类似的事情。每个JSVirtualMachine独占线程，拥有独立的空间和管理，但是可以包含多个JSContext。
	- `JSContext`：提供了JS运行的上下文环境和接口。可以不准确的理解为，就是创建了一个Javascript中的Window对象。
	- `JSValue`：提供了OC和JS间数据类型的封装和转换[Type Conversions](https://developer.apple.com/documentation/javascriptcore/jsvalue)。除了基本的数据类型，需要注意OC中的Block转换为JS中的function，Class转换为Constructor等等。
	- `JSManagedValue`：Javascript使用GC机制管理内存，而OC采用引用计数的方式管理内存。所以在JavascriptCore使用过程中，难免会遇到`循环引用`以及`提前释放`的问题。JSManagedValue解决了在两种环境中的内存管理问题。
	- `JSExport`：提供了类、属性和实例方法的调用接口。内部实现是在ProtoType & Constructor中实现对应的属性和方法。
<br>
<center>
<img width="25%" height="25%" src="https://raw.githubusercontent.com/dequan1331/dequan1331.github.io/master/assets/img/2/7.png">
</center>

- #### 使用JavascriptCore进行通信

	- Native - Web: 通过JavascriptCore，Native可以直接在Context中执行JS语句，和Web侧进行通信和交互。

		```objc
		JSValue *value = [self.jsContext evaluateScript:@"document.cookie"];
		```
	- Web - Native: 对于Web侧向Native的通信，JavascriptCore提供 **两种** 方式，注册Block & Export协议。
	
		```objc
		//Native
		self.jsContext[@"addMethod"] = ^ NSInteger(NSInteger a, NSInteger b) {
	      return a + b;
		};
		
		//JS
		console.log(addMethod(1, 2));    //3

		//Native
		@protocol testJSExportProtocol <JSExport>
		@property (readonly) NSString *string;
		...
		@interface OCClass : NSObject <testJSExportProtocol>
		
		//JS
		var OCClass = new OCClass();
		console.log(OCClass.string);
		```
		
	<br>
	对于JavascriptCore粗浅的理解，可以认为使用Block方法，内部是将Block保存到保存到一个Web环境中的全局的Object中，例如window。而使用JSExport方法，则是在Web环境中Object的`prototype`中创建属性、实例方法；在`constructor`对象中创建类方法，从而实现Web中的调用。
	
## 3. App中的应用场景

-  对于基于WebView的通信，主要用于App向H5页面中注入的Javascript Open Api，如提供Native的拍照、音视频、定位；以及App内的登录、分享等等功能。
<br>
-  对于JavaScriptCore，则催生了动态化、跨平台以及热修复等一系列技术的蓬勃发展。

<br>
# <center>- 跨平台与热修复 -</center>
---

近几年来国内外移动端各种方案如雨后春笋般涌现，“Write once, run anywhere”不再是开发者的向往。剥离跨平台技术在Web侧DSL、virtualDom等方面的优化，以及Native侧Runtime的应用与封装，对于两端通信的核心，依然是JavascriptCore。

<center>
<img width="70%" height="70%" src="https://raw.githubusercontent.com/dequan1331/dequan1331.github.io/master/assets/img/2/9.png">
</center>

而不同于国外开发者对跨平台技术的积极探索，国内开发者也对热修复技术产生了极大的热情。同样作为Native和Web的交叉 - JavascriptCore，依然承担着整个技术结构中的通信任务。

## 1. 基于Web的热修复技术

对于国内的iOS开发者来说，审核周期、敏感业务、支付分成以及bug修复都催生了热修复方向的不断探索。在苹果加强审核之前，几乎所有大型的App都把热修复当成了iOS开发的基础能力，最近在[《移动开发还有救么》](https://mp.weixin.qq.com/s/6znQEtQ9L2zptohSTx9-Ew)中也详细的介绍了相关黑科技的前世今生。在所有iOS热修复的方案中，基于Javascript、同时也是影响最大的就是JSPatch。

- 基于上文的分析，对于脱离WebView的Native和Web间的通信，我们只能使用JavascriptCore。而在JavascriptCore中提供了两种方式用于通信，即Context注册Block的回调，以及JSExport。对于热修复的场景来说，我们不可能把潜在需要修复的函数都一一使用协议进行注册，更不能对新增方法和删除方法等进行处理，所以在Native和Web通信这个维度，我们只能采用Context注册Block的方式。

	```objc
	// 注册回调
	context[@"_OC_callI"] = ^id(JSValue *obj, NSString *selectorName, JSValue *arguments, BOOL isSuper) {
	    return callSelector(nil, selectorName, arguments, obj, isSuper);
	};
	context[@"_OC_callC"] = ^id(NSString *className, NSString *selectorName, JSValue *arguments) {
	    return callSelector(className, selectorName, arguments, nil, NO);
	};
	```

- 确定了通信采用Block回调的方式后，热修复就面临着如何在JS中调用类以及类的方法问题。由于没有使用JSExport等方式，JS是无法找到相应类等属性和方法，在JSPathc中，通过简单的字符串替换，将所有方法都替换成通用函数（__c），然后就可以将相关信息传递给Native，进而使用runtime接口调用方法。

	```objc
	// 替换全部方法调用
	static NSString *_replaceStr = @".__c(\"$1\")(";
	
	// 调用方法
	__c: function(methodName) {
		...
		return function(){
			...
        	var ret = instance ? _OC_callI(instance, selectorName, args, isSuper):
                         		 _OC_callC(clsName, selectorName, args)
    		return _formatOCToJS(ret)
      }
	```
- 当然对于JSPatch以及其他热修复的项目来说，Web和Native通信只是整个框架中的一个技术点，更多的实现原理和细节由于篇幅的关系暂且不做介绍。

## 2. 基于Web的跨平台技术

随着Google开源了基于Dart语言的Flutter，跨平台的技术又进入了一个新的发展阶段。对于传统的跨平台技术来讲，各个公司以JavascriptCore作为通信桥梁，围绕着DSL的解析、方法表的注册、模块注册通信、参数传递的设计以及OC Runtime的运用等不同方向，封装成了一个又一个跨平台的项目。

<center>
	<img width="70%" height="70%" src="https://raw.githubusercontent.com/dequan1331/dequan1331.github.io/master/assets/img/2/8.png">
</center>

而在其中，以Javascript作为前端DSL的跨平台技术方案里，FaceBook的[react-native](https://github.com/facebook/react-native)以及阿里(目前托管给了Apache)的[Weex](https://github.com/apache/incubator-weex)最为流行。在网络上两者的比较文章有很多，集中在学习成本、框架生态、代码侵入、性能以及包大小等,各个业务可以根据自己的重点选择合理的技术结构。

而不管是`react-native`还是`Weex`,Web和Native的通信桥梁仍然是JavascriptCore。

```objc
//weex 举例
JSValue* (^callNativeBlock)(JSValue *, JSValue *, JSValue *) = ^JSValue*(JSValue *instance, JSValue *tasks, JSValue *callback){
	...
  return [JSValue valueWithInt32:(int32_t)callNative(instanceId, tasksArray, callbackId) inContext:[JSContext currentContext]];
};
_jsContext[@"callNative"] = callNativeBlock; 
```

和热修复技术一样，跨平台又是一个庞大的技术体系，JavascriptCore仅仅是作为整个体系运转中的一个小小的部分，而整个跨平台的技术方案就需要另开多个篇幅进行介绍了。

<br>
# <center>- iOS中Web相关优化策略 -</center>
---

随着Web技术的不断升级以及App动态性业务需求的增多，越来越多的Web页面加入到了iOS App当中。与之对应的，首屏展示速度——这个对于移动客户端Web的最重要体验优化，也成为了移动客户端中Web业务最重要的优化方向。

这一章节更为详细的设计与实现，请移步[iOS新闻类App内容页技术探索](./hybrid-page-kit.html);

## 1. 不同业务场景的优化策略

对于单纯的Web页面来说，业界早已有了合理的优化方向以及成熟的优化方案，而对于移动客户端中的Web来说，开发者在进行单一的Web优化同时，还可以通过优化Web容器以及Web页面中数据加载方式等多个途径做出优化。

所以对于iOS开发中的优化来说，就是通过Native和Web两个维度的优化关键渲染路径，保证WebView优先渲染完毕。由此我们梳理了常规Web页面整体的加载顺序，从中找出关键渲染路径，继而逐个分析、优化。

<center>
	<img width="50%" height="50%" src="https://raw.githubusercontent.com/dequan1331/dequan1331.github.io/master/assets/img/2/18.png">
</center>

## 2. Web维度的优化

- #### 通用Web优化
	 
	 对于Web的通用优化方案，一般来说在网络层面，可以通过DNS和CDN技术减少网络延迟、通过各种HTTP缓存技术减少网络请求次数、通过资源压缩和合并减少请求内容等。在渲染层面可以通过精简和优化业务代码、按需加载、防止阻塞、调整加载顺序优化等等。对于这个老生常谈的问题，业内已经有十分成熟和完整的总结，例如[Best Practices for Speeding Up Your Web Site](https://developer.yahoo.com/performance/rules.html?guce_referrer=aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTA2NDg1NTUvYXJ0aWNsZS9kZXRhaWxzLzUwNzIxNzUx&guce_referrer_sig=AQAAALFDmgeec4o4M3qDGMF4pJ7mfwbrvNzWRSoHOuVgUBMqfpOi8fCJWX9_zYqnLeQlvhGa39OdzymsPP-4C5wB0XqzU7oux3SZqBfbgoNOrf_ScV24a05ZfBxcwZ7oQaG8nrdgAAjNCKhFRAwQu18yQdsMJuY1M_CReqF8_dbbZJlV&guccounter=1)，在这里暂不做深入的展开。

- #### 其他
	
	脱离较为通用的优化，在对代码侵入宽容度较高的场景中，开发者对Web优化有着更为激进的做法。例如在[VasSonic](https://github.com/Tencent/VasSonic)中，除了Web容器复用、数据分离预拉取和通用的优化方式外，通过自定义VasSonic标签将HTML页面进行划分，分段进行缓存控制。
	

## 3. Native维度的优化

- #### 容器复用和预热

	WKWebView虽然JIT大幅优化了JS的执行速度，但是单纯的加载渲染HTML，WKWebView比UIWebView慢了很多。根据渲染的不同阶段分别对耗时进行测试，同时对比UIWebView，我们发现WKWebView在初始化及渲染开始前的耗时较多。

	<center>
	<img width="55%" height="55%" src="https://raw.githubusercontent.com/dequan1331/dequan1331.github.io/master/assets/img/2/22.png">
	</center>

	针对这种情况，业界主流的做法就是复用 & 预热。预热即时在App启动时创建一个WKWebView，使其内部部分逻辑预热已提升加载速度。而复用又分为两种，较为复杂的是处理边界条件已达到真正的复用，还有一种较为Triky的办法就是常驻一个空WKWebView在内存。在[HybridPageKit](https://github.com/dequan1331/HybridPageKit)中，就提供了易于集成的完整WKWebView重用机制实现。


- #### Native并行资源请求 & 离线包

	由于Web页面内请求流程不可控以及网络环境的影响，对于Web的加载来说，网络请求一直是优化的重点。开发者较为常用的做法是使用Native并行代理数据请求，替代Web内核的资源加载。在客户端初始化页面的同时，并行开始网络请求数据；当Web页面渲染时向Native获取其代理请求的数据。	
	
	而将并行加载和预加载做到极致的优化，就是离线包的使用。将常用的需要下载资源（HTML模板、JS文件、CSS文件、占位图片）打包，App选择合适的时机全部下载到本地，当Web页面渲染时向Native获取其数据。通过离线包的使用，Web页面可以并行（提前）加载页面资源，同时摆脱了网络的影响，提高了页面的加载速度和成功率。当然离线包作为资源动态更新的一个方式，合理的下载时机、增量更新、加密和校验等方面都是需要进行设计和思考的方向，后文会简单介绍。

- #### 替换任意标签Native实现，地图、音视频

	当并行请求资源，客户端代理数据请求的技术方案逐渐成熟时，由于WKWebView的限制，开发者不得不面对业务调整和适配。其中保留原有代理逻辑、采用LocalServer的方式最为普遍。但是由于WKWebView的进程间通信、LocalServer Socket建立与连接、资源的重复编解码都影响了代理请求的效率。
	
	<center>
	<img width="50%" height="50%" src="https://raw.githubusercontent.com/dequan1331/dequan1331.github.io/master/assets/img/2/15.png">
	</center>
	
	所以对于一些资讯类App，通常采用Native的方式实现复杂节点，如图片、地图、音视频等模块。这样不但能减少通信和请求的建立、提供更加友好的交互、也能并行的进行View的渲染和处理，减少Web页面的业务逻辑。在[HybridPageKit](https://github.com/dequan1331/HybridPageKit)中就提供封装好的功能框架。

- #### 按优先级划分业务逻辑

	从App的维度上看，一个Web页面从入口点击到渲染完成，或多或少都会有Native的业务逻辑并行执行。所以这个角度的优化关键渲染路径，就是优先保证WebView以及其他在首屏直接展示的Native模块优先渲染。所以承载Web页面的Native容器，可以根据业务逻辑的优先级，在保证WebView模块展示之后，选择合适的时机进行数据加载、视图渲染等。这样就能保证在内容页的维度上，关键路径优先渲染。
	
	<center>
	<img width="60%" height="60%" src="https://raw.githubusercontent.com/dequan1331/dequan1331.github.io/master/assets/img/2/17.png">
	</center>

## 4. 优化整体流程

所以整体上对于客户端来说，我们可以从Native维度（容器和数据加载）以及Web维度两个方向提升加载速度，按照页面的加载流程，整体的优化方向如下：

<center>
	<img width="60%" height="60%" src="https://raw.githubusercontent.com/dequan1331/dequan1331.github.io/master/assets/img/2/16.png">
</center>

<br>
# <center>- iOS中Web相关延伸业务 -</center>
---


## 1. 模板引擎

- 为了达到并行加载数据以及并行处理复杂的展示逻辑，对于非直出类型的Web页面，绝大部分App都采用数据和模板分离下发的方式。而这样的技术架构，导致在客户端内需要增加替换对应DSL的模板标签，形成最终的HTML的业务逻辑。简单的字符串替换逻辑不但低效，还无法做到合理的组件化管理，以及组件合理的与Native交互，而模板引擎相关技术会使这种逻辑和表现分离的业务场景实现的更加简洁和优雅。

- 基于模板引擎与数据分离，客户端可以根据数据并行创建子业务模块，同时在子业务模块中处理和Native交互的部分如图片裁剪适配、点击跳转等等，生成HTML代码片段。之后基于模板进行替换生成完整的页面。这样不但减少了大量的字符串替换逻辑，同时业务也得到了合理拆分。

<center>
	<img width="70%" height="70%" src="https://raw.githubusercontent.com/dequan1331/dequan1331.github.io/master/assets/img/2/10.png">
</center>
<br>

- 模板引擎的本质就是字符串的解析和替换拼接。在Web端不同的使用场景有很多不同语法的引擎类型，而在客户端较为流行的，有使用较为复杂的[MGTemplateEngine](https://github.com/mattgemmell/MGTemplateEngine)，它类似于Smarty，支持部分模板逻辑。也有基于[mustache](http://mustache.github.io/)，Logic-less的[GRMustache](https://github.com/groue/GRMustache)可供选择。

## 2. 资源动态更新和管理

无论是离线包、本地注入的JS、CSS文件、以及Web中的默认图片，目的都是通过打包或提前下载，替换网络请求为本地读取来优化Web的加载体验和成功率。而对于这些资源的管理，开发者需要从合理的下载与更新，以及Web中合理的访问这两个方面进行设计。

- #### 下载与更新
	
	下载与重试：对于资源或是离线包的下载，选择合适的时机、失败重载时机、失败重载次数都要根据业务灵活调整。通常为了增加成功率和及时更新，在冷启动、前后台切换、关键的操作节点，或者采用定时轮循的方式，都需要进行资源版本号或MD5的判断，用以触发下载逻辑。当然对于服务端来说，合理的灰度控制，也是保证业务稳定的重要途径。
	
	签名校验：对于动态下载的资源，我们都需要将原文件的签名进行校验，防止在传输过程中被篡改。对于单项加密的办法就是双端对数据进行MD5的加密，之后客户端校验MD5是否符合预期；而双向加密可以采用DES等加密算法，客户端使用公钥对资源验证使用。
	
	增量更新：为了减少资源和离线包的重复下载，业内大部分使用离线包的场景都采用了增量更新的方式。即客户端在触发请求资源时，带上本地已存在资源的标示，服务端根据标示和最新资源做对比，之后只提供新增或修改的Patch供客户端下载。

- #### 基于LocalServer的访问

	离线包自身的下载业务，需要考虑下载时机、是否采用增量下载、校验等等通用的问题。而对于已经下载好的离线包，如何将Web请求重定向到本地，大部分App都依赖于NSURLProtocol。上文提到在WKWebView中虽然可以使用私有函数实现（或者iOS11+提供系统函数），但是仍然有许多问题。目前业界一部分App，都采用了集成LocalServer的方式，接管部分Web请求，从而达到访问本地资源的目的。同时集成了LocalServer，通过将本地资源封装成Response，利用HTTP的缓存技术，进一步的优化了读取的时间和性能，实现层次化的缓存结构。	
	而使用了本地资源的HTTP缓存，就需要考虑缓存的控制和过期时间。通常可以通过在URL上增加本地文件的修改时间、或本地文件的MD5来确保缓存的有效性。
	
	<center>
	<img width="40%" height="40%" src="https://raw.githubusercontent.com/dequan1331/dequan1331.github.io/master/assets/img/2/11.png">
	</center>
	
- #### GCDWebServer浅析
	排除Socket类型，业界流行的Objc版针对HTTP开源的WebServer，不外乎年久失修的[CocoaHTTPServer](https://github.com/robbiehanson/CocoaHTTPServer)以及[GCDWebServer](https://github.com/swisspol/GCDWebServer)。GCDWebServer是一个基于GCD的轻量级服务器，简单的四个模块 - Server / Connection / Request / Reponse，以及通过维护LIFO的Handler队列传入业务逻辑生成响应。在排除了基于RFC的Request/Response协议设计之外，关键的代码和流程如下：

	```objc
	//GCDWebServer 端口绑定
	bind(listeningSocket, address, length)
	listen(listeningSocket, (int)maxPendingConnections)
    
    //GCDWebServer 绑定Socket端口并接收数据源
    dispatch_source_t source = dispatch_source_create(DISPATCH_SOURCE_TYPE_READ, listeningSocket, 0, dispatch_get_global_queue(_dispatchQueuePriority, 0));
	
	//GCDWebServer 接收数据并创建Connection
	dispatch_source_set_event_handler(source, ^{
		...
       GCDWebServerConnection* connection = [(GCDWebServerConnection*)[self->_connectionClass alloc] initWithServer:self localAddress:localAddress remoteAddress:remoteAddress socket:socket]; 
	
	//GCDWebServerConnection 读取数据
	dispatch_read(_socket, length, dispatch_get_global_queue(_server.dispatchQueuePriority, 0), ^(dispatch_data_t buffer, int error) {
	
	//GCDWebServerConnection 处理GCDWebServerMatchBlock和GCDWebServerAsyncProcessBlock
    self->_request = self->_handler.matchBlock(requestMethod, requestURL, requestHeaders, requestPath, requestQuery);
    ...
    _handler.asyncProcessBlock(request, [completion copy]);
	```
	在LocalServer的使用上，也要注意端口的选择[ports used by Apple](https://support.apple.com/en-us/HT202944)，以及前后台切换时suspendInBackground的设置和业务处理。

## 3. Javascript Open Api

- 随着App业务的不断发展，单纯的Web加载与渲染无法满足复杂的交互逻辑如拍照、音视频、蓝牙、定位等，同时App内需要统一的登录态，统一的分享逻辑以及支付逻辑等。所以针对第三方的Web页面，Native需要注册相应的Javascript接口供Web使用。

- 当然对于Api需要提供的能力、接口设计和文档规范，不同的业务逻辑和团队代码风格会有不同的定义，[微信JS-SDK说明文档](https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421141115) 就是一个很好的例子。而脱离Javascript Open Api对外的接口设计和封装，在内部的实现上也有一些通用的关键因素，这里简单列举几个：

	- #### 注入方式和时机
	
		对于Javascript文件的注入，最简单的就是将文件打包到项目中，使用WKWebView提供的系统函数进行注入。这种方式无需网络加载，可以合理的选择注入时机，但是无法动态的进行修改和调整。而对于这部分业务需求需要经常调整的App来说，也可以把文件存储到CDN，通过模板替换或者和Web合作者约定，在Web的HTML中通过URL的方式进行加载，这种的方式虽然动态话程度较高，但是需要合作方的配合，同时对于JS Api也不能做到拆分的注入。
		
		针对上面的两种方式的不足，一个较为合理的方式是建立资源的动态更新系统（下文具体介绍），同时Javascript文件采用本地注入的方式。这样一方面支持了动态更新，也无需合作方的配合，同时对于不同的业务场景可以拆分不同的Api进行注入，保证安全。

	- #### 安全控制

		对于Javascript Open Api设计实现的另一个重要方面，就是安全性的控制。由于完整的Api需要支持Native登录、Cookies等较为敏感的信息获取，同时也支持一些对UI和体验影响较多的功能如页面跳转、分享等，所以App需要一套权限分级的逻辑控制Web相关的接口调用，保证体验和安全。
		
		常规的做法就是在Javascript Open Api建立分级的管理，不同权限的Web页面只能调用各自权限内的接口。客户端通过Domain进行分级，同时支持动态拉取权限Domain白名单，灵活的配置Web页面的权限。在此基础上App内部也可以通过业务逻辑的划分，在Native层面使用不同的容器加载页面，而容器根据业务逻辑的不同，注入不同的JS文件进行权限控制。

<center>
	<img width="30%" height="30%" src="https://raw.githubusercontent.com/dequan1331/dequan1331.github.io/master/assets/img/2/12.png">
</center>
# <center>- 其他 -</center>

> ***
>_插播广告 —— 几十行代码完成资讯类App多种形式内容页_ 
>
>_[HybridPageKit](https://github.com/dequan1331/HybridPageKit) ：一个针对资讯类App高性能、易扩展、组件化的通用内容页实现框架。_
>
> ***



