---
title: "Pandoc generate pdf from markdown"
date: 2024-07-25T14:52:12+08:00
lang: zh-CN
mainfont: "PingFang SC"
monofont: "FantasqueSansM Nerd Font Mono"
geometry:
- margin=2cm
---


## Intro

始终不太习惯 word/wps 类工具编辑文本，再一次尝试用 `pandoc` 来生成 pdf，算是初步搞明白一点玩法，本文略过 markdown 的部分主要记录 `latex` 相关设定内容。

* `wkhtmltopdf` 也是一个选项，但是转 HTML 再生成 pdf 就没啥挑战了额哈哈

## Setup

首先完整安装 latex 包
```bash
$ brew uninstall --cask basictex
$ brew install --cask mactex
```

## Latex template

```yaml
---
title: "Your article title here"
date: 2023-02-20
fontsize: 12pt
# fc-list :lang=zh
mainfont: "PingFang SC"
monofont: "FantasqueSansM Nerd Font Mono"
# 中文换行支持
lang: zh-CN
geometry: margin=2cm

# 设置双栏
# classoption=twocolumn
---
```

在 markdown 文档的最前面加入以上 metadata 简化命令参数设定，实际效果等价于

```bash
$ pandoc --pdf-engine=xelatex -f gfm  -H head.latex \
  -V geometry:"top=2cm, bottom=2cm, left=2cm, right=2cm" \
  -V mainfont="PingFang SC" \
  input.md -o output.pdf
# simplify
$ pandoc --pdf-engine=xelatex -H head.tex -f gfm input.md -o output.pdf
```

```latex
% head.tex，使用文档内联的 header-includes 指令一直运行不成功，求指导~

\pagenumbering{gobble}
```


## Vim plugin

```vim
command! ToPDF call s:md_to_pdf()

function! s:md_to_pdf() abort
  " check if pandoc is installed
  if executable('pandoc') != 1
    echoerr "pandoc not found"
    return
  endif

  let l:md_path = expand("%:p")
  let l:pdf_path = fnamemodify(l:md_path, ":r") .. ".pdf"

  let l:cmd = "pandoc --pdf-engine=xelatex -f gfm " .
    \ "-H ~/.local/share/pandoc/head.tex '" .
    \ l:md_path . "' -o '" . l:pdf_path . "' && ql '" . l:pdf_path . "'"

  " echomsg l:cmd
  let l:id = jobstart(l:cmd)

  if l:id == 0 || l:id == -1
    echoerr "Error running command"
  endif
endfunction

```

[^manual]:  https://pandoc.org/MANUAL.html#creating-a-pdf
