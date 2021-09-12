# Maven系列（五）工程管理
- [Maven系列（五）工程管理](#maven系列五工程管理)
  - [1.统一构建](#1统一构建)
  - [2.版本管理](#2版本管理)
    - [定义版本号](#定义版本号)
  - [3.第三方组件强制约束依赖方版本号](#3第三方组件强制约束依赖方版本号)

## 1.统一构建

假设现在有一种情况，有20个模块，依赖了某个基础模块，而基础模块作出了改动，那么这20个模块就需要更新依赖，然后执行单元测试套件，确保依赖没有问题，那么就要依次手工运行20次构建的命令，让20个模块都依次去构建和测试？

这种方式很麻烦，所以maven的解决方案就是聚合功能，将各个模块，聚合成一个大的模块，给它一个父工程。
只要对父模块运行一次构建命令，此时maven会自动对这个父模块下面的所有子模块都运行相应的构建命令，这样就可以保证一键自动化构建所有的模块，不要一个一个依次去构建。

如果要一次性构建多个模块的工程，那么就需要创建一个父工程，我们可以创建一个os-parent工程，在其pom.xml中加入以下内容：
```
<groupId>com.binbinshan.oa</groupId>
<artifactId>oa-parent</artifactId>
<version>1.0.0-SNAPSHOT</version>
<packaging>pom</packaging>
<name>oa parent project</name>
<!-- 这里是三个子模块 -->
<modules>
	<module>oa-organ</module>
	<module>oa-auth</module>
	<module>oa-flow</module>
</modules>
```

接着对oa-parent运行mvn clean install，此时就会对oa-parent中所有的工程都进行统一的构建

## 2.版本管理

对同一个系统，需要将各个模块的工程，其中的基础性的、相同的依赖，全部放入一个父工程中，集中式统一约束所有模块的依赖，避免每个模块随意依赖，避免后续因为版本不同导致的问题。

而maven的解决方案就是继承，可以将一个项目中所有模块通用的一些配置，比如依赖和插件，全部放在一个父工程中，oa-parent，然后所有的子工程，声明一下从父工程中去继承。继承的格式如下：
```
//子工程的pom.xml文件中
<parent>
	<groupId>com.binbinshan.oa</groupId>
	<artifactId>oa-parent</artifactId>
	<version>1.0.0-SNAPSHOT</version>
</parent>
```

这里有两种方式:
1. 在父工程中直接用\<dependencies\>和\<plugins>\来声明依赖和插件,然后子工程用\<parent\>元素声明继承，此时会强制性继承父工程中所有的依赖和插件。

2. 在父工程中使用\<depdendencyManagement\>和\<pluginManagement\>来声明依赖和插件,然后子工程用<parent>元素声明继承父工程，此时不会强制性继承所有的依赖和插件，子工程需要同时声明，自己要继承哪些依赖和插件，但是只要声明groupId和artifactId即可，不需要声明version版本号，因为version全部放在父工程中统一声明，强制约束所有子工程的相同依赖的版本要一样。
3. 上面两种方式同时使用，使用<dependencies\>和\<plugins>\来声明依赖和插件，让子工程强制依赖，再使用\<depdendencyManagement\>和\<pluginManagement\>来声明依赖和插件，让子工程可选依赖。

一般情况下，每个子工程都是继承自己需要的依赖，而不会全部依赖，全部依赖就会导致打出的Jar包过大。
这里很关键的一个知识点，是要搞清楚，dependencyManagement和pluginManagement的意义，就是按需依赖，不要全部依赖。

### 定义版本号
版本号管理可以在properties元素里面，统一定义一个版本号，全部用${}占位符来引用一个版本号，那么每次修改，，就直接修改properties元素中的一个地方就可以。
```
<properties>
    <!-- SpringBoot -->
    <spring-boot.version>2.1.6.RELEASE</spring-boot.version>
</properties>

....
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-dependencies</artifactId>
    <version>${spring-boot.version}</version>
    <type>pom</type>
    <scope>import</scope>
</dependency>
```

## 3.第三方组件强制约束依赖方版本号

假设我们自己开发了组件，依赖了很多重要的第三方的开源框架，然后你为了保证依赖你的组件的人，不会自行导入一些开源框架的过旧的版本，导致跟你的组件出现依赖冲突。

1. 需要为你的组件开发一个类型为pom的工程，后缀名为bom，这个工程，要声明一套dependencyManagement，里面声明好对你的组件的依赖的版本号，还有你的组件使用的重要的第三方开源框架的版本号

2. 然后依赖方在引用你的组件的时候，需要在自己的dependencyManagement中，加一个scope范围为import，类型为pom的这么依赖导入，此时就可以将你的bom包声明的那一套依赖的版本号导入他那里，作为版本的约束

    ```
    <dependencyManagement>
       <dependencies>
           <dependency>
               <groupId>com.binbinshan.commons</groupId>
               <artifactId>commons-flow-bom</artifactId>
               <version>1.2.9</version> 
               <type>pom</type>
               <scope>import</scope>
           </dependency>
       </dependencies>
    </dependencyManagement>
    ```

3. 然后依赖方接着在dependencies里面可以声明对你的组件的依赖，此时版本号都不用写，因为已经被你约束了


这里解释一下Bom，Bom（Bill of Materials）是由Maven提供的功能,它通过定义一整套相互兼容的jar包版本集合，使用时只需要依赖该BOM文件，即可放心的使用需要的依赖jar包，且无需再指定版本号。BOM的维护方负责版本升级，并保证BOM中定义的jar包版本之间的兼容性。

BOM本质上是一个普通的POM文件，区别是对于使用方而言，生效的只有\<dependencyManagement\>这一个部分。在\<dependencyManagement\>定义对外发布的依赖。
而在使用方则是需要配置，上述第二步中的依赖即可。需要使用相关JAR包的pom.xml文件中<dependencies></dependencies>节点下引入，不需要写版本号，如需要使用不同于当前bom中所维护的jar包版本，则加上<version>覆盖即可，



