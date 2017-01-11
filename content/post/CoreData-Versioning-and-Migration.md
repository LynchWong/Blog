---
title: "CoreData: Versioning and Migration"
date: 2015-07-24 20:10:27
categories: 
- iOS
tags: 
- CoreData
metaAlignment: center
autoThumbnailImage: no
coverImage: //d1u9biwaxjngwg.cloudfront.net/welcome-to-tranquilpeak/city.jpg

---

什么时候迁移是必须的？早期的答案就是当你需要改变数据模型的时候。
<!--more-->

然而有些情况你也可以避免迁移。比如你使用CoreData只是用作离线缓存，当你更新你的程序的时候，你完全可以删除存储的数据。在其他情况下，你都需要安全的保护你用户的数据。

有时候大部分程序设计或者功能需求都离不开要修改数据模型，所以你需要创建新版本的数据模型而且提供迁移的途径。

To be continue...

# A lightweight migration
# A manual migration
# A complex mapping model
# Migrating non-sequential versions
