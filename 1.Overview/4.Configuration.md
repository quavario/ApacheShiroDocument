# [Apache Shiro配置](http://shiro.apache.org/configuration.html#apache-shiro-configuration)

- [程序化配置](http://shiro.apache.org/configuration.html#Configuration-ProgrammaticConfiguration)
  - [SecurityManager对象图](http://shiro.apache.org/configuration.html#Configuration-SecurityManagerObjectGraph)
- [INI配置](http://shiro.apache.org/configuration.html#Configuration-INIConfiguration)
  - [从INI创建SecurityManager](http://shiro.apache.org/configuration.html#Configuration-CreatingaSecurityManagerfromINI)
    - [来自INI资源的SecurityManager](http://shiro.apache.org/configuration.html#Configuration-SecurityManagerfromanINIresource)
    - [来自INI实例的SecurityManager](http://shiro.apache.org/configuration.html#Configuration-SecurityManagerfromanINIinstance)
  - [INI部分](http://shiro.apache.org/configuration.html#Configuration-INISections)
    - [`[main]`](http://shiro.apache.org/configuration.html#Configuration-%5Cmain%5C)
      - [定义一个对象](http://shiro.apache.org/configuration.html#Configuration-Defininganobject)
      - 设置对象属性
        - [原始价值观](http://shiro.apache.org/configuration.html#Configuration-PrimitiveValues)
        - [参考值](http://shiro.apache.org/configuration.html#Configuration-ReferenceValues)
        - [嵌套属性](http://shiro.apache.org/configuration.html#Configuration-NestedProperties)
        - [字节数组值](http://shiro.apache.org/configuration.html#Configuration-ByteArrayValues)
        - [集合属性](http://shiro.apache.org/configuration.html#Configuration-CollectionProperties)
      - [注意事项](http://shiro.apache.org/configuration.html#Configuration-Considerations)
        - [订单事项](http://shiro.apache.org/configuration.html#Configuration-OrderMatters)
        - [覆盖实例](http://shiro.apache.org/configuration.html#Configuration-OverridingInstances)
        - [默认SecurityManager](http://shiro.apache.org/configuration.html#Configuration-DefaultSecurityManager)
    - [`[users]`](http://shiro.apache.org/configuration.html#Configuration-%5Cusers%5C)
      - [行格式](http://shiro.apache.org/configuration.html#Configuration-LineFormat)
      - [加密密码](http://shiro.apache.org/configuration.html#Configuration-EncryptingPasswords)
    - [`[roles]`](http://shiro.apache.org/configuration.html#Configuration-%5Croles%5C)
      - [行格式](http://shiro.apache.org/configuration.html#Configuration-LineFormat)
    - [`[urls]`](http://shiro.apache.org/configuration.html#Configuration-%5Curls%5C)

Shiro旨在适用于任何环境，从简单的命令行到大型的企业集群。由于生产环境的多样性，有多种不同的配置方式。本节仅介绍Shiro核心支持的配置方式。

 

许多配置选项

------

Shiro的`SecurityManager`实现和所有支持组件都兼容JavaBeans。这使得Shiro可以使用任何配置格式，例如常规Java，XML（Spring，JBoss，Guice等），[YAML](http://www.yaml.org/)，JSON，Groovy Builder标记等。

## [Java代码配置方式](http://shiro.apache.org/configuration.html#programmatic-configuration)

创建一个SecurityManager最简单的方法是创建一个`org.apache.shiro.mgt.DefaultSecurityManager`。例如：

```
Realm realm = //这里需要一个Realm,这个我们一会再说.
SecurityManager securityManager = new DefaultSecurityManager(realm);

//使用这个方法可以单例化SecurityManager
SecurityUtils.setSecurityManager(securityManager);
```



### [SecurityManager对象图](http://shiro.apache.org/configuration.html#securitymanager-object-graph)

正如[架构](http://shiro.apache.org/architecture.html)章节中所讨论的，Shiro的`SecurityManager`实现本质上是嵌套安全特定组件的模块化对象图。因为它们也是JavaBeans兼容的，所以可以调用任何嵌套的组件`getter`和`setter`方法来配置`SecurityManager`它及其内部对象图。

例如，如果要将`SecurityManager`实例配置为使用自定义`SessionDAO`来自定义[会话管理](http://shiro.apache.org/session-management.html)，则可以`SessionDAO`使用嵌套的SessionManager `setSessionDAO`方法直接设置：

```java
...

DefaultSecurityManager securityManager = new DefaultSecurityManager(realm);

SessionDAO sessionDAO = new CustomSessionDAO();

((DefaultSessionManager)securityManager.getSessionManager()).setSessionDAO(sessionDAO);
...
```

使用直接方法调用，您可以配置`SecurityManager`对象的任何内容。

但是，就像程序化定制一样简单，它并不代表大多数真实环境的理想配置。Java代码配置可能不适合您的应用程序有几个原因：

- 需要知道直接创建实例的方法。
- 由于Java的类型安全性，您需要将通过`get*`方法获得的对象转换为其特定的实现。如此多的类型转换是冗长的，并且将你与实现类紧密结合在一起。
- 通过`SecurityUtils.setSecurityManager`方法实例化`SecurityManager`，在虚拟机中是单例静态的，大多数情况是可行的，但是如果多个项目在同一个JVM中运行,  可能会导致一些问题,  这是`SecurityManager` 在这个项目中是单例的比较好,  而不是在JVM内存中作为单例形式,  否则可能导致安全问题。
- 每次更改Shiro配置时，都重新编译应用程序。

然而，即使有这些缺点，直接编程操作方法在内存受限的环境中仍然很有价值，例如智能手机应用程序。如果您的应用程序不在内存受限的环境中运行，您将发现基于文本的配置更易于使用和阅读。

## [INI配置方式](http://shiro.apache.org/configuration.html#ini-configuration)

大部分框架都是通过配置文件方式进行配置，这使得配置信息可以和代码相互独立.

Shiro支持[INI格式](https://en.wikipedia.org/wiki/INI_file)来构建`SecurityManager`对象及其支持组件。

### [通过INI创建SecurityManager](http://shiro.apache.org/configuration.html#creating-a-securitymanager-from-ini)

以下是如何通过INI配置构建SecurityManager的两个示例。

#### [通过INI文件创建SecurityManager](http://shiro.apache.org/configuration.html#securitymanager-from-an-ini-resource)

我们可以通过INI文件创建SecurityManager实例。可以通过文件路径，类路径或URL获取资源,  分别用`file:` , `classpath:` , `url:`  作为参数的前缀。本例中使用`IniSecurityManagerFactory` 从类路径的根目录中获取文件并返回`SecurityManager`实例

```java
import org.apache.shiro.SecurityUtils;
import org.apache.shiro.util.Factory;
import org.apache.shiro.mgt.SecurityManager;
import org.apache.shiro.config.IniSecurityManagerFactory;

...

Factory<SecurityManager> factory = new IniSecurityManagerFactory("classpath:shiro.ini");
SecurityManager securityManager = factory.getInstance();
SecurityUtils.setSecurityManager(securityManager);
```

#### [通过INI对象创建SecurityManager](http://shiro.apache.org/configuration.html#securitymanager-from-an-ini-instance)

如果需要，也可以通过[`org.apache.shiro.config.Ini`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/config/Ini.html)类来编程地构造INI配置。Ini类的功能与JDK [`java.util.Properties`](http://download.oracle.com/javase/6/docs/api/java/util/Properties.html)类类似，但还支持按节名称进行分段。

例如：

```java
import org.apache.shiro.SecurityUtils;
import org.apache.shiro.util.Factory;
import org.apache.shiro.mgt.SecurityManager;
import org.apache.shiro.config.Ini;
import org.apache.shiro.config.IniSecurityManagerFactory;

...

Ini ini = new Ini();
//populate the Ini instance as necessary
...
Factory<SecurityManager> factory = new IniSecurityManagerFactory(ini);
SecurityManager securityManager = factory.getInstance();
SecurityUtils.setSecurityManager(securityManager);
```

现在我们知道如何通过INI创建一个`SecurityManager`，让我们看一下对INI文件进行配置。

### [INI部分](http://shiro.apache.org/configuration.html#ini-sections)

INI基本上是一个文本配置方式，由唯一命名的区域组成的键值。key仅在他所属的区域是唯一的，而不是在整个配置上.  然而，每个部分可以被视为单个定义。

注释行可以从#开始

以下是Shiro可以识别的区域示例：

```ini
# =======================
# Shiro INI configuration
# =======================

[main]
# 可以在main下定义对象和和对象的属性
# 例如securityManager,Realms等

[users]
# users这个区域可以简单的定义用户信息,不过都是死数据

[roles]
# roles这个区域用于定义用户身份,也是死数据,实际工作
# 环境中很少这样配置

[urls]
# The 'urls' section is used for url-based security
# in web applications.  We'll discuss this section in the
# Web documentation
```

#### [main]

**[main]**区域用于配置`SecurityManager`实例及其依赖项的部分，例如[Realm](http://shiro.apache.org/realm.html)。

通过INI文件配置对象实例(例如SecurityManager获取其依赖的Realm)听起来好像挺难的, 而且仅仅是用过键值对方式.

虽然不像成熟的的Spring / Guice / JBoss XML文件那么强大，但是你会发现它在没有太多复杂性的情况下完成了很多工作。当然，那些其他配置机制也是可用的，但它们不需要使用Shiro。

为了满足你的胃口，这里是一个有效`[main]`配置的例子。我们将在下面详细介绍它，但是你可能会发现你已经通过直觉了解了很多已经发生的事情：

```
[main]
sha256Matcher = org.apache.shiro.authc.credential.Sha256CredentialsMatcher

myRealm = com.company.security.shiro.DatabaseRealm
myRealm.connectionTimeout = 30000
myRealm.username = jsmith
myRealm.password = secret
myRealm.credentialsMatcher = $sha256Matcher

securityManager.sessionManager.globalSessionTimeout = 1800000
```

##### [定义一个对象](http://shiro.apache.org/configuration.html#defining-an-object)

请考虑以下`[main]`部分代码段：

```
[main]
myRealm = com.company.shiro.realm.MyRealm
...
```

这行代码实例化一个`com.company.shiro.realm.MyRealm`对象,  并命名为`myRealm`。

如果实例化的对象实现了`org.apache.shiro.util.Nameable`接口，`Nameable.setName`被被调用,  `myRealm` 会作为setName的参数传递给对象）。

##### [设置对象属性](http://shiro.apache.org/configuration.html#setting-object-properties)

###### [初始化属性](http://shiro.apache.org/configuration.html#primitive-values)

只需使用等号分配简单的初始属性：

```
...
myRealm.connectionTimeout = 30000
myRealm.username = jsmith
...
```

通过方法调用配置：

```
...
myRealm.setConnectionTimeout(30000);
myRealm.setUsername("jsmith");
...
```

这怎么可能？它假定所有对象都是[Java Bean](https://en.wikipedia.org/wiki/JavaBean)兼容的[POJO](https://en.wikipedia.org/wiki/Plain_Old_Java_Object)。

在封面下，Shiro默认使用Apache Commons [BeanUtils](http://commons.apache.org/proper/commons-beanutils/)来设置这些属性时完成所有繁重工作。因此，尽管INI值是文本，但BeanUtils知道如何将字符串值转换为正确的基本类型，然后调用相应的JavaBeans setter方法。

###### [参考值](http://shiro.apache.org/configuration.html#reference-values)

如果您需要设置的值不是值，而是另一个对象，该怎么办？好吧，您可以使用美元符号（$）来引用先前定义的实例。例如：

```
...
sha256Matcher = org.apache.shiro.authc.credential.Sha256CredentialsMatcher
...
myRealm.credentialsMatcher = $sha256Matcher
...
```

这里通过**sha256Matcher**这个名称找到了他引用的 对象，然后使用BeanUtils在**myRealm**上设置该对象（通过调用`myRealm.setCredentialsMatcher(sha256Matcher)`方法）。

###### [嵌套属性](http://shiro.apache.org/configuration.html#nested-properties)

通过`.`，您可以遍历对象以获取要设置的对象/属性。例如：

```
...
securityManager.sessionManager.globalSessionTimeout = 1800000
...
```

将（通过BeanUtils）转换为以下逻辑：

```
securityManager.getSessionManager().setGlobalSessionTimeout(1800000);
```

图遍历可以根据需要进行深入处理： `object.property1.property2....propertyN.value = blah`

 

BeanUtils属性支持

------

BeanUtils支持的任何属性赋值操作。[setProperty](https://commons.apache.org/proper/commons-beanutils/apidocs/org/apache/commons/beanutils/BeanUtils.html#setProperty-java.lang.Object-java.lang.String-java.lang.Object-)方法将在Shiro的[main]部分中工作，包括set / list / map元素赋值。有关更多信息，请参阅[Apache Commons BeanUtils网站](http://commons.apache.org/proper/commons-beanutils/)和文档。

###### [字节数组值](http://shiro.apache.org/configuration.html#byte-array-values)

因为原始字节数组不能以文本格式本地指定，所以我们必须使用字节数组的文本编码。可以将值指定为Base64编码的字符串（默认值）或Hex编码的字符串。默认值为Base64，因为Base64编码需要较少的实际文本来表示值 - 它具有更大的编码字母表，这意味着您的标记更短（文本配置更好一点）。

```
# The 'cipherKey' attribute is a byte array.    By default, text values
# for all byte array properties are expected to be Base64 encoded:

securityManager.rememberMeManager.cipherKey = kPH+bIxk5D2deZiIxcaaaA==
...
```

但是，如果您更喜欢使用十六进制编码，则必须在String标记前加上`0x`（'zero''x'）：

```
securityManager.rememberMeManager.cipherKey = 0x3707344A4093822299F31D008
```

###### [集合属性](http://shiro.apache.org/configuration.html#collection-properties)

列表，集合和映射可以像任何其他属性一样设置 - 直接或作为嵌套属性。对于集合和列表，只需指定逗号分隔的值集或对象引用。

例如，一些SessionListeners：

```
sessionListener1 = com.company.my.SessionListenerImplementation
...
sessionListener2 = com.company.my.other.SessionListenerImplementation
...
securityManager.sessionManager.sessionListeners = $sessionListener1, $sessionListener2
```

对于Maps，您可以指定以逗号分隔的键值对列表，其中每个键值对由冒号'：'分隔。

```
object1 = com.company.some.Class
object2 = com.company.another.Class
...
anObject = some.class.with.a.Map.property

anObject.mapProperty = key1:$object1, key2:$object2
```

在上面的示例中，引用的对象`$object1`将位于String键下的映射中`key1`，即`map.get("key1")`返回`object1`。您还可以使用其他对象作为键：

```
anObject.map = $objectKey1:$objectValue1, $objectKey2:$objectValue2
...
```

##### [注意事项](http://shiro.apache.org/configuration.html#considerations)

###### [订单事项](http://shiro.apache.org/configuration.html#order-matters)

上面的INI格式和约定非常方便且易于理解，但它没有其他基于文本/ XML的配置机制那么强大。使用上述机制时最重要的一点是**订单很重要！**

 

> **注意 **
>
> 每个对象实例化和每个值赋值*按它们在[main]部分中出现的顺序*执行。这些信息最终转换为JavaBeans getter / setter方法调用，因此这些方法以相同的顺序调用！

写配置文件时请牢记这一点。

###### [覆盖实例](http://shiro.apache.org/configuration.html#overriding-instances)

如果配置相同的key值,那么后面的value值将会覆盖前面的value值。例如，第二个`myRealm`定义将覆盖第一个：

```
...
myRealm = com.company.security.MyRealm
...
myRealm = com.company.security.DatabaseRealm
...
```

这将导致`myRealm`成为一个`com.company.security.DatabaseRealm`实例，并且永远不会使用前一个实例

###### [默认SecurityManager](http://shiro.apache.org/configuration.html#default-securitymanager)

您可能已经在上面的完整示例中注意到，未定义SecurityManager实例的类，我们直接跳转到仅设置嵌套属性：

```
myRealm = ...

securityManager.sessionManager.globalSessionTimeout = 1800000
...
```

这是因为`securityManager`实例是一个特殊的实例 - 它已经为您实例化并准备好了，所以您不需要知道`SecurityManager`要实例化的特定实现类。

当然，如果您确实*想要*指定自己的实现，则可以按照上面“覆盖实例”部分中的说明定义实现：

```
...
securityManager = com.company.security.shiro.MyCustomSecurityManager
...
```

当然，这很少需要 - Shiro的SecurityManager实现是非常可定制的，通常可以配置任何必要的东西。您可能想问自己（或用户列表）是否真的需要这样做。
[用户](http://shiro.apache.org/configuration.html#Configuration-users)

#### [users]

该**[users]**部分允许您定义一组静态用户。这在少量用户的环境中或者在运行时不需要动态创建用户的环境中非常有用。这是一个例子：

```
[users]
admin = secret
lonestarr = vespa, goodguy, schwartz
darkhelmet = ludicrousspeed, badguy, schwartz
```

 

自动IniRealm

------

仅定义非空[users]或[roles]部分将自动触发[`org.apache.shiro.realm.text.IniRealm`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/realm/text/IniRealm.html)实例的创建，并使其在名称下的[main]部分中可用**iniRealm**。您可以像上面描述的任何其他对象一样配置它。

##### [行格式](http://shiro.apache.org/configuration.html#line-format)

[users]部分中的每一行必须符合以下格式：

`username`= `password`，*roleName1*，*roleName2*，...，*roleNameN*

- 等号左侧的值是用户名
- 等号右侧  。密码是必需的。
- 密码后面的任何逗号分隔值是分配给该用户的角色的名称。角色名称是可选的。

##### [加密密码](http://shiro.apache.org/configuration.html#encrypting-passwords)

如果您不希望[users]部分密码为纯文本，您可以使用您喜欢的哈希算法（MD5，Sha1，Sha256等）加密它们，但是您喜欢并使用生成的字符串作为密码值。默认情况下，密码字符串应为十六进制编码，但可以配置为Base64编码（见下文）。

 

> **简易安全密码**
>
> 为了节省时间并使用最佳实践，您可能需要使用Shiro的[Command Line Hasher](http://shiro.apache.org/command-line-hasher.html)，它将散列密码以及任何其他类型的资源。加密INI `[users]`密码特别方便。

一旦指定了散列文本密码值，就必须告诉Shiro这些是加密的。您可以通过配置`iniRealm`[main]部分中隐式创建的方法来使用`CredentialsMatcher`与您指定的哈希算法相对应的适当实现：

```
[main]
...
sha256Matcher = org.apache.shiro.authc.credential.Sha256CredentialsMatcher
...
iniRealm.credentialsMatcher = $sha256Matcher
...

[users]
# user1 = sha256-hashed-hex-encoded password, role1, role2, ...
user1 = 2bb80d537b1da3e38bd30361aa855686bde0eacd7162fef6a25fe97bf527a25b, role1, role2, ...
```

您可以在`CredentialsMatcher`任何其他对象上配置任何属性以反映您的哈希策略，例如，指定是否使用salting或执行多少哈希迭代。请参阅[`org.apache.shiro.authc.credential.HashedCredentialsMatcher`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authc/credential/HashedCredentialsMatcher.html)JavaDoc以更好地了解散列策略以及它们是否对您有用。

例如，如果用户的密码字符串是Base64编码而不是默认的Hex，则可以指定：

```
[main]
...
# true = hex, false = base64:
sha256Matcher.storedCredentialsHexEncoded = false
```

#### [roles]

该**[roles]**部分允许您将[权限](http://shiro.apache.org/permissions.html)与[users]区域中定义的角色相关联。同样，这在具有少量角色的环境中或在运行时不需要动态创建角色时非常有用。这是一个例子：

```
[roles]
# 'admin' role has all permissions, indicated by the wildcard '*'
admin = * 
# The 'schwartz' role can do anything (*) with any lightsaber:
schwartz = lightsaber:*
# The 'goodguy' role is allowed to 'drive' (action) the winnebago (type) with
# license plate 'eagle5' (instance specific id)
goodguy = winnebago:drive:eagle5
```

##### [行格式](http://shiro.apache.org/configuration.html#line-format)

[roles]部分中的每一行必须使用以下格式定义角色到权限的键/值映射：

`rolename`= *permissionDefinition1*，*permissionDefinition2*，...，*permissionDefinitionN*

其中*permissionDefinition*是一个任意字符串，但大多数人将要使用符合串
到[`org.apache.shiro.authz.permission.WildcardPermission`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authz/permission/WildcardPermission.html)了易用性和灵活性的格式。有关权限以及如何从中受益的详细信息，请参阅[权限](http://shiro.apache.org/permissions.html)文档。

 

> **内部逗号**
>
> 请注意，如果单个*permissionDefinition*需要在内部以逗号分隔（例如`printer:5thFloor:print,info`），则需要用双引号（“）括起该定义以避免解析错误：`"printer:5thFloor:print,info"`

 

> **没有权限的角色**
>
> 如果您的角色不需要权限关联，则不需要在[roles]部分中列出它们。只是在[users]部分中定义角色名称就足以创建角色（如果它尚不存在）。