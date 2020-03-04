
## 下载SDK
放到指定目录,配置环境变量 
具体步骤结合自己电脑参考[flutter中文网](https://flutterchina.club/get-started/install/)
## 安装

### Android Stdio + SDK + tool SDK + 创建模拟器 + 插件(flutter和dart) 

下载`Android Stdio`网上很多资源,我推荐这个[网站](https://www.androiddevtools.cn/),资源比较集中

`~/.zshrc`或者`~/.bash_profile`配置环境变量
```shell
# android sdk
export ANDROID_HOME="/Users/wangguibin/Library/Android/sdk" 
export PATH=${PATH}:${ANDROID_HOME}/tools
export PATH=${PATH}:${ANDROID_HOME}/platform-tools
```

**由于大陆有墙,可在`/etc/hosts`文件中加入以下镜像或者代理,下载`Android SDK`顺畅无比**
(PS. 当前时间2020年2月19日,亲测可用!)
```shell
# Android Start
119.28.87.227	android.com

119.28.87.227	www.android.com

119.28.87.227	a.android.com

119.28.87.227	connectivitycheck.android.com

119.28.87.227	d.android.com

119.28.87.227	dev.android.com

119.28.87.227	developer.android.com

119.28.87.227	market.android.com

119.28.87.227	r.android.com

119.28.87.227	source.android.com

119.28.87.227	android-china.l.google.com

119.28.87.227	android.clients.google.com

119.28.87.227	android-market.l.google.com

119.28.87.227	android.l.google.com

119.28.87.227	android.googleblog.com

119.28.87.227	androidstudio.googleblog.com

119.28.87.227	android-developers.googleblog.com

119.28.87.227	android-developers.blogspot.com

119.28.87.227	android-developers.blogspot.hk

119.28.87.227	officialandroid.blogspot.com

119.28.87.227	android.googlecode.com

119.28.87.227	android.googlesource.com

119.28.87.227	android-review.googlesource.com

119.28.87.227	androidmarket.googleusercontent.com

119.28.87.227	android.googleapis.com

119.28.87.227	jmoore-dot-android-experiments.appspot.com

119.28.87.227	b.android.com

64.233.188.121	m.android.com

64.233.188.121	tools.android.com

64.233.191.121	jmoore-dot-android-experiments.appspot.com

64.233.191.121	maven.google.com
# Android End
```

### Xcode + cocoapods
#### `Xcode`在`App Store`或者[开发者中心](https://developer.apple.com/xcode/)下载即可
#### `Cocoapods`安装
```shell
sudo gem install cocoapods

```

```shell
flutter doctor 检测环境是否可行
flutter doctor --android-licenses 安卓验证SDK
```

## VSCode + Flutter开发必备辅助插件

- `Flutter`语法及调试插件
- `Dart`	语法插件
- `Awesome Flutter Snippets` 代码块
- `Flutter Widget Snippets` 组件代码块
- `Bracket Pair Colorizer` 作用域描线
- `Flutter Stylizer` API使用提示
- `Material Icon Theme` 图标主题 
