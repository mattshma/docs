pom.xml说明
===

The Basics
---
pom.xml存储了一个工程的所有信息和build过程中plugin的配置信息。其中`groupId:artifactId:version`是所有pom.xml的必须字段。`groupId`相当于java的package这个概念。如果`groupId`中有".", 将会被操作系统替换为相应的文件分割符（如Unix系统中被替换成"/"）。`artifactId`是工程的名字，其和`groupId`一起唯一指定了这个工程。`version`是版本信息。这三个字段指出了该工程的指定版本让maven知道谁在处理，何时处理处理这个工程。

POM关系
---
Maven有三种关系：dependencies、inheritance、aggregation

### Dependencies（依赖）
依赖列表是POM的基石。maven将会根据pom文件下载和链接这些依赖。

```java
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                      http://maven.apache.org/xsd/maven-4.0.0.xsd">
  ...
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.0</version>
      <type>jar</type>
      <scope>test</scope>
      <optional>true</optional>
    </dependency>
    ...
  </dependencies>
  ...
</project>
```

### exclusions
如果我们不想包括传递依赖，如只想使用maven-core，而不想使用它的依赖，可以把这个文件放到exclusion里面。

```java
<project xmlns="http://maven.apache.org/POM/4.0.0"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0
                      http://maven.apache.org/xsd/maven-4.0.0.xsd">
  ...
  <dependencies>
    <dependency>
      <groupId>org.apache.maven</groupId>
      <artifactId>maven-embedder</artifactId>
      <version>2.0</version>
      <exclusions>
        <exclusion>
          <groupId>org.apache.maven</groupId>
          <artifactId>maven-core</artifactId>
        </exclusion>
      </exclusions>
    </dependency>
    ...
  </dependencies>
  ...
</project>
```


Reference
---
- [POM Reference](http://maven.apache.org/pom.html)
- [Maven Assembly Plugin](http://maven.apache.org/plugins/maven-assembly-plugin/)

