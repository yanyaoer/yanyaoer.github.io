---
title: brick intro
date: 2013-08-28 10:47:17
---

#Introducing Brick: Minimal-markup Web Components for Faster App Development
#介绍 brick: 用于快速开发 webapp 的自定义标签组件 


<https://hacks.mozilla.org/2013/08/introducing-brick-minimal-markup-web-components-for-faster-app-development/>


Those of you on the cutting HTML5 edge may have already heard of the exciting Web Components specification. If you haven’t, you’ll probably want to read up on what makes this so exciting, but long story short, Web Components promise to open up a new realm of development by letting web developers write custom, reusable HTML tags. Think of them as JavaScript plugins without the need for additional code initialization or boilerplate markup/styling.


也许你曾听说过这个令人激动的 web 组件。要是没有的话，[也许该看看是什么让它如此振奋人心](http://techcrunch.com/2013/05/19/google-believes-web-components-are-the-future-of-web-development/) 长话短说，web 组件开启了一个的新领域，让开发者自定义可重用的 HTML 标签。不需要多余代码来初始化的 javascript 插件和模板，样式。


Unfortunately, it will be a while before we see native browser support for the spec, but that doesn’t mean developers can’t start taking advantage of the component concept now, thanks to Google’s Polymer framework and Mozilla’s x-tags polyfill library (both X-Tag and Polymer share the same low-level, Web Component polyfills).


遗憾的是并非所有浏览器都原生支持该规范，不过这并不意味着开发者现在无法享受组件概念的好处，感谢 [google's Polymer](http://www.polymer-project.org/)和 [Mozilla's x-tag](https://hacks.mozilla.org/2013/05/speed-up-app-development-with-x-tag-and-web-components/) （X-Tag 和 Polymer 共享同样的底层，web 组件）


We’re proud to announce the beta release of Brick, a cross-browser library that provides new custom HTML tags to abstract away common user interface patterns into easy-to-use, flexible, and semantic Web Components. Built on Mozilla’s x-tags library, Brick allows you to plug simple HTML tags into your markup to implement widgets like sliders or datepickers, speeding up development by saving you from having to initially think about the under-the-hood HTML/CSS/JavaScript.


我们自豪的宣布释出测试版  Brick，一个跨浏览器，易用，灵活，语意化的 web 组件库，提供新的自定义 HTML 标签抽象成常见的用户界面模式。基于 Mozilla 的 [x-tags](http://www.x-tags.org/) Brick 能让你做到插入简单的 HTML 标签即可调用控件，比如滑动条或日期选择器。减少你花在思考如何关联 HTML/CSS/Javscript 上的时间以加速开发过程。


##Putting Brick into Action

##把 Brick 用起来

Say that you wanted to implement a cross-browser, mobile-friendly calendar widget in your application. With current JavaScript plug-ins, such as jQuery UI, this would require putting boilerplate, non-semantic markup into your HTML, as well as explicitly initializing and managing it through JavaScript. However, with Brick, you can implement such a component simply by adding a custom HTML tag that you can treat as a normal native tag.


想要在应用里实现一个跨浏览器，移动设备友好的日历控件。目前的 JavaScript 插件，比如 jQuery UI 需要插入无关语意的标记到 HTML 并以 JavaScript 明确的初始化和管理。然而有了 Brick 只需要简单的添加一些有别于原生标签的自定义 HTML 标签便能实现那些组件了。


For our calendar example, this means just including the library’s CSS and Javascript file in your application, then adding the following tag to your markup:


在我们的日历示例中，只需要在应用中引用了库里的 CSS 和 Javascript 文件，再添加如下标签


`<x-calendar></x-calendar>`


which creates a DOM element that looks like this:


就创建了一个这样 DOM 元素


![x-calendar](https://hacks.mozilla.org/wp-content/uploads/2013/08/x-calendar.png)


Want to edit how the component behaves, such as by adding navigational controls or pre-selecting a date? Like any other native tag, you can change how a component behaves just by changing the attributes of the tag!


想要修改组件行为比如添加导航控制或日期预选？就像其他的原生标签一下，只要修改标签属性就能改变组件行为！


`<x-calendar controls chosen='2012-05-17'></x-calendar>`

![x-calendar-attr](https://hacks.mozilla.org/wp-content/uploads/2013/08/x-calendar-attr.png)


##Available Bricks

##可用的 Bricks


At the time of writing, Brick consists of thirteen different tags, most of which are completely independent of one another, and can even be downloaded separately instead of a single bundle.

在写这片文章的时候，Brick 由 13 个不同标签构成，大部分都是与其他标签完全独立的，你可以单独下载其中的部分代替整个包


Some tags abstract away complex widgets into simple HTML tags, such as:


一些标签被抽象成了复合控件，比如：


[\<x-calendar\>](http://mozilla.github.io/brick/demos/calendar/index.html) (calendars, as seen from the example)

[\<x-deck\>](http://mozilla.github.io/brick/demos/deck/index.html) (a cyclable slide gallery)

[\<x-tooltip\>](http://mozilla.github.io/brick/demos/tooltip/index.html) (exactly as it sounds).


Others are cross-browser polyfill implementations of existing native not-yet-globally-supported elements, such as:


其他依靠修复跨浏览器实现的非全局原生支持元素，例如


[\<x-slider\>](http://mozilla.github.io/brick/demos/slider/index.html)

[\<x-datepicker\>](http://mozilla.github.io/brick/demos/datepicker/index.html)


which polyfill \<input type="range"\> and \<input type="date"\>, respectively. Still others are structural components simplifying the styling and markup of certain components, such as \<x-layout\>, which ensures that content, headers, and footers can fill a container element without explicit styling markup.


都用来支持 \<input type="range"\> 和 \<input type="date"\>。还有一些结构组件用于简化样式和标记，例如 \<x-layout\> 保证内容和头尾不需要样式化的标记也能填充到容器元素中。


Each tag comes with a flexible attribute/JavaScript API and can be fully styled to match your application.

每个标签都有一些扩展属性/JavaScript 接口，样式也完全可以符合应用的风格


##Start Building with Bricks

##用 Bricks 开始创造

Want to start using components in your own applications? Head to <mozilla.github.io/brick> to download a release bundles, view demos, and read the documentation for the available tags. Alternatively, visit the [Brick Github page](https://github.com/mozilla/brick/) to view the source code and contribute to the effort!


想在你的应用中开始用组件，前去 <mozilla.github.io/brick> 下载发行包，查看示例，阅读可用标签的文档吧。也可以选择浏览 [Brick Github 页面](https://github.com/mozilla/brick/) 来查看源代码或作出贡献！


The library is still in a beta release, so we appreciate all user feedback! Brick is already [starting to crop up in the wild](https://developer.mozilla.org/en-US/docs/Web/Apps/app_layout/responsive_design_building_blocks), so we’d love to hear about how you’re using it!


这个库还处于测试版本， 所以我们非常重视所有的用户反馈！Brick 已经有了一些[实例](https://developer.mozilla.org/en-US/docs/Web/Apps/app_layout/responsive_design_building_blocks)
，而我们很乐于看到你是如何使用它的！
