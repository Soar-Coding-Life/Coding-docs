# pod相关知识点 

## 1.安装和卸载`cocoapods`
##### 安装
```ruby
sudo gem install cocoapods
#建立本地索引 
pod setup 
# 1.8.0以后支持CDN 大可不必如此 只需在Podfile文件头部加上一句 source 'https://cdn.cocoapods.org'
source 'https://cdn.cocoapods.org'
```
##### 卸载
```ruby
sudo gem list --local | grep cocoapods
sudo gem uninstall cocoapods
sudo gem uninstall cocoapods-core
sudo gem uninstall cocoapods-downloader
sudo gem uninstall cocoapods-plugins
sudo gem uninstall cocoapods-search
sudo gem uninstall cocoapods-stats
sudo gem uninstall cocoapods-trunk
sudo gem uninstall cocoapods-try
sudo gem uninstall cocoapods-deintegrate
```

## 2.常规问题解决思路
##### 50%报错问题可以通过 `pod install`或者`pod update`解决
```ruby
pod install #有时可能需要删除pods或者Podfile.lock 协作开发的时候最好是使用统一版本配置 避免删除Podfile.lock来解决问题
或者
pod update
或者
pod repo update
或者
pod install --repo-update
``` 
##### 指定`swift`编译版本
```ruby
post_install do |installer|
    installer.pods_project.targets.each do |target|
        target.build_configurations.each do |configuration|
            configuration.build_settings['SWIFT_VERSION'] = "3.0"
            # Objective-C 跟 Swift 混编的项目, 想要引入 OC 的第三方库的话, 还需要添加另一项参数
            configuration.build_settings['ALWAYS_EMBED_SWIFT_STANDARD_LIBRARIES'] = 'NO'
        end
    end
end
```
##### 由于墙的原因，可能会`install`失败,`Gem`换淘宝的源
```ruby
gem sources -a https://ruby.taobao.org/ #替换源
gem sources -l #查看源
```
##### `BitCode`一般来说是选择关闭的,如果有类似报错,pod也可以加上这一设置,排查问题
```ruby
post_install do |installer| 
  installer.pods_project.targets.each do |target| 
    target.build_configurations.each do |config| 
      config.build_settings['ENABLE_BITCODE'] = 'NO' 
    end 
  end 
end 
```
## 3.添加`Podfile`
 **在项目根目录下`pod init`会生成一个`Podfile`文件此时`vim Podfile`进入编辑模式编辑内容**
```ruby
platform :ios, '9.0'
use_frameworks!

target "DemoProj" do
#私有库链接
source 'git@gitblit.local.com:XXX/iOSLibs.git'
#公有库源 1.8.0之后支持CDN 本来应该默认是source 'https://github.com/CocoaPods/Specs.git' 可省略不写
source 'https://cdn.cocoapods.org'
#默认是使用最新版本 
pod 'YYModel'
# 指定版本号避免升级不兼容api 
pod 'ReactiveObjC','~>3.1.1'
# 指定链接地址 可能是fork的库
pod 'YYText' , :git => 'https://github.com/ibireme/YYText.git'
#本地库
pod 'WGBSelectPhotoView',:path=> './LocalLibs/WGBSelectPhotoView' 
end
```

## 4.创建自己组件
1. 克隆远程库
```shell
git clone git@gitblit.local.com:XXX/iOSLibs.git
```
2. 创建组件

```ruby 
pod lib create HelloWorldLib
# 执行之后 一步一步陆续出现以下几个提问
1.What platform do you want to use? [ iOS / macOS ]
选择平台 输入iOS即可
2.What language do you want to use? [ Swift / ObjC ]
选择语言
3.Would you like to include a demo application with your library? [ Yes / No ]
是否包含demo
4.Which testing frameworks will you use? [ Specta / Kiwi / None ]
测试框架选择
5.Would you like to do view based testing? [ Yes / No ]
是否查看测试过程
6.What is your class prefix?
设置类前缀 
```
查看目录结构 需安装`brew install tree`插件
```ruby
tree -L 2 #查看2级目录结构

├── Example
│   ├── Podfile
│   ├── Podfile.lock
│   ├── Pods
│   ├── Tests
│   ├── HelloWorldLib
│   ├── HelloWorldLib.xcodeproj
│   └── HelloWorldLib.xcworkspace
├── LICENSE
├── README.md
├── HelloWorldLib
│   ├── Assets
│   └── Classes
├── HelloWorldLib.podspec
└── _Pods.xcodeproj -> Example/Pods/Pods.xcodeproj
以上 HelloWorldLib 目录下的 Classes 里替换你的库文件, Assets 放资源文件  
```

编辑 `HelloWorldLib.podspec`文件
```ruby
Pod::Spec.new do |s|
s.name             = 'HelloWorldLib'
s.version          = '1.0.0' #版本号与git tag 保持一致
s.summary          = 'xxxxx的功能组件.' #组件描述
s.description      = <<-DESC
TODO: Add long description of the pod here.
        DESC
#添加一大段的描述 一般为更新说明
s.homepage         = 'https://github.com/xxx/HelloWorldLib' #项目主页 一般是github pages 或者 文档博客
# s.screenshots     = 'www.example.com/screenshots_1', 'www.example.com/screenshots_2'
s.license          = { :type => 'MIT', :file => 'LICENSE' } #开源证书类型
s.author           = { 'XXX' => 'xxxxxxx@qq.com' } #作者用户名和邮箱
s.source           = { :git => 'https://github.com/XXXX/HelloWorldLib.git', :tag => s.version.to_s }#公有库写法
#s.source           = { :git => 'git@gitblit.local.com:XXX/iOSLibs.git', :tag => s.version.to_s }#私有库一般放在自己服务器上  

# s.social_media_url = 'https://twitter.com/<TWITTER_USERNAME>' #社交媒体 

s.ios.deployment_target = '8.0'

s.source_files = 'HelloWorldLib/Classes/**/*' 
#这一步很关键 如果是使用命令模板`pod lib create HelloWorldLib`生成的 直接把库文件放进HelloWorldLib/Classes目录下即可,要是自己创建的则需要匹配库文件的路径,能访问到即可

# s.resource_bundles = {
#   'HelloWorldLib' => ['HelloWorldLib/Assets/*.png']
# } #资源文件 

# s.public_header_files = 'Pod/Classes/**/*.h' #类库头文件
# s.frameworks = 'UIKit', 'MapKit' #依赖系统框架
# s.dependency 'AFNetworking', '~> 2.3' #依赖库
end
```
3. 使用本地库调试
**使用本地库调试，直接编辑Podfile，pod本地库的相对路径访问即可**
```ruby
 pod 'HelloWorldLib', :path => '../' # 使用`pod lib create HelloWorldLib`生成的工程自带本地库调试  其他工程取决于本地库相对于工程根目录所在的相对路径
 #这个 :path => '../' 路径只是访问HelloWorldLib.podspec文件的所在,如果路径正确,使用 `pod install` 即可安装正确
```
4. 发布远程库 
若是尚未关联远程库，则需加一下关联操作
```ruby
#若是没有关联远程库 可以执行以下操作
git remote add origin https://gitlab.com/xxx.git #（替换为自己的实际git地址）
git push --set-upstream origin master
```
发布私有库如下流程: 
```ruby
git add .
git commit -am  "commit HelloWorldLib v1.0.0"
git tag  1.0.0 #版本号映射
git push origin master --tags
pod repo push git@gitblit.local.com:XXX/iOSLibSpecs.git HelloWorldLib.podspec # 需要自己创建一个索引库iOSLibSpecs存储 .podspec
```
5. 发布远程公有库
在`HelloWorldLib.podspec`所在目录下
```shell
git add .
git commit -am  "commit HelloWorldLib v1.0.0 "
git tag  1.0.0 #版本号映射
git push origin master --tags
pod trunk push HelloWorldLib.podspec --allow-warnings --verbose #发布推送
```