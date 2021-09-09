# Maven系列（三）依赖管理
- [Maven系列（三）依赖管理](#maven系列三依赖管理)
  - [1.坐标](#1坐标)
    - [1.坐标的意义](#1坐标的意义)
    - [2.坐标的元素](#2坐标的元素)
  - [2.依赖管理机制](#2依赖管理机制)
    - [1.依赖范围](#1依赖范围)
    - [2.传递性依赖](#2传递性依赖)
    - [3.传递性依赖的多层级的依赖范围](#3传递性依赖的多层级的依赖范围)
    - [4.依赖调解](#4依赖调解)
    - [5.可选依赖](#5可选依赖)
  - [3.依赖冲突](#3依赖冲突)
    - [1.常见的依赖冲突问题](#1常见的依赖冲突问题)
    - [2.如何解决依赖冲突](#2如何解决依赖冲突)
  - [4.仓库](#4仓库)
    - [1.仓库的类型](#1仓库的类型)
    - [2.使用nexus搭建私服](#2使用nexus搭建私服)
    - [3.nexus中的仓库类型](#3nexus中的仓库类型)
    - [4.私服设置](#4私服设置)
    - [5.镜像+repository强制从私服下载依赖](#5镜像repository强制从私服下载依赖)
      - [1.settings配置](#1settings配置)
    - [6.使用专用账号部署构建包到私服](#6使用专用账号部署构建包到私服)
      - [1.新建角色](#1新建角色)
      - [2.新建用户](#2新建用户)
      - [3.修改settings.xml](#3修改settingsxml)
      - [4.修改maven项目的pom.xml](#4修改maven项目的pomxml)
      - [5.执行deploy命令上传](#5执行deploy命令上传)
本篇将会介绍 Maven 的依赖管理，包括坐标、依赖管理机制、依赖冲突、仓库几部分。

## 1.坐标
每个 Maven 项目都有一个坐标，而坐标是通过 groupId + artifactId + version + packaging + classifier，五个维度，唯一定位一个依赖包。前三个常用，后两个packaging + classifier 几乎没用。

### 1.坐标的意义
1. 在 maven 项目的 pom.xml 里配置了需要的依赖之后，通过 groupId+artifactId+version 就定位了这个依赖在某个时间点的一个特定版本的代码，也就是一个特定的版本的代码的jar包 ，然后 maven 自动从中央仓库下载后缓存到本地仓库里，并且在打包的时候，可以通过插件直接将本地仓库中的依赖打入jar包中，形成一个完整可用的发布包。

2. 自己的maven项目如果要提供给别的项目使用，就需要将这个版本的代码打成一个jar包，放到仓库里去，给公司里其他人去依赖和使用。所以也有一个坐标，才能让别人唯一定位自己的项目某个版本的代码的jar包，然后让maven下载了之后给别人用。


### 2.坐标的元素
* groupId : 一般使用公司或者组织的官网的域名倒序来开头，例如com.baidu 、com.feihe
* artifactId : 项目中的某个模块，或者某个服务
* version : 这个工程的版本号
* packaging : 这个工程的发布包打包方式，一般常用的就jar和war两种
* classifier : 很少用，定义某个工程的附属项目，比如hello-world工程的，hello-world-source工程，就是源码，可能是类似于hello-world-1.0-SNAPSHOT-source.jar这样的东西


## 2.依赖管理机制

在pom文件中用\<dependency\>可以引用任何需要的依赖，比如spring、mybatis、log4j...，通过在声明 groupId、artifactId、version 三要素，一个坐标唯一定位了一个依赖的某个版本的jar包，maven会自动到远程的中央仓库里面去，给我们去找，下载到本地来，在打包、编译的时候，就会自动使用。

### 1.依赖范围
通过\<scope\>\</scope\> 可以声明依赖的作用范围。通过不同的依赖范围，会导致依赖包可能在编译、测试、打包运行的时候，有时候可以使用，有时候不能够使用。

```
classpath: classpath是JVM用到的一个环境变量，就是一组目录的集合，它用来指示JVM如何搜索class。

比如classpath 为 .;C:\work\project1\bin;C:\shared ,当JVM在加载abc.xyz.Hello这个类时，会依次查找：
//.代表当前目录
<当前目录>\abc\xyz\Hello.class
C:\work\project1\bin\abc\xyz\Hello.class
C:\shared\abc\xyz\Hello.class

如果JVM在某个路径下找到了对应的class文件，就不再往后继续搜索。如果所有路径下都没有找到，就报错。
```
在 Maven 中有三套 classpath ：
* 编译源代码的时候有一套 classpath 
* 在编译测试代码以及执行测试代码的时候，有一套 classpath ；
* 运行项目的时候，有一套 classpath ；

依赖范围就是用来控制依赖包与这三种classpath的关系的。

四种依赖范围：
1. compile：默认，对编译、测试和运行的 classpath 都有效。一般都是用这种scope。

3. test：仅仅对于运行测试代码的 classpath 有效，编译或者运行主代码的时候无效，仅仅测试代码需要用的依赖一般都会设置为这个范围，比如junit。
4. provided：编译和测试的时候有效，但是在运行的时候无效，因为可能环境已经提供了，比如servlet-api，servlet容器会提供依赖。
5. runtime：测试和运行classpath有效，但是编译代码时无效，比如jdbc的驱动实现类、mysql驱动


### 2.传递性依赖
每个依赖可能又有其他的依赖，其他依赖又有其他的依赖，循环往复。比如mybatis又依赖了cglib、log4j、commons-logging等依赖。

纯手工时期，可能就是先加入第一层依赖，然后报错；接着加入第二层依赖，再报错，继续加入第三层依赖；循环往复，极其麻烦。

有了 Maven 之后，自动递归解析所有的依赖，然后负责将依赖下载下来，接着所有层级的依赖，都会成为我们的项目的依赖，不需要手工干预。不管有多少层级，所有需要的依赖都会全部下载下来。这个就是Maven 的传递性依赖。

### 3.传递性依赖的多层级的依赖范围
比如说，我们依赖于 A，依赖范围是 compile； A依赖于 B，是 test； 我们对 B的依赖范围就是空，不会去依赖B，因为 A只有在测试的时候才会使用 B。我们依赖 A是生产用的，B是给 A测试的。

下面的表格展示了传递性依赖机制对依赖范围影响，第一列是一级依赖，第一行是二级依赖：


|  |compile|test|provided|runtime|
| --- | --- | --- | --- | --- |
|compile|compile|  |  |runtime|
|test|test|  |  |test|
|provided|provided|  |provided|provided|
|runtime|runtime|  |  |runtime|


### 4.依赖调解
maven会自动解析所有层级的依赖，自动下载所有的依赖，就可能会出现依赖冲突的问题，maven 会自动进行依赖调解。

依赖调节的两个原则：
1. 就近原则
2. 第一声明原则

比如 A->B->C->X(1.0)，A->D->X(2.0)，A有两个传递性依赖X，不同的版本。

就近原则：离A最近的选用，就是X的2.0版本
第一声明原则：如果A->B->X(1.0)和A->D->X(2.0)，路径等长，B和D哪个依赖在A的 pom.xml里先声明，就用哪个。


### 5.可选依赖
通过 \<optional\>true\</optional\> 可以设置依赖是否为可选依赖。

如果A依赖于B，B依赖于C，B对C的依赖是optional，那么A就不会依赖于C。反之，如果没有optional，根据传递性依赖机制，A会依赖于C。


## 3.依赖冲突
maven 会自动依赖调解，对依赖项目不同的版本选择一个版本来使用，那么调解的是个错误的版本呢？

### 1.常见的依赖冲突问题
我们的项目依赖了 A 和 B，此时 A 依赖了 C 的1.0版本，B 依赖了 D， D 依赖了C的2.0。
因为 A 先于 B 声明，根据依赖调解的就近原则和第一声明原则，默认使用 C 的1.0版本。而在1.0版本中没有我们需要的方法，那么就会报错。

### 2.如何解决依赖冲突

1. 定位冲突依赖
    执行 mvn dependency:tree ，用 mvn dependency:tree 这个命令看一下整个项目的依赖路径树，看看所有的依赖路径里，有哪几个版本的依赖重复了。

1. 手动排除依赖
    在需要排除的依赖，比如排除依赖 A 中的 C，那么就在 A 的依赖声明中 排除 C。
    ```java
    <dependency>
    <groupId>A</groupId>
    <artifactId>A</artifactId>
    <version>1.0</version>
    <exclusions>
    		<exclusion>
    			<groupId>C</groupId>
    			<artifactId>C</artifactId>
    		</exclusion>
    </exclusions>
    </dependency>
    ```


## 4.仓库
仓库是统一存放各种依赖的地方，哪怕是有几十个工程，但是每个工程如果有相同的依赖，那么那个依赖在仓库里只会存在一次。

### 1.仓库的类型
maven仓库的大类分成本地仓库和远程仓库两种，本地仓库没有想要的依赖话就会去远程仓库寻找。

* 本地仓库：默认是在.m2/repository下，可以在settings.xml配置文件里设置本地仓库路径。
* 中央仓库：maven有一个自带的超级pom.xml文件，里面配置了一个默认的中央仓库，如果本地仓库没有，就会去maven自带配置的这个中央仓库去拉取。
* 私服：一般在公司局域网内部，都会有公司的私服，在本地配置远程仓库为私服，如果本地仓库没有，先去私服找，如果私服没有，再去中央仓库找。在中央仓库找到后，先缓存在私服中，然后再缓存在本地仓库中。
* 其他远程仓库：比如jboss的仓库。java.net，google，codehaus，jboss，还有一些其他公司自己搞的Maven仓库
* 镜像仓库：国内的一些大型的互联网公司，比如阿里云，会搞一个镜像仓库，完全跟中央仓库一模一样的，代理了中央仓库所有的请求。就可以通过镜像仓库下载。

其中多层仓库的架构图如下：
<div style="width:700px;height:200px;display:flex;justify-content:center;align-items:center;">
    <img src="https://github.com/binbinshan/binbinshan-blog/blob/master/maven/img/16308392581454/16309378055718.jpg" ></img>
</div>

### 2.使用nexus搭建私服

1. 通过 https://help.sonatype.com/repomanager3/download 下载指定版本的nexus。

2. 解压 nexus，包含两个子目录  nexus-3.34.0-01 （包含nexus运行需要的文件） 和  sonatype-work (包含nexus的配置文件、日志文件、仓库文件)
3. 进入 nexus -> bin ，执行 ./nexus run 启动 nexus ,默认端口 8081
4. 通过 ip+端口 访问，账户默认 admin,密码可以在 /sonatype-work/nexus3/admin.password 中找到。
5. 登录后会有向导，指引下一步：
    1. 设置密码
    <div style="width:500px;height:280px;display:flex;justify-content:center;align-items:center;">
        <img src="https://github.com/binbinshan/binbinshan-blog/blob/master/maven/img/16308392581454/16309726793849.jpg" ></img>
    </div>
    
    2. 是否启用匿名访问
    <div style="width:500px;height:280px;display:flex;justify-content:center;align-items:center;">
        <img src="https://github.com/binbinshan/binbinshan-blog/blob/master/maven/img/16308392581454/16309727594755.jpg" ></img>
    </div>
    
    3. 是否帮助改进计划
    <div style="width:500px;height:280px;display:flex;justify-content:center;align-items:center;">
        <img src="https://github.com/binbinshan/binbinshan-blog/blob/master/maven/img/16308392581454/16309729091932.jpg" ></img>
    </div>
    

### 3.nexus中的仓库类型
nexus安装好之后本身就内置了一些仓库

* maven-central：maven中央仓库的代理仓库，策略为 Release ，只会缓存中央仓库中的发布版本的构件

* maven-releases ： 该仓库是个宿主仓库，用于部署公司内部的release版本的发布包

* maven-snapshots ：该仓库是个宿主仓库，用于部署公司内部的snapshots版本的发布包

* maven-public ：仓库组，所有存储策略为 Release 的仓库聚合并通过统一的地址提供服务

下图为nexus仓库的架构图：

<div style="width:1100px;height:400px;display:flex;justify-content:center;align-items:center;">
        <img src="https://github.com/binbinshan/binbinshan-blog/blob/master/maven/img/16308392581454/16310196605677.jpg" ></img>
    </div>


### 4.私服设置
1. 修改maven-central 从直接中央仓库代理 改为 阿里云代理

    <div style="width:800px;height:400px;display:flex;justify-content:center;align-items:center;">
        <img src="https://github.com/binbinshan/binbinshan-blog/blob/master/maven/img/16308392581454/16310252967006.jpg" ></img>
    </div>

1. 创建一个3rd-party仓库，存放第三方厂商的不开源商用依赖
    <div style="width:800px;height:400px;display:flex;justify-content:center;align-items:center;">
        <img src="https://github.com/binbinshan/binbinshan-blog/blob/master/maven/img/16308392581454/16310254351662.jpg" ></img>
    </div>
    
    选择类型
    <div style="width:300px;height:400px;display:flex;justify-content:center;align-items:center;">
        <img src="https://github.com/binbinshan/binbinshan-blog/blob/master/maven/img/16308392581454/16310254928771.jpg" ></img>
    </div>
    
    <div style="width:600px;height:450px;display:flex;justify-content:center;align-items:center;">
        <img src="https://github.com/binbinshan/binbinshan-blog/blob/master/maven/img/16308392581454/16310256283021.jpg" ></img>
    </div>
    
    加入public组
    <div style="width:600px;height:450px;display:flex;justify-content:center;align-items:center;">
        <img src="https://github.com/binbinshan/binbinshan-blog/blob/master/maven/img/16308392581454/16310257338928.jpg" ></img>
    </div>

### 5.镜像+repository强制从私服下载依赖

#### 1.settings配置
在settings文件中添加配置，强制maven从私服下载，不在从中央仓库下载。

在/<profiles/>标签中添加下列内容，覆盖maven默认中央仓库central
```
<profile>
      <id>nexus</id>
          <repositories>
              <repository>
                  <id>central</id>
                  <name>Nexus </name>
                <url>http://central</url>
                  <releases><enabled>true</enabled></releases>
                  <snapshots><enabled>true</enabled></snapshots>
              </repository>
          </repositories>
          <pluginRepositories>
              <pluginRepository>
                  <id>central</id>
                  <name>Nexus Plugin Repository</name>
              <url>http://central</url>
                  <releases><enabled>true</enabled></releases>
                  <snapshots><enabled>true</enabled></snapshots>
              </pluginRepository>
          </pluginRepositories>
    </profile>
```

在/<mirrors/>标签中配置mirror，强制所有依赖从私服下载

```
  <mirror>
    <id>nexus</id>
    <mirrorOf>*</mirrorOf>
    <url>http://ip地址:8081/repository/maven-public/</url>
  </mirror>
```

在/<server/>中配置私服用户名密码，以免权限不够，无法下载
```
<server>
  <id>nexus</id>
  <username>admin</username>
  <password>admin</password>
</server>
```

配置完成后，在maven项目pom文件中依赖kafka，在私服的仓库中就会下载并缓存该依赖包。

<div style="width:1000px;height:600px;display:flex;justify-content:center;align-items:center;">
        <img src="https://github.com/binbinshan/binbinshan-blog/blob/master/maven/img/16308392581454/16311146140132.jpg" ></img>
    </div>



### 6.使用专用账号部署构建包到私服
在平常的开发中，需要将自己的工程打包后发布到私服，那么需要有一个专用的部署账号，通过账号认证，才能部署发布包到nexus私服的。这个账号最少包括：
1. 涵盖所有匿名账号的权限，至少可以搜索仓库，下载依赖
2. 对仓库有所有的管理权限，就可以往仓库中去部署发布包

#### 1.新建角色
<div style="width:1000px;height:550px;display:flex;justify-content:center;align-items:center;">
        <img src="https://github.com/binbinshan/binbinshan-blog/blob/master/maven/img/16308392581454/16312293139772.jpg" ></img>
    </div>


创建一个deployment角色，包含了一下权限：
<div style="width:800px;height:550px;display:flex;justify-content:center;align-items:center;">
        <img src="https://github.com/binbinshan/binbinshan-blog/blob/master/maven/img/16308392581454/16312294103664.jpg" ></img>
    </div>

保存角色

#### 2.新建用户

<div style="width:800px;height:500px;display:flex;justify-content:center;align-items:center;">
        <img src="https://github.com/binbinshan/binbinshan-blog/blob/master/maven/img/16308392581454/16312294720540.jpg" ></img>
    </div>

创建用户，赋值刚刚创建的角色

<div style="width:800px;height:450px;display:flex;justify-content:center;align-items:center;">
        <img src="https://github.com/binbinshan/binbinshan-blog/blob/master/maven/img/16308392581454/16312295314754.jpg" ></img>
    </div>

保存用户

#### 3.修改settings.xml
使用刚创建好的deployment用户，替换原本的admin用户。
```
     <server>
      <id>nexus-releases</id>
      <username>deployment</username>
      <password>deployment</password>
    </server>

    <server>
      <id>nexus-snapshots</id>
      <username>deployment</username>
      <password>deployment</password>
    </server>
```

#### 4.修改maven项目的pom.xml
在pom文件中，指定发布仓库的配置，也就是nexus私服中release和snapshot仓库的地址

```
  <distributionManagement>
    <repository>
      <id> nexus-releases</id>
      <name> Nexus Release Repository</name>
      <url>http://ip:8081/repository/maven-releases/</url>
    </repository>
    <snapshotRepository>
      <id> nexus-snapshots</id>
      <name> Nexus Snapshot Repository</name>
      <url>http://ip:8081/repository/maven-snapshots/</url>
    </snapshotRepository>
  </distributionManagement>
```

#### 5.执行deploy命令上传

执行mvn clean deploy，就会打包到nexus私服中

<div style="width:500px;height:300px;display:flex;justify-content:center;align-items:center;">
        <img src="https://github.com/binbinshan/binbinshan-blog/blob/master/maven/img/16308392581454/16312300465474.jpg" ></img>
    </div>







