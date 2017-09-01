---
title: hexo折腾记-1
date: 2017-08-28 17:01:43
tags:
- hexo
categories:
- 博客搭建
description: hexo折腾记-1
---
之前一直用github存md文档做日记，现在想来不如直接搭个hexo博客。既美观又能能学点东西。所以今天就学着搭建一下
<!-- more -->
# 0-环境准备
0.1-安装node
0.2-安装git
0.3-github新建一个xxx.github.io的页面
0.4-hexo操作
```
npm install hexo -g
//进入项目目录 执行初始化操作
hexo init
//安装依赖包
npm install 
//生成静态页面
hexo g
//启动本地服务
hexo s

//如果一切正确的话，按照提示输入localhost:4000页面会有一个页面展示出来

//如果你经常使用github想必公钥这些早就已经配好了。
//然后在_config.yml中配置如下代码：
deploy:
  type: git
  repository: git@github.com:xxx/xxx.github.io.git
  branch: master
//执行命令
hexo d
```

如果出现:deployer not found错误，执行下面命令再重新执行d命令

```
 npm install hexo-deployer-git --save 
```

另外对于把博客当成日志来写的猿们，一定要熟记下面的命令(及命令缩写)
```
hexo new "postName" #新建文章
hexo new page "pageName" #新建页面
hexo generate #生成静态页面至public目录
hexo server #开启预览访问端口（默认端口4000，'ctrl + c'关闭server）
hexo deploy #将.deploy目录部署到指定空间
hexo help # 查看帮助
hexo version #查看Hexo的版本
```

# 1-替换hexo的主题（本猿选的是经典主题NexT）
1.1-进入项目目录
```
git clone https://github.com/iissnan/hexo-theme-next themes/next
```

但是这里我们就发现再hexo deploy就没什么用了

1.2-备份项目目录
最好的方式是我们的项目目录也有个git仓库,这样随时更改，随时提交
```
//切换到项目目录
git init
git add .
git commit -m "xx"
git remote add git@github.com:xxx/blog.git
git push origin master
```

这一步，只是备份了我们的项目目录。但还不会更新github.io的样式

1.3-更新github目录,项目目录下执行
```
hexo clean
hexo g
hexo d
```

# 2-如何删除一篇文章
当我们进入url xxx.github.io的时候，默认的第一篇blog是"hello world"，这不是本猿想要的，本猿真正的第一篇blog应该是当下写的这篇。

我们找到source/_post/下的日期目录，这个目录就是我们所有的文章，删除文章
```
git rm blog.md hello-world.md
```

然后回到项目根目录,执行命令
```
hexo clean
hexo g
hexo d
```

# 3-新建一篇博客
```
hexo new "使用hexo和github搭建博客-1"
//打开source/下对应的md文件
title: 使用hexo和github搭建博客-1
date: 2017-08-28 17:01:43
tags:
- hexo
```
然后把文档粘进去就可以了，这就是本猿第一篇blog啦！