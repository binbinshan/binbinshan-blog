# Maven系列（四）构建原理
- [Maven系列（四）构建原理](#maven系列四构建原理)
  - [1.构建生命周期](#1构建生命周期)
    - [1.生命周期](#1生命周期)
    - [2.phase](#2phase)
    - [3.示例](#3示例)
  - [2.默认的phase和plugin绑定](#2默认的phase和plugin绑定)
  - [3.maven的命令行和生命周期](#3maven的命令行和生命周期)
  - [4. plugin和goal](#4-plugin和goal)
    - [1.将插件的goal绑定到phase上](#1将插件的goal绑定到phase上)
    - [2.配置插件](#2配置插件)
    - [3.插件解析](#3插件解析)

## 1.构建生命周期

### 1.生命周期
Maven 构建生命周期定义了一个项目构建跟发布的过程。一个典型的 Maven 构建（build）生命周期是由以下几个阶段的序列组成的：

<div style="width:1000px;height:100px;display:flex;justify-content:center;align-items:center;">
    <img src="https://github.com/binbinshan/binbinshan-blog/blob/master/maven/img/16308392826979/16314399456810.jpg" ></img>
</div>


而maven有三套完全独立的生命周期 clean，default和site。
每套生命周期都可以独立运行，每个生命周期的运行都会包含**多个phase**，每个phase又是由各种**插件的goal**来完成的，一个插件的goal可以认为是一个功能。

<div style="width:800px;height:300px;display:flex;justify-content:center;align-items:center;">
    <img src="https://github.com/binbinshan/binbinshan-blog/blob/master/maven/img/16308392826979/16314391638510.jpg" ></img>
</div>

所以每次执行一个生命周期，都会依次执行这个生命周期内部的多个phase，每个phase执行时都会执行某个插件的goal完成具体的功能。

### 2.phase

1. clean生命周期包含的phase 
    * pre-clean、clean、post-clean

2. default生命周期包含的23个phase
    ```
    validate：校验这个项目的一些配置信息是否正确
    initialize：初始化构建状态，比如设置一些属性，或者创建一些目录
    generate-sources：自动生成一些源代码，然后包含在项目代码中一起编译
    process-sources：处理源代码，比如做一些占位符的替换
    generate-resources：生成资源文件
    process-resources：将资源文件拷贝到目标目录中，方便后面打包
    compile：编译项目的源代码
    process-classes：处理编译后的代码文件，比如对java class进行字节码增强
    generate-test-sources：自动化生成测试代码
    process-test-sources：处理测试代码，比如过滤一些占位符
    generate-test-resources：生成测试用的资源文件
    process-test-resources：拷贝测试用的资源文件到目标目录中
    test-compile：编译测试代码
    process-test-classes：对编译后的测试代码进行处理，比如进行字节码增强
    test：使用单元测试框架运行测试
    prepare-package：在打包之前进行准备工作，比如处理package的版本号
    package：将代码进行打包，比如jar包
    pre-integration-test：在集成测试之前进行准备工作，比如建立好需要的环境
    integration-test：将package部署到一个环境中以运行集成测试
    post-integration-test：在集成测试之后执行一些操作，比如清理测试环境
    verify：对package进行一些检查来确保质量过关
    install：将package安装到本地仓库中，这样开发人员自己在本地就可以使用了
    deploy：将package上传到远程仓库中，这样公司内其他开发人员也可以使用了
    ```


3. site生命周期的包含的phase
    * pre-site、site、post-site、site-deploy

    
### 3.示例
假设执行mvn clean命令，Maven会调用clean 生命周期，在一个生命周期中，运行某个阶段的时候，它之前的所有阶段都会被运行，也就是说，如果执行 mvn clean 将运行以下两个生命周期阶段 mvn pre-clean 、mvn clean ，而 mvn post-clean则不会执行。

假设执行mvn install命令，Maven会调用 default 生命周期，并且执行全部23个phase中的22个，也就是deploy前面的所有phase。 


## 2.默认的phase和plugin绑定
在构建环境中，使用 mvn clean package 来纯净地构建和部署项目到共享仓库中，那么每个phase都是由插件的goal来完成的，phase和plugin绑定关系是？

其实默认maven就绑定了一些plugin goal到phase上去，比如：
* process-resources 绑定了 resources:resources
* compile	绑定了 compiler:compile
* test-compile 绑定了 compiler:testCompile
* test	绑定了 surefire:test
* package 绑定了 jar:jar或者war:war
* install	绑定了 install:install
* deploy	绑定了 deploy:deploy
* clean	绑定了 clean:clean

tips：resources:resources这种格式，说的就是resources这个plugin的resources goal（resources功能，负责处理资源文件）

## 3.maven的命令行和生命周期

比如 mvn clean package，clean是指的clean生命周期中的clean phase，package是指的default生命周期中的package phase。

此时就会执行clean生命周期中，在clean phase之前的所有phase和clean phase，pre clean，clean，同时会执行default生命周期中，在package phase之前的所有phase和package phase。
但是在clean 生命周期中，pre clean phase 默认是没有绑定任何一个plugin goal的，所以默认什么也不会干；而 clean phase默认是绑定了clean:clean，也就是 clean plugin的clean goal，所以就会去执行clean插件的clean goal，就会实现一个功能，就是清理target目录下的文件。

当然也可以不执行任何一个生命周期的任何一个phase，直接执行指定的插件的一个goal：
* 直接maven执行 mvn dependency:tree，直接执行dependency这个插件的tree这个goal，然后递归解析所有的依赖，然后打印出一颗依赖树
* 直接maven执行 mvn deploy:depoy-file，就是直接执行deploy这个插件的deploy-file这个goal，将指定目录的jar包，以指定的坐标，部署到指标的maven私服仓库里去，同时使用指定仓库id对应的server的账号和密码。

## 4. plugin和goal

默认情况下，少数一些phase绑定了一些maven内置的插件的goal，大量的phase其实是空置的。而一些特殊的功能maven默认其实是不支持的。
比如说现在需要打出来的一个jar包，放到生产环境去跑的时候，那么肯定需要将所有依赖的jar包都打在这个jar包里面的，这样在实际程序运行的时候，才能找到所有需要的依赖。就可以用assembly 插件，配置在里面，打包的时候，将所有依赖的jar包都放入我们自己工程的jar包里。

如果要使用别人的插件在我们的项目里实现某些特殊的功能，此时就可以在pom.xml里面配置那个插件，包括最重要的，配置那个插件绑定到哪个phase上去。而在maven的命令执行的时候，默认会执行生命周期中的各种phase，如果绑定了某个第三方的插件到phase，此时插件就会执行，然后实现我们想要的功能。

### 1.将插件的goal绑定到phase上

每个插件都有多个goal，每个goal都是一个具体的功能，举个例子，dependency插件有十几个goal，可以进行分析项目依赖，列出依赖树，等等。

比如将source插件的jar-no-fork goal绑定到verify phase，在完成集成测试之后，就生成源码的jar包，这里可以看到在 pom.xml 文件中绑定plugin的语法：

```
<build>
	<plugins>
		<plugin>
			<groupId>org.apache.maven.plugins</groupId>
			<artifactId>maven-source-plugin</artifactId>
			<version>2.1.1</version>
			<executions>
				<execution>
					<id>attach-sources</id>
					<phase>verify</phase>
					<goals>
						<goal>jar-no-fork</goal>
					</goals>
				</execution>
			</executions>
		</plugin>
	</plugins>
</build>
```

即使不配置绑定的phase也可以，因为大多数插件都默认内置了要绑定的phase，比如这个插件就内置绑定在package phase

### 2.配置插件
1. 命令行执行
    如果在命令行执行插件，可以用-Dkey=value来进行插件的设置，比如mvn install -Dmaven.test.skip=true，就是surefire插件在测试的时候提供的参数，设置为true就会跳过测试。

1. pom文件配置
    也可以在pom.xml中用来配置
    ```
    <build>
    	<plugins>
    		<plugin>
    			<groupId>org.apache.maven.plugins</groupId>
    			<artifactId>maven-compiler-plugin</artifactId>
    			<version>2.1</version>
    			<configuration>
    				<source>1.5</source>
    				<target>1.5</target>
    			</configuration>
    		</plugin>
    	</plugins>
    </build>
    ```

### 3.插件解析

首先，在 [Maven 可用插件](http://maven.apache.org/plugins/index.html) 中能找到所有的Maven 插件。
而插件解析则是也是本地仓库找插件，没有则从远程仓库找插件。所以插件的远程仓库也需要配置，maven默认配置了远程的插件仓库：
```
<pluginRepository>
		<id>central</id>
		<name>Maven Plugin Repository</name>
		<url>http://repo1.maven.org/maven2</url>
		<layout>default</layout>
		<snapshots>
			<enabled>false</enabled>
		</snapshots>
		<releases>
			<updatePolicy>never</updatePolicy>
		</releases>
	</pluginRepository>
</pluginRepositories>
```
当然我们也可以配置为公司的私服，从私服获取插件。






    