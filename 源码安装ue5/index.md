# 源码安装UE5



<!--more-->

## 为什么选择用源码版
1. 由于官方的二进制版本是无法发布专用服务器（Dedicated Server）的。
1. 项目需要修改了源码的内容

{{< admonition type=tips title="建议" open=true >}}
如果不是出于以上任何一个原因都不推荐用源码版。二进制版本直接开箱即用多香。
{{< /admonition >}}
<br>

## 源码下载
官方源码下载教程：


{{< friend "从GitHub下载虚幻引擎源代码" "https://dev.epicgames.com/documentation/zh-cn/unreal-engine/downloading-source-code-in-unreal-engine" "https://www.epicgames.com/favicon.ico" "https://dev.epicgames.com/documentation/zh-cn/unreal-engine/downloading-source-code-in-unreal-engine" >}}

<br><br>

虚幻GitHub仓库：
{{< friend "GitHub: EpicGames/UnrealEngine" "https://github.com/EpicGames/UnrealEngine" "https://github.com/favicon.ico" "https://github.com/EpicGames/UnrealEngine" >}}



## 环境配置


{{< admonition type=warning title="注意" open=true >}}
一定要根据当前源码版的SDK配置要求来安装本地环境！！！  
一定要根据当前源码版的SDK配置要求来安装本地环境！！！  
一定要根据当前源码版的SDK配置要求来安装本地环境！！！  
{{< /admonition >}}

`主要还是要保证SDK版本一致，有时候最新的会导致编译报错，修改了相关的SDK版本后，需要重新生成.sln文件，然后再编译。`

每个大版本发布的时候，虚幻官网都有对应的版本说明，找到对应版本的SDK要求，按照要求去下载，每个版本都不一样！！！

下面是虚幻5.4的版本说明：《平台sdk升级》中有详细的SDK要求
{{< friend "虚幻文档：虚幻引擎5.4版本说明" "https://dev.epicgames.com/documentation/zh-cn/unreal-engine/unreal-engine-5.4-release-notes?application_version=5.4#%E5%B9%B3%E5%8F%B0sdk%E5%8D%87%E7%BA%A7" "https://www.epicgames.com/favicon.ico" "https://dev.epicgames.com/documentation/zh-cn/unreal-engine/unreal-engine-5.4-release-notes?application_version=5.4#%E5%B9%B3%E5%8F%B0sdk%E5%8D%87%E7%BA%A7" >}}

![](/images/源码安装UE5/1.png)

所以我的本地编译版本的详细信息如下：
* Visual Studio 2022 - v17.4 (可以根据实际情况选择更高的)
* MSVC - 14.38.33130 (可以根据实际情况选择更高的)
* Windows SDK - 10.0.18362 (可以根据实际情况选择更高的)
* LLVM clang - 16.0.6
* .NET 4.6.2 Targeting Pack (定死了的版本)
* .NET 6.0 (定死了的版本)

{{< admonition type=tips title="建议" open=true >}}
MSVC和SDK的版本我建议按官网的推荐版本就可以了，不要用更高版本的，不要问为什么![alt text](007A3AD5.png)
{{< /admonition >}}
{{< friend "虚幻文档：设置Visual Studio" "https://dev.epicgames.com/documentation/zh-cn/unreal-engine/setting-up-visual-studio-development-environment-for-cplusplus-projects-in-unreal-engine" "https://www.epicgames.com/favicon.ico" "https://dev.epicgames.com/documentation/zh-cn/unreal-engine/setting-up-visual-studio-development-environment-for-cplusplus-projects-in-unreal-engine" >}}


## 下载引擎依赖文件

以管理员身份运行引擎目录下的Setup.bat进行依赖的下载

时间相当长，有20G多的内容，而且国内下着下着就断开了，如果断开了就再次运行Setup.bat就可以，支持断点续传，不用全部重新下载。

## 生成对应的工程文件

确认所有依赖下载完之后，以管理员身份运行引擎目录下的GenerateProjectFiles.bat

## 官方参考
剩下的内容参考虚幻官方的教程即可，参考链接：
{{< friend "虚幻文档：从源代码构建虚幻引擎" "https://dev.epicgames.com/documentation/zh-cn/unreal-engine/building-unreal-engine-from-source" "https://www.epicgames.com/favicon.ico" "https://dev.epicgames.com/documentation/zh-cn/unreal-engine/building-unreal-engine-from-source" >}}


## 用源码版本遇到的问题

部分问题是从网友处摘抄

### 编译完成时遇到1 失败，其他都成功了
用VS2022重新生成项目每次完成时都会遇到 1 失败, 11 最新。再次点生成就又能正常编译通过。暂时没找到是什么原因导致的

### 编译时遇到 error C4756: 常量算法中溢出
UE默认应该用的是你安装过的最新的MSVC来做的编译，这有可能超过了该版本UE指定的MSVC。这个可以通过正确设置MSVC版本来解决。以UE5.4为例,应该使用 14.38.33130 版本的MSVC。

通过修改BuildConfiguration.xml来配置
{{< admonition type=tips title="提示" open=true >}}
通常在以下两个路径
* 引擎所在目录/Engine/Saved/UnrealBuildTool
* %appdata%/Unreal Engine/UnrealBuildTool
{{< /admonition >}}

``` BuildConfiguration.xml
<?xml version="1.0" encoding="utf-8" ?>
<Configuration xmlns="https://www.unrealengine.com/BuildConfiguration">
	<WindowsPlatform>
		<CompilerVersion>14.38.33130</CompilerVersion>
		<WindowsSdkVersion>10.0.22621.0</WindowsSdkVersion>
	</WindowsPlatform>
</Configuration>
```

以下链接可查看UE5推荐的MSVC版本
{{< friend "虚幻文档：设置Visual Studio" "https://dev.epicgames.com/documentation/zh-cn/unreal-engine/setting-up-visual-studio-development-environment-for-cplusplus-projects-in-unreal-engine" "https://www.epicgames.com/favicon.ico" "https://dev.epicgames.com/documentation/zh-cn/unreal-engine/setting-up-visual-studio-development-environment-for-cplusplus-projects-in-unreal-engine" >}}

<br><br>
参考于以下文章：
{{< friend "UE5:为特定版本引擎指定MSVC版本的方法" "https://zhuanlan.zhihu.com/p/6959954923" "https://zhuanlan.zhihu.com/favicon.ico" "[友链页](https://zhuanlan.zhihu.com/p/6959954923)" >}}

### 编译时遇到 error C1001: 内部编译器错误。

### 奇怪的引擎版本名称
位于项目的XX.uproject文件里面  
`"EngineAssociation": "5.4",`  
版本会变成一串长字符串  

![](/images/源码安装UE5/2.png "源码版")

![](/images/源码安装UE5/3.png "二进制版")

EngineAssociation的值其实指向的是当前的引擎路径，这个路径可在注册表中找到

    计算机\HKEY_CURRENT_USER\Software\Epic Games\Unreal Engine\Builds

![](/images/源码安装UE5/4.png)

那么这个值不一样会导致哪些问题呢？
1. 项目文件混乱：
   * 如果你用一个源码编译的引擎打开一个之前用官方5.3版本创建的项目，引擎会提示“项目是由其他版本的引擎创建的，是否要转换？”。如果你选择“是”，它会将项目的 EngineAssociation 更新为你编译引擎的GUID。
   * 此后，这个项目在Epic启动器中可能无法正确识别或打开，因为启动器不认识你这个自定义的GUID。
2. 团队协作问题：
   * 如果团队中有人用官方二进制版本，有人用源码编译版本，那么 .uproject 文件中的 EngineAssociation 会频繁被不同的人修改，导致版本控制上的冲突和混乱。

### 警告：文件保存自空版本引擎
这个每次启动项目的时候都会提示，通常出现在源码版跟二进制版本混用的情况

### 不定期出现需要重新编译引擎/插件代码的情况
{{< friend "B站专栏：定制UE4引擎，如何避免C++项目重编引擎" "https://www.bilibili.com/opus/464162093554185164/?from=readlist" "https://www.bilibili.com/favicon.ico" "https://www.bilibili.com/opus/464162093554185164/?from=readlist" >}}

