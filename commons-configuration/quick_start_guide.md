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

## 从properties文件获取属性信息

**[Configuration](https://commons.apache.org/proper/commons-configuration/apidocs/org/apache/commons/configuration2/Configuration.html)**对象是我们读取properties文件中存储信息的最后一步媒介，其中包含一系列get方法来适配不同类型的值，对于上面提到的properties文件，可以通过如下方式读取：

```java
String dbHost = config.getString("database.host");
int dbPort = config.getInt("database.port");
String dbUser = config.getString("database.user");
String dbPassword = config.getString("database.password", "secret");  // provide a default
long dbTimeout = config.getLong("database.timeout");
```
注：传入方法的key必须与properties文件中的key相匹配。如果不能被解析，将会返回`null`。返回原始类的方法将会抛出异常，如果业务场景允许，可以附上默认值，防止不能解析的情况出现。

## 读取XML属性文件
xml文件也适合用来存储信息，尤其是复杂的信息，例如一系列有相同tag的属性列表。这部分的样例文件存储了一系列路径信息，文件名为`paths.xml`。

```xml
<?xml version="1.0" encoding="ISO-8859-1" ?>
<configuration>
  <processing stage="qa">
    <paths>
      <path>/data/path1</path>
      <path>/data/otherpath</path>
      <path>/var/log</path>
    </paths>
  </processing>
</configuration>
```

读取xml问价与读取properties文件十分类似。同样创建一个**[Configuration](https://commons.apache.org/proper/commons-configuration/apidocs/org/apache/commons/configuration2/Configuration.html)**实例（顺便说一下，这个类是线程安全的，其对象可以被安全的分享复用来读取不同数据源），但是这次使用`xml()`方法而不是`properties()`方法。

```java
Configurations configs = new Configurations();
try
{
    XMLConfiguration config = configs.xml("paths.xml");
    // access configuration properties
    ...
}
catch (ConfigurationException cex)
{
    // Something went wrong
}
```

`xml()`方法返回一个**[XMLConfiguration](https://commons.apache.org/proper/commons-configuration/apidocs/org/apache/commons/configuration2/XMLConfiguration.html)**实例，它实现了**[Configuration](https://commons.apache.org/proper/commons-configuration/apidocs/org/apache/commons/configuration2/Configuration.html)**接口，并且提供了更多功能，来更有条理地读取属性信息。同时，读者应注意到在`xml()`方法中传入的是一个String而非一个`java.io.File`对象，事实上这是工具提供的读取方法的重载，用以不同方式表示数据源：可以是一个file对象，一个URL，或一个String。在用String表示数据源的时候，工具类会在不同的位置查找文件，包括绝对路径，相对路径，资源路径或者当前用户的home目录下。

## 从XML文件获取属性信息

在XML文件（或其他分层配置）中读取属性时，有相同的查询方法。有一些额外的机制用来保证获取分层配置的属性。在样例文件中的属性可以通过如下方式获取：

```java
String stage = config.getString("processing[@stage]");
List<String> paths = config.getList(String.class, "processing.paths.path");
```

属性的键是通过连接XML文档中可能嵌套的标签名称生成的(忽略root节点),对于属性，有一个特殊的语法，如stage属性所示。由于路径元素出现多次的情况，它实际上定义了一个列表。使用getList()方法，所有值都可以一次查询。
分层配置支持键的高级语法，允许导航到源文档中的特定元素。这是通过在单个关键部分之后的括号中添加数字索引来实现的。例如，为了引用列表中的第二个路径元素，可以使用以下键（索引是基于0的）：

```java
String secondPath = config.getString("processing.paths.path(1)");
```

## 更新一个configuration

**[XMLConfiguration](https://commons.apache.org/proper/commons-configuration/apidocs/org/apache/commons/configuration2/XMLConfiguration.html)**实例，它实现了**[Configuration](https://commons.apache.org/proper/commons-configuration/apidocs/org/apache/commons/configuration2/Configuration.html)**接口定义了一些方法来操作属性信息，典型的CRUD操作适用于所有属性，以下代码片段显示了示例属性配置如何进行操作，数据库的端口更改为新值，并添加新属性：

```java
config.setProperty("database.port", 8200);
config.addProperty("database.type", "production");
```
`addProperty()`总是为配置添加一个新值。如果key已经存在，则将该值添加到此key下，以使其成为列表。相反，`setProperty()`会覆盖现有的值（如果该key不存在，则创建一个新的值）。这两种方法都可以传递一个任意的值对象。可以是一个数组或集合，在一个操作中添加多个值。

## 保存configuration

在对一个congiguration实例操作完成后，应保存以使之持久化。否则，更改仅在内存中。如果要更改configuration，则最好通过不同的机制获取它们：配置生成器（configuration builder）。Builders是构建congiguration的最强大和最灵活的方式。它们支持许多configuration数据加载方式和加载的configuration对象行为的设置。基于文件的configuration的构建器还提供`save()`方法，将所有配置数据写回磁盘。configuration builder通常使用流式API创建，以便对builders进行方便和灵活的配置。该API在“配置构建器”一节中进行了介绍。对于简单的使用情况，我们已经使用的Configurations类还有一些便利方法。以下代码片段显示了如何通过构建器读取配置，并进行操作并最终再次保存configuration：

```java
Configurations configs = new Configurations();
try
{
    // obtain the configuration
    FileBasedConfigurationBuilder<XMLConfiguration> builder = configs.xmlBuilder("paths.xml");
    XMLConfiguration config = builder.getConfiguration();

    // update property
    config.addProperty("newProperty", "newValue");

    // save configuration
    builder.save();
}
catch (ConfigurationException cex)
{
    // Something went wrong
}
```
