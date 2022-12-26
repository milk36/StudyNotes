Maven使用笔记
---
## 安装与配置
### 参考:
   * [maven安装、本地仓库路径设置以及仓库配置](https://cloud.tencent.com/developer/article/1864614)
   * [Maven的安装与配置](https://blog.csdn.net/pan_junbiao/article/details/104264644)
### 设置环境变量: `MAVEN_HOME` 和 `MAVEN_OPTS` 

   * 设置MAVEN_OPTS环境变量不是必须的，但建议设置。因为Java默认的最大可用内存往往不能够满足Maven运行的需要，比如在项目较大时，使用Maven生成项目站点需要占用大量的内存，如果没有该配置，则很容易得到java.lang.OutOfMemeoryError。因此，一开始就配置该变量是推荐的做法。

      变量值:`-Xms128m -Xmx512m`
### 配置用户范围的settings.xml文件
* 配置文件路径: `D:\XXXX\apache-maven-3.8.6\conf\settings.xml`
* 设置本地仓库
   ```xml
   <localRepository>D:\maven-local-repository</localRepository>
   ``` 
   修改本地仓库路径以后 `Gradle`配置`mavenlocal`可能会出现问题:    
    1. 方法一: [解决 从本地 Maven 存储库依赖关系](https://stackoverflow.com/questions/32107205/gradle-does-not-use-the-maven-local-repository-for-a-new-dependency)

        `build.gradle`文件配置
        ```gradle
        repositories {
            maven { url 'D:/maven-local-repository' }
        }
        ```
    1. 方法二: 添加环境变量 `M2_HOME`
        
        参考:
        * [Gradle通过mavenLocal()指向本地仓库 -Gradle依赖包的存储位置](https://blog.csdn.net/shuair/article/details/107319204)
        * [Windows配置gradle本地仓库](https://juejin.cn/post/7096505187677765669)
            
        gradle寻找本地maven仓库位置的策略:

        `USER_HOME/.m2/settings.xml >> M2_HOME/conf/settings.xml >> USER_HOME/.m2/repository`

        1. 我们一般在maven的安装目录/conf/settings.xml（也就是我们配置的maven环境变量）中配置本地仓库位置，所以我们需要让gradle选择该路径，从而使用我们配置的maven本地仓库

        1. gradle先寻找`USER_HOME/.m2/settings.xml`，所以我们要删掉该文件（其实也可以将安装目录下的settings.xml复制过来）

        1. maven环境变量我们习惯配置成`MAVEN_HOME`，但是gradle寻找的是`M2_HOME`，所以我们需要配置`M2_HOME`环境变量

        > 使用此方式, 需要重启系统, 以确保环境变量生效, `M2_HOME` == `MAVEN_HOME`
* 设置阿里云中央仓库镜像
   ```xml
   <mirror>
     <id>aliyunmaven</id>
     <mirrorOf>*</mirrorOf>
     <name>阿里云公共仓库</name>
     <url>https://maven.aliyun.com/repository/public</url>
   </mirror>
   ``` 
## 发布jar到仓库
* 参考:[如何发布JAR包到Maven本地仓库](https://www.jianshu.com/p/5a737de5b52c)
* 执行命令:
  ```sh
  mvn install:install-file -Dfile=server-core-1.2.11.jar -DgroupId=com.janlr.ag.server-core-framework -DartifactId=server-core -Dversion=1.2.11.17 -Dpackaging=jar
  ```