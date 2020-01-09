# iOS 安全防护相关
> 道高一尺,魔高一丈,关于安全是个攻防不断博弈的过程,拥抱变化.

#### 越狱相关
##### 越狱设备或者环境判断
**1. 越狱设备可能会存在以下文件**
```bash
    "/Applications/Cydia.app",
    "/Library/MobileSubstrate/MobileSubstrate.dylib",
    "/bin/bash",
    "/usr/sbin/sshd",
    "/etc/apt"
```
**2. 判断是否存在[cydia](https://cydia-app.com/)应用**

**3. 能够读取所有应用名称**
> 非越狱手机是无法获取系统应用参数的

**4. 读取环境变量 `DYLD_INSERT_LIBRARIES`**
> 非越狱手机DYLD_INSERT_LIBRARIES获取到的环境变量为NULL。

**综合以上观点:**
```swift
// 常见越狱文件
const char *Jailbreak_Tool_pathes[] = {
    "/Applications/Cydia.app",
    "/Library/MobileSubstrate/MobileSubstrate.dylib",
    "/bin/bash",
    "/usr/sbin/sshd",
    "/etc/apt"
};

char *printEnv(void){
    char *env = getenv("DYLD_INSERT_LIBRARIES");
    return env;
}

/** 当前设备是否越狱 */
+ (BOOL)isDeviceJailbreak
{
    // 判断是否存在越狱文件
    for (int i = 0; i < 5; i++) {
        if ([[NSFileManager defaultManager] fileExistsAtPath:[NSString stringWithUTF8String:Jailbreak_Tool_pathes[i]]]) {
            NSLog(@"此设备越狱!");
            return YES;
        }
    }
    // 判断是否存在cydia应用
    if([[UIApplication sharedApplication] canOpenURL:[NSURL URLWithString:@"cydia://package/com.example.package"]]){
        NSLog(@"此设备越狱!");
        return YES;
    }
    
    // 读取系统所有的应用名称
    if ([[NSFileManager defaultManager] fileExistsAtPath:@"/User/Applications/"]){
        NSLog(@"此设备越狱!");
        return YES;
    }
    
    // 读取环境变量
    if(printEnv()){
        NSLog(@"此设备越狱!");
        return YES;
    }
    
    NSLog(@"此设备没有越狱");
    return NO;
}
```
[防止越狱代码可参考](https://github.com/thii/DTTJailbreakDetection/blob/master/Sources/DTTJailbreakDetection/DTTJailbreakDetection.m)

#### IPA文件被篡改
##### 检测ipa文件是否被篡改可以通过以下方法：
 **1. 通过对`CodeResources`读取资源文件原始`hash`，和当前`hash`进行对比，判断是否经过篡改，被篡改过的文件应从服务器重新请求资源文件进行替换;**

**2. 可以通过检测`SignerIdentity`判断是`Mach-O`文件否被篡改;**

**3. 可以通过检测`cryptid`的值来检测`Mach-O`文件是否被篡改，篡改过`cryptid`的值为`1`**

**4. IPA包上传到TestFlight或者App Store后，计算安装包中重要文件的MD5 值，服务器记录，在应用运行前首先将本地计算的 MD5 值和服务器记录的 MD5 值 进行对比，如不同，则退出应用**

*这里介绍通过检测`SignerIdentity`判断是`Mach-O`文件否被篡改*

**原理:**

> `SignerIdentity`的值在`Info.plist`中是不存在的，开发者不会加上去，苹果也不会，只是当`ipa`包被反编译后篡改文件再次打包，需要伪造`SignerIdentity`

```swift
    NSBundle *bundle = [NSBundle mainBundle];
    NSDictionary *info = [bundle infoDictionary];
    if ([info objectForKey:@"SignerIdentity"] != nil)
    {
        return YES;
    }
    return NO;
```

#### 防止`gdb/lldb` 调试
```swift
// 阻止 gdb/lldb 调试
// 调用 ptrace 设置参数 PT_DENY_ATTACH，如果有调试器依附，则会产生错误并退出
#import <dlfcn.h>
#import <sys/types.h>
 
typedef int (*ptrace_ptr_t)(int _request, pid_t _pid, caddr_t _addr, int _data);
#if !defined(PT_DENY_ATTACH)
#define PT_DENY_ATTACH 31
#endif
 
void anti_gdb_debug() {
    void *handle = dlopen(0, RTLD_GLOBAL | RTLD_NOW);
    ptrace_ptr_t ptrace_ptr = dlsym(handle, "ptrace");
    ptrace_ptr(PT_DENY_ATTACH, 0, 0, 0);
    dlclose(handle);
}
 
int main(int argc, char * argv[]) {
#ifndef DEBUG
    // 非 DEBUG 模式下禁止调试
    anti_gdb_debug();
#endif
    @autoreleasepool {
        return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
    }
}
```
#### 防止抓包
##### 1. 检测代理
```swift
对于抓包，现在的手段基本是设置代理，所以我们可以通过判断是否设置了代理的方式来进行下一步的防范。
在网络请求前插入这个方法，再根据需求做相应的防范
+ (BOOL)getDelegateStatus
{
    NSDictionary *proxySettings = CFBridgingRelease((__bridge CFTypeRef _Nullable)((__bridge NSDictionary *)CFNetworkCopySystemProxySettings()));
    NSArray *proxies = CFBridgingRelease((__bridge CFTypeRef _Nullable)((__bridge NSArray *)CFNetworkCopyProxiesForURL((__bridge CFURLRef)[NSURL URLWithString:@"http://www.google.com"], (__bridge CFDictionaryRef)proxySettings)));
    NSDictionary *settings = [proxies objectAtIndex:0];
    NSLog(@"host=%@", [settings objectForKey:(NSString *)kCFProxyHostNameKey]);
    NSLog(@"port=%@", [settings objectForKey:(NSString *)kCFProxyPortNumberKey]);
    NSLog(@"type=%@", [settings objectForKey:(NSString *)kCFProxyTypeKey]);
    if ([[settings objectForKey:(NSString *)kCFProxyTypeKey] isEqualToString:@"kCFProxyTypeNone"])
    {
        //没有设置代理
        return NO;
        
    } else {
        //设置代理了
        return YES;
    }
}
``` 
##### 2. 接口参数和数据使用对称加密处理
* AES128 
* RSA
  
#### 代码混淆加固等
* 一些核心数据的加密混淆，因为大面积混淆容易出现不可预知的问题,比如苹果商店拒绝上架之类的

#### 相关博客介绍
* [【iOS应用安全】hook及越狱的基本防护与检测(动态库注入检测、hook检测与防护、越狱检测、签名校验、IDA反编译分析加密协议示例)](https://github.com/SmileZXLee/ZXHookDetection)
* [iOS应用安全 － 完整性检测](https://www.jianshu.com/p/f043b72db736)
* [iOS App的几种安全防范](https://www.cnblogs.com/tangyuanby2/p/11384481.html)
* [防止App被修改,加签名判断](https://makezl.github.io/2016/06/21/CodeSign/)
* [iOS安全防护---越狱检测、二次打包检测、反调试](https://blog.csdn.net/u013602835/article/details/86545106)
* [如何防止客户端被破解](https://tanqisen.github.io/blog/2014/06/06/how-to-prevent-app-crack/)
* [iOS——防止反编译总结](https://icocos.github.io/2017/10/26/iOS%E2%80%94%E2%80%94%E9%98%B2%E6%AD%A2%E5%8F%8D%E7%BC%96%E8%AF%91%E6%80%BB%E7%BB%93/)



