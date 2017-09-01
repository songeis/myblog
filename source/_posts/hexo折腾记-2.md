---
title: hexo折腾记-2
date: 2017-08-29 18:18:53
tags:
- hexo
categories:
- 博客搭建
description: hexo折腾记-2
---
昨天虽然把博客搭完了，但是效果却不理想，至少和本猿在挑选hexo主题模板预览中看到的不一样。所以今天就稍微改动一下。
<!-- more -->
# 1-切换theme样式
【预览的样式】http://notes.iissnan.com/

所以今天本猿决定，要把它改成像预览这样的带着侧边导航的。

修改next主题目录下的_config.yml文件中的,注意切换注释即可。

```
scheme: Pisces
//sliderbar
display: always
//然后再执行
hexo clean
hexo g
hexo d
```

如果你发现修改了但是没有效果，如果你发现修改后没有效果，可以手动删除项目目录下的db.json文件。

# 2-设置站点语言
改完了之后，本猿又发现标题什么的都是英文，不爽改成中文。
```
# Site
title: songeis
subtitle:
description:
author: songeis
language: zh-Hans
timezone:
```

# 3-修改个人头像,修改主题目录下的配置文件
```
#avatar:
avatar: /uploads/avatar.jpg
```

# 4-设置tags和分类
另外改版之后,我也希望我的的导航可以多显示点东西。注意从昨天到今天我们都没有在博客中上传图片，至于原因，明确的告诉你本猿还没学会!!!

首先我们先修改下next主题下的配置文件，切换几个menu开关
```
menu:
  home: / || home
  tags: /tags
  categories: /categories
  archives: /archives
```

然后我们发现点击的时候会有404的报错，这是什么原因呢？是因为博客没有找到hexo的tags文件，所以我们同样需要命令行执行
```
hexo new page tags
```

同理 categories也是一样的。
```
hexo new page categories
```

同时编辑对应目录下的index.md
```
title: All tags
date: 2017-08-29 10:24:26
type: "tags"//如果是分类就是categories
```

PS：另外最近公司开始大搞小程序，开发日志也会逐渐放上来方便大家填坑。

我不是很爱写东西，也不太能坚持写东西。但是说实话，与其东一榔头西一棒头的学习，不如脚踏实地的记录一个日志。读书也是一样的，读过却没有笔记的不算读，技术书又不是红楼梦，不太可能反反复复的读，更不会背，所以笔记才显得尤为重要。在记笔记的过程中去反思和回顾。

博学从来不是件值的夸耀的事情。东一榔头西一棒头也不是什么无可奈何的事情。

我对自己只有两点要求：
1- 坚持写，每天写，或多或少都行，哪怕没有知识点，记录一下心情也可以。重点不是你写了什么，而是你坚持。
2- 每周都去找一篇外国技术博客去读一读，翻译或是感想也记下来,如果能实践最好，读读也不是件坏事儿。
