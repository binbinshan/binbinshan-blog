# Maven系列（二）快速入门
- [Maven系列（二）快速入门](#maven系列二快速入门)
  - [1.快速体验Maven](#1快速体验maven)
    - [1.安装Maven](#1安装maven)
    - [2.快速入门HelloWorld](#2快速入门helloworld)
    - [3.pom.xml的简单介绍](#3pomxml的简单介绍)
    - [4.对项目进行打包](#4对项目进行打包)
    - [5.执行Jar包](#5执行jar包)
  - [2.Maven的体系结构](#2maven的体系结构)
  - [3.IntelliJ IDEA 生成Maven工程](#3intellij-idea-生成maven工程)
## 1.快速体验Maven
### 1.安装Maven
安装Maven的过程这里不进行介绍，谷歌即可，这里主要说几个重点：
1. 确保安装JDK
2. 需要配置Maven环境变量
3. 设置MAVEN_OPTS环境变量，Maven也是Jave的项目，所以也可能出现内存不够，导致构建失败，设置MAVEN_OPTS环境变量，设置为-Xms128m -Xmx512m
4. settings.xml默认是在%M2_HOME%/conf目录下，如果升级maven的版本，那么可能导致settings文件被覆盖为空，所以可以放在当前用户的目录下的~/.m2/settings.xml


### 2.快速入门HelloWorld

为了加快国内的速度，可以使用阿里云镜像仓库进行，在settings.xml中加一段配置。
```
<mirror>
    <id>nexus-aliyun</id>
    <mirrorOf>*</mirrorOf>
    <name>Nexus aliyun</name>
    <url>http://maven.aliyun.com/nexus/content/groups/public</url>
</mirror>
```

在某个目录下，通过CMD/终端执行如下命令
```
mvn archetype:generate -DgroupId=com.binbinshan.maven -DartifactId=helloWorld -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
```

如图，就会得到一个工程名为helloWorld的工程，也包括了基本的maven工程目录结构
* src/main/java 目录包含了这个项目的java源码
* src/test/java 目录包含了测试代码
* pom.xml 文件是 maven 的核心配置文件，是这个项目的 Project Object Model。

<div style="width:500px;height:500px;display:flex;justify-content:center;align-items:center;">
    <img src="https://github.com/binbinshan/binbinshan-blog/maven/img/16308391670305/16308401703908.jpg" />
</div>

### 3.pom.xml的简单介绍
pom.xml文件是一个项目最核心的maven配置文件，maven正是基于这里的配置信息来对工程进行构建等管理工作。

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.binbinshan.maven</groupId>
  <artifactId>helloWorld</artifactId>
  <packaging>jar</packaging>
  <version>1.0-SNAPSHOT</version>
  <name>helloWorld</name>
  <url>http://maven.apache.org</url>
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
</project>
```

这是通过快速入门构建出来的一个基础Pom.xml，其中包含了
* \<project\>：pom.xml中的顶层元素
* \<modelVersion\>：POM本身的版本号，一般很少变化
* \<groupId\>：创建这个项目的公司或者组织,比如com.baidu，或者cn.binbinshan
* \<artifactId\>：这个项目的唯一标识，一般生成的jar包名称
* \<packaging\>：要用的打包类型，比如jar，war
* \<version\>：这个项目的版本号
* \<name\>：这个项目用于展示的名称，一般在生成文档的时候使用
* \<url\>：这是这个项目的文档能下载的站点url，一般用于生成文档
* \<description\>：用于项目的描述，比如依赖了Junit

### 4.对项目进行打包
基于快速入门的Maven示例，就可以进行自动化运行测试用例+编译+打包。在项目目录下使用mvn package命令，对一个工程进行构建，构建出来一个可以执行的java jar包。

通过执行该命令可以发现，Maven执行compile、testResources、testCompile、test、jar。
* 自动下载了Junit的Jar包
* 自动化运行了单元测试的用例
* 自动化把java源代码编译成了.class文件
* 自动化把代码打包成了一个jar包

### 5.执行Jar包
java -cp target/helloWorld-1.0-SNAPSHOT.jar com.binbinshan.maven.App


## 2.Maven的体系结构

一张图解释Maven的体系结构

<div style="width:500px;height:200px;display:flex;justify-content:center;align-items:center;">
    <img src="https://github.com/binbinshan/binbinshan-blog/maven/img/16308391670305/16308420669711.jpg" />
</div>


1. settings.xml 是 Maven 配置相关的东西，所以 Maven 一定是依赖于 settings.xml 的。
2. pom.xml 则是工程相关的配置和依赖，那么 Maven 肯定也是需要进行解析的。
3. 当工程需要一个依赖的时候，Maven 会现在本地仓库寻找。
4. 如果本地仓库不存在，则会去远程仓库寻找，如果找到，则会在本地仓库放一份。

Tips: Maven远程中央仓库的地址配置可以在 %M2_HOME%/lib 下面找到 maven-model-builder-3.5.2.jar，在pom-4.0.0.xml中找到 https://repo.maven.apache.org/maven2/

## 3.IntelliJ IDEA 生成Maven工程

前面快速入门中，使用的是命令的方式生成Maven工程，这里通过 IntelliJ IDEA来生成一个Maven工程。
1. 使用archetype quickstart原型创建
<div style="width:678px;height:600px;display:flex;justify-content:center;align-items:center;">
    <img src="https://github.com/binbinshan/binbinshan-blog/maven/img/16308391670305/16308428948089.jpg" />
</div>

2. 设置GroupId、ArtifactId、Version
<div style="width:678px;height:600px;display:flex;justify-content:center;align-items:center;">
    <img src="https://github.com/binbinshan/binbinshan-blog/maven/img/16308391670305/16308430541945.jpg" />
</div>

3. 编写测试代码
```
//src/main/java
public class App {

    public static void main(String[] args) {
        System.out.println("hello world......");
    }

    public String sayHello(String name) {
        return "hello, " + name;
    }
}

//src/main/test
public class AppTest {
    @Test
    public void testSayHello() {
        App app = new App();
        String result = app.sayHello("binbinshan");
        assertEquals("hello, binbinshan", result);
    }
}
```

4. 创建Maven 命令
创建一个 Maven clean package
<div style="width:500px;height:300px;display:flex;justify-content:center;align-items:center;">
    <img src="https://github.com/binbinshan/binbinshan-blog/maven/img/16308391670305/16308437089821.jpg" ></img>
</div>

执行后，Maven 就会进行clean 和 package


