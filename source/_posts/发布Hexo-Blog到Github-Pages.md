---
title: 发布Hexo Blog到Github Pages
date: 2018-06-22 14:01:40
tags:
toc: true
description: While there is life,there is hope. 
---
## 准备环境：
- 安装[Git](https://git-scm.com/)
- 安装[Node.js](https://nodejs.org/)
- 安装[hexo](https://hexo.io/zh-cn/docs/index.html)
利用npm命令安装
```
npm install -g hexo-cli
```
问题

- npm ERR! registry error parsing json 错误
可能需要设置npm代理，执行命令
```
npm config set registry http://registry.npmjs.org/
```
- hexo:command not found
删除刚刚安装的npm目录，重新执行命令
```
npm install -h hexo
```
## 创建hexo文件夹
执行命令，hexo会自动在目标文件夹建立博客网站所需的所有文件
```
hexo init
```
## 安装依赖包
```
npm install
```
## 本地查看
在hexo文件夹执行以下命令，然后到浏览器输入```http://localhost:4000```查看
```
hexo generate
hexo server
```
问题

- WARN No layout: index.html?...
查看主题目录是否为空，如果为空下载主题
```
git clone https://github.com/hexojs/hexo-theme-landscape.git themes/landscape
```
- npm WARN optional SKIPPING OPTIONAL DEPENDENCY: fsevents@^1.0.0 (node_modules\chokidar\node_modules\fsevents):
npm WARN notsup SKIPPING OPTIONAL DEPENDENCY: Unsupported platform for [fsevents@1.0.15](mailto:fsevents@1.0.15): wanted {“os”:”darwin”,”arch”:”any”} (current: {“os”:”win32”,”arch”:”x64”})
官网给出的方案solution:you are both experiencing a warning that is perfectly normal and will not cause any issues for development. When using OS X there’s a nice filesystem feature provided by the OS by which file changes emit events, making “watching” files for changes the reverse, where they’re passively “listened” for (Change Detection vs an Event Emitter if you need an analogy).
This is made possible by fsevents, a package that is only available for OS X and macOS installations due to dependence on the OS’s functionality. Windows and *nix will all see this warning. I haven’t tested it, but the only non-proprietary OS that might have support would be the Darwin open source project.
所以这个警告信息可以忽略
## 创建页面仓库
地址：[https://github.com/](https://github.com/)
这个仓库的名字需要和你的账号对应，格式: yourname.github.io
问题
- 生成SSH密钥
```
ssh-keygen -t rsa -C "你的邮箱地址"
```
## hexo部署使用
编辑_config.yml文件
```
deploy:
     type: git
     repo: git@github.com:yourname/yourname.github.io.git
     branch: master
```
- 配置文件的冒号“:”后面有一个空格
repo: 刚刚 GitHub 创库地址.git
- 部署步骤
```
  hexo clean
  hexo generate
  hexo deploy
```
问题
- ERROR Deployer not found: git
```
 npm install hexo-deployer-git --save
```
---------------------------------------------------------------------------------
## hexo常用命令使用
```
hexo help #查看帮助
hexo init #初始化一个目录
hexo new "postName" #新建文章
hexo new page "pageName" #新建页面
hexo generate #生成网页，可以在 public 目录查看整个网站的文件
hexo server #本地预览，'Ctrl+C'关闭
hexo deploy #部署.deploy目录
hexo clean #清除缓存，**强烈建议每次执行命令前先清理缓存，每次部署前先删除 .deploy 文件夹**
```
简写
```
hexo n == hexo new
hexo g == hexo generate
hexo s == hexo server
hexo d == hexo deploy
```
## 编辑文章
```
hexo new "标题"
```
在 _posts 目录下会生成文件标题.md：
```
title: Hello World
date: 2015-07-30 07:56:29 #发表日期，一般不改动
categories: hexo #文章文类
tags: [hexo,github] #文章标签，多于一项时用这种格式
---
正文，使用 Markdown 语法书写
```
编辑完后保存，hexo server 预览
## hexo 部署
```
  hexo clean
  hexo generate
  hexo deploy
```
## hexo目录结构
```
├── .deploy       #需要部署的文件
├── node_modules  #Hexo插件
├── public        #生成的静态网页文件
├── scaffolds     #模板
├── source        #博客正文和其他源文件，404、favicon、CNAME 都应该放在这里
|   ├── _drafts   #草稿
|   └── _posts    #文章
├── themes        #主题
├── _config.yml   #全局配置文件
└── package.json
```
参考连接： [http://wuxiaolong.me/2015/07/31/build-blog-by-hexo/](http://wuxiaolong.me/2015/07/31/build-blog-by-hexo/ "手把手教你建github技术博客by hexo")

