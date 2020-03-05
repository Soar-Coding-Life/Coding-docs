# 尝试了一波 Git Book,在此处与docsify做个对比记录📝

### 1. 安装
*需要node环境*
```shell
#gitbook
npm i -g gitbook-cli
#docsify
npm i -g docsify-cli 
```
### 2. 初始化一个项目
```shell
#gitbook
gitbook init
#docsify
docsify init
```
会生成`README.md`和`SUMMARY.md`两个文件,一个是介绍项目的一个概述,一个是目录索引,都需要自己去编写
```shell
|---SUMMARY.md
|---README.md
├── 第一章
│   ├── 前言.md
│   ├── 001.md
│   ├── 002.md
│   ├── 003.md 
│   └── README.md #章节预览 写本章介绍啥的 当然也可以写总结顺序在`SUMMARY.md`调整即可
└── 第二章
    ├── 004.md
    ├── 005.md
    ├── xxx.md
    └── README.md
```

### 3. 组织目录结构
*SUMMARY.md*
```markdown
# 目录
* [介绍](README.md)
* [第一章](第一章/README.md)
	* [Hello World](第一章/helloworld.md)
	* [Dart基础语法](第一章/Dart基础语法.md)
	* [Dart字符串](第一章/Dart字符串.md)
	* [Dart数组和Map集合类型](第一章/Dart数组和Map集合类型.md)
	* [Dart函数](第一章/Dart函数.md)
* [第二章](第二章/README.md)
	* [Flutter是什么](第二章/Flutter是什么.md)
	* [Flutter文本样式等组件](第二章/Flutter文本样式等组件.md)
	* [Flutter图片和图标组件](第二章/Flutter图片和图标组件.md)
```

### 4. 本地web服务预览
```shell
#gitbook
gitbook serve 
#docsify
docsify serve
```
### 5. 编译打包
```shell
gitbook build
```
### 6. 部署到github
编译之后会生成一些文件,其中只有`_book`目录内的文件才是最终需要部署的,也就是需要在`github-page`里单独开一条分支出来来存放该目录的文件,当然也可以将`_book`里的内容放到`docs`目录下. 至于私有的一些部署,原理大同小异,只不过需要读取到`index.html`能够打开页面正常运行即可,关键在于读取`_book`目录下的内容.


### 7. 总结:
与`docsify`功能相类似,各有各的优缺点.
`docsify`可以在线编译不用提前把`md`转`html`,而`gitbook`发布之前必须先编译转成`html`文件
`gitbook`貌似不支持两级目录以上的分级,`docsify`最多可支持五级

### 参考文档
[docsify官方文档](https://docsify.js.org/#/zh-cn/)

