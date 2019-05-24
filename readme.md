# Hexo 博客 next 主题应用

## 常用
docs: https://hexo.io/zh-cn/docs/writing

### Hexo 命令
`hexo init <folder>` 新建一个网站。
`hexo new <layout> "title" | hexo n` 新建文章或页面。
`hexo generate | hexo g` 生成静态页面
`hexo deploy | hexo d` 将内容部署到网站，部署到github需先`npm install hexo-deployer-git --save`
`hexo publish <layout> <filename>` 发布内容，实际上是将内容从drafts（草稿）文件夹移到posts（文章）文件夹。
`hexo server | hexo s` 启动服务器，默认情况下，访问网站为http://localhost:4000/
`hexo new page categories` 创建“分类”页面
添加分类，添加标签。请看： https://www.jianshu.com/p/21c94eb7bcd1

## 安装Hexo
安装Node.js和Git
通过npm来安装Hexo 
```
npm install -g hexo-cli
```

## 建站
```
hexo init <folder>
cd <folder>
npm install
```

## 目录

**config.yml**
博客的配置文件，博客的名称、关键词、作者、语言、博客主题...设置都在里面。
**package.json**
应用程序信息，新添加的插件内容也会出现在这里面，我们可以不修改这里的内容。
**scaffolds**
scaffolds就是脚手架的意思，这里放了三个模板文件，分别是新添加博客文章（posts）、新添加博客页（page）和新添加草稿（draft）的目标样式。
这部分可以修改的内容是，我们可以在模板上添加比如categories等自定义内容
**source**
source是放置我们博客内容的地方，里面初始只有两个文件夹，一个是drafts（草稿），一个posts（文章），但之后我们通过命令新建tags（标签）还有categories（分类）页后，这里会相应地增加文件夹。
**themes**
放置主题文件包的地方。Hexo会根据这个文件来生成静态页面。
初始状态下只有landscape一个文件夹，后续我们可以添加自己喜欢的。



