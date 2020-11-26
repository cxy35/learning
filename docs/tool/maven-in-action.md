从零开始学习 Maven ，从入门到精通。
<!-- more -->

## 1 坐标

### 1.1 groupId

必须，定义当前项目隶属的**实际项目**。如 com.cxy35.account 。

### 1.2 artifactId

必须，定义实际项目中的一个**Maven项目或模块**，推荐使用实际项目名称作为 artifactId 的前缀。如account-web 。

### 1.3 version

必须，定义 Maven 项目当前所处的版本。如 1.0.0 或 1.0.0-SNAPSHOT 。

### 1.4 packaging

定义 Maven 项目的打包方式，默认 jar 。如 jar 或 war 。

### 1.5 classifier

用来帮助定义构建输出的一些附属附件，不能直接定义。

## 2 依赖

### 2.1 依赖的配置

在项目 pom.xml 文件中配置，当前项目有效，如：**`<dependencies><dependency>...</dependency>......</dependencies>`** 。元素包括：

- groupId、artifactId、version：依赖的基本坐标，必须。对于任何一个依赖来说，基本坐标是最重要的， Maven 根据坐标才能找到需要的依赖。
- type：依赖的类型，默认 jar ，对应坐标中的 packaging 。
- scope：依赖的范围，默认 compile 。
- optional：标记依赖是否可选。
- exclusions：用来排除传递性依赖。

### 2.2 依赖范围

- **compile**：编译依赖范围，默认该值。对编译、测试、运行三种 classpath 都有效，如 spring-core 。
- **test**：测试依赖范围。只对测试 classpath 有效，如 jUnit 。
- **provided**：已提供依赖范围。对编译和测试 classpath 有效，因为运行时容器已经提供，如 servlet-api 。
- **runtime**：运行时依赖范围。对测试和运行 classpath 有效，因为编译时用的是 JDK 提供的 JDBC 接口，如 JDBC 驱动实现，只有在执行测试或者运行项目的时候才需要实现上述接口的具体 JDBC 驱动。
- **system**：系统依赖范围。对编译和测试 classpath 有效，通过 systemPath 元素显示地指定本地依赖文件的路径，可能造成构建的不可移植，因此应该谨慎使用。
- **import**：导入依赖范围。对编译、测试、运行三种 classpath 都无效。

依赖范围与 classpath 的关系，如下图：

![](https://static.oschina.net/uploads/space/2018/0504/084726_tsrG_593078.png)

### 2.3 传递性依赖

如项目 A 有这样的依赖关系：A->B->C->X ，则 X 是 A 的传递性依赖。依赖范围影响传递性依赖，第一列表示第一直接依赖范围，第一行表示第二直接依赖范围，中间的交叉单元格则表示传递性依赖范围，如下图：

![](https://static.oschina.net/uploads/space/2018/0504/084833_5aW6_593078.png)

### 2.4 依赖调解

- **第一原则：路径最近者优先**：如项目 A 有这样的依赖关系：A->B->C->X(1.0) 和 A->D->X(2.0) ， X 是 A 的传递性依赖，但有 2 个版本，最终 X(2.0) 会被解析使用。
- **第二原则：第一申明者优先**：如项目A有这样的依赖关系：A->B->X(1.0) 和 A->D->X(2.0) ， X 是 A 的传递性依赖，但有 2 个版本，且路径相同，最终 X(1.0) 会被解析使用。

### 2.5 可选依赖

有时候我们不想让依赖传递，那么可配置该依赖为可选依赖，即将参数 optional 设置为 true ，这样其他项目用到时就不会得到此依赖的传递。

### 2.6 排除依赖

当我们引入第三方 jar 包的时候，难免会引入传递性依赖。如果不想引入，则可在依赖中配置去除对应的传递性依赖，如： **`<dependency><exclusions><exclusion>...</exclusion>......</exclusions></dependency>`** 。

## 3 仓库

在 Maven 世界中，任何一个依赖、插件或者项目构建的输出，都可以称为**构件**。仓库分为两类：**本地仓库和远程仓库**，如下图。

![](https://static.oschina.net/uploads/space/2018/0504/085317_IwJM_593078.png)

### 3.1 本地仓库

默认在当前用户的目录下，如：C:\Users\Administrator\.m2\repository 下。可在 settings.xml 文件中修改。 **`<localRepository>D:\.m2\repository</localRepository>`** 。

### 3.2 远程仓库

远程仓库分为中央仓库( http://repo1.maven.org/maven2 )、私服（一般搭建在局域网内）、其他公共库。

#### 3.2.1 远程仓库的配置

在项目 pom.xml 文件中配置，当前项目有效。在 setting.xml 文件中配置，全局有效。如 ：**`<repositories><repository>...</repository>......</repositories>`** 。

#### 3.2.3 远程仓库的认证

在 setting.xml 文件中配置，全局有效，如： **`<servers><server>...</server>......</servers>`** 。

#### 3.2.3 部署到远程仓库

在项目 pom.xml 文件中配置，当前项目有效，如： **`<distributionManagement><repository>...</repository><snapshotRepository>...</snapshotRepository></distributionManagement>`** 。

### 3.3 镜像

在 setting.xml 文件中配置，全局有效，如： **`<mirrors><mirror>...</mirror>......</mirrors>`** 。

## 4 生命周期

Maven 的生命周期是抽象的，其实际行为都**由插件来完成**，如 clean 阶段的任务由 maven-clean-plugin 完成。主要包含了项目的清理、初始化、编译、测试、打包、集成测试、验证、部署、站点生成等构建步骤。有 3 套生命周期，**每套生命周期包含一些阶段（phase）**，这些阶段有顺序，且后面的阶段依赖前面的阶段，但每套生命周期本身是相互独立的。

### 4.1 clean 生命周期（清理项目）

pre-clean、clean、post-clean 。

### 4.2 dafault 生命周期（构建项目）

- validate：验证项目是否正确和所有需要的相关资源是否可用。
- initialize：初始化构建。
- generate-sources  
- process-sources：处理项目主资源文件。一般来说，是对 src/main/resources 目录的内容进行变量替换等工作后，复制到项目输出的主 classpath 目录中。
- generate-resources  
- process-resources  
- **compile**：编译项目的主源码。一般来说，是编译 src/main/java 目录下的 Java 文件至项目输出的主 classpath 目录中。
- process-classes  
- generate-test-sources  
- process-test-sources：处理项目测试资源文件。一般来说，是对 src/test/resources 目录的内容进行变量替换等工作后，复制到项目输出的测试 classpath 目录中。
- generate-test-resources  
- process-test-resources  
- test-compile：编译项目的测试代码。一般来说，是编译 src/test/java 目录下的 Java 文件至项目输出的测试 classpath 目录中。
- process-test-classes  
- **test**：使用单元测试框架运行测试，测试代码不会被打包或部署。
- prepare-package：做好打包的准备。
- **package**：接受编译好的代码，打包成可发布的格式，如 JAR 。
- pre-integration-test  
- integration-test  
- post-integration-test  
- verify  
- **install**：将包安装到 Maven 本地仓库，供本地其他 Maven 项目使用。
- **deploy**：将最终的包复制到远程仓库，供其他开发人员和 Maven 项目使用。

### 4.3 site 生命周期（建立项目站点）

pre-site、site、post-site、site-deploy。

## 5 插件

### 5.1 插件目标( Plugin Goal )

**一个插件可以完成很多功能，每个功能就是一个插件目标**，如 dependency:list(或 maven-dependency-plugin:list )、dependency:tree( 或maven-dependency-plugin:tree )，**冒号前面的是插件前缀或插件，冒号后面的是目标**。

### 5.2 插件绑定

一些生命周期中的阶段会**内置绑定某个插件目标**，如 clean 生命周期的 clean 阶段内置绑定的插件目标是 maven-clean-plugin:clean 。

### 5.3 插件配置

在项目 pom.xml 文件中配置，当前项目有效，如： **`<build><plugins><plugin>...</plugin>......</plugins></build>`** 。在命令行中中配置，当前项目有效，使用 -D 参数，如： `mvn clean install -Dmaven.test.skip=true `。

## 6 聚合与继承

### 6.1 聚合

**目的是为了方便快速构建项目，分为聚合模块和被聚合模块**。聚合模块命名一般为： xxx-aggregator 或 xxx ，打包方式 packaging 为 pom ，可以是父子目录结构（推荐，聚合模块放在项目目录的最顶层），也可以是平行目录结构。在聚合模块的 pom.xml 文件中配置，如： **`<modules><module>...</module>......</modules>`** 。

### 6.2 继承

**目的是为了消除重复配置，分为父模块和子模块**。父模块命名一般为： xxx-parent ，打包方式 packaging 为 pom 。在子模块 pom.xml 文件中配置，如： **`<parent>...</parent>`** ，可由 relativePath 属性指定父模块的位置，默认认为在子模块的上一级。任何一个 maven项目都隐式地继承 1 个默认的超级 pom ，里面定义了一些默认的配置。

### 6.3 依赖管理

dependencyManagement 元素能让子模块继承到父模块的依赖配置，但不会引入实际的依赖，实际引入由子模块灵活控制，类似于接口与实现类。在父项目 pom.xml 文件中配置，如： **`<dependencyManagement><dependencies><dependency>...</dependency>......</dependencies></dependencyManagement>`** 。

### 6.4 插件管理

类似于依赖管理。在父项目 pom.xml 文件中配置，如： **`<pluginManagement><plugins><plugin>...</plugin>......</plugins></pluginManagement>`** 。

### 6.5 反应堆( Reactor )

所有模块组成的一个构建结构。对于单模块的项目，反应堆就是该模块本身，但对于多模块的项目，反应堆就包含了各模块之间继承与依赖的关系，从而能够自动计算出合理的模块构建顺序。

### 6.6 聚合与继承合并

可以用 xxx 或 xxx-parent 来**合并聚合与继承的功能**，既是聚合 pom ，又是父 pom 。

## 7 灵活的构建

Maven 为了支持构建的灵活性，内置了三大特性，即**属性、Profile 和资源过滤**。 Maven 用户可以在 POM 和资源文件中使用 Maven 属性表示那些可能变化的量，通过不同 profile 中的属性值和资源过滤特性为不同环境执行不同的构建。

### 7.1 Maven 属性（6类）

#### 7.1.1 内置属性

主要有两个常用内置属性， ${basedir} 表示项目根目录，即包含 pom.xml 文件的目录； ${version} 表示项目版本。

#### 7.1.2 POM 属性

用户可以使用该类属性引用 POM 文件中对应元素的值。例如 ${project.artifactId} 就对应了 `<project><artifactId>` 元素的值。常用的 POM 属性包括：

- ${project.build.sourceDirectory}：项目的主源码目录，默认为 src/main/java/ 。
- ${project.build.testSourceDirectory}：项目的测试源码目录，默认为 src/test/java/ 。
- ${project.build.directory}：项目构建输出目录，默认为 target/ 。
- ${project.outputDirectory}：项目的主代码编译输出目录，默认为 target/classes/ 。
- ${project.testOutputDirectory}：项目测试代码编译输出目录，默认为 target/test-classes/ 。
- ${project.groupId}：项目的 groupId 。
- ${project.artifactId}：项目的 artifactId 。
- ${project.version}：项目的 version ，与 ${version} 等价。
- ${project.build.finalName}：项目打包输出文件的名称，默认为 ${project.artifactId}-${project.version} 。

这些属性都对应了一个 POM 元素，它们中一些属性的默认值都是在超级 POM 中定义的。

#### 7.1.3 自定义属性

用户可以在 POM 的 `<properties>` 元素下自定义 Maven 属性。例如：

```xml
<project>  
    ...  
     <properties>  
          <my.prop>hello</my.prop>  
     </properties>  
     ...  
</project>  
```

然后在 POM 中其他地方使用 ${my.prop} 的时候会被替换成 hello 。

#### 7.1.4 Settings 属性

与 POM 属性同理，用户使用以 settings. 开头的属性引用 settings.xml 文件中 XML 元素的值，如常用的 `${settings.localRepository}` 指向用户本地仓库的地址。

#### 7.1.5 Java 系统属性

所有 Java 系统属性都可以使用 Maven 属性引用，例如 ${user.home} 指向了用户目录。用户可以使用 `mvn help:system` 查看所有的 Java 系统属性。

#### 7.1.6 环境变量属性

所有环境变量都可以使用以 env. 开头的 Maven 属性引用。例如 `${env.JAVA_HOME}` 指代了 JAVA_HOME 环境变量的值。用户可以使用 `mvn help:system` 查看所有的环境变量。

### 7.2 资源过滤

在项目 pom.xml 文件中配置，当前项目有效。如： `<build><resources><resource>...</resource>......</resources></build>` 。会使用 maven-resources-plugin 插件解析资源文件（如src/main/resources ）中的 Maven 属性，替换成对应的值，如数据库配置信息。

### 7.3 Maven Profile

在项目 pom.xml 文件中配置，当前项目有效。在 setting.xml 文件中配置，全局有效。如： **`<profiles><profile>...</profile>......</profiles>`** 。常用于配置不同环境下的数据库配置信息，如开发环境、测试环境、测试环境，再激活其中一个 profile ，如可用命令激活： `mvn clean install -Pdev` ， -P 参数表示在命令行激活一个 profile 。也可配置默认激活某一个 profile 。**当执行 Maven 构建的时候，激活的 profile 会将配置应用到项目中去。**

### 7.4 Web 资源过滤

在项目 pom.xml 文件中配置，当前项目有效。会使用 maven-war-plugin 插件解析 web 资源文件（如 src/main/webapp ）中的 Maven 属性，替换成对应的值，如 logo 图片或 css 主题。

## 8 使用 Nexus 搭建 Maven 私服

- Nexus 有两种安装包：一种是包含 Jetty 容器的 Bundle 包，另一种是不包含Web容器的 war 包。默认端口是 **8081** ，默认管理员用户名和密码是 **admin/admin123** 。
- Nexus 仓库类型有四种： **group** （仓库组，如 Public Repositories ）、 **hosted** （宿主，如 3rd party/Releases/Snapshots ）、 **proxy** （代理，如 Central ）、 **virtual** （虚拟）。
- Nexus通过维护仓库的索引来提供搜索功能。注意默认情况下，中央仓库的代理仓库（ Central ）中下载远程索引的配置（ **Download Remote Indexes** ）是关闭的，需要在配置中开启，这样才能搜索中央仓库中的构件。注意：对于在仓库浏览器中浏览或者搜索构件，其搜索的是存在 nexus 服务器本地 ${bundleBasedir}/../sonatype-work /nexus/storage 目录下的构件，如果找不到就找不到了。这些构件当 maven 应用配置指向 nexus 地址，并且 nexus 在自己层面找不到构件，才从相应的所代理的远程仓库中下载构件，并且存入 **storage** 目录。对于点击某仓库，查看其 "**Browse Remote"** ，则只要网络通，就总能看到某版本构件，因为你直接查看了远程仓库的构件索引。

## 9 常用命令

[Maven 常用命令](https://www.cxy35.com/2018/06/05/Maven/Maven%E5%B8%B8%E7%94%A8%E5%91%BD%E4%BB%A4/)

- mvn clean：清理 target 目录。
- mvn clean compile：清理，编译主代码与资源。
- mvn clean test：清理，编译主代码与资源，编译测试代码与资源，测试。
- mvn clean package：清理，编译主代码与资源，编译测试代码与资源，测试，打包。
- mvn clean install：清理，编译主代码与资源，编译测试代码与资源，测试，打包，安装到本地仓库。
- mvn clean deploy：清理，编译主代码与资源，编译测试代码与资源，测试，打包，安装到本地仓库，部署到远程仓库。
- mvn archetype:generate ：快速创建项目骨架。
- mvn dependency:list ：查看当前项目的已解析依赖。
- mvn dependency:tree ：查看当前项目的依赖属。
- mvn dependency:analyze ：分析当前项目的依赖。可分析出“项目中使用到的但没有显式声明的依赖”和“项目中未使用的但显式声明的依赖（注意只会分析编译时需要用到的依赖，测试和运行时的发现不了，删除时务必小心）”。

## 10 常用插件

- maven-clean-plugin（清理）
- maven-compiler-plugin（编译，可指定编译版本）
- maven-resources-plugin
- maven-war-plugin
- maven-source-plugin（创建源码jar包）
- maven-surefire-plugin（测试，自动运行单元测试，测试类的命名或目录需保持一定的规则）
- maven-shade-plugin（生成可执行的jar文件）
- maven-dependency-plugin（依赖相关的插件）
- maven-help-plugin（获取插件的详细信息）
- jetty-maven-plugin（测试，自动扫描项目并更新部署到 Web 容器中，帮助日常的快速开发和测试）
- cargo-maven2-plugin（自动化部署）
- mybatis-generator-maven-plugin（ mybatis 代码生成器）

## 11 常用仓库与查询

- 中央仓库：[http://repo1.maven.org/maven2/](http://repo1.maven.org/maven2/)
- 中央仓库：[https://repo.maven.apache.org/maven2/](https://repo.maven.apache.org/maven2/)
- 查询：[http://search.maven.org/](http://search.maven.org/)
- 查询：[http://mvnrepository.com/](http://mvnrepository.com/)


---

扫码关注微信公众号 **程序员35** ，获取最新技术干货，畅聊 ##程序员的35，35的程序员## 。独立站点：[https://cxy35.com](https://cxy35.com)

![](https://oscimg.oschina.net/oscnet/up-285838b9c516db5bb1ba760f292f2346078.JPEG)