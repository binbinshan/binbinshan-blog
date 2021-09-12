# Maven系列（六）开发流程管理
- [Maven系列（六）开发流程管理](#maven系列六开发流程管理)
  - [1.使用surefire插件进行单元测试](#1使用surefire插件进行单元测试)
    - [1.1 跳过单元测试](#11-跳过单元测试)
    - [1.2 指定单元测试](#12-指定单元测试)
    - [1.3 自定义包含与排除测试类](#13-自定义包含与排除测试类)
  - [2.使用jetty插件测试本地web工程](#2使用jetty插件测试本地web工程)
    - [2.1 加入jetty插件的配置](#21-加入jetty插件的配置)
    - [2.2设置pluginGroups](#22设置plugingroups)
    - [2.3 执行mvn jetty:run](#23-执行mvn-jettyrun)
  - [3.使用cargo插件部署web工程](#3使用cargo插件部署web工程)
    - [3.1 引入cargo插件](#31-引入cargo插件)
    - [3.2 本地tomcat配置](#32-本地tomcat配置)
  - [4.maven的版本管理](#4maven的版本管理)
## 1.使用surefire插件进行单元测试

通常我们在开发完成之后，都会编写一些单元测试，然后可以利用surefire插件进行单元测试。在default生命周期的test阶段，会运行surefire插件的test goal，然后执行src/test/java下面的所有单元测试的。

而surefire插件会根据一定的规则在sre/test/java下面找单元测试类，具体规则如下：
```
**/Test*.java
**/*Test.java
**/*TestCase.java
```

### 1.1 跳过单元测试
如果某次你的确不想要执行单元测试就打包，那么可以跳过单元测试，mvn package -Dmaven.test.skip=true 。

### 1.2 指定单元测试
当然你也可以指定想要运行的测试类，mvn test -Dtest=**Test，直接指定你要运行的类即可。

### 1.3 自定义包含与排除测试类
引入surefire插件，然后指定排除的测试类或者包含的测试类。
```
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-surefire-plugin</artifactId>
  <version>2.5</version>
  <configuration>
    <includes>
      <include>**/*Tests.java</include>
    </includes>
    <excludes>
      <exclude>**/*TempTest.java</exclude>
    </excludes>
  </configuration>
</plugin>
```

## 2.使用jetty插件测试本地web工程

除了在本地单元测试以外，你还需要在本地启动web服务，然后调用一下web接口，做一个简单的冒烟测试，看看基本的功能是否正常。

### 2.1 加入jetty插件的配置
```
<plugin>
	<groupId>org.mortbay.jetty</groupId>
	<artifactId>jetty-maven-plugin</artifactId>
	<version>7.1.6.v20100715</version>
	<configuration>
		<scanIntervalSeconds>10</scanIntervalSeconds>
		<webAppConfig>
			<contextPath>/test</contextPath>
		</webAppConfig>
	</configuration>
</plugin>
```
scanIntervalSeconds，指的是自动扫描项目代码变更的时间。

contextPath，就是部署之后的contextPath。

### 2.2设置pluginGroups
jetty插件的groupId不是默认的org.apache.maven.plugins，因此如果要执行该插件，需要在settings.xml中进行配置：
```
<pluginGroups>
	<pluginGroup>org.mortbay.jetty</pluginGroup>
</pluginGroups>
```

### 2.3 执行mvn jetty:run
执行mvn jetty:run，执行jetty插件的run goal，会用jetty容器启动web应用，默认绑定8080端口，用mvn jetty:run -Djetty.port=8081可以修改端口号



## 3.使用cargo插件部署web工程
cargo插件专门用来进行将本地的web工程打包成一个war包，然后扔到远程的服务器上的Tomcat中去，进行自动化的部署。

### 3.1 引入cargo插件
```
<plugin>
	<groupId>org.codehaus.cargo</groupId>
	<artifactId>cargo-maven2-plugin</artifactId>
	<version>1.0</version>
	<configuration>
		<container>
			<containerId>tomcat6x</containerId>
			<type>remote</type>
		</container>
		<configuration>
			<type>runtime</type>
			<properties>
				<cargo.remote.username>admin</cargo.remote.username>
				<cargo.remote.password>admin</cargo.remote.password>
				<cargo.tomcat.manager.url>http://localhost:8080/manager</cargo.tomcat.manager.url>
				<cargo.servlet.port>8080</cargo.servlet.port>
			</properties>
		</configuration>
	</configuration>
</plugin>
```

在settings.xml中加入pluginGroup：org.codehaus.cargo

### 3.2 本地tomcat配置

1. 本地安装一个tomcat
2. 给tomcat 设置一个管理员，在conf/tomcat-users.xml
```
<user username="admin" password="admin" roles="manager"/>
```

3. 修改tomcat的conf/server.xml，里面在8080那个地方，加入两个配置
```
URIEncoding="UTF-8" useBodyEncodingForURI="true"
```

4. 启动tomcat

5. 在项目的pom.xml里加入cargo插件的配置
6. 在settings.xml里加入pluginGroup，org.codehaus.cargo
7. 解决中文编码的问题，先在web.xml里加入一个字符集编码的过滤器
8. 运行mvn clean package命令，打包，必须先有一个war包
9. 到这一步为止，就可以部署了，直接cargo:deploy，第一次部署，在http://localhost:8080/oa-web/，跟上接口，就可以去访问了
10. 以后重复部署，cargo:redeploy，自动会先卸载，然后再重新部署


## 4.maven的版本管理
版本分为两种，一种是快照（snapshot）版本，一种是发布（release）版本

* 快照版本就是版本号后面加上SNAPSHOT，比如1.0.0-SNAPSHOT

* 发布版本就是版本号上不加SNAPSHOT的，比如1.0.0

而且版本通常而言是三位的:
* 1.0.0中的第一个1指的是一个重大版本，比如已经基本成型的一个系统架构
    * 也有不少项目一开始是0.0.1这样的版本，这个指的就是这个系统刚开始起步，甚至都没能形成一个完整的东西出来，只是少数核心功能也出来了
* 1.0.0中的第二个0指的是一个次要版本，比如1.1.0，那么就是加了一些新的功能或者开发了一些新的模块，或者做了一些较大的代码重构，技术升级改造
* 1.0.0中的第三个0指的是日常迭代的一个增量版本，比如1.1.1，一般对应着修复了一个bug，或者对某些代码做了轻微的优化或者调整

