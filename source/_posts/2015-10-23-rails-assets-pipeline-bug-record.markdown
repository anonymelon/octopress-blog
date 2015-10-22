---
layout: post
title: "Rails assets pipeline bug小记"
date: 2015-10-23 00:08:39 +0800
comments: true
categories: 
---

前段时间在用Rails部署生产环境时遇到一个诡异的情况，在生产环境的集群下面每台机器build出来的assets摘要（digest）后缀不一样，这导致了在loadbalance访问时随机出现assets 404 的情况。

经过debug后发现是由于一个第三方的前端类库（[angular-rails-templates][1]）在做assets pipeline的时候根据项目的path改了`Rails.application.config.assets.version`，这导致只要项目的path不一样算出来的asset version都不一样。而asset version是用来计算所有assets的digest后缀的key。

在我们生产集群环境中的每个project root path都不一样（根据自动化部署的时间），所以每台机器经过assets pipeline打出来的assets后缀也都不同了。（Rails assets pipeline是个坑？也许。。）

我们总是在挖坑填坑中积累经验，所以在debug期间研究了一下Rails做assets pipeline的过程，简单记录一下。

## Rails.application.assets.digest

在development环境下`Rails.application.assets`是一个`Sprockets::Environment`对象（关于[Sprockets][2]），但是在production环境下面，`Rails.application.assets`确是一个`Sprockets::Index`对象（虽然都继承自`Base`但需要继续研究为什么），里面几乎包含了Rails所需要的所有assets信息。虽然对象不一样，但是都有一个`digest`方法（来自`Sprockets::Base`），能够拿到一个经过SHA1或者MD5算过的digest。这个digest会用来计算之后所有assets的digest后缀。
而这个`Rails.application.assets.digest`是根据`Rails.application.config.assets.version`计算出来的。

##总结

废话了这么多，简单用个过程表示吧：

```
::Rails环境准备
    --> 计算Rails.application.config.assets.version
        --> 通过version计算Rails::application.assets.digest
            --> 通过digest计算assets digest后缀
```
在这期间还有很多复杂的过程，这里只是简单做过记录。



  [1]: https://github.com/pitr/angular-rails-templates
  [2]: https://github.com/sstephenson/sprockets
