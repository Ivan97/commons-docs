# 快速开始
这篇文档是使用Commons Configuration基本功能的快速教程，之后的文档会详尽解释本篇中出现的概念。

## 读取properties属性文件

在应用中，配置信息经常会储存在proerties文件中。例如下面这个简单的文件，它定义了连接数据库所需的一些属性信息，假设它以`database.properties`为文件名存储在本地。

```
database.host = db.acme.com
database.port = 8199
database.user = admin
database.password = ???
database.timeout = 60000
```

读取该文件的最简单的方法是使用**[Configurations](https://commons.apache.org/proper/commons-configuration/apidocs/org/apache/commons/configuration2/builder/fluent/Configurations.html)**工具类，它提供了一系列方便的方法，来从不同的数据源创建configuration对象。对于properties文件来说，读取方式如下：

```java
Configurations configs = new Configurations();
try
{
    Configuration config = configs.properties(new File("config.properties"));
    // access configuration properties
    ...
}
catch (ConfigurationException cex)
{
    // Something went wrong
}
```

## 获取属性信息

**[Configuration](https://commons.apache.org/proper/commons-configuration/apidocs/org/apache/commons/configuration2/Configuration.html)**对象是我们读取properties文件中存储信息的最后一步媒介，其中包含一系列get方法来适配不同类型的值，对于上面提到的properties文件，可以通过如下方式读取：

```java
String dbHost = config.getString("database.host");
int dbPort = config.getInt("database.port");
String dbUser = config.getString("database.user");
String dbPassword = config.getString("database.password", "secret");  // provide a default
long dbTimeout = config.getLong("database.timeout");
```
