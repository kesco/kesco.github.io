---
title: 用Vimwiki写笔记
date: 2015-06-01T22:13:04+08:00
categories: [生活, 折腾]
tags: [无用功, Vim]
---

![Wiki-Index](http://7mnom1.com1.z0.glb.clouddn.com/vimwiki-index-page.png)

以前试过很多种笔记应用，如EverNote、为知和OneNote等等，始终用得不习惯。比如说OneNote和EverNote在Mac上无法很好的显示代码(马克飞象不算)，为知的App在我自己MBP上不稳定，运行的像个Demo一样，老是Crush。接着，因为公司的API文档用Gitbook写的，自己也因此用上Gitbook来写笔记。本以为这样就可以了，谁知道Gitbook后来不开发PC端Client了，只保留Web端的，所以最后我转到了[Vimwiki][0]上。

而且[Vimwiki][0]依赖于Vim，恰好解决了编辑器的问题，我本来就是Vim党^-^。通过私有Git仓库，同步笔记，笔记是纯文本的，转换也比较方便。

## 安装和配置Vim

Vimwiki使用Vim的第三方插件管理器非常方便，我平常使用Vim-plug来管理插件，感觉比Vundle好用点(UI做得漂亮点):

```viml
Plug 'vimwiki/vimwiki'          " VimWiki
```

安装完后，要对Vimwiki设置下:

```viml
let g:vimwiki_menu = ''         " 不在菜单栏上显示Vimwiki
let g:vimwiki_use_mouse = 1     " 使用鼠标
let g:vimwiki_diary_months = {
    \ 1: '一月', 2: '二月', 3: '三月', 4: '四月', 5: '五月', 6: '六月',
    \ 7: '七月', 8: '八月', 9: '九月', 10: '十月', 11: '十一月', 12: '十二月'
    \ }                         " 设置日期显示文字
autocmd FileType vimwiki setlocal wrap " 折行
let g:vimwiki_valid_html_tags = 'b,i,s,u,sub,sup,kbd,br,hr,img' " 设置可以在笔记中使用的Html Tag
let develop_notes = {}          " 个人开发笔记
let develop_notes.path = '~/Documents/note/develop-notes'                       " 笔记文件路径
let develop_notes.path_html = '~/Documents/note/develop-notes/output/'          " 笔记转换为HTML输出路径
let develop_notes.template_path = '~/Documents/note/develop-notes/template/'    " 用于生成HTML页面的模板
let develop_notes.template_default = 'kesco.tpl'                                " 默认模板
let develop_notes.nested_syntaxes = {'python': 'python', 'c++': 'cpp', 'java': 'java', 'sh': 'sh',
    \ 'viml': 'vim', 'xml': 'xml'
    \ }                                                                         " 启用的代码语法高亮
let g:vimwiki_list = [develop_notes]                                            " 笔记列表
nmap <leader>wc :VimwikiAll2HTML<CR>
```

要注意的一点是Vimwiki自带的模板比较简单，所以如果想要漂亮点的样式的话，要自己写，上图就是我用Pure CSS渲染的笔记目录。遗憾的是，[Vimwiki][0]人气不高，我在Github上没找到什么好的样式，都是好几年前的了，不知道Emacs的Org-mode在这方面怎么样。

## 使用[Vimwiki][0]

[Vimwiki][0]文稿有自己一套标记语言，类似于Markdown。[Vimwiki][0]也支持Markdown，但是效果不好，有语法高亮不支持种种问题。下图就是用Vimwiki语法写的笔记。

![Vimwiki-syntax](http://7mnom1.com1.z0.glb.clouddn.com/vimwiki-sample.png)

### 基本语法

```
= 一级标题 =  
== 二级标题 ==
=== 三级标题 ===
此次类推。

当标题前面有空白时，标题文本居中对齐。
       = 我是居中的标题 =

*粗体*  _斜体_  ~~删除线~~   `Some Code 代码` 

注意 这几个针对文本格式的标签，都要求左右留有空白。
请注意你的代码高亮，一般来说，有了相应的高亮，你用的wiki标签才生效。

^上^标  ,,下,,标

    四个空格缩进的内容会被转成blockquote
    
\{\{\{ class="brush:php"
这中间的内容会被放到一个 pre 里，适合贴代码。
上面的 class 是可选的，一般用来安排代码高亮。
事实上，这一块代码展示就是放在了一个 pre 里。
\}\}\}

WikiItem  大写开头的驼峰英文会被自动当作一个维基词条，并添加链接
[[Wiki Item]]  这是手动建立维基词条的方式
[[wiki item|description]]  输出HTML时显示description，链到 wiki item
http://ktmud.com/  外部URL会被自动转换成链接
[http://ktmud.com Ktmud]  带文字的外链
[images/hello.jpg] 输出 <img src="images/hello.jpg" />
[[images/hello.jpg]] 输出图片，并链向图片地址

* 无序列表 条目一
* 无序列表 条目二 
  - 子列表 条目一
  - 自列表 条目二

# 有序列表 条目一
# 有序列表 条目二

* 和 - 是等价的，后面必须跟一个空格
```

另外，你也可以Vim的Helper里查看，`:h vimwiki-syntax`。

可以看到，Vimwiki自带的模板功能和语法上都比Markdown弱一点的，而且Vimwiki这两年都没有更新了，不过作为笔记工具来说已经够了，而且自己本身是Vimer，叫我用其它编辑器都不习惯，于是就一直用Vimwiki了。

[0]: https://github.com/vimwiki/vimwiki
