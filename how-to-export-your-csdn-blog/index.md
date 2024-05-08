# 批量导出CSDN博客至hexo的解决办法


最近利用gitpage+hexo搭建好自己的博客后，想将自己的之前的csdn博客批量迁移到自己新的hexo博客上，网上搜索了一下， 发现有人已经用python写了个工具，可以将博客导出为Markdown和HTML格式：[csdn-blog-export](https://github.com/gaocegege/csdn-blog-export)  。 

用法也很简单：`./main.py -u 你的CSDN用户名 -f markdown` 或者 `./main.py -u cecesjtu -f html`   

 然而脚本直接download下来发现不行，打开源码debug一下。将自己的**（博客主题需切回旧的主题“编程工作室”，我的“大白”主题失效）**  ，修改部分源码：修复获取博客页数的bug、去除正文中博文标题。 

终于拿到自己在csdn上的博客。但是此时导出的博文还不满足hexo博客的格式，于是用python写了个工具转换成满足hexo格式的.md文件。 

```markdown
 title: 批量导出CSDN博客至hexo的解决办法 

	date: 2018-03-31 23:34:06 

	tags: [hexo,csdn博客,博客迁移] 

	categories: ["经验总结"]

```



用法： 

1. 利用[导出脚本](https://github.com/Jordanzheng/csdn-blog-export/blob/master/main.py)同时导出`.md`格式和`.html`格式  
2. 利用[转换脚本](https://github.com/Jordanzheng/csdn-blog-export/blob/master/md2hexo.py) 转换导出的csdn博客为hexo格式的博客

特性：`Python`脚本从`.html`文件中提取出博文标题、博文创建时间、标签、分类，将它们插入对应的`.md`文件

转换后最终结果： 

``` markdown
title: 读《王二的经济学故事》

date: 2018-01-18  23:25:44

tags: [经济学, 理财]

category:      读书札记

前段时间看到一个微信公众号推荐一本通俗的经济读物《王二的经济学故事》，在亚马逊试读来一下，觉得很有趣，便从亚马逊下单买了kindle电子版。  

作者用生活中通俗易懂的故事来解释经济学，主要通过主人公王二来阐述解释中国主要的经济政策和经济活动。该书读起来一点也不晦涩难懂，十分有趣，读完后对中国一些常见的经济现象有一定的思考启发。

```

搞定：）代码已上传到github上，入口：[https://github.com/Jordanzheng/csdn-blog-export](https://github.com/Jordanzheng/csdn-blog-export)  






