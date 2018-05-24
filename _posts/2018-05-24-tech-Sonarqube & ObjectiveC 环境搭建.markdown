---
layout: post
title:  "Sonarqube & ObjectiveC 环境搭建"
date:   2017-12-25 00:00:00
categories: tech
author: "hotacool"
description: 本篇介绍Mac上sonarqube的安装，以及添加开源plugin对objective c支持。
---

###前言
团队开发中，代码质量的把关，往往决定了一个团队的开发维护效率。成员的增长，业务的扩大，不同风格、不严谨的代码，直接导致后续维护的高成本。
每个团队都有自己的一套代码检查方式。对于小团队，也有很多高效可扩展的开源工具。结合自己的实践，介绍下一种解决方案：sonarqube + gerrit + Jenkins。
本篇介绍Mac上sonarqube的安装，以及添加开源plugin对objective c支持。环境配置比较多，文字较长...


#####所需安装工具一览
* homebrew
* Java JDK （推荐jdk而不是jre，最新的即可）
* maven
* xcode
* xcpretty
* sonarqube
* sonar-scanner
* oclint
* SonarQube Plugin for Objective C

注： 安装顺序推荐如上排序。建议先阅读下文，了解各工具作用后再进行安装

###sonarqube 安装与使用
[SonarQube](http://www.sonarqube.org/)® software (previously called Sonar) is an open source quality management platform, dedicated to continuously analyze and measure technical quality, from project portfolio to method. If you wish to extend the SonarQube platform with open source plugins, have a look at our [plugin library](https://docs.sonarqube.org/display/PLUG/Plugin+Library).

#####1.  环境要求（具体见官网）
    *   at least 2GB of RAM to run efficiently and 1GB of free RAM for the OS
    *   disk space you need will depend on how much code you analyze
    *   Database： [MySQL](http://www.mysql.com/)
    *   Web Browser

#####2. 安装sonarqube
  *   download [Download Page](http://www.sonarsource.org/downloads/)
  *   Unzip & execute:  sonar.sh console(解压后目录里有系统对应脚本)
  ```
  # 主要命令一览， 建议添加alias到~/.bash_profile
  ./sonar.sh console #Debug信息
  ./sonar.sh start #启动服务
  ./sonar.sh stop #停止服务
  ./sonar.sh restart #重启服务
  ```
  *   Log in to [http://localhost:9000](http://localhost:9000/) & 默认登录/密码: admin/admin
  *   follow the tutorial to analyze your first project （无法添加project，需要安装数据库和scanner）
  *   **汉化方式**：setting->market->搜Chinese

#####3. 安装 &amp; 配置数据库(这里使用MySQL)
sonarqube内嵌有数据库，但不能有额外操作，所以需要另外添加数据库
```
# 直接使用homebrew，homebrew安装方式请自查
brew install mysql #安装
mysql.server start #启动
mysql_secure_installation #基本配置
brew services start mysql #配置为系统服务
```
mysql管理gui，optional：
* [sequelpro]( https://sequelpro.com/)
* [phpmyadmin](https://www.phpmyadmin.net/)

**配置sonar的用户和数据库实例:**
* 创建sonar用户和数据库sonar，赋予sonar权限
```
mysql -u root -p #进入MySQL, root默认密码为空
CREATE DATABASE sonar;
CREATE USER 'sonar'@'localhost' IDENTIFIED BY '你要设置的密码’;
GRANT ALL PRIVILEGES ON sonar_qube.* TO 'sonar'@'localhost’;
FLUSH PRIVILEGES;
```
**修改sonarqube数据库配置**
* 到sonarquebe目录配置\conf\sonar.properties中数据库设置，原文件相应行被注释
```
sonar.jdbc.username=sonar
sonar.jdbc.password=你要设置的密码
sonar.jdbc.url=jdbc:mysql://localhost:3306/sonar_qube?useUnicode=true&amp;characterEncoding=utf8&amp;rewriteBatchedStatements=true&amp;useConfigs=maxPerformance&amp;useSSL=false
```
* restart sonarqube server，正常启动，且不再有内嵌数据库提示

#####4. 安装sonar-scanner & 配置
The SonarQube&nbsp;Scanner is recommended as the default launcher to&nbsp;analyze a project with SonarQube.
* [Installation](https://docs.sonarqube.org/display/SCAN/Analyzing+with+SonarQube+Scanner#AnalyzingwithSonarQubeScanner-Installation)
* 解压到安装目录，推荐添加bin目录路径到$PATH
```
export PATH=`pwd`/bin:$PATH >> ~/.bash_profile  # 或直接copy bin路径到~/.bash_profile
```
* 配置安装目录下的sonar-scanner.properties & 添加host.url
```
sonar.host.url=http://localhost:9000
```
**对需检测project执行scanner命令**
到project所在目录，添加sonar-project.properties(**注意命名不要错😪**)。添加project相关信息sonar-project.properties中，例如
```
sonar.projectKey=testOC #projectKey required
sonar.sources=/Users/hotacool/dev/RACTest #project source required
sonar.projectName=test OC #server 展示名字
sonar.projectVersion=1.0 # version
```
在该工程sonar-project.properties所在目录下，执行sonar-scanner，进行扫描
打开server（[http://localhost:9000/projects](http://localhost:9000/projects)），可以看到projects扫描结果

#####5.  [oclint](http://oclint.org/) 安装 & 使用
sonarqube对objectivec的代码检测，需要依赖oclint对xcode工程的编译log的分析结果。简单说下过程，对xcode工程进行xcodebuild，输出编译log，oclint对编译log进行分析，输出xml，sonarqube根据xml来显示可视化结果。
OCLint is a [static code analysis](https://en.wikipedia.org/wiki/Static_program_analysis) tool for improving quality and reducing defects by inspecting C, C++ and Objective-C code and looking for potential problems.
* 安装方式见官网。
  * Homebrew 简单，但可能外网连不上
  * 推荐直接download包，解压执行文件路径加入$PATH
* 安装完毕，终端执行oclint
  * xcodebuild | tee xcodebuild.log | xcpretty -r json-compilation-database
  * 编译工程并输出log，格式化log为json文件
  * Xcodebuild 指令需要添加编译相关参数
```
xcodebuild archive -workspace 项目名称.xcworkspace 
                       -scheme 项目名称 
                       -configuration 构建配置 
                       -archivePath archive包存储路径 
                       CODE_SIGN_IDENTITY=证书 
                       PROVISIONING_PROFILE=描述文件UUID
```
  * oclint的执行，需要依赖xcpretty
  * Xcpretty 是编译log format工具，直接gem安装
```
sudo gem install xcpretty
```
  * -r json-compilation-database 制定的数据的输出格式为json格式。输出的数据为build/reports/compilation_db.json
  * 使用oclint时需要将build/reports/compilation_db.json 重新命名为 compile_commands.json 并移动至当前目录（工程文件目录）,执行：
  ```
  oclint-json-compilation-database -- -report-type pmd -o oclint.xml #输出xml
  oclint-json-compilation-database -e Pods -v -- -report-type html -o report.html #输出HTML，oclint使用xml
  ```
#####6. SonarQube Plugin for Objective C
* Sonarqube market有收费plugin可供安装，but，感受一下：
![image.png](https://upload-images.jianshu.io/upload_images/2263884-745f26d9271072ab.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* 选用开源[https://github.com/Backelite/sonar-objective-c](https://github.com/Backelite/sonar-objective-c)(还在保持更新的)，不过可能存在一些坑，需要自填。笔者fork了一下，更新了一下rule：[Hotacool/sonar-objective-c](https://github.com/Hotacool/sonar-objective-c)。下面以Backelite为例：
  * github download sonar-objective-c，unzip
  * Java代码，需要maven来编译jar包，安装maven： 
  ```
  brew install maven
  ```
  * sonar-objective-c 目录(pom.xml所在目录)编译：
  ```
  mvn package
  ```
  * /sonar-objective-c-plugin/target 目录下输出backelite-sonar-objective-c-plugin-0.6.2.jar
  * Copy jar包到sonarqube安装目录下的/extensions/plugins文件夹
  * 修改project的sonar-project.properties（参照sonar-objective-c-plugin的/sample/sonar-project.properties）
  ```
  sonar.objectivec.oclint.report=oclint.xml
  sonar.objectivec.project=myApp.xcodeproj
  sonar.objectivec.appScheme=myApp
  sonar.objectivec.excludedPathsFromCoverage=.*Tests.*
  ```
  * 重启sonarqube server & 重新运行sonar-scanner
  * 在server上可看到objectivec代码被检查，bingo~
  * 可能存在问题：
    * ERROR: The rule 'OCLint:compiler warning' does not exist.
    * 原因是开源的sonar-objective-c plugin没有包含这条rule，而oclint检出了这个问题。几种解决方案：
      * oclint.xml 去掉相关报错
      * 修改sonar-objective-c plugin，添加rule并打包—推荐
        * 实际上，rule文件被存放在/sonar-objective-c-plugin/src/main/resources/org/sonar/plugins/oclint文件夹下，包含两个文件：profile-oclint.xml 和 rules.txt
        * 参照其他rule，添加compiler warning即可
        * 重新打jar包、替换、重启server 、 sonar-scanner...


  
参考链接：
1.  [iOS 测试 iOS 静态代码扫描平台 Sonarqube 实战 Objective-C、Swift](https://testerhome.com/topics/13158?locale=en)
2.  [jenkins和sonar环境配置MAC版](https://bobo892589.github.io/2017/07/10/jenkins-sonar-mac/)
3.  [ERROR: The rule 'OCLint:compiler warning' does not exist.](https://github.com/Backelite/sonar-objective-c/issues/45)
4.  [Objective C静态代码扫描和代码质量管理 OClint + SonarQube](http://www.cnblogs.com/ishawn/p/3959521.html)





