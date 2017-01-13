---
title: webpack resolve local module
date: 2017-01-13 23:19:49
---

最近的项目刚开始，设计的目录层级有点深  
经常会在好几层本地路径之间互相引用

`import Image from '../../../../components/image'`

这层层叠叠的路径写起来实在丑陋  
不由得想起 Python 从项目根目录引用模块  
然后研究了一下 Node.js 里的几种简易实现

干脆利落的软连接: `ln -s node_modules src`

修改环境变量: `NODE_PATH=. node app`

从本地目录安装:
```javascript
// package.json 
// 需要运行 npm install
{
  "name": "baz",
  "dependencies": {
    "foo": "file: ./src",
  }
}
```

另外还有些修改 global，或者引入其他 require 实现的方法就不再一一列出了  
最终选择的是修改 webpack 配置

```
// webpack.config.js
resolve: {
  modulesDirectories: [__dirname, 'node_modules'],
}
```

<https://gist.github.com/branneman/8048520>
<http://stackoverflow.com/questions/10860244/how-to-make-the-require-in-node-js-to-be-always-relative-to-the-root-folder-of-t/41078266#41078266>
<https://webpack.github.io/docs/configuration.html#resolve-modulesdirectories>
