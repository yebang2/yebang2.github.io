# 如何在Github上搭建自己的博客

要问全球最大的基佬交友网站是哪个，我相信不少答案必须是咱们的github。而github.io便是其出品，品质必须是有保证的，最重要的一点是基于github的repo管理，这意味着咱们对其是有觉得的控制，这个跟放在第三方的平台比，可控性要好太多。下面咱们将详细讲述如何基于github.io打造属于自己的博客网站。

要完成自己的github.io博客网站，总共分四步：

- 开通自己的github.io repo
- 修改配置
- 本地查看自己的博客
- 编写发布博客


# 一、开通自己github.io repo
- 创建repo

你需要有一个github的账号，在github上新建一个repo(仓库)。repo的名字叫做username.github.io其中
username替换成自己的github的用户名。

如果你不想自己创建github.io repo,可以推荐自己去fork[huxpro.github.io](https://github.com/Huxpro/huxpro.github.io)这个仓库
Fork好之后，repo名字修改成自己的用户名。举个栗子：我Fork了[huxpro.github.io](https://github.com/Huxpro/huxpro.github.io)这个仓库,Fork后repo的
名字是huxpro.github.io,我们可以在repo的Settings里面修改repo的名称

![settings](/img/in-post/2018-06-16-post-images-github-io/settings.png)

- 将创建的repo clone到本地

```
$ git clone https://github.com/username/username.github.io
```
其中username是自己github的用户名

打开博客网站https://username.github.io

不出意外，你就可以看到你的博客首页了。如果不小心出了意外，通常情况下，你只需等一会再刷新就会好。

# 修改配置

将repo clone到本地之后，对项目里面的_config.yml进行修改，改成自己需要的,详细的配置信息可以参考
[Hux blog 模板](https://github.com/Huxpro/huxpro.github.io/blob/master/README.zh.md)

```
# Site settings
title: JoouA Blog  #博客的标题
SEOTitle: JoouA的博客  
header-img: img/home-bg.jpg  # 博客首页的图片
email: tangwtna@163.com
description: "这里是 @JoouA的个人博客，与你一起发现更大的世界。"
keyword: "博客, 个人网站, 互联网, Web"
url: "jooua.top"       # 这个是你自己的域名，如果你没有域名就留空
baseurl: ""          #  默认为空 for example, '/blog' if your blog hosted on 'host/blog'

# Publish posts or collection documents with a future date.
future: true

# SNS settings
RSS: true
weibo_username:     1501344020  # 微博的的ID
zhihu_username:     snk-26      # 知乎的用户名 
github_username:    JoouA       # github的用户名
#twitter_username:   huxpro
#facebook_username:  huxpro
#linkedin_username:  firstname-lastname-idxxxx
```

# 本地查看自己的博客
首先你要在本地安装Jekyll,具体的安装步骤这边就不陈序了可以参考如下的文档[安装Jekyll](https://www.jekyll.com.cn/docs/installation/)
下的安装，还有Jekyll的各种用法。注意： 安装ruby建议安装最新的版本，不然会有意想不到的问题。

本地环境搭建还之后，在repo目录下执行
```
$ jekyll serve --watch
```
默认情况下，该服务会侦听在本地4000的端口上，可以打开浏览器访问http://127.0.0.1:4000，这样就可以在本地查看自己的博文效果了。

![content](/img/in-post/2018-06-16-post-images-github-io/content.png)


# 编写发布博客

Jekyll对于博文，都是要求放在_posts目录下面，同时对博文的文件名有严格的规定，必须保持格式YEAR-MONTH-DAY-title.MARKUP，
通常情况下，咱们采用推荐的Markdown撰写博文，基于该格式，本博文的文件名为2018-06-16-how-to-build-your-github-blog.markdown。

写好博文之后，就可以通过git提交博文了：
```
$ git status
$ git add -A
$ git commit -m "Add how to build github.io blog"
$ git push origin master
```












