---
title: jekyll grammar
author: lijiabao
categories: Blog
tags: Blog
---


#### 语法规则

`{{ page.title }}`输出预先定义好的变量，作为页面的content内容，示例展示的page.title变量的值

`{% control/logic flow%}`这个标签包裹的是逻辑流或者控制结构，下面是两个示例：
```
{% if page.show_sidebar %}
	<div class="sidebar">
		sidebar content
	</div>
{% endif %}

{% for url in site.urls %}

{% endfor %}
```

##### Filter

`{{ "hi" | capitalize }}`展示大写的HI而不是小写的
`{{ "/assets/style.css" | relative_url}}`使用相对路径，抹除掉`baseurl`变量值，在网站布置在子域名下的时候很有用
`{{ "/assets/style.css" | absolute_url`展示绝对路径
`{{ site.members | group_by:"graduation_year" }}`根据特定的内容进行分组，还可以用正则匹配，`group_by_exp`
`{{ site.members | find/find_exp:"item","item.graduation_year < 2014" }}`查找命令

##### use liquid

在page的开头加上：Front Matter
```
---
# 加上这个是为了告诉jekyll处理liquid
mynumber: 5
# 可以使用page.mynumber来访问这个变量
---

```

##### Layout
