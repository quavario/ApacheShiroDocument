# Apache Shiro Web Support

- [配置](/account/** = ssl, authc/account/signup = anon)
  - [`web.xml`](#web.xml)
    - [Shiro 1.2及以后](http://shiro.apache.org/web.html#Web-Shiro1.2andlater)
      - [自定义`WebEnvironment`类](http://shiro.apache.org/web.html#Web-Custom%7B%7BWebEnvironment%7D%7DClass)
      - [自定义配置位置](http://shiro.apache.org/web.html#Web-CustomConfigurationLocations)
    - [Shiro 1.1及更早版本](http://shiro.apache.org/web.html#Web-Shiro1.1andearlier)
      - [自定义路径](http://shiro.apache.org/web.html#Web-CustomPath)
      - [内联配置](http://shiro.apache.org/web.html#Web-InlineConfig)
  - [Web INI配置](http://shiro.apache.org/web.html#Web-WebINIconfiguration)
    - [`[urls\]`](http://shiro.apache.org/web.html#Web-%7B%7B%5Curls%5C%7D%7D)
      - [URL路径表达式](http://shiro.apache.org/web.html#Web-URLPathExpressions)
      - 过滤链定义
        - [可用过滤器](http://shiro.apache.org/web.html#Web-AvailableFilters)
- [默认过滤器](http://shiro.apache.org/web.html#Web-DefaultFilters)
- 启用和禁用过滤器
  - [常规启用/禁用](http://shiro.apache.org/web.html#Web-GeneralEnabling%2FDisabling)
  - [特定于请求的启用/禁用](http://shiro.apache.org/web.html#Web-RequestspecificEnabling%2FDisabling)
  - [特定于路径的启用/禁用](http://shiro.apache.org/web.html#Web-PathspecificEnabling%2FDisabling)
- [会话管理](http://shiro.apache.org/web.html#Web-SessionManagement)
  - [Servlet容器会话](http://shiro.apache.org/web.html#Web-ServletContainerSessions)
    - [Servlet容器会话超时](http://shiro.apache.org/web.html#Web-ServletContainerSessionTimeout)
  - [原生会话](http://shiro.apache.org/web.html#Web-NativeSessions)
    - [`DefaultWebSessionManager`](http://shiro.apache.org/web.html#Web-%7B%7BDefaultWebSessionManager%7D%7D)
      - [本机会话超时](http://shiro.apache.org/web.html#Web-NativeSessionTimeout)
      - 会话Cookie
        - [会话Cookie配置](http://shiro.apache.org/web.html#Web-SessionCookieConfiguration)
        - [禁用会话Cookie](http://shiro.apache.org/web.html#Web-DisablingtheSessionCookie)
- [记住我的服务](http://shiro.apache.org/web.html#Web-RememberMeServices)
  - [计划支持](http://shiro.apache.org/web.html#Web-ProgrammaticSupport)
  - [基于表单的登录](http://shiro.apache.org/web.html#Web-FormbasedLogin)
  - [Cookie配置](http://shiro.apache.org/web.html#Web-Cookieconfiguration)
  - [习惯 `RememberMeManager`](http://shiro.apache.org/web.html#Web-Custom%7B%7BRememberMeManager%7D%7D)
- [JSP / GSP标记库](http://shiro.apache.org/web.html#Web-JSP%2FGSPTagLibrary)
  - [标记库配置](http://shiro.apache.org/web.html#Web-TagLibraryConfiguration)
  - [该`guest`标签](http://shiro.apache.org/web.html#Web-The%7B%7Bguest%7D%7Dtag)
  - [该`user`标签](http://shiro.apache.org/web.html#Web-The%7B%7Buser%7D%7Dtag)
  - [该`authenticated`标签](http://shiro.apache.org/web.html#Web-The%7B%7Bauthenticated%7D%7Dtag)
  - [该`notAuthenticated`标签](http://shiro.apache.org/web.html#Web-The%7B%7BnotAuthenticated%7D%7Dtag)
  - 该`principal`标签
    - [键入的主体](http://shiro.apache.org/web.html#Web-Typedprincipal)
    - [主要财产](http://shiro.apache.org/web.html#Web-Principalproperty)
  - [该`hasRole`标签](http://shiro.apache.org/web.html#Web-The%7B%7BhasRole%7D%7Dtag)
  - [该`lacksRole`标签](http://shiro.apache.org/web.html#Web-The%7B%7BlacksRole%7D%7Dtag)
  - [该`hasAnyRole`标签](http://shiro.apache.org/web.html#Web-The%7B%7BhasAnyRole%7D%7Dtag)
  - [该`hasPermission`标签](http://shiro.apache.org/web.html#Web-The%7B%7BhasPermission%7D%7Dtag)
  - [该`lacksPermission`标签](http://shiro.apache.org/web.html#Web-The%7B%7BlacksPermission%7D%7Dtag)

## 配置

将Shiro集成到Web应用程序最简单方法是在web.xml中配置Servlet ContextListener和Filter，它们知道如何读取Shiro的INI配置文件。大部分INI配置方式在[INI章节](http://shiro.apache.org/configuration.html#Configuration-INISections)部分中定义，但我们将在此处介绍一些其他特定于Web的部分。

 

> **集成Spring**
>
> Spring Framework用户不会使用本章节介绍的方式。如果您使用Spring，阅读有关[特定](http://shiro.apache.org/spring.html#[[#]]#Spring-WebApplications)于[Spring的Web配置](http://shiro.apache.org/spring.html#[[#]]#Spring-WebApplications)。

### [web.xml中](http://shiro.apache.org/web.html#web-xml)

#### Shiro 1.2及以后

在Shiro 1.2及更高版本中，配置以下代码到`web.xml` 中来初始化shiro：

```xml
<listener>
    <listener-class>org.apache.shiro.web.env.EnvironmentLoaderListener</listener-class>
</listener>

...

<filter>
    <filter-name>ShiroFilter</filter-name>
    <filter-class>org.apache.shiro.web.servlet.ShiroFilter</filter-class>
</filter>

<filter-mapping>
    <filter-name>ShiroFilter</filter-name>
    <url-pattern>/*</url-pattern>
    <dispatcher>REQUEST</dispatcher>
    <dispatcher>FORWARD</dispatcher>
    <dispatcher>INCLUDE</dispatcher>
    <dispatcher>ERROR</dispatcher>
</filter-mapping>
```

想要配置生效,  需要在以下两个路径之一配置`shiro.ini`：

1. `/WEB-INF/shiro.ini`

2. `shiro.ini` 文件位于类路径的根目录。

   `web.xml`配置详解：

- 在`EnvironmentLoaderListener` 初始化 一个 `WebEnvironment`实例（包含所有shiro需要的操作，也包括`SecurityManager`），并使其在可接入`ServletContext`。可以调用`WebUtils.getRequiredWebEnvironment(servletContext)` 方法获取`WebEnvironment` 对象。
- 在`ShiroFilter`将使用这个`WebEnvironment ` 执行需要的安全操作。
- 最后，`filter-mapping`定义确保所有请求都会经过`ShiroFilter`，建议大多数Web应用程序使用此配置。


> **filter-mapping 配置位置**
>
> 建议在配置其他filter之前配置shiro过滤器,  这样可以确保所有请求都会经过shiro过滤

##### 自定义`WebEnvironment`类

默认情况下，`EnvironmentLoaderListener`将创建一个`IniWebEnvironment`实例，该实例默认使用ini配置方式。如果您愿意，你可以在`web.xml` 中ServletContext的context-param,  如下：

```xml
<context-param>
    <param-name>shiroEnvironmentClass</param-name>
    <param-value>com.foo.bar.shiro.MyWebEnvironment</param-value>
</context-param>
```

你可以自定义一个`WebEnvironment` 子类来自定义配置文件的格式(如ini或者xml)。你可以创建`IniWebEnvironment` 子类，或者完全支持不同的配置格式。例如，如果有人想用XML而不是INI配置Shiro，他们可以创建一个基于XML的实现，例如`com.foo.bar.shiro.XmlWebEnvironment`。

##### [自定义配置位置](http://shiro.apache.org/web.html#custom-configuration-locations)

`IniWebEnvironment ` 通过  `convertPathToIni` 方法将shiro.ini 转换成ini对象 。默认情况下，会自动加载以下位置的shiro.ini（按顺序）：

1. `/WEB-INF/shiro.ini`
2. `classpath:shiro.ini`

**通过convertPathToIni将路径转换成ini对象**

![54444738677](..\image\1544447386774.png)

他会使用最先找的一个。

但是，如果你想将你的配置在其他位置，你可以与另一个指定的位置`context-param`在`web.xml`：

```xml
<context-param>
    <param-name>shiroConfigLocations</param-name>
    <param-value>YOUR_RESOURCE_LOCATION_HERE</param-value>
</context-param>
```

默认情况下，`param-value` 会被 `ServletContext.getResource`方法进行解析。这里我们可以定义param-value为shiro.ini 路径 `/WEB-INF/some/path/shiro.ini` . 

你也可以下面三种格式设置param-value的值，例如：

- `file:/home/foobar/myapp/shiro.ini`
- `classpath:com/foo/bar/shiro.ini`
- `url:http://confighost.mycompany.com/myapp/shiro.ini`

#### Shiro 1.1及更早版本

在1.1或更早版本的Web应用程序中启用Shiro的最简单方法是定义IniShiroFilter并指定`filter-mapping`：

```xml
<filter>
    <filter-name>ShiroFilter</filter-name>
    <filter-class>org.apache.shiro.web.servlet.IniShiroFilter</filter-class>
</filter>

...

<!-- Make sure any request you want accessible to Shiro is filtered. /* catches all -->
<!-- requests.  Usually this filter mapping is defined first (before all others) to -->
<!-- ensure that Shiro works in subsequent filters in the filter chain:             -->
<filter-mapping>
    <filter-name>ShiroFilter</filter-name>
    <url-pattern>/*</url-pattern>
    <dispatcher>REQUEST</dispatcher>
    <dispatcher>FORWARD</dispatcher>
    <dispatcher>INCLUDE</dispatcher>
    <dispatcher>ERROR</dispatcher>
</filter-mapping>
```

此定义要求您的INI配置位于类路径根目录下的shiro.ini文件中（例如`classpath:shiro.ini`）。

##### 自定义路径

如果你不想在`/WEB-INF/shiro.ini`或者`classpath:shiro.ini` 配置，你可以指定一个路径,  并将该路径配置为 `init-param`  下name为`configPath` 的值  示例配置：

```xml
<filter>
    <filter-name>ShiroFilter</filter-name>
    <filter-class>org.apache.shiro.web.servlet.IniShiroFilter</filter-class>
    <init-param>
        <!-- param-name必须为configPath -->
        <param-name>configPath</param-name>
        <!--此处配置ini的路径-->
        <param-value>/WEB-INF/anotherFile.ini</param-value>
    </init-param>
</filter>

...
```

没有配置前缀的的configPath的值, 会通过ServletContext.getResource方法解析

> **ServletContext资源路径** - Shiro1.2版本之后
>
> 在Shiro1.2版本之后可以不写资源前缀(就是classpath: 和url:等)。在1.1和更早的版本中，所有`configPath`定义必须指定`classpath:`，`file:`或`url:`前缀。

你也可以指定其他非`ServletContext` 资源路径,  通过使用`classpath:`，`url:`或`file:`前缀分别表示classpath中，URL或文件系统位置。例如：

```xml
...
<init-param>
    <param-name>configPath</param-name>
    <param-value>url:http://configHost/myApp/shiro.ini</param-value>
</init-param>
...
```

##### 内联配置

最后，还可以在不使用INI文件的情况下将您的INI配置嵌入到web.xml中。您可以使用`config init-param`而不是`configPath`：

```xml
<filter>
    <filter-name>ShiroFilter</filter-name>
    <filter-class>org.apache.shiro.web.servlet.IniShiroFilter</filter-class>
    <init-param>
        <param-name>config</param-name>
    	<param-value>
    	# INI Config Here
   		</param-value>
    </init-param>
</filter>
...
```

内联配置通常适用于小型或简单的应用程序，但通常更方便的是将其外部化到专用的shiro.ini文件中，原因如下：

- 您可能会编辑很多安全配置，并且不希望添加到web.xml文件中
- 您可能希望将安全配置部分与web.xml配置的其余部分分开
- 您的配置文件可能会变得很大，并且您希望将web.xml保持精简并且更易于阅读
- 您有一个复杂的系统，可能需要在多个位置引用相同的shiro配置

这取决于您。

### Web INI配置

除了之前我们提到过的`[main]`，`[users]` 和`[roles]` 区域，你还可以在`shiro.ini` 中指定一个特定与的web应用的`[urls]`中的区域：

```ini
# [main], [users] and [roles] above here
...
[urls]
...
```

`[urls]` 区域可以做一些现有框架不支持的一些事情,  例如:  我们可以定义所有URL路径所匹配的点对点过滤器链！

#### [[urls]](http://shiro.apache.org/web.html#urls-)

该部分中每一行的格式如下：

```ini
_URL_Ant_Path_Expression_ = _Path_Specific_Filter_Chain_
```

例如：

```ini
...
[urls]

/index.html = anon
/user/create = anon
/user/** = authc
/admin/** = authc, roles[administrator]
/rest/** = authc, rest
/remoting/rpc/** = authc, perms["remote:invoke"]
```

接下来我们将详细介绍这些代码的含义。

等号（=）左侧的标记是相对于Web应用程序的上下文根的[Ant](http://ant.apache.org/)风格路径表达式。

例如，假设您有以下`[urls]`：

```ini
/account/** = ssl, authc
```

此行说明任何访问`/account` 或者他的子目录的请求都会被触发`ssl,authc` 过滤器链,  过滤器链我们稍后讲解

请注意，所有路径表达式都与应用程序的上下文根相关。这意味着，如果您将应用程序部署到，`www.somehost.com/myapp`然后再将其部署到`www.anotherhost.com`（没有'myapp'子路径），则模式匹配仍然有效。所有路径都相对于[HttpServletRequest.getContextPath（）](http://docs.oracle.com/javaee/1.3/api/javax/servlet/http/HttpServletRequest.html#getContextPath--)值。

> **配置顺序**
>
> URL路径表达式按照传入请求的定义顺序和*FIRST MATCH WINS*进行评估。例如，让我们假设有以下链定义：
>
> ```ini
> /account/** = ssl, authc
> /account/signup = anon
> ```
>
> 如果请求`/account/signup`路径,  那么会被`ssl,authc` 处理,  永远不会被后面的处理.

##### 过滤链定义

等号（=）右侧的标记是逗号分隔的过滤器列表，用于匹配该路径的请求。它必须符合以下格式：

```ini
filter1[optional_config1], filter2[optional_config2], ..., filterN[optional_configN]
```

哪里：

- *filterN*是`[main]`节和中定义的过滤器bean的名称
- `[optional_configN]`是一个可选的括号中的字符串，对于*该特定路径的*特定过滤器具有意义（每个过滤器，*特定于路径的*配置！）。如果过滤器不需要该URL路径的特定配置，您可以丢弃括号，`filterN[]`只需变为`filterN`。

并且因为过滤器令牌定义了链（也就是列表），所以请记住顺序很重要！按照希望请求流经链的顺序定义逗号分隔的列表。

最后，如果不满足其必要条件，每个过滤器都可以自由地处理响应（例如，执行重定向，使用HTTP错误代码进行响应，直接呈现等）。否则，它应该允许请求继续通过链到最终目标视图。

 

> **Tip**
>
> 能够对路径特定配置做出反应，即`[optional_configN]`过滤器令牌的一部分，是Shiro过滤器可用的独特功能。
>
> 如果您想创建自己的`javax.servlet.Filter`实现,  你可以创建[org.apache.shiro.web.filter.PathMatchingFilter](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/web/filter/PathMatchingFilter.html) 子类

###### [可用过滤器](http://shiro.apache.org/web.html#available-filters)

可用于过滤器链定义的过滤器“池”在本`[main]`节中定义。在主要部分中分配给它们的名称是要在过滤器链定义中使用的名称。例如：

```ini
[main]
...
myFilter = com.company.web.some.FilterImplementation
myFilter.property1 = value1
...

[urls]
...
/some/path/** = myFilter
```

## 默认过滤器

运行Web应用程序时，Shiro将创建一些有用的`Filter`实例，并 自动在`[main] ` 部分中使用它们。您可以像配置`main`任何其他bean一样配置它们，并在链定义中引用它们。例如：

```ini
[main]
...
# Notice how we didn't define the class for the FormAuthenticationFilter ('authc') - it is instantiated and available already:
authc.loginUrl = /login.jsp
...

[urls]
...
# 确保前端用户是通过认证的. 否则就跳转到我们上面配置的`authc.loginUrl`对应的路径
# 通过认证过,在跳转会之前的页面
# were trying to view:
/account/** = authc
...
```

默认可用的`Filter` 对象是由枚举类型的`DefaultFilter` 定义的,  枚举的name字段是可用于配置的名称 

| 过滤器名称        | 类                                                           |
| ----------------- | ------------------------------------------------------------ |
| anon              | [org.apache.shiro.web.filter.authc.AnonymousFilter](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/web/filter/authc/AnonymousFilter.html) |
| authc             | [org.apache.shiro.web.filter.authc.FormAuthenticationFilter](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/web/filter/authc/FormAuthenticationFilter.html) |
| authcBasic        | [org.apache.shiro.web.filter.authc.BasicHttpAuthenticationFilter](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/web/filter/authc/BasicHttpAuthenticationFilter.html) |
| logout            | [org.apache.shiro.web.filter.authc.LogoutFilter](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/web/filter/authc/LogoutFilter.html) |
| noSessionCreation | [org.apache.shiro.web.filter.session.NoSessionCreationFilter](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/web/filter/session/NoSessionCreationFilter.html) |
| perms             | [org.apache.shiro.web.filter.authz.PermissionsAuthorizationFilter](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/web/filter/authz/PermissionsAuthorizationFilter.html) |
| port              | [org.apache.shiro.web.filter.authz.PortFilter](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/web/filter/authz/PortFilter.html) |
| rest              | [org.apache.shiro.web.filter.authz.HttpMethodPermissionFilter](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/web/filter/authz/HttpMethodPermissionFilter.html) |
| roles             | [org.apache.shiro.web.filter.authz.RolesAuthorizationFilter](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/web/filter/authz/RolesAuthorizationFilter.html) |
| ssl               | [org.apache.shiro.web.filter.authz.SslFilter](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/web/filter/authz/SslFilter.html) |
| user              | [org.apache.shiro.web.filter.authc.UserFilter](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/web/filter/authc/UserFilter.html) |

## 启用和禁用过滤器

与其他过滤器链定义机制一样(例如:web.xml, ini配置),  只需要将过滤器包含在过滤器链中,  就可以开启该过滤器,  通过移除过滤器来禁用.

但Shiro 1.2 新增了一个功能,  无需从过滤器链中移除过滤器, 即可禁用或起用他们.  如果开启过滤器,  请求会被过滤器拦截,  如果关闭,  filter会允许请求立即直接通过, 到过滤器链中下一个过滤器. 

这是一个强大的概念，比起更改固定而且复杂的过滤器链定义,  这种方式在大部分环境下,  启用和禁用过滤器更加方便

Shiro通过[OncePerRequestFilter](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/web/servlet/OncePerRequestFilter.html)抽象父类完成此操作。shiro提供的所有的filter实现都是这一类的子类，因此可以启用或禁用它们，而无需从过滤器链中删除它们。如果您还需要此功能，则可以将此类子类化为您自己的过滤器实现*。

* [SHIRO-224](https://issues.apache.org/jira/browse/SHIRO-224)有望为任何过滤器启用此功能，而不仅仅是`OncePerRequestFilter` 的子类。如果这对您很重要，请投票支持该问题。

### 常规启用/禁用

[OncePerRequestFilter](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/web/servlet/OncePerRequestFilter.html)（和所有它的子类）支持启用/跨所有请求以及基于每个请求基础禁用。

通过将其`enabled`属性设置为true或false来为所有请求启用或禁用filter。默认设置是`true`因为如果在链中配置了大多数过滤器，则必须执行这些过滤器。

例如，在shiro.ini中：

```ini
[main]
...
# 禁用ssl过滤器
ssl.enabled = false

[urls]
...
/some/path = ssl, authc
/another/path = ssl, roles[admin]
...
```

此示例说明大部属URL路径都需被SSL连接保护,  但是在开发环境中,  设置SSL可能不是很方便,  而且浪费时间. 所以你可以在开发时关闭ssl过滤器.  当部署到生产环境中,  在配饰ssl的enabled属性为true.   这比手动更改URL路径或者设置两个shiro配置要简单的多

### 基于请求的启用/禁用

`OncePerRequestFilter`实际上确定是否根据其`isEnabled(request, response)`方法启用或禁用过滤器。

![1545104309505](..\image\1545104309505.png)

此方法默认返回`enabled`属性的值，该属性通常用于启用/禁用上述所有请求。如果要根据*请求特定*条件启用或禁用过滤器，可以覆盖该`OncePerRequestFilter` `isEnabled(request,response)`方法以执行更具体的检查。



### 基于路径的启用/禁用

Shiro的[PathMatchingFilter](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/web/filter/PathMatchingFilter.html)（`OncePerRequestFilter` 的子类) 可以根据需要被过滤的*路径*对配置作出反应。这意味着除了传入请求和响应之外，您还可以根据路径和基于路径的配置启用或禁用过滤器。

如果您需要能够对匹配路径和基于路径的配置做出反应来确定是启用还是禁用过滤器，你需要重写`PathMatchingFilter.isEnabled(request,response,path,pathConfig) ` ,  而不是`OncePerRequestFilter.isEnabled(request,response)`.方法。

## 会话管理

### Servlet容器会话

在Web环境中，Shiro默认的会话管理器[`SessionManager`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/session/mgt/SessionManager.html)实现是[`ServletContainerSessionManager`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/web/session/mgt/ServletContainerSessionManager.html)。这个非常简单的实现将所有会话管理职责（包括servlet容器支持的会话集群）委托给运行时Servlet容器。它本质上是shiro session API和servlet容器之前的桥梁.

使用这个默认的会话管理好处就是,  可以兼容现在的servelt容器的配置(超时,  或者特定于容器的集群机制等)

这个默认值的缺点是你被绑定到servlet容器的特定会话行为。例如，如果要集群会话，但是在生产中使用Jetty进行测试和使用Tomcat，则特定于容器的配置（或代码）将不可移植。

#### Servlet容器会话超时

如果使用默认的servlet容器支持，session的过期时间就是在web.xml中配置的时间。例如：

```xml
<session-config>
  <!-- 30分钟过期, 这是默认值 -->
  <session-timeout>30</session-timeout>
</session-config>
```

> 译者: **什么是Servlet容器?**
>
> 就是Tomcat/Jetty/Jboss等

### 原生Session

如果你希望你的session配置信息和集群配置可以跨servlet容器或者你希望控制指定的session和集群配置,  你可以使用shiro自带的session.

Shiro自己的原生企业会话管理对象将支持所有`Subject`和`HttpServletRequest`会话并完全绕过servlet容器。但请放心--Shiro直接实现Servlet规范的相关部分，因此任何现有的Web / http相关代码都能正常工作，并且永远不需要“知道”Shiro是如何管理会话的。

> **原生session**
>
> 这里我们也叫做本地session或者本机session,  就是shiro自己实现的session管理

#### DefaultWebSessionManager

要为Web应用程序启用原生session管理，您需要配置一个支持本机Web的session管理器来覆盖默认的基于servlet容器的session管理器。你可以为`securityManager`配置`DefaultWebSessionManager`。例如，在`shiro.ini` 中：

**shiro.ini本地Web会话管理**

```ini
[main]
...
sessionManager = org.apache.shiro.web.session.mgt.DefaultWebSessionManager
# configure properties (like session timeout) here if desired

# 使用原生会话管理:
securityManager.sessionManager = $sessionManager
```

声明后，您可以为 `DefaultWebSessionManager` 配置本地会话选项，如会话超时和群集配置，如[会话管理](http://shiro.apache.org/session-management.html)部分中所述。

##### 本地session过期

配置`DefaultWebSessionManager`实例后，会话超时的配置如[会话管理：会话超时](http://shiro.apache.org/session-management.html#SessionManagement-sessionTimeout) 中所述

##### 会话Cookie

`DefaultWebSessionManager`支持用于web配置的属性：

- `sessionIdCookieEnabled` （布尔值）
- `sessionIdCookie`，一个[Cookie](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/web/servlet/Cookie.html)实例。

> **Cookie作为模板**
>
> `sessionIdCookie`属性本质上是一个模板.  当您配置了`Cookie`属性，`DefaultWebSessionManager` 会调用`response.addHeader` 方法将设置cookie
>
> **DefaultWebSessionManager的属性**
>
> ![1545121520729](..\image\1545121520729.png)
>
> **Cookie对象(Shiro自己实现的Cookie对象实例)**
>
> ![1545122899580](..\image\1545122899580.png)

###### 会话Cookie配置

DefaultWebSessionManager的`sessionIdCookie`默认实例是 [`SimpleCookie`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/web/servlet/SimpleCookie.html)。这个简单的实现支持JavaBean风格设置cookie属性

例如，您可以设置Cookie的domain属性：

```ini
[main]
...
securityManager.sessionManager.sessionIdCookie.domain = foo.com
```

有关其他属性，请参阅[SimpleCookie JavaDoc](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/web/servlet/SimpleCookie.html)。

cookie的默认名称`JSESSIONID`与servlet规范一致。此外，Shiro的cookie支持[`HttpOnly`](https://en.wikipedia.org/wiki/HTTP_cookie#HttpOnly_cookie) 标签。`sessionIdCookie.httpOnly`  默认为`true`

> **注意**
>
> 即使在Servlet 2.4和2.5版本中，Shiro的`Cookie`概念也支持`HttpOnly ` 属性（而Servlet API仅在2.6或更高版本中支持它）。

###### 禁用会话Cookie

如果您不想使用会话cookie，可以通过将`sessionIdCookieEnabled`属性配置为false 来禁用它们。例如：

**禁用本机会话cookie**

```ini
[main]
...
securityManager.sessionManager.sessionIdCookieEnabled = false
```

## 记住我服务

如果`AuthenticationToken`实现[`org.apache.shiro.authc.RememberMeAuthenticationToken`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authc/RememberMeAuthenticationToken.html) 接口，Shiro将执行'rememberMe'服务。此接口指定一个方法：

```java
boolean isRememberMe();
```

如果此方法返回`true`，Shiro将记住最终用户在会话中的身份。

 

UsernamePasswordToken和RememberMe

------

经常使用的`UsernamePasswordToken`已实现`RememberMeAuthenticationToken`接口并支持rememberMe登录。

### [计划支持](http://shiro.apache.org/web.html#programmatic-support)

要以编程方式使用rememberMe，可以将值设置为`true`支持此配置的类。例如，使用标准`UsernamePasswordToken`：

```
UsernamePasswordToken token = new UsernamePasswordToken(username, password);

token.setRememberMe(true);

SecurityUtils.getSubject().login(token);
...
```

### 基于表单的登录

对于Web应用程序，`authc`默认情况下过滤器为a [`FormAuthenticationFilter`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/web/filter/authc/FormAuthenticationFilter.html)。这支持将“rememberMe”布尔值作为表单/请求参数读取。默认情况下，它期望命名请求参数`rememberMe`。这是一个支持这个的示例shiro.ini配置：

```
[main]
authc.loginUrl = /login.jsp

[urls]

# your login form page here:
login.jsp = authc
```

在您的网络表单中，有一个名为“rememberMe”的复选框：

```
<form ...>

   Username: <input type="text" name="username"/> <br/>
   Password: <input type="password" name="password"/>
    ...
   <input type="checkbox" name="rememberMe" value="true"/>Remember Me?
   ...
</form>
```

默认情况下，`FormAuthenticationFilter`会寻找名为请求参数`username`，`password`和`rememberMe`。如果这些与您在表单中使用的表单字段名称不同，则需要在其上配置名称`FormAuthenticationFilter`。例如，在`shiro.ini`：

```
[main]
...
authc.loginUrl = /whatever.jsp
authc.usernameParam = somethingOtherThanUsername
authc.passwordParam = somethingOtherThanPassword
authc.rememberMeParam = somethingOtherThanRememberMe
...
```

### [Cookie配置](http://shiro.apache.org/web.html#cookie-configuration)

您可以`rememberMe`通过设置默认的{{RememberMeManager}}各种cookie属性来配置cookie的功能。例如，在shiro.ini中：

```
[main]
...

securityManager.rememberMeManager.cookie.name = foo
securityManager.rememberMeManager.cookie.maxAge = blah
...
```

请参阅[`CookieRememberMeManager`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/web/mgt/CookieRememberMeManager.html)支持[`SimpleCookie`](http://shiro.apache.org/static/current/apidocs/src-html/org/apache/shiro/web/servlet/SimpleCookie.html)JavaDoc以获取配置属性。

### 习惯 `RememberMeManager`

应该注意的是，如果默认的基于cookie的`RememberMeManager`实现不能满足您的需求，您可以插入您喜欢的`securityManager`任何其他对象引用：

```ini
[main]
...
rememberMeManager = com.my.impl.RememberMeManager
securityManager.rememberMeManager = $rememberMeManager
```

## [JSP / GSP标记库](http://shiro.apache.org/web.html#jsp-gsp-tag-library)

Apache Shiro提供了一个`Subject`-aware JSP / GSP标记库，允许您根据当前Subject的状态控制JSP，JSTL或GSP页面输出。这对于基于查看网页的当前用户的身份和授权状态来个性化视图非常有用。

### [标记库配置](http://shiro.apache.org/web.html#tag-library-configuration)

标签库描述符（TLD）文件捆绑`shiro-web.jar`中`META-INF/shiro.tld`的文件。要使用任何标记，请将以下行添加到JSP页面的顶部（或者在您定义页面指令的任何位置）：

```jsp
<%@ taglib prefix="shiro" uri="http://shiro.apache.org/tags" %>
```

我们使用`shiro`前缀来指示shiro标记库命名空间，但您可以指定任何您喜欢的名称。

现在我们将介绍每个标记并显示它如何用于呈现页面。

### 该`guest`标签

`guest`仅当当前`Subject`被视为“访客”时，标签才会显示其包装内容。客人是任何`Subject`没有身份的客人。也就是说，我们不知道用户是谁，因为他们没有登录，并且他们没有被记住（来自记住我的服务）来自之前的站点访问。

例：

```jsp
<shiro:guest>
    Hi there!  Please <a href="login.jsp">Login</a> or <a href="signup.jsp">Signup</a> today!
</shiro:guest>
```

该`guest`标签是的逻辑相反的[`user`](http://shiro.apache.org/web.html#Web-usertag)标签。

### 该`user`标签

`user`仅当当前`Subject`被视为“用户”时，标签才会显示其包装内容。此上下文中的“用户”被定义为`Subject`具有已知身份的“用户” ，可以是成功的身份验证，也可以是“RememberMe”服务。请注意，此标记在语义上与经过[身份验证的](http://shiro.apache.org/web.html#Web-authenticatedtag)标记不同，后者比此标记更具限制性。

例：

```jsp
<shiro:user>
    Welcome back John!  Not John? Click <a href="login.jsp">here<a> to login.
</shiro:user>
```

该`user`标签是的逻辑相反的[`guest`](http://shiro.apache.org/web.html#Web-guesttag)标签。

### 该`authenticated`标签

仅在当前用户*在其当前会话期间*成功通过身份验证时才显示正文内容。它比'user'标签更具限制性。它在逻辑上与'notAuthenticated'标签相反。

`authenticated`仅当*当前会话期间*当前`Subject`已成功验证*时*，标记才会显示其包装内容。它是一个比[用户](http://shiro.apache.org/web.html#Web-usertag)更严格的标签，用于保证敏感工作流程中的身份。

例：

```
<shiro:authenticated>
    <a href="updateAccount.jsp">Update your contact information</a>.
</shiro:authenticated>
```

该`authenticated`标签是的逻辑相反的[`notAuthenticated`](http://shiro.apache.org/web.html#Web-notauthenticatedtag)标签。

### 该`notAuthenticated`标签

该`notAuthenticated`如果当前标签将显示它的包裹内容`Subject`已**不**还成功地在本届会议期间通过身份验证。

例：

```
<shiro:notAuthenticated>
    Please <a href="login.jsp">login</a> in order to update your credit card information.
</shiro:notAuthenticated>
```

该`notAuthenticated`标签是的逻辑相反的[`authenticated`](http://shiro.apache.org/web.html#Web-authenticatedtag)标签。

### 该`principal`标签

该`principal`标签将输出Subject的[`principal`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/subject/Subject.html#getPrincipal--)（识别属性），或主要的属性。

如果没有任何标记属性，标记将呈现`toString()`主体的值。例如（假设主体是String用户名）：

```
Hello, <shiro:principal/>, how are you today?
```

这（大部分）相当于以下内容：

```
Hello, <%= SecurityUtils.getSubject().getPrincipal().toString() %>, how are you today?
```

#### [键入的主体](http://shiro.apache.org/web.html#typed-principal)

该`principal`标签假定默认情况下，校长打印的是`subject.getPrincipal()`价值。但是，如果您想要打印一个*不是*主要主体的值，而是在主体的{ [主要集合中](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/subject/Subject.html#getPrincipals--)打印另一个值，则可以按类型获取该主体并打印该值。

例如，假设ID在主要集合中，打印主题的用户ID（而不是用户名）：

```
User ID: <principal type="java.lang.Integer"/>
```

这（大部分）相当于以下内容：

```
User ID: <%= SecurityUtils.getSubject().getPrincipals().oneByType(Integer.class).toString() %>
```

#### [主要财产](http://shiro.apache.org/web.html#principal-property)

但是，如果主体（上面的默认主要主体或“类型”主体）是复杂对象而不是简单字符串，并且您想引用该主体上的属性，该怎么办？您可以使用该`property`属性指示要读取的属性的名称（必须可通过兼容JavaBeans的getter方法访问）。例如（假设主要主体是User对象）：

```
Hello, <shiro:principal property="firstName"/>, how are you today?
```

这（大部分）相当于以下内容：

```
Hello, <%= SecurityUtils.getSubject().getPrincipal().getFirstName().toString() %>, how are you today?
```

或者，与type属性结合使用：

```
Hello, <shiro:principal type="com.foo.User" property="firstName"/>, how are you today?
```

这大致等同于以下内容：

```
Hello, <%= SecurityUtils.getSubject().getPrincipals().oneByType(com.foo.User.class).getFirstName().toString() %>, how are you today?
```

### 该`hasRole`标签

该`hasRole`标签将显示它的包裹内容只有当电流`Subject`被分配指定的角色。

例如：

```
<shiro:hasRole name="administrator">
    <a href="admin.jsp">Administer the system</a>
</shiro:hasRole>
```

该`hasRole`标签是的逻辑相反[lacksRole](http://shiro.apache.org/web.html#Web-lacksroletag)标签。

### 该`lacksRole`标签

`lacksRole`仅当未为当前`Subject` **未**分配指定角色时，标记才会显示其包装内容。

例如：

```
<shiro:lacksRole name="administrator">
    Sorry, you are not allowed to administer the system.
</shiro:lacksRole>
```

该`lacksRole`标签是的逻辑相反[hasRole](http://shiro.apache.org/web.html#Web-hasroletag)标签。

### 该`hasAnyRole`标签

该`hasAnyRole`如果当前标签会显示它的包裹内容`Subject`被分配*任何*指定的角色从一个逗号分隔的角色名称列表。

例如：

```
<shiro:hasAnyRoles name="developer, project manager, administrator">
    You are either a developer, project manager, or administrator.
</shiro:hasAnyRoles>
```

该`hasAnyRole`标签目前还没有一个逻辑相反的标签。

### 该`hasPermission`标签

`hasPermission`仅当当前`Subject`“具有”（暗示）指定的权限时，标记才会显示其包装内容。也就是说，用户具有指定的能力。

例如：

```
<shiro:hasPermission name="user:create">
    <a href="createUser.jsp">Create a new User</a>
</shiro:hasPermission>
```

该`hasPermission`标签是的逻辑相反[lacksPermission](http://shiro.apache.org/web.html#Web-lackspermissiontag)标签。

### 该`lacksPermission`标签

`lacksPermission`仅当当前`Subject` **DOES没有**（暗示）指定的权限时，标记才会显示其包装内容。也就是说，用户**没有**指定的能力。

例如：

```
<shiro:lacksPermission name="user:delete">
    Sorry, you are not allowed to delete user accounts.
</shiro:lacksPermission>
```

该`lacksPermission`标签是的逻辑相反[调用hasPermission](http://shiro.apache.org/web.html#Web-haspermissiontag)标签。