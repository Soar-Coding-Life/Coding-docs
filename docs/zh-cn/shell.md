# 我折腾的shell笔记
[shell 代码集合](https://github.com/WangGuibin/WGBToolsConfigRepository/blob/master/Shell/README.md)

## Mac一些常用的快捷键记录

### 命令行
* `ctrl + w` 按单词(word)单步删除输入的命令
* `ctrl + a` 光标移动到命令行最前面
* `ctrl + e` 光标移动到命令行最前面
* `ctrl + k` , `ctrl + q` , `ctrl + u`清除输入的命令行
* `open ./` 弹出当前目录`finder`

### Mac桌面上或者某目录下操作
* `cmd + shift + G` 前往文件夹
* `cmd + space` 聚焦搜索即全局搜索
* `cmd + shift + .` 隐藏/显示隐藏文件或文件夹
* `cmd + shift + 3` 全屏截图  
* `cmd + shift + 4` 可选取范围截图
* `ctrl + cmd + space` 弹出`emoji`选择窗口
* `ctrl + ←` 向左切换桌面
* `ctrl + →` 向右切换桌面
* `ctrl + ↑` 显示全部桌面选择
* `ctrl + ↓` 显示最近打开编辑过的文件
* `cmd + tab` 切换已打开程序坞上的应用
* `cmd + q` 关闭当前程序
* `cmd + w` 关闭当前窗口
* `cmd + n` 新建文件
* `cmd + s` 保存
* `cmd + ,` 当前应用的偏好设置
* `ctrl + space` 切换输入法




## 一些实用脚本示例

### 代码无提示或者其他抽风症状,清除Xcode缓存
```shell
#!/bin/bash
defaults write com.apple.dt.XCode IDEIndexDisable 0
rm -rf ~/Library/Developer/Xcode/DerivedData
rm -rf ~/Library/Caches/com.apple.dt.Xcode
# 关闭Xcode 
killall Xcode
```
### 查看当前网络ip地址
```shell
#!/bin/bash
curl ip.sb
```
### 日常提交推送git代码
```shell
#!/bin/bash

read -p "输入提交日志信息: " commit_message
read -p "输入分支名(默认为dev分支): " branch_name

if [[ -n "${commit_message}" ]]; then
	#statements
	echo "日志信息参数为: ${commit_message}"
else
	echo "日志信息参数为空，已使用默认模板: 🚀update~"
	commit_message="🚀update~"
fi

if [[ -n "${branch_name}" ]]; then
	#statements
	echo "分支名参数为: ${branch_name}"
else
	echo "分支名参数未输入,默认为dev"
	branch_name="dev"
fi

git add .
git commit -am  "${commit_message}"
git push origin ${branch_name}
```

### 统计iOS代码行数
```shell
#!/bin/bash
	read -p "输入代码文件所在路径: " code_path
	find ${code_path} "(" -name "*.h" -or -name "*.mm" -or -name "*.m" -or -name "*.swift" ")" -print | xargs wc -l 

```