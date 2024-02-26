+++
title = 'Hugo网站 配置gitalk评论系统 流程及问题解决'
date = 2024-02-26T16:19:20+08:00
draft = false
description = "记录Hugo网站使用gitalk评论系统时遇到的问题，接口401和403，登陆网络错误等" 
tags = [ "建站", "Hugo", "GitHub","Gitalk"] 
categories = [ "建站"]
+++

本文的前提是使用的Hugo主题带有gitalk功能。

这里使用主题[pure](https://github.com/xiaoheiAh/hugo-theme-pure)作为例子，其他主题的配置方式大致都是一样的。

使用gitalk首先要[创建一个application](https://github.com/settings/applications/new)

![创建gitalk application](https://minio.xudashuai.blog/blog/20240226162904.jpg)

- Application name：随便写
- Homepage URL：博客首页地址 **最好是写https地址，http的我没试过**
- Authorization callback URL：博客首页地址 **最好是写https地址，http的我没试过**

创建后会得到一个`ClientID`并需要生成一个`ClientSecret`，记录这两个值！

然后看下gitalk必要的参数，[官方文档](https://github.com/gitalk/gitalk/blob/master/readme-cn.md)

- clientID
  - 类型：string
  - 上面👆已得到

- clientSecret
  - 类型：string
  - 上面👆已得到

- repo
  - 类型：string
  - gitalk所依赖的repository的名字，⚠️是名字，不是地址

- owner
  - 类型：string
  - 填GitHub账号用户名即可。

- admin
  - 类型：array
  - 填GitHub账号用户名即可。⚠️类型是array


同时要留意两个非必要参数：

- id：页面的唯一标识。长度必须小于50。默认是`location.href`
- proxy：GitHub oauth 请求到反向代理，为了支持 CORS。默认是`https://cors-anywhere.azm.workers.dev/https://github.com/login/oauth/access_token` ⚠️因为时间问题，你看到的也许默认不是这个，使用你看到的最新的值，别复制这里的。

接下来打开主题的配置文件（hugo.toml或者hugo.yaml），找到gitalk配置（pure主题默认给了以下配置）：

```yaml
  # Comment
  comment:
    type:  gitalk # type disqus/gitalk/valine 启用哪种评论系统
    gitalk: # gitalk. https://gitalk.github.io/
      owner:  #必须. GitHub repository 所有者，可以是个人或者组织。
      admin:   #必须. GitHub repository 的所有者和合作者 (对这个 repository 有写权限的用户)。
      repo:   #必须. GitHub repository.
      ClientID:  #必须. GitHub Application Client ID.
      ClientSecret:  #必须. GitHub Application Client Secret.
```

我们按照上面看到的gitalk字段要求，将值填入，得到以下配置（值为虚构）：

```yaml
  # Comment
  comment:
    type:  gitalk # type disqus/gitalk/valine 启用哪种评论系统
    gitalk: # gitalk. https://gitalk.github.io/
      owner: zhangsan #必须. GitHub repository 所有者，可以是个人或者组织。
      admin:  ['zhangsan'] #必须. GitHub repository 的所有者和合作者 (对这个 repository 有写权限的用户)。
      repo: hugoblog  #必须. GitHub repository.
      ClientID: 123123  #必须. GitHub Application Client ID.
      ClientSecret: 234234  #必须. GitHub Application Client Secret.
```

配置提交后，打开页面，底部会出现评论区域，同时会得到以下问题：

### issues 未初始化

遇到此问题，打开浏览器网络面板，查看`https://api.github.com/user`接口，是否有异常，返回401；

经查看pure主题源码，发现`themes/pure/layouts/partials/_script/comment.html`中关于gitalk是这么初始化的：

```javascript
<script type="text/javascript">
    var gitalk = new Gitalk({
        clientID: '{{- $comment.gitalk.ClientID }}',
        clientSecret: '{{- $comment.gitalk.ClientSecret }}',
        repo: '{{- $comment.gitalk.repo }}',
        owner: '{{- $comment.gitalk.owner }}',
        admin: ['{{- $comment.gitalk.admin }}'],
        id: md5(location.pathname),
        distractionFreeMode: true
    });
    gitalk.render('comments');
</script>
```

其中`admin`字段，作者在提交的时候，加了中括号，我们在配置页面配置的时候，配置了`['zhangsan']`出现冲突，改之。

### 登陆网络异常 接口403

打开浏览器网络面板，看看是不是有个`https://cors-anywhere.herokuapp.com/https://github.com/login/oauth/access_token`请求返回了403。

这是gitalk的proxy参数出了问题，默认使用了`https://cors-anywhere.herokuapp.com`，已经不能用了，我换成[官方文档](https://github.com/gitalk/gitalk/blob/master/readme-cn.md)中最新默认的proxy参数解决。

pure主题配置proxy参数流程，其他主题可参考

首先hugo.yaml中的gitalk部分是没有proxy配置选项的，我们加一个，值填写文档中最新的：

```yaml
  # Comment
  comment:
    type:  gitalk # type disqus/gitalk/valine 启用哪种评论系统
    gitalk: # gitalk. https://gitalk.github.io/
      owner:  #必须. GitHub repository 所有者，可以是个人或者组织。
      admin:   #必须. GitHub repository 的所有者和合作者 (对这个 repository 有写权限的用户)。
      repo:   #必须. GitHub repository.
      ClientID:  #必须. GitHub Application Client ID.
      ClientSecret:  #必须. GitHub Application Client Secret.
      proxy: https://cors-anywhere.azm.workers.dev/https://github.com/login/oauth/access_token
```

然后打开`themes/pure/layouts/partials/_script/comment.html`添加对应参数，取配置文件中的proxy：

```javascript
<script type="text/javascript">
    var gitalk = new Gitalk({
        clientID: '{{- $comment.gitalk.ClientID }}',
        clientSecret: '{{- $comment.gitalk.ClientSecret }}',
        repo: '{{- $comment.gitalk.repo }}',
        owner: '{{- $comment.gitalk.owner }}',
        admin: ['{{- $comment.gitalk.admin }}'],
        id: md5(location.pathname),
        distractionFreeMode: true,
        proxy: '{{- $comment.gitalk.proxy }}'
    });
    gitalk.render('comments');
</script>
```

### 网络err not found

此问题有可能涉及到gitalk id参数，具体我没验证，可试试

[官方文档](https://github.com/gitalk/gitalk/blob/master/readme-cn.md)中提到id默认是`location.href`，我们在自己的项目中改成`location.pathname`。

