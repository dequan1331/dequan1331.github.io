---
layout: web-crossplatform

---

随着互联网红利的消失，整个移动市场的关注从“流量”转成了“留量”，大部分的移动产品也都告别了初期的抢占市场，进入了 A/B 实验和快速试错的阶段，迭代速度、效果验证的压力与日俱增，效率变成了移动 App 的核心竞争力。

同时随着红海期结束，现有 App 都开始对用户的时间进行激烈的争夺。普通的 App 不断的扩展领域和内容来满足长尾需求、超级 App 们也不断的提高护城河构建自有生态。所以无论是流量分发、精细化运营、还是提升时长，都需要 App 不断的增加平台化的属性，高效的响应调整和变化。

技术趋势一定是顺应行业发展的，在**提升效率**这样的行业背景下，App 整体的架构和技术选型，也都到了进行服务于效率的架构升级，拥抱**工程化敏捷**的时候了。

# <center>- 目录-</center>

<br>

<center>
<img width="50%" height="50%" src="https://raw.githubusercontent.com/dequan1331/dequan1331.github.io/master/assets/img/3/00.png">
</center>

# <center>- 基于 Web 技术栈的敏捷提效-</center>

---

无论激进的改革派们选择哪一种单一的技术栈进行重构、也无论动态化和跨平台技术中台团队如何分享和布道，面对存量代码和历史包袱，除去中台和组件化等优化，为了提升 App 整体的工程化效率，最实际和平稳的架构升级方案，就是基于 Web 和 JS 技术栈，选择和搭配多种框架或优化来满足业务需求，解决业务痛点。

- 面对 A/B 实验和快速试错，动态化 UI 的方案一定是较好的选择。无论是基于 DSL + Layout、还是基于 Web & JS 的技术栈，结合成熟框架和二次封装优化，面向动态化重构 UI 框架可以极大的提升 App 的交付效率。
- 面对平台化和精细运营，App 的职责也会逐渐向通用容器、能力和引擎的提供者转变。App 会从功能的实施者，逐渐向连接、分发和赋能的方向转变。这就需要 App 不断优化提供的容器和引擎，而其中最常用的容器就是 WebView。
- 面对小程序这种超级流量 App 形成的新平台。开发者目前不光要开发 iOS / Android 的 App，还要兼容 Web 以及各种平台的小程序。所以跨平台的方案也需要不断的进化升级，来减少开发和沟通成本，释放人力提升开发效率。

<center>
<img width="50%" height="50%" src="https://raw.githubusercontent.com/dequan1331/dequan1331.github.io/master/assets/img/3/1.png">
</center>

当然基于 Web 技术栈的优化只是未来 App 架构升级的方向之一，App 面对效率的提升、平台化建设以及中台的技术趋势，也需要不断的升级和扩展其他技术栈和平台新技术来寻求突破，赋能产品。在这里我们只围绕**动态化布局和框架**、**Web 容器优化**和**小程序跨平台**三个方面，简单介绍 Web 技术栈的方案。

# <center>- 动态化布局 -</center>

---

无论任何形态的 App 产品，对于客户端开发的大部分的工作都集中在了不断的 UI 样式调整、交互调整、A/B 实验等等。那么对于工程化敏捷的实现，无论是提升开发效率、减少 Leadtime 还是释放人力成本，对这部分能力的升级都是极为关键和重要的， 这就要求 App 必须要实现 **动态化 UI 布局** 的能力。

## 1. DSL + Layout 方案

回归到动态化 UI 到本质，我们希望做到 UI 布局动态化，最简单的想法就是动态下发 UI 的布局信息，Native 使用解析之后的数据对 View 进行布局。对于这种"配置性"的文件，我们通常选用 JSON / XML / YAML 等等来进行描述，客户端根据约定进行解析和使用即可。

随着 UI 复杂性的不断变高，简单的解析已经满足不了互相依赖的视图组织。这个时候就需要一个高效的布局引擎对解析之后的数据进行计算。同时为了缓存和高效的对比计算，我们也需要 "virtual node" 来存储视图信息和依赖关系，防止频繁对 View 进行绘制。

至此，一个基本的基于 DSL + Layout 的动态化 UI 方案已经初具雏形。我们使用 DSL 来配置 UI 的布局信息，客户端解析DSL 中存储的节点属性和依赖关系，形成虚拟节点视图树，通过高效的 Layout 引擎计算依赖后对 Native View 进行赋值。

<center>
<img width="60%" height="60%" src="https://raw.githubusercontent.com/dequan1331/dequan1331.github.io/master/assets/img/3/2.png">
</center>

在整体方案的应用过程中，我们自然会对 DSL 以及协议的约定进行能力扩充、对布局计算不断的优化、对 iOS / Andorid 多平台的一致性支持，以及对整个方案做工程化的建设和优化。在能力支持上，DSL 中可以预埋一些状态管理，交互事件、动画属性甚至是数据交互能力，通过合理的协议约定与解析，我们就可以不断的扩展自定义的功能需求。

而对于跨平台一致的布局协议约定，前端已有成熟的 Flexbox 布局协议，同时对应 Flexbox 布局的开源 [Yoga](https://github.com/facebook/yoga) 引擎正好提供了跨平台的高效计算能力。在实际应用中的工程化建设和优化，从配置文件的更新和版本管理、到整合视图形成通用组件、可视化的预览平台，以及在整个链路中节点缓存、视图层级优化、事件处理优化和生命周期管理等等。

其实以上，也就是目前流行的 DSL + Layout 动态化方案的整体实现。天猫的 [Tangram](http://tangram.pingguohe.net/)、美团的 [MTFlexbox]() 和 Picasso 等等都是在此基础上进行的完善和封装。同时由于 Flexbox + Yoga 的出现，这种技术方案甚至已经发展到了几乎每个小团队都有一个轮子的程度。

## 2. JavaScript 运行时方案

通过 DSL + Layout 的方式，我们就可以实现交互较少的基础 UI 布局的动态化能力。那么随着动态化 UI 的功能边界不断扩大，这种方式的局限性也渐渐的显露出来：对于自定义视图的扩展、交互能力处理的限制、复杂 UI 状态的管理复杂性等等。

解决这些问题的办法，就是使静态的配置性语言"动"起来，那么就需要一个语言的引擎、或者 Virtual Machine 来运行我们的语言。由于 WebView 在 iOS / Android 中的广泛应用，自然就想到了 Javascript + V8 / JSCore 这样的组合。随着语言引擎的引入就需要扩展和完善 JS / Native 的通信机制、封装完善的组件和能力，这样就形成了 RN / Weex 框架的雏形。

<center>
<img width="80%" height="80%" src="https://raw.githubusercontent.com/dequan1331/dequan1331.github.io/master/assets/img/3/3.png">
</center>

在这类框架中，伴随着上文提到的主路径和扩展组件以及能力，一般就是由4个大的模块构成：JS 引擎模块提供分平台的 JS 执行能力，Native 组件和能力分别作为 Modules 模块和 Component 模块。而作为 JS 和 Native 之间的桥梁，在framework 模块就提供了链接整个链路的功能和对外接口。

具体的来说，JS 引擎模块一般使用 C++ 封装平台无关的 Wrapper 层，然后针对不同的平台，使用JSCore 或者 V8 来实现 JS 的运行环境。 对于 UI 组件，一般由一个 Component 配合着对应的 View 来实现，其中 Component 提供了该组件对外的接口、属性等等。而非 UI 的组件，一般称为 Modules 或者 Manager，常用的包括网络、导航、以及结合 Layout 处理 UI 布局的 UI Modules。作为整体的流程的链接，在 framework 模块提供执行 JS 的 Executor、Modules 和Components 的注册管理 Register 、函数调用和数据通信的解析和转换 Method、以及供开发者使用的 Bridge 以及 RoootView 等等。

<center>
<img width="50%" height="50%" src="https://raw.githubusercontent.com/dequan1331/dequan1331.github.io/master/assets/img/3/4.png">
</center>

当然作为一个功能完备的框架，在这类框架中我们不光要关注整体功能的模块分布，其中 JS、布局、UI 等业务线程的管理、跨栈数据传递的优化、组件注册以及布局、更新逻辑，以及基于 JS 驱动异步布局的设计思想，都值得不断的学习和体会。

## 3.框架优化和工程化

随着这类技术方案的普及，各大公司和团队也都有了定制化的优化方案， 比如基于 RN 腾讯的 [Hippy]()、携程的 [CRN](https://github.com/ctripcorp/CRN)，使用 TypeScript 实现的 [Titanium](https://github.com/appcelerator/titanium_mobile) 等等。同时相应的方案也被更上层的框架所内置，比如后文介绍的跨平台方案如 [chameleon](https://github.com/didi/chameleon) , [uni-app](https://github.com/dcloudio/uni-app) 等也都内置了 Weex 来实现 Native App 。整体上看，为了更好的和自身的历史逻辑和产品形态结合，不同的开发团队在保证核心链路的基础上，对整个框架方案的各个层面都进行了思考和优化。

在描述和语言层，不断的扩展语言支持以及转换逻辑来兼容不同的前端历史代码；在 Framework 框架层面，通过对 Widget 的深度优化，比如列表 Cell 复用、动画优化以及分包和自动化工具等等完成更多的场景覆盖；在 VM 及引擎内核层使用定制的 JS 内核、封装 C++ 跨平台能力提高通用逻辑的性能；而随着 Flutter 带来的高效渲染引擎 Skia，在平台渲染层也都纷纷尝试替换提供更加高效的一致性渲染。

<center>
<img width="60%" height="60%" src="https://raw.githubusercontent.com/dequan1331/dequan1331.github.io/master/assets/img/3/5.png">
</center>

同时与现有业务融合的工程化建设和方法论，也是近几年各大演讲分享的热门话题。从自动化构建打包集成、JS 文件的检测、版本兼容以及离线资源发布，到各种工具链和脚手架、完善的debug能力和可视化的发布平台。从各项性能指标的完整监控、统计和告警，到对历史业务和逻辑的重构经验。

# <center>- Web 容器优化 -</center>

---

面对用户关注和时间的激烈争夺，大部分 App 中都增加了大量的运营需求，比如红包礼包、福利任务、用户等级等来增加留存和提高用户时长，同时对热点事件、爆款的运营和玩法也变得越来越灵活和复杂。针对这种突发、多变、甚至政策风险的场景，产品需求对 App 提供的 Web 容器也提出了更高的要求。

## 1. WebView 优化

App 作为 WebView 容器和 Native 能力提供方，就需要对第三方提供高性能和高质量的 Webview、灵活的展示场景、以及安全和全面的 Native 能力和数据。

与之对应的在 App 中的技术落地，除了前端角度的 Web 优化，客户端作为容器的提供者，常规的会从 WebView 的复用回收和预热、扩展安全的方法和接口封装、全面安全的 JSApi 设计管理等，来提升 Web 容器的加载速度、稳定性和易用性等，比如 [微信 JS-SDK](https://developers.weixin.qq.com/doc/offiaccount/OA_Web_Apps/JS-SDK.html) 的设计或 mPaas 中的 [H5容器和离线包](https://tech.antfin.com/docs/2/59192)。同时结合业务场景，扩展 Web 容器以页面、卡片、窗口浮层等多种展示手段；提供离线包和预加载功能或类似 [VasSonic](https://github.com/Tencent/VasSonic) 这样更侵入和激进的离线缓存机制；完整细致的监控平台保障稳定等等。

而更进一步，我们可以通过更加底层的优化，提供定制化的浏览器的内核来满足业务场景。比如 [X5内核](https://x5.tencent.com/) 自定义的浏览器Cache、内存管理、安全的连接等等；比如微信在发展到 [JS SDK](https://developers.weixin.qq.com/doc/offiaccount/OA_Web_Apps/JS-SDK.html) 之后就因为 Web 的各种瓶颈转而开始发展 [小程序](https://developers.weixin.qq.com/ebook?action=get_post_info&docid=0008aeea9a8978ab0086a685851c0a) ，进而实现了逻辑渲染分离、结合业务的同层渲染、更加安全的管控能力等等。

<center>
<img width="45%" height="45%" src="https://raw.githubusercontent.com/dequan1331/dequan1331.github.io/master/assets/img/3/6.png">
</center>

总的来说，客户端中的 Web 容器优化，就是围绕着 WebView，从自身加载性能、Native 能力和数据支持、对外接口安全扩展、离线包和丰富的 Native 展示场景等角度提供完整的优化方案。这其中并没有完整的通用框架支持，需要开发者整合多个优化点，围绕业务层面制定优化策略。

## 2. Web App

当然以上都是从 App 开发者的视角进行的容器优化，而从前端的角度，如何使用 WebView 进行场景和功能实现，将 Web 内容迁移到移动设备上形成独立 App 才是对于他们的最优方案。所以无论是 [cordova](https://cordova.apache.org/)、 [PhoneGap](https://phonegap.com/)、[capacitor](https://capacitor.ionicframework.com/)，还是国内的 [Kerkee](https://github.com/kercer)、 [appcan](https://www.appcan.cn/)、 [WeX5](http://www.wex5.com/wex5/) 等等都是这一类的解决方案和开源框架。这些框架主要实现了 Native App 的基本框架、加载展示 Web 容器、最重要的就是解决 JS 和 Native 间的通信问题，封装Native 能力，支持 JS 使用 Native 的能力。

随着 Apple 审核的愈加严格，这类 App 今后也面临着越来越大的压力。与上文提到的动态化布局和框架不同，无论是 WebView 容器的优化还是 Web App，这类方案不但使用 JS 引擎作为逻辑实现，同时还使用 WebView 作为渲染展示。而对于这类方案更加深入的优化，就不得不对提到小程序了。Web 页面通过 Web App 框架在操作系统上构建完整的 App，而通过小程序框架，在各自的平台里就构建出了小程序这样的功能形态。

<center>
<img width="45%" height="45%" src="https://raw.githubusercontent.com/dequan1331/dequan1331.github.io/master/assets/img/3/9.png">
</center>

# <center>- 小程序和跨平台 -</center>

---

构建 App 敏捷型架构的目的，除了快速满足用户需求、提升用户体验之外，还有的就是对开发效率的提升和人员成本的优化。那么相应的对于客户端的 Web 技术栈，跨平台技术的应用正是解决的这个问题。

## 1. 小程序

从技术实现上说，小程序就是 JS-Api 的升级优化版，核心解决的问题也是 Web 侧对 Native / 宿主 App 能力的调用、和 Native / 宿主 App UI 组件的深度结合等。当然以微信小程序举例，小程序在以 WebView 为渲染核心的基础上有很多优化，比如为了提升渲染性能和安全管控采用的双线程模型以及背后的优化、解决 Native 组件在 WebView 中的展示层级使用了同层渲染技术的优化、以及渲染层面采用 Skia 的尝试等等。

从产品形态来看，小程序的出现是流量寡头们为了自己的护城河，建立自有生态从而进行流量分发。所以目前各大超级 App 都纷纷入场推出自己的小程序平台。从这个角度讲，短时间内小程序很难有统一的标准或者协议，或者提供统一的接入方式，开发者必然会进入多平台小程序适配和开发的阶段。

小程序平台的不断碎片化也为业务开发者带来了负担。为了提升开发效率，目前业界大部分的开源项目也都集中在这个角度的优化和提效，比如一份代码生成多平台小程序框架 [Taro](https://taro.aotu.io/)、 [mpvue](https://github.com/Meituan-Dianping/mpvue) ；小程序和 Web 端的同构 [kbone](https://github.com/Tencent/kbone)、[alita](https://github.com/areslabs/alita)、[remax](https://github.com/remaxjs/remax) 等等。

<center>
<img width="50%" height="50%" src="https://raw.githubusercontent.com/dequan1331/dequan1331.github.io/master/assets/img/3/7.png">
</center>

未来普通 App 在提供小程序的同时，为了增加留存和用户时长，也需要打破信息壁垒扩展更多的第三方内容。同时为了尽可能的挖掘每一个用户，提供差异化的内容和服务满足长尾，App 或许会逐步的增加面向 A/B 实验或者特定人群的功能入口。面对这种需求，传统的 SDK 接入或数据接口服务相较于小程序，在侵入性、平台相关性、动态扩展、移植性和开发效率上，都有极大的差距。

如何生成自己的引擎提供能力，利用小程序的生态，让超级 App 平台的小程序能在我们的 App 中运行，或许是一个不错的解决方案。目前小程序平台间的转换框架 [wwto](https://github.com/wuba/wwto) 在类似的场景有了一定的探索，相信未来在这个方向上会有更多的框架涌现出来，也会有更多的想象空间在等待着我们。

## 2. 跨平台

无论是最初的 Web，还是使用 DSL 实现统一 UI 的方案，到后来的 RN、Weex 方案，其实都是有“跨平台”的“ Write once, run anywhere”思想。但是从他们的实现上我们可以发现，这些技术方案并没有真正抹平平台的差异，只是对于开发者使用统一的语言来实现而已。

而在渲染层面，Flutter 的到来才真正的抹平了在渲染层面的平台差异化，提供了平台一致性的渲染引擎 Skia。

对于 Web 和 JS 技术栈的框架来说，。那么 Flutter 的到来就为跨平台的方案提供了一种新的思路，所以类 RN / Weex 框架目前大多数也都在与 Skia 进行结合。

在跨平台方向的探索上，除了 Flutter 或者类 RN / Weex 使用统一的语言进行分平台渲染的方案，还有一类就是编译转换的方案，比如将 iOS 转换成 Andriod 的框架 [MyAppConverter](https://www.myappconverter.com/v2/)、Java 转换成 OC 的工具 [j2objc](https://developers.google.com/j2objc)、以及使用 JS 开发 Flutter 的框架 [mxflutter](https://github.com/mxflutter/mxflutter)、 [Kraken](https://www.infoq.cn/article/V6IuINi-9UJ21Tk1qEAY) 、甚至是使用 Dart 开发 iOS / Android 的框架 [[dart_native](https://github.com/dart-native/dart_native)](https://github.com/dart-native/dart_native)。

随着互联网流量的马太效应，超级 App 逐渐成为新的底层平台。开发者在面对 iOS / Android / Web 三个平台的基础上，又要不断的适配微信小程序、支付宝小程序、qq 小程序等等一众超级 App 平台。那么相应的基于 Javascript 的跨平台方案自然也就扩展到了更多平台的支持上。

业界比如 [chameleon](https://github.com/didi/chameleon) , [uni-app](https://github.com/dcloudio/uni-app) 都是在这样的场景下催生的跨平台框架。

<center>
<img width="40%" height="40%" src="https://raw.githubusercontent.com/dequan1331/dequan1331.github.io/master/assets/img/3/8.png">
</center>

# <center>- 结语 -</center>

---

从 Hybrid App 到 小程序，从 React Native / Weex 到 Flutter，近几年动态化和跨平台技术层出不穷，各个公司和团队也都积极的投身到其中。但是，目前动态化和跨平台的领域还没有最终的解决方案，各个框架面向的开发者、解决的问题、自身缺陷和瓶颈都很明显，比如 Web 的交互体验和稳定性、基于 DSL 方案的灵活性和扩展性、基于 JS 方案的性能和维护成本、Flutter 对混合工程的支持等等。




