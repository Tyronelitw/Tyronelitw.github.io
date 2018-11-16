---
title: hexo给文章添加分类和标签
date: 2018-04-02 15:05:51
categories:
- 教程
tags:
- 摘抄
- 文档
---
[本文为转转，原文链接](https://linlif.github.io/2017/05/27/Hexo%E4%BD%BF%E7%94%A8%E6%94%BB%E7%95%A5-%E6%B7%BB%E5%8A%A0%E5%88%86%E7%B1%BB%E5%8F%8A%E6%A0%87%E7%AD%BE/)

    本教程针对的是Mac环境下，nexT主题的文章分类和标签设置，其他主题也应该是类似的。添加成功后会在侧边栏或导航栏生成“分类”和“标签”这两个选项，看下图：

文章分类、标签

点击“分类”后的页面：

文章分类页

点击“标签”后的页面：

标签

ok，大概效果就是这样的，下面进入教程

1、创建“分类”选项
1.1 生成“分类”页并添加tpye属性
打开命令行，进入博客所在文件夹。执行命令

$ hexo new page categories
成功后会提示：

INFO  Created: ~/Documents/blog/source/categories/index.md
根据上面的路径，找到index.md这个文件，打开后默认内容是这样的：

---
title: 文章分类
date: 2017-05-27 13:47:40
---
添加type: "categories"到内容中，添加后是这样的：

---
title: 文章分类
date: 2017-05-27 13:47:40
type: "categories"
---
保存并关闭文件。

1.2 给文章添加“categories”属性
打开需要添加分类的文章，为其添加categories属性。下方的categories: web前端表示添加这篇文章到“web前端”这个分类。注意：hexo一篇文章只能属于一个分类，也就是说如果在“- web前端”下方添加“-xxx”，hexo不会产生两个分类，而是把分类嵌套（即该文章属于 “- web前端”下的 “-xxx ”分类）。

---
title: jQuery对表单的操作及更多应用
date: 2017-05-26 12:12:57
categories: 
- web前端
---
至此，成功给文章添加分类，点击首页的“分类”可以看到该分类下的所有文章。当然，只有添加了categories: xxx的文章才会被收录到首页的“分类”中。

2、创建“标签”选项
2.1 生成“标签”页并添加tpye属性
打开命令行，进入博客所在文件夹。执行命令

$ hexo new page tags
成功后会提示：

INFO  Created: ~/Documents/blog/source/tags/index.md
根据上面的路径，找到index.md这个文件，打开后默认内容是这样的：

---
title: 标签
date: 2017-05-27 14:22:08
---
添加type: "tags"到内容中，添加后是这样的：

---
title: 文章分类
date: 2017-05-27 13:47:40
type: "tags"
---
保存并关闭文件。

2.2 给文章添加“tags”属性
打开需要添加标签的文章，为其添加tags属性。下方的tags:下方的- jQuery - 表格
- 表单验证就是这篇文章的标签了

---
title: jQuery对表单的操作及更多应用
date: 2017-05-26 12:12:57
categories: 
- web前端
tags:
- jQuery
- 表格
- 表单验证
---
至此，成功给文章添加分类，点击首页的“标签”可以看到该标签下的所有文章。当然，只有添加了tags: xxx的文章才会被收录到首页的“标签”中。

细心的朋友可能已经发现，这两个的设置几乎一模一样！是的，没错，思路都是一样的。所以我们可以打开scaffolds/post.md文件，在tages:上面加入categories:,保存后，之后执行hexo new 文章名命令生成的文件，页面里就有categories:项了。

scaffolds目录下，是新建页面的模板，执行新建命令时，是根据这里的模板页来完成的，所以可以在这里根据你自己的需求添加一些默认值。

教程结束，赶紧去设置吧！
