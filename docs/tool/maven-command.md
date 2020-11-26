---
title: Maven 常用命令
date: 2018-06-05 10:21:41
categories: Maven
tags: [Maven, 命令]
toc: true
---
Maven 常用命令。
<!-- more -->

- 创建 Maven 的普通 java 项目：mvn archetype:generate -DgroupId=com.demo -DartifactId=demo_maven -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
- 创建 Maven 的 Web 项目：mvn archetype:generate -DgroupId=com.demo -DartifactId=demo_maven2 -DarchetypeArtifactId=maven-archetype-webapp -DinteractiveMode=false
- 编译源代码： mvn compile
- 编译测试代码：mvn test-compile
- 运行测试：mvn test
- 产生 site：mvn site
- 打包：mvn package
- 在本地 Repository 中安装 jar：mvn install
- 清除产生的项目：mvn clean
- 跳过测试运行：mvn package -DskipTests
- 跳过测试编译和运行：mvn package -Dmaven.test.skip=true
- 只打 jar 包: mvn jar:jar
- 生成 idea 项目：mvn idea:idea
- 生成 eclipse 项目：mvn eclipse:eclipse
- 清除 eclipse 的一些系统设置:mvn eclipse:clean
- 运行项目于 jetty 上：mvn jetty:run
- 显示版本信息：mvn -version/-v
- 显示详细错误信息：mvn -e
- 验证工程是否正确，所有需要的资源是否可用：mvn validate
- 在集成测试可以运行的环境中处理和发布包：mvn integration-test
- 运行任何检查，验证包是否有效且达到质量标准：mvn verify
- 产生应用需要的任何额外的源代码，如 xdoclet：mvn generate-sources
- 删除再编译：mvn clean install
- 打印出已解决依赖的列表：mvn dependency:resolve
- 打印整个依赖树：mvn dependency:tree
- 想要查看完整的依赖踪迹，包含那些因为冲突或者其它原因而被拒绝引入的构件，打开 Maven 的调试标记运行：mvn install -X 
- 构建装配 Maven Assembly 插件是一个用来创建你应用程序特有分发包的插件：mvn install assembly:assembly
- 使用 Hibernate3 插件构造数据库：mvn hibernate3:hbm2ddl
- 使用 help 插件的 describe 目标来输出 Maven Help 插件的信息：mvn help:describe -Dplugin=help
- 使用 Help 插件输出完整的带有参数的目标列：mvn help:describe -Dplugin=help -Dfull
- 获取单个目标的信息,设置 mojo 参数和 plugin 参数。此命令列出了 Compiler 插件的 compile 目标的所有信息：mvn help:describe -Dplugin=compiler -Dmojo=compile -Dfull
- 列出所有 Maven Exec 插件可用的目标：mvn help:describe -Dplugin=exec -Dfull
- 看这个“有效的 (effective)”POM ，它暴露了 Maven 的默认设置：mvn help:effective-pom
- Main Exec 插件让我们能够在不往 classpath 载入适当的依赖的情况下，运行这个程序：mvn exec:java -Dexec.mainClass=org.sonatype.mavenbook.weather
- 打包同时生成源码包：mvn clean source:jar install

---

- 参数1：-DdownloadSources=true（构建项目时下载源码 jar ）
- 参数2：-DdownloadJavadocs=true（构建项目时下载 javadoc 包）
- 参数3：-Dwtpversion=2.0（构建项目时表示是 web 项目,而不是简单的 java 项目）
- 参数4：-Dmaven.test.skip=true（ install 时跳过测试）

---

- 示例1：【**eclipse.bat**】

SET MAVEN_OPTS= -Xms512M -Xmx512M -XX:PermSize=128M -XX:MaxPermSize=128M -XX:ReservedCodeCacheSize=64M  
mvn eclipse:clean eclipse:eclipse -DdownloadSources=true -Dwtpversion=2.0

- 示例2：【**install.bat**】

SET MAVEN_OPTS= -Xms512M -Xmx512M -XX:PermSize=128M -XX:MaxPermSize=128M -XX:ReservedCodeCacheSize=64M  
mvn clean source:jar install -Dmaven.test.skip=true


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 #程序员的35，35的程序员# 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)