Maven
===

这里记录下在 使用Maven时的一些知识点。

Installation
---
  在[Apache Maven Project](http://maven.apache.org/download.cgi)中下载maven，解压并按照`README.txt`配置。运行`mvn -v`查看是否配置成功。

Configuring Maven
---
  根据`setting.xml`知，有两种级别的配置: User Level 和 Global Level。`localRepository`默认为` ${user.home}/.m2/repository/`。

Creating a project
---
  参考官方文档，创建一个简单的maven应用。运行命令如下：  
- `mvn archetype:generate -DgroupId=com.mycompany.app -DartifactId=my-app -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false`。  
  通过`mvn -h`查看各参数意思。`-DartifactId`定义了应用的名称。生成的`pom.xml`是maven的核心配置文件。

- `mvn package`  
  package是[build lifecycle](http://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html)中的一步。这条命令将lifecycle从头一直执行到package这一步。这步可以看到生成的jar包。

- `java -cp target/my-app-1.0-SNAPSHOT.jar com.mycompany.app.App`  
  检查生成的jar包，这条命令的结果是`Hello World!`

Others
---
- 删除一个project  
  在project目录上，运行`mvn clean packege`即可删除一个project。




