# 使用Configuration

Commons Configuration允许从各种不同的来源访问配置属性。无论它们是存储在properties文件，XML文档还是JNDI树中，它们都可以通过通用**[Configuration](https://commons.apache.org/proper/commons-configuration/apidocs/org/apache/commons/configuration2/Configuration.html)**接口以相同方式访问。

 Commons Configuration的另一个优势能力是它可以把多个不同的配置数据源混合在一起作为逻辑上唯一的配置处理。本节将向您介绍可用的不同配置，并向您展示如何组合它们。

 ## Configuration Sources

 目前，Commons Configuration支持相当多的Configuration对象数据源。但是，通过使用Configuration对象而不是特定的类型（如XMLConfiguration或JNDIConfiguration），您将无法获得实际检索配置值的机制。这些不同来源包括：

- `PropertiesConfiguration`从属性文件中加载配置值。
- `XMLConfiguration`从XML文档中获取值。
- `INIConfiguration`加载Windows使用的.ini文件中的值。
- `PropertyListConfiguration`从OpenStep .plist文件中加载值。XMLPropertyListConfiguration也可用于读取Mac OS X使用的XML变体。
- `JNDIConfiguration`使用JNDI树中的键，可以检索值作为配置属性。
- `BaseConfiguration`填充Configuration对象的内存中方法。
- `HierarchicalConfiguration`能够处理复杂结构化数据的内存中配置对象。
- `SystemConfiguration`使用系统属性的配置
- `ConfigurationConverter`获取`java.util.Properties`或`org.apache.commons.collections.ExtendedProperties`并将其转换为Configuration对象。

## The Configuration interface

代表不同类型配置源的这个包中的所有类共享一个接口：**[Configuration](https://commons.apache.org/proper/commons-configuration/apidocs/org/apache/commons/configuration2/Configuration.html)**。该接口允许您以通用方式访问和操作配置属性。
**[Configuration](https://commons.apache.org/proper/commons-configuration/apidocs/org/apache/commons/configuration2/Configuration.html)**接口中定义的方法可以分为查询配置对象中的数据的方法和更改配置对象的方法。事实上，Configuration接口扩展了一个名为**[ImmutableConfiguration](https://commons.apache.org/proper/commons-configuration/apidocs/org/apache/commons/configuration2/ImmutableConfiguration.html)**的Base接口。`ImmutableConfiguration`定义了从配置对象读取数据而不改变其状态的所有方法。`Configuration`添加了操作配置的方法。`ImmutableConfiguration`接口中定义的大部分方法用于处理检索不同数据类型的属性。所有这些方法都将一个key作为指向所需属性的参数。这是一个字符串值，其确切含义取决于所使用的`Configuration`具体实现类。方法尝试找到由传入的key指定的属性并将其转换为其目标类型;这个转换后的值将被返回。另外，所有方法都有允许指定默认值的方法的重载变体，如果找不到属性，将返回该默认值。开箱即用支持以下数据类型：

- BigDecimal
- BigInteger
- boolean
- byte
- double
- float
- int
- long
- short
- String

这些方法的名称以`get`后跟数据类型开头。 例如，`getString()`方法将返回String值，`getInt()`将对整数进行操作。
属性可以具有多个值，因此也可以查询包含所有可用值的列表或数组。这是使用`getList()`或`getArray()`方法完成的。

此外，还有一些通用的get方法试图将请求的属性值转换为指定的数据类型。这些转换也支持集合或数组的元素。有关转换的更多详细信息，请参见[数据类型转换部分](https://commons.apache.org/proper/commons-configuration/userguide/howto_basicfeatures.html#Data_type_conversions)。

如果configuration设置按特定结构进行组织并且应用程序的模块仅对此结构的一部分感兴趣，则subset()方法很有用。subset()传递一个带有键前缀的String，并返回一个Configuration对象，该对象仅包含以此前缀开头的键。

为了操作属性或它们的值，可以使用以下方法：

- `addProperty()`

将新属性添加到配置。如果此属性已经存在，则向其添加另一个值（以便它变成多值属性）。

- `clearProperty()`

从配置中删除指定的属性。

- `setProperty()`

覆盖指定属性的值。这与删除属性然后使用新属性值调用`addProperty()`相同。

- `clear()`

擦除整个配置

## Immutable Configurations
Commons Configuration库提供的大多数类都实现了Configuration接口，即它们允许客户端代码更改其内部状态。对于某些使用情况，这可能不是期望的做法。例如，应用程序可能希望保护中央配置对象免受子模块完成的不受控制的修改。
有一种简单的方法可以将普通的`Configuration`对象转换为`ImmutableConfiguration`：只需将相关配置传递给**[ConfigurationUtils](https://commons.apache.org/proper/commons-configuration/apidocs/org/apache/commons/configuration2/ConfigurationUtils.html)**实用程序类的`unmodifiableConfiguration()`方法即可。这会导致包含与原始配置相同的数据的不可变配置。

## Threading issues

当从多个线程访问配置时（无论是只读还是操纵方式），问题出现在Configuration实现是否是线程安全的。当使用上一节所述的不可变配置时，您通常是安全的，因为不可变对象可以安全地在多个线程之间共享。但是，`由ConfigurationUtils`创建的`ImmutableConfiguration`对象只是可变配置对象的包装器。所以如果代码持有对底层配置的引用，它仍然可以改变。

由于并发是一个复杂的主题，因此本用户指南包含有关此主题的专用章节：[配置和并发访问](https://commons.apache.org/proper/commons-configuration/userguide/howto_concurrency.html)。
