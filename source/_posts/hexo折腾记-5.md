---
title: hexo折腾记-5
date: 2017-09-01 09:35:59
tags:
- hexo
categories:
- 博客搭建
description: 今天最主要的任务就是为自己的博客文章添上插图。

---
# 1- 七牛相关操作
1.1-首先你要注册七牛账户并完成实名认证。
1.2-对象存储中新建存储空间，因为是放blog文章中的插图的，所以本猿新建的空间名就叫blog，其中控制访问中，选择公有。然后记住你的 xxxx.bkt.clouddn.com 测试域名
1.3-上传文件，成功后到空间管理，文件列表中选择你要放在blog中的图片，在操作选择复制外链。
然后使用"!\[给图片起个名字\](url =100x100)"这样的语法在blog中插入一章图片，如我们的就是"\!\[炮姐镇楼\](http://ovkvsx7hv.bkt.clouddn.com/timg%20%281%29.jpg)"
![炮姐镇楼](http://ovkvsx7hv.bkt.clouddn.com/timg%20%281%29.jpg)

现在我们发现，图片虽然正常显示了，但是太大了。这个时候怎么办？
没关系，你可以插入一段HTML代码，在代码中设定宽高。
        
<img src="http://ovkvsx7hv.bkt.clouddn.com/timg%20%281%29.jpg" width=200 align=center />

这个时候你又说了这不行，这图不居中不好看.

没关系咱们再在外面裹个align=center的div呗!
<div align="center">    
<img src="http://ovkvsx7hv.bkt.clouddn.com/timg%20%281%29.jpg" width=200/>
</div>

好，在md文档里加上html标签也就算了，忍忍就过去了。但是每次添加图片还要到七牛控制面板，上传，然后复制外链，这也太不方便了吧。这一步能不能再简化点？

# 2-hexo-qiniu插件
https://github.com/gyk001/hexo-qiniu-sync

好，我们就按照文档，一步步来，听说很多人都遇到了坑，不过我们还是先试试吧。

2.1-安装
```
npm install hexo-qiniu-sync --save
```

2.2-修改项目_config.yml
```
#七牛云存储设置
##offline       是否离线. 离线状态将使用本地地址渲染
##sync          是否同步
##bucket        空间名称.
##access_key    上传密钥AccessKey
##secret_key    上传密钥SecretKey
##secret_file   秘钥文件路径，可以将上述两个属性配置到文件内，防止泄露，json格式。绝对路径相对路径均可
##dirPrefix     上传的资源子目录前缀.如设置，需与urlPrefix同步 
##urlPrefix     外链前缀.
##up_host      上传服务器路径,如选择华北区域的话配置为http://up-z1.qiniu.com
##local_dir     本地目录.
##update_exist  是否更新已经上传过的文件(仅文件大小不同或在上次上传后进行更新的才会重新上传)
##image/js/css  子参数folder为不同静态资源种类的目录名称，一般不需要改动
##image.extend  这是个特殊参数，用于生成缩略图或加水印等操作。具体请参考http://developer.qiniu.com/docs/v6/api/reference/fop/image/ 
##              可使用基本图片处理、高级图片处理、图片水印处理这3个接口。例如 ?imageView2/2/w/500 即生成宽度最多500px的缩略图
qiniu:
  offline: false
  sync: true
  bucket: bucket_name
  secret_file: sec/qn.json or C:
  access_key: AccessKey
  secret_key: SecretKey
  dirPrefix: static
  urlPrefix: http://bucket_name.qiniudn.com/static
  up_host: http://upload.qiniu.com
  local_dir: static
  update_exist: true
  image: 
    folder: images
    extend: 
  js:
    folder: js
  css:
    folder: css
```

到这儿有人会问了，access_key和secret_key在哪儿啊，我没有啊?
请登录七牛,到个人中心，秘钥管理中查找。

其实上面的参数我也不完全懂，不过先不管，先执行下命令试试
```
//在文档中加入{ % qnimg test/demo.png title:图片标题 alt:图片说明 ‘class:class1 class2’ % }然后执行命令
hexo qiniu s2
```

{ % qnimg static/images/sai.jpg title:看到没好了 alt:看到没好了 ‘class:class1 class2’ % }

{% qnimg sai.jpg title:图片标题 alt:图片说明 'class:class1 class2' extend:?imageView2/2/w/600 %}

执行完命令我们发现在项目目录生成了一个Static目录子目录为css，images，js
这个时候我们往images里存一张图片sai。然后再执行s2命令。
没有报错，同时也输出了下面的信息
```
INFO  -----------------------------------------------------------
INFO  qiniu state: online
INFO  qiniu sync:  true
INFO  qiniu local dir:  static
INFO  qiniu url:   http://ovkvsx7hv.bkt.clouddn.com/static
INFO  -----------------------------------------------------------

qiniu sync plugin for hexo

INFO  Now start qiniu sync.
INFO  Need upload file: D:\myblog\static\images\sai.jpg
INFO  blog static/images/sai.jpg D:\myblog\static\images\sai.jpg
```

那这个时候我们再去七牛看看，发现sai已经成功的上传了。

然后我们再把文档上传看看，文章中是否正确显示了这个图片。

事实上，第一次没有正确显示，我们按照路径查找回去发现多了一对static/images/

```
//http://xxxxxxxxx.bkt.clouddn.com/static/images/static/images/sai.jpg?imageView2/2/w/600

//更改前
{ % qnimg static/images/sai.jpg title:看到没好了 alt:看到没好了 ‘class:class1 class2’ % }

//更改后
{% qnimg sai.jpg title:图片标题 alt:图片说明 'class:class1 class2' extend:?imageView2/2/w/600 %}
```

既然如此，我们再找个图片试试。
{% qnimg bear.jpg title:图片标题 alt:图片说明 'class:class1 class2' extend:?imageView2/2/w/600 %}

{% qnimg 170901/bear.jpg title:图片标题 alt:图片说明 'class:class1 class2' extend:?imageView2/2/w/600 %}

当我们在images图片中建了时间日期分类的时候，我们再用图片就要多加一个时间日期目录
所以正确的调用方式为
```
{% qnimg 170901/bear.jpg title:图片标题 alt:图片说明 'class:class1 class2' extend:?imageView2/2/w/600 %}
```

另外命令hexo clean什么时候是成功的，就是命令行出现Deleted database的时候。
