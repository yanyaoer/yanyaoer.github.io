---
title: xtag-and-shadowdom
date: 2015-10-04 12:20:52
---

最近在做的项目重构, 原本打算用 reactjs, 写了一些实验代码后心累无爱
找了个 domdiff 配合自定义标签和 shadowdom 的独有作用域也是爽 YY

可惜的是测试红米上的 android webview 版本(32?)不支持 ::shadow 伪类
inline 方式覆盖样式略嫌繁琐

## 示例代码

### xtag.js

```javascript
let dom = {
  shadow(el) {
    return el.createShadowRoot ? el.createShadowRoot() : el.webkitCreateShadowRoot();
  },
  attr(el, prefix='') {
    return Object.keys(el.dataset).map((d)=> `${prefix}${d}="${el.dataset[d]}"`).join(' ')
  }
}

document.registerElement('x-image', {
  prototype: Object.create(HTMLElement.prototype, {
    createdCallback: {
      value() {
        //xtag 嵌套时这里读不到attr, 放到 attach
        console.log('onCreate::image');
      }
    },
    attachedCallback: {
      value() {
        console.log('onAttach::image');
        let shadow = dom.shadow(this);
        shadow.innerHTML = `<style>
            img { max-width: 100%; }
          </style>
          <img ${dom.attr(this)} />`;

        this.onclick = (e) => {
          console.log(this);
        }
      }
    }
  })
});

// other x-tag;
```

### index.js

```javascript
import 'babel/polyfill';
import './xtag.js';

class BaseView {
  constructor(opt) {
    Object.assign(this, {
      root: dom('#app')
    }, opt);
  }
  mount() {
    console.log('event::mount');
  }
  onCreate() {
    console.log('event::create');
  }
  onUpdate() {
    console.log('event::update');
  }
}

export default class BannerView extends BaseView {
  init(...args) {
    this.mount(...args);
  }
  render() {
    return `<x-image data-src="${this.src}"
                     data-href="${this.redirect_url}" />`;
  }
}
```

## links

<http://www.html5rocks.com/en/tutorials/webcomponents/customelements/>
<http://www.html5rocks.com/en/tutorials/webcomponents/shadowdom-301/>
