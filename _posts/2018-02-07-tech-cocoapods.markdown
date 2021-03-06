---
layout: post
title:  "基于cocoapods的私有库搭建"
date:   2017-12-25 00:00:00
categories: tech
author: "hotacool"
description: 基于cocoapods管理工具，创建私有库，模块化管理项目工程
---

*最近想为公司搭建cocoapods私有库框架，老早之前做过，踩过不少坑，想不到又一次掉坑里。果真是好记性不如烂笔头，这次得记下来。*

# 1. 前言
cocoapods早已不是新鲜技术，作为iOS平台上的依赖管理工具，方便我们日常开发过程中管理和更新第三方库。基本使用方式不再赘述，这里说一下需求。项目规模大了之后，往往需要将各个业务功能拆分封装，一方面解耦，另一方面可作为第三方库供其他项目组使用。
所以产生一个需求：公司各开发团队，将自己的业务，封装为私有库，上传到公司私有git上，利用cocoapods统一管理。
###### 分解需求，我们需要：
* 创建私有spec repo（相当于cocoapods私有库资源中心https://github.com/CocoaPods/Specs.git），所有的私有库上传记录在私有spec中
* 工程封装为私有库，上传到私有spec repo中
* 项目工程集成私有库

# 2. 搭建
#### 1. 创建私有spec repo
##### 什么是spec repo
在Podfile中，我们通过

```
pod 'repoName'
```

来配置要加载的第三方库。repoName对应的repo地址并没有写出来，因为我们会到https://github.com/CocoaPods/Specs.git中去查找。它相当于一个容器，将所有私有库的podspec文件放到这个spec repo中，建立索引，提供私有库配置信息。
通过pod repo list命令，查看mac 上的repo列表：

```
HotacooldeMacBook-Pro:0-Venus hotacool$ pod repo list

master
- Type: git (master)
- URL:  https://github.com/CocoaPods/Specs.git
- Path: /Users/hotacool/.cocoapods/repos/master
```

本地spec repo的大致结构：

```
.
├── Specs
    └── [SPEC_NAME]
        └── [VERSION]
            └── [SPEC_NAME].podspec
```

进入/Users/hotacool/.cocoapods/repos/master，可以看到目录结构：
![1514169190670.jpg](http://upload-images.jianshu.io/upload_images/2263884-a79fd104b71ac493.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##### 创建remote&本地clone
理解了上面这个知识点，我们知道了需要创建一个自己的spec repo。
* 在代码托管平台（github，Bitbucket，码云等），创建spec repo的托管空间
例如在码云上创建一个spec repo：https://gitee.com/hotacool/HACSpec.git
* 利用cocoapods提供command，在terminal上运行：

```
// Clones `URL` in the local spec-repos directory at `~/.cocoapods/repos/`. The
      remote can later be referred to by `NAME`.
pod repo add HACSpec https://gitee.com/hotacool/HACSpec.git
```
会看到从remote地址git clone spec repo到本地，再次查看本地repo列表，可以看到我们创建的spec repo：

	```
	HotacooldeMacBook-Pro:0-Venus hotacool$ pod repo list

	HACSpec
	- Type: git (master)
	- URL:  https://gitee.com/hotacool/HACSpec.git
	- Path: /Users/hotacool/.cocoapods/repos/HACSpec

	master
	- Type: git (master)
	- URL:  https://github.com/CocoaPods/Specs.git
	- Path: /Users/hotacool/.cocoapods/repos/master
	```

这一步是在本地添加私有库source，其他开发人员Podfile需要集成私有spec上的私有库，都需要首先通过pod repo add命令，本地clone spec repo。
另外一些命令也可能会用到：

```
// 查看本地spec repo列表
pod repo list
// 更新本地specName的spec repo。私有库podspec文件更改并push到spec repo后，往往需要update
pod repo update specName
// 本地删除spec repo
pod repo remove specName
```

#### 2. 私有库制作
##### 创建一个基本（不包含代码）的私有库
创建私有库之前，首先需要解耦独立出单独的组件项目。如果已有独立的组件项目，可以通过pod spec create直接创建podspec文件，然后配置，生成私有库。这里首先讲一下，如何完整的创建一个新的私有库，并push到我么自己的spec repo中。
官网上有详细的命令[解释文档](http://guides.cocoapods.org/making/using-pod-lib-create)。下面以创建一个名为HStockCharts的私有库为例：
* 创建新的lib库

	```
	pod lib create HStockCharts
	```

cocoapods会clone pod-template模板来生成HStockCharts，创建过程中会问几个问题:

	```
	What language do you want to use?? [ Swift / ObjC ]
	 > ObjC

	Would you like to include a demo application with your library? [ Yes / No ]
	 > Yes

	Which testing frameworks will you use? [ Specta / Kiwi / None ]
	 > None

	Would you like to do view based testing? [ Yes / No ]
	 > No

	What is your class prefix?
	 > H

	Running pod install on your new library.
	```

一般我会按上面代码来生成一个带example的例子工程，这样生成的example相对干净。也可以选择自动集成testing framework，或者view based testing，这样cocoapods会自动帮你集成一些额外的三方库。
这样生成的目录结构：
```
HotacooldeMacBook-Pro:HStockCharts hotacool$ tree -L 2
.
├── Example
│   ├── HStockCharts
│   ├── HStockCharts.xcodeproj
│   ├── HStockCharts.xcworkspace
│   ├── Podfile
│   ├── Podfile.lock
│   ├── Pods
│   └── Tests
├── HStockCharts
│   ├── Assets
│   └── Classes
├── HStockCharts.podspec
├── LICENSE
├── README.md
└── _Pods.xcodeproj -> Example/Pods/Pods.xcodeproj
10 directories, 5 files
```
 HStockCharts.podspec是私有库的配置文件；HStockCharts文件夹下是私有库代码文件，Assets是放资源，Classes下是编译的源文件；Example是自动生成的测试工程。
* 编辑.podspec文件
.podspec文件是私有库的配置文件。新创建的lib中，自动生成了HStockCharts.podspec，填充了一些默认配置，我们需要手动去编辑。cocoapods是基于Ruby语音写的，所以可以设置为Ruby语法高亮。
	```
	Pod::Spec.new do |s|
	  s.name             = 'HStockCharts'
	  s.version          = '0.1.0'
	  s.summary          = 'A short description of HStockCharts.'

	# This description is used to generate tags and improve search results.
	#   * Think: What does it do? Why did you write it? What is the focus?
	#   * Try to keep it short, snappy and to the point.
	#   * Write the description between the DESC delimiters below.
	#   * Finally, don't worry about the indent, CocoaPods strips it!

	  s.description      = <<-DESC
	TODO: Add long description of the pod here.
	                       DESC

	  s.homepage         = 'https://github.com/shisosen@163.com/HStockCharts'
	  # s.screenshots     = 'www.example.com/screenshots_1', 'www.example.com/screenshots_2'
	  s.license          = { :type => 'MIT', :file => 'LICENSE' }
	  s.author           = { 'shisosen@163.com' => 'shisosen@163.com' }
	  s.source           = { :git => 'https://github.com/shisosen@163.com/HStockCharts.git', :tag => s.version.to_s }
	  # s.social_media_url = 'https://twitter.com/<TWITTER_USERNAME>'

	  s.ios.deployment_target = '8.0'

	  s.source_files = 'HStockCharts/Classes/**/*'
	  
	  # s.resource_bundles = {
	  #   'HStockCharts' => ['HStockCharts/Assets/*.png']
	  # }

	  # s.public_header_files = 'Pod/Classes/**/*.h'
	  # s.frameworks = 'UIKit', 'MapKit'
	  # s.dependency 'AFNetworking', '~> 2.3'
	end
	```
各标签(s.name等)的具体意义可以参照[官方文档](http://guides.cocoapods.org/syntax/podspec.html)。这里先列举几个重要的地方：
	```
	s.name //名称，pod search 查找
	s.version //版本号，私有库git tag。如果s.source里设置了s.version.to_s，需要注意与私有库git的tag相匹配。
	s.source //重要，私有库的remote 地址和tag。也可以:branch => 'master'设置分支。
	s.source_files //#代码源文件地址，**/*表示Classes目录及其子目录下所有文件；如果有多个目录下则用逗号分开如['','','']形式
	s.resource_bundles //资源文件地址
	s.public_header_files //公开的头文件地址
	s.frameworks //所需的framework，多个用逗号隔开
	s.dependency //依赖关系，该项目所依赖的其他库，如果有多个需要填写多个s.dependency。cocoapods会自动将依赖的其他库集成到工程中。
	```
现在只简单的对source等配置（s.source，s.description）做一下修改：
	```
	Pod::Spec.new do |s|
	  s.name             = 'HStockCharts'
	  s.version          = '0.1.0'
	  s.summary          = 'HStockCharts.'

	  s.description      = <<-DESC
	HStockCharts is a library for stock charts.
	                       DESC

	  s.homepage         = 'https://gitee.com/hotacool'
	  s.license          = { :type => 'MIT', :file => 'LICENSE' }
	  s.author           = { 'hotacool' => 'shisosen@163.com' }
	  s.source           = { :git => 'https://gitee.com/hotacool/HStockCharts.git', :tag => s.version.to_s }

	  s.ios.deployment_target = '8.0'

	  s.source_files = 'HStockCharts/Classes/**/*'
	  
	  # s.resource_bundles = {
	  #   'HStockCharts' => ['HStockCharts/Assets/*.png']
	  # }

	  # s.public_header_files = 'Pod/Classes/**/*.h'
	  # s.frameworks = 'UIKit', 'MapKit'
	  # s.dependency 'AFNetworking', '~> 2.3'
	end
	```

* push 私有库到remote，validation
现在在本地已完成私有库的创建，需要将私有库push到远程仓库，这样其他人才能到远程仓库clone。这里将本地HStockCharts push到https://gitee.com/hotacool/HStockCharts.git，即podspec设置的source地址。
因为我们podspec s.source设置了:tag => s.version.to_s，version是0.1.0，所以需要打tag，注意tag应与version保持一致：
	```
	git tag -m "release v.0.1.0 "0.1.0"
	git push --tags     //推送tag到远端仓库
	```
然后开始通过pod lib lint 对私有库进行validation：
	```
	HotacooldeMacBook-Pro:HStockCharts hotacool$ pod lib lint

	 -> HStockCharts (0.1.0)

	HStockCharts passed validation.
	```
验证通过。默认情况下，如果验证出现warning，是不能通过验证的。可以通过添加--allow-warnings来允许warnings。

* push私有库到spec repo
我们已经在本地搭建好了一个私有库，在Example中，Podfile通过相对地址，引入HStockCharts。
	```
	pod 'HStockCharts', :path => '../'
	```
现在需要将私有库推送到我们的spec repo中，从而可以从远程clone。
之前已经在本地通过pod repo add，新加入了我们自己的spec repo: HACSpec。通过命令：
	```
	pod repo push HACSpec HStockCharts.podspec  //前面是本地Repo名字 后面是podspec名字
	```
将HStockCharts推送到spec repo。执行pod repo update HACSpec，再cd 到本地repo文件夹/Users/hotacool/.cocoapods/repos/HACSpec，我们可以看到:
	```
	.
	├── HStockCharts
	│   └── 0.1.0
	├── README.md
	```
HStockCharts已从remote加入本地spec，版本号0.1.0。

* 集成私有库
现在我们可以在其他项目中，集成我们自己的私有库。Podfile加入：
	```
	pod 'HStockCharts'
	```
运行pod install，报错[!] Unable to find a specification for `HStockCharts`
因为HStockCharts在我们自己的spec repo中，默认的source地址是cocoapods repo。Podfile头部加入：
	```
	source 'https://gitee.com/hotacool/HACSpec.git' #private spec repo
	```
再次pod install，成功。
同样的，因为我们的spec是自建的，没有fork cocoapods的，所以不包含其他公共的私有库，如果你用到pod 'AFNetworking'，Podfile也需要添加：
	```
	source 'https://github.com/CocoaPods/Specs.git'
	```
至此，我们已经完整的建立了私有库，并加入私有的spec repo。但这个库什么都干不了，如果加入我们自己的代码，需要做更多的配置和处理。

##### 私有库设置
上面的介绍，没有涉及功能代码，实际的私有库配置，往往会遇到各种问题。下面对遇到的具体问题，做一些对应。
* 私有库依赖其他framework、library设置
解耦出的私有库工程，往往有对某些系统或者第三方framework、.a静态库的依赖。所以需要对私有库的podspec进行设置，使其他工程Podfile集成私有库时，能够自动引入相关依赖。
我们会对下面几个标签进行设置：
	```
	s.framework = 'SystemConfiguration','CoreData' //对系统framework的依赖
	s.libraries   = 'c++','stdc++.6.0.9' //对系统.a的依赖，注意去掉lib前缀

	s.ios.vendored_libraries = 'libXXX.a' //对第三方.a的依赖，注意路径，libXXX.a在.podspec目录下
	s.ios.vendored_frameworks = 'XXX.framework' //对第三方framework的依赖
	```
在项目的Pods工程的私有库target的build setting中，查看Other linker Flags，可以看到依赖的framework和.a:
![1514193548251.jpg](http://upload-images.jianshu.io/upload_images/2263884-753b3bc336c03f33.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* 包含.a的私有库
上面说了对第三方.a的依赖，可以通过s.ios.vendored_libraries来配置。公开的header files，按照官方说明，可以再s.public_header_files中设置。但貌似没达不到效果。目前我是通过s.source_files，将.h头文件统一放在s.source_files目录下输出。
另一个问题，直接对包含.a的私有库使用pod lib lint来validation是可以通过的，但pod repo push验证会出错会报错。报错内容类似：

```
- ERROR | [iOS] unknown: Encountered an unknown error (The 'Pods-App' target has transitive dependencies that include static binaries: (/private/var/folders/vx/466sl6v913z1xvbssfgpnjb00000gn/T/CocoaPods-Lint-20171220-2701-11qtxh8-MarketModule/Pods/XXXLib/libXXX.a)) during validation.
```

虽然这并不影响Pod的使用，但是验证是无法通过的。可以通过 --use-libraries 来让验证通过：
	
```
pod repo push HACSpec MarketModule.podspec --allow-warnings --use-libraries
```	
还有一个类似问题，如果在工程Podfile中使用use_frameworks!，会报错：
```
[!] The 'Pods-Venus' target has transitive dependencies that include static binaries: (/Users/hotacool/work/0-Venus/0-Venus/Pods/SZYDataSource/libMApi.a)
```
这里一个可行的办法，只能注释掉'use_frameworks!'。[参考链接](https://stackoverflow.com/questions/30910852/the-pods-target-has-transitive-dependencies-that-include-static-binaries-whe)。
* 通配符地址匹配报错
可能会出现类似报错：

```
- ERROR | [iOS] file patterns: The `source_files` pattern did not match any file.
```

一般是文件路径无法匹配造成。例如：

```
s.source_files = ['HStockCharts/Classes/**/*.{c,h,hh,m,mm,cpp}' //可以匹配HStockCharts/Classes/A/A.m，但无法匹配HStockCharts/Classes/A.m。
s.source_files = ['HStockCharts/Classes/**/*.{c,h,hh,m,mm,cpp}', 'HStockCharts/Classes/*.{c,h,hh,m,mm,cpp}'] //可以实现上述匹配
```

* 私有库对私有库的依赖
私有库B需要依赖私有库A，可以在B.podspec中设置：

```
s.dependency 'A'
```

如果执行pod lib lint 或者pod repo push 会报错找不到A。这是肯定的，默认的spec repo是cocoapods的公共库，所以需要对source单独设置：

```
pod lib lint --sources='https://gitee.com/hotacool/HACSpec.git'
pod repo push HACSpec B.podspec --sources='https://gitee.com/hotacool/HACSpec.git'
```

* 添加资源文件
有以下几种添加资源方法：

```
// 1.默认方式，cocoapods会将匹配路径'HStockCharts/Assets/*.png’下资源打包为名称HStockCharts的bundle，代码中通过bundle名来查找资源。
  s.resource_bundles = {
    'HStockCharts' => ['HStockCharts/Assets/*.png']
  }
// 调用方式：
NSBundle *bundle = [NSBundle bundleForClass:[MYSomeClass class]];
NSURL *bundleURL = [bundle URLForResource:@"HStockCharts" withExtension:@"bundle"];
NSBundle *resourceBundle = [NSBundle bundleWithURL: bundleURL];
UIImage *img = [UIImage imageNamed:icon inBundle:bundle compatibleWithTraitCollection:nil];
// 或者
UIImage *image = [UIImage imageNamed:@"HStockCharts.bundle/imagename"];

// 2.资源会在打包的时候直接拷贝的main bundle中，可能会和其它资源产生命名冲突
spec.resources = ["Images/*.png", "Sounds/*"]

// 3.把资源都放在bundle中，然后打包时候这个bundle会直接拷贝进app的mainBundle中。使用的时候在mainBundle中查找这个bundle然后再搜索具体资源
spec.resource = "Resources/MYLibrary.bundle"
//调用方式：
NSURL *bundleURL = [[NSBundle mainBundle] URLForResource:@"JZShare" withExtension:@"bundle"];
    NSBundle *bundle = [NSBundle bundleWithURL:bundleURL];
    UIImage *img = [UIImage imageNamed:icon inBundle:bundle compatibleWithTraitCollection:nil];
```

* pod lib lint 和 pod spec lint
两者都是对私有库进行validation。不过区别是：

```
pod lib lint是只从本地验证你的pod能否通过验证
pod spec lint是从本地和远程验证你的pod能否通过验证
```

所以如果你使用pod spec lint，需要保证remote及时更新，否则会发现本地已经做了修改，但一直无法validation成功...
* 更新维护podspec
在开发初始阶段，可能会频繁对podspec进行更改。在pod repo push后，记得及时pod repo update specName，否则可能拉取不到最新的私有库配置。

* mrc文件设置
某些文件需要设置mrc，podspec可以如下设置：

```
  non_arc_files = 'MyLib/Classes/xxx.m'
  s.exclude_files = non_arc_files

  s.subspec 'no-arc' do |sp|
    sp.source_files = non_arc_files
    sp.requires_arc = false
  end
```

but，不知是否是cocoapods的版本原因（1.3.1），采用这种方式我仍旧无法通过验证。通过--no-clean，进入验证的APP工程，发现私有库下的编译文件只有non_arc_files文件。
怀疑是否默认使用no-arc 的subspec，之后参照[链接](https://segmentfault.com/q/1010000002972392)，设置default_subspec和subspec。但仍旧无法验证通过。
最后，只能通过将requires_arc设为mrc，单独添加arc配置，来实现对mrc文件的编译。如下：

```
#source_files需要包含mrc和arc全部文件路径
s.source_files = ["xxx/MRC/*.{c,h,hh,m,mm,cpp}", 'xxx/Classes/**/*.{c,h,hh,m,mm,cpp}', 'xxx/Classes/*.{c,h,hh,m,mm,cpp}'] 
s.requires_arc = false
s.requires_arc = ['xxx/Classes/**/*.{c,h,hh,m,mm,cpp}', 'xxx/Classes/*.{c,h,hh,m,mm,cpp}']
```

* pod 本地repo缓存清理
私有库开发阶段，会经常修改文件，这时候极易出现remote上已更新，但本地pod install/update 的私有库还是老版本的情况。这往往是因为pod本地缓存导致，需要更新本地缓存：

```
//1. 更新本地spec
pod repo update xxxSpec
//2. 去本地pods缓存文件夹(~/Library/Caches/CocoaPods/Pods/Release)，删除需要更新的库
open ~/Library/Caches/CocoaPods/Pods/
```

# 3. 项目集成私有库
私有库已搭建完成，私有库的集成方式与cocoapods公共库集成方式无异，只需按照上述，在Podfile的source加入私有spec repo地址。无需赘述。
这里提一下本地相对路径的集成方式。我们可以通过pod xxx，直接从remote clone私有库到项目中。也可以通过pod 'HStockCharts', :path => '../'的方式，将本地库链到工程中。对于需要开发的各个私有库，可以将其clone到本地，然后本地相对路径的方式链接到工程中，对私有库进行修改，然后push到对应私有库。或许可以以此来解耦复杂工程，结合脚本，搭建环境，完成业务的独立开发，这也是目前我做的事情。

# 4. 总结
这里总结了一下基于cocoapods的私有库搭建，归纳了一些基本的方法，记录了一些遇到的问题。因为需求不大，涉及深度有限。如在后续改造优化过程中有更多的问题出现，也在此同步更新。


参考资料：
[使用Cocoapods创建私有podspec](http://www.cocoachina.com/ios/20150228/11206.html)
[Cocoapods使用私有库中遇到的坑](https://www.jianshu.com/p/1e5927eeb341)
