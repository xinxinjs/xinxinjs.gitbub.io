---
title: 搭建博客，换了电脑之后踩坑记录
date: 2019-06-27 16:22:58
tags: 其他
---

搭建：

1. [全民博客时代的到来——20 分钟简要教程](https://www.jianshu.com/p/e99ed60390a8)
2. [Hexo博客的跨设备同步](https://www.jianshu.com/p/6fb0b287f950)


换了电脑之后 hexo 配置都没有了

### 思路：

新建一个分支 hexo，管理 hexo 配置和主题文件，原来的 master 分支继续存放生成的静态页面即要发布的内容

## 操作：

1. 新建 hexo 分支，把 hexo 的配置和主题文件复制进去，然后执行：git add ... git push ，push 到 hexo 分支
2. 生成并发布到 master 执行：hexo g，然后 hexo d

换电脑之后：

1. 先 clone 一下 hexo 分支的代码
2. 安裝 npm install hexo-cli -g
3. 安装 hexo npm install hexo
4. 安装其他依赖 npm install
5. 安装 hexo-deployer-git 执行命令：npm install hexo-deployer-git --save
6. 安装 hexo server 命令：npm install hexo-server
   npm install hexo-server --save
7. 生成添加密钥: 执行命令 ssh-keygen -t rsa -C "Github 的注册邮箱地址"，一路回车，待秘钥生成完毕，会得到两个文件 id_rsa 和 id_rsa.pub，用带格式的记事本打开 id_rsa.pub，Ctrl + a 复制里面的所有内容，然后进入https://github.com/settings/keys，添加密钥

踩坑记录：

- 执行 hexo s 启动本地服务，页面空白，有时候还一直转圈圈，网上百度，说是 hexo server 默认的端口号是 4000，这个端口号被占用了，依照网上的操作，hexo s -p 4001 切换端口号，还是没用

使用了主题，比如 next ,在根目录执行：git clone https://github.com/iissnan/hexo-theme-next themes/next，安装主题，否则不能生成静态目录

- 又发现另外一个待解决的问题： 如果使用的主题皮肤，主题皮肤是另外一个仓库的，有自己的 git 配置，不能被我的 git 仓库管理，解决方法：https://juejin.im/post/5c2e22fcf265da615d72c596