---
title: 快速注释和取消注释
author: lijiabao
date: 2021-02-20 21:23:59.714998900 +0800
category: vim使用
categories: vim使用
tags: vim使用
---
## 快速注释

1.  按`ESC`进入命令行模式；
2.  `ctrl+v`进入块选择模式；
3.  操作`k`和`j`进行上下行选择；
4.  按大写`I`进入插入模式，输入注释符`//`或`#`(shell脚本注释符)；
5.  最后按下`ESC`即可完成注释。

## 取消注释

1.  按`ESC`进入命令行模式；
2.  `ctrl+v`进入块选择模式；
3.  操作`k`和`j`进行上下行选择,还可以操作`h`和`l`键进行左右控制；
4.  按`d`键删除注释符；
5.  最后按下`ESC`即可完成取消注释。


## 插件的方式

[nerdcommenter](https://github.com/preservim/nerdcommenter)
下载插件，放置到`～/.vim/pack/vendor/start`下
添加配置项到配置文件中
```
" Create default mappings
let g:NERDCreateDefaultMappings \= 1

" Add spaces after comment delimiters by default
let g:NERDSpaceDelims \= 1

" Use compact syntax for prettified multi-line comments
let g:NERDCompactSexyComs \= 1

" Align line-wise comment delimiters flush left instead of following code indentation
let g:NERDDefaultAlign \= 'left'

" Set a language to use its alternate delimiters by default
let g:NERDAltDelims\_java \= 1

" Add your own custom formats or override the defaults
let g:NERDCustomDelimiters \= { 'c': { 'left': '/\*\*','right': '\*/' } }

" Allow commenting and inverting empty lines (useful when commenting a region)
let g:NERDCommentEmptyLines \= 1

" Enable trimming of trailing whitespace when uncommenting
let g:NERDTrimTrailingWhitespace \= 1

" Enable NERDCommenterToggle to check all selected lines is commented or not 
let g:NERDToggleCheckAllLines \= 1
```

然后就可以使用带前缀的快捷键来进行快速注释/取消注释了
每个命令都支持前面带数字的多行注释/取消注释：一般是在visualmode下的按键设置
`3<leader>cc`：快速注释当前行及其之后的两行
	其中的leader是配置文件中设置的leader符号`let mapleader=","`
-   `[count]<leader>cc` **|NERDCommenterComment|**
    
    Comment out the current line or text selected in visual mode.
    
-   `[count]<leader>cn` **|NERDCommenterNested|**
    
    Same as cc but forces nesting.
    
-   `[count]<leader>c<space>` **|NERDCommenterToggle|**
    
    Toggles the comment state of the selected line(s). If the topmost selected line is commented, all selected lines are uncommented and vice versa.
    
-   `[count]<leader>cm` **|NERDCommenterMinimal|**
    
    Comments the given lines using only one set of multipart delimiters.
    
-   `[count]<leader>ci` **|NERDCommenterInvert|**
    
    Toggles the comment state of the selected line(s) individually.
    
-   `[count]<leader>cs` **|NERDCommenterSexy|**
    
    Comments out the selected lines with a pretty block formatted layout.
    
-   `[count]<leader>cy` **|NERDCommenterYank|**
    
    Same as cc except that the commented line(s) are yanked first.
    
-   `<leader>c$` **|NERDCommenterToEOL|**
    
    Comments the current line from the cursor to the end of line.
    
-   `<leader>cA` **|NERDCommenterAppend|**
    
    Adds comment delimiters to the end of line and goes into insert mode between them.
    
-   **|NERDCommenterInsert|**
    
    Adds comment delimiters at the current cursor position and inserts between. Disabled by default.
    
-   `<leader>ca` **|NERDCommenterAltDelims|**
    
    Switches to the alternative set of delimiters.
    
-   `[count]<leader>cl` **|NERDCommenterAlignLeft** `[count]<leader>cb` **|NERDCommenterAlignBoth**
    
    Same as **|NERDCommenterComment|** except that the delimiters are aligned down the left side (`<leader>cl`) or both sides (`<leader>cb`).
    
-   `[count]<leader>cu` **|NERDCommenterUncomment|**
    
    Uncomments the selected line(s).