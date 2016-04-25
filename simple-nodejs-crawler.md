---
title: simple crawler
date: 2015-10-17 12:06:21
tags: ['nodejs', 'crawler']
---

半夜看了本[小说][1]觉得翻页太累

首先要解决的问题是找一个质量还过得去的小说站
然后写个[脚本][2]去把它爬下来章节合并到一起

```javascript
import fs from "fs";
import { argv } from "process";
import request from "request";
import cheerio from "cheerio";
import iconv from "iconv-lite";
import sanitize from 'sanitize-html';


class main {
  constructor() {
    Object.assign(this, {
      path: './chapter.json',
      html: './reader.html',
      url: {
        list: 'http://www.piaotian.net/html/6/6658/'
      },
      store: []
    })
  }
  fetch(url, callback) {
    request.get(url, {encoding: null}, (error, response, body)=> {
      if (!error && response.statusCode == 200) {
        let $ = cheerio.load(iconv.decode(body, 'GBK'));
        callback($, body);
      } else {
        console.log(url);
      }
    })
  }
  runchapter() {
    this.store.map(d=> {
      if (!d.content) {
        this.fetch(this.url.list + d.href, ($, body)=> {
          let content = iconv.decode(body, 'GBK');
          d.content = sanitize(content);
          this.save();
        })
      } else {
        console.log('runchapter fail: ', d.id, d.title);
      }
    })
    this.output();
  }
  output() {
    //let content = this.store.slice(0, 2)
    let content = this.store
      .map(d=> d.content)
      .join('<hr />');
    let html = `<html>
    <head>
      <meta charset="utf8" />
      <style>
        ul, table, div {
          display: none;
        }
        hr {
          height: 1px;
          margin: 4rem 0;
        }
        body {
          padding: 0 20%;
          font:24px/1.5 'Songti Sc';
        }
      </style>
    </head>
    <body>${content}</body></html>`;
    fs.writeFileSync(this.html, html, 'utf8');
  }
  reload() {
    this.fetch(this.url.list, ($)=> {
      this.store = [];
      $('.mainbody .centent ul li a').map((id, d)=> {
        let el = $(d);
        let href = el.attr('href');
        if (!href.endsWith('.html')) {
          return
        }
        this.store.push({ id, href, title: el.text() })
      })
      this.save();
      this.runchapter();
    })
  }
  config(force){
    if (force) {
      this.reload();
    } esle {
      let text = fs.readFileSync(this.path, 'utf8').toString();
      this.store = JSON.parse(text);
      this.runchapter();
    }
  }
  save() {
    fs.writeFileSync(this.path, JSON.stringify(this.store), 'utf8');
  }
  init() {
    let force = argv[2] === '-f';
    console.log('run: ', argv[2], force);
    this.config(force);
  }
}

new main().init();
```

于是就水出来一篇日志啦

最近各种写 es6
都快忘了
python 怎么用
上周
花一天就
写2个脚本
得服老...

[1]: http://www.qidian.com/Book/3313422.aspx
[2]: https://github.com/yanyaoer/crawler
