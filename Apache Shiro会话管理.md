# 会话管理

安全框架领域Apache Shiro提供了一些独特功能：适用于任何应用程序的完整企业级会话解决方案，从最简单的命令行和移动App到最大的企业级Web应用程序集群。

这对许多应用程序有很大的影响 - 在Shiro之前，如果您需要会话支持，则需要在Web容器中部署应用程序或使用EJB Stateful Session Bean。与这两种机制中的任何一种相比，Shiro的Session支持更易于使用和管理，并且无论容器如何，它都可以在任何应用程序中使用。

即使您在Servlet或EJB容器中部署应用程序，仍然有令人信服的理由使用Shiro的Session支持而不是容器的。以下列出了Shiro Session支持提供的最令人满意的功能：

**特性**

- **基于POJO / J2SE（IoC友好）** - Shiro中的所有内容（包括会话和会话管理的所有方面）都是基于接口的，并使用POJO实现。这允许您使用任何与JavaBeans兼容的配置格式（如JSON，YAML，Spring XML或类似机制）轻松配置所有会话组件。您还可以根据需要轻松扩展Shiro的组件或编写自己的组件，甚至完全自定义会话管理功能。
- **自定义会话存储** - 由于Shiro的会话对象是基于POJO的，因此会话数据可以轻松存储在任何数据库中。这允许您自定义会话数据的存放位置 - 例如，文件系统，内存中，网络分布式缓存，关系数据库或专有数据存储。
- **与容器无关的聚类！** - Shiro的会话可以使用任何易于使用的网络缓存产品轻松集群，如Ehcache + Terracotta，Coherence，GigaSpaces等。人。这意味着您可以为Shiro配置一次且仅一次的会话群集，无论您部署到哪个容器，您的会话都将以相同的方式进行群集。无需特定于容器的配置！
- **异构客户端访问** - 与EJB或Web会话不同，Shiro会话可以通过各种客户端技术“共享”。例如，桌面应用程序可以“查看”和“共享”Web应用程序中同一用户使用的相同物理会话。我们不知道Shiro以外的任何框架都可以支持这一点。
- **事件监听器** - 事件监听器允许您在会话的生命周期内监听生命周期事件。您可以侦听这些事件并对它们做出反应以获取自定义应用程序行为 - 例如，在会话过期时更新用户记录。
- **主机地址保留** - Shiro会话保留启动会话的主机的IP地址或主机名。这允许您确定用户所在的位置并做出相应的反应（通常在IP关联具有确定性的Intranet环境中很有用）。
- **不活动/过期支持** - 会话由于预期的不活动而到期，但是`touch()`如果需要，可以通过一种方法延长它们以保持它们“活着”。这在用户可能正在使用桌面应用程序的Rich Internet Application（RIA）环境中很有用，但可能无法定期与服务器通信，但服务器会话不应过期。
- **透明Web使用** - Shiro的Web支持完全实现并支持Sessions的Servlet 2.5规范（`HttpSession`接口及其所有相关API）。这意味着您可以在现有Web应用程序中使用Shiro会话，而无需更改任何现有Web代码。
- **可以用于SSO** - 因为Shiro会话是基于POJO的，所以它们很容易存储在任何数据源中，并且如果需要，它们可以跨应用程序“共享”。我们将此称为“穷人的SSO”，它可用于提供简单的登录体验，因为共享会话可以保留身份验证状态。

## 使用Session 

通过当前线程的`Subject` 获得`Session` ：

```java
Subject currentUser = SecurityUtils.getSubject();

Session session = currentUser.getSession();
session.setAttribute( "someKey", someValue);
```

`currentUser.getSession()`  会调用`currentUser.getSession(true)`。

![](D:\Dev\Shiro文档\ApacheShiroDocument\image\1543458963449.png)

对于那些熟悉`HttpServletRequest`API的人来说，`Subject.getSession(boolean create)`的功能与`HttpServletRequest.getSession(boolean create)` 相同：

- 如果`Subject`已经有a `Session`，则忽略boolean参数并`Session`立即返回
- 如果`Subject`还没有a `Session`和`create`boolean参数`true`，则会创建并返回一个新会话。
- 如果`Subject`还没有a `Session`和`create`boolean参数`false`，则不会创建新会话并`null`返回。

 

> **适用任何应用程序**
>
> `getSession` 调用适用于任何应用程序，甚至非Web应用程序。

`subject.getSession(false)` 可以在开发框架代码时发挥良好作用，以确保不会不必要地创建会话。

获得`Session` 后,  您可以使用它执行许多操作，例如设置和获取属性，设置session过期时间等。请参阅[Session JavaDoc](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/session/Session.html) 了解详情。

## SessionManager

SessionManager，正如其名称所暗示的那样，管理应用程序中*所有*主题的会话- 创建，删除，不活动和验证等。与Shiro中的其他核心架构组件一样，`SessionManager` 是由`SecurityManager` 管理的顶级组件。

`DefaultSecurityManager` 默认使用[`DefaultSessionManager`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/mgt/DefaultSecurityManager.html) (SessionManager 默认实现是DefaultSecurityMangager)。`DefaultSessionManager`提供了应用程序所需的所有企业级会话管理功能，如会话验证，孤立清理等。这可以在任何应用程序中使用。

 

> **Web应用程序**
>
> Web应用程序使用不同的`SessionManager`实现 有关特定于Web的会话管理信息，请参阅[Web](http://shiro.apache.org/web.html)文档。

与其他SecurityManager管理的组件一样,  可以通过`SecurityManager`  的`getSessionManager()`/`setSessionManager()`  方法获取/设置 `SessionManager`。或者，如果使用`shiro.ini` [配置](http://shiro.apache.org/configuration.html) ,  例如：

**在shiro.ini中配置新的SessionManager**

```ini
[main]
...
sessionManager = com.foo.my.SessionManagerImplementation
securityManager.sessionManager = $sessionManager
```



### Session过期 

默认情况下，Shiro的`SessionManager`实现默认为30分钟的会话超时。也就是说，如果`Session`创建的后,  没有使用（[`lastAccessedTime`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/session/Session.html#getLastAccessTime--)未更新）超过30分钟，`Session`则认为已过期且不再允许使用。

您可以设置` SessionManager`的`globalSessionTimeout`属性，以定义所有会话的默认超时值。例如，如果您希望超时为一小时而不是30分钟：

**在shiro.ini中设置默认会话超时**

```ini
[main]
...
# 3,600,000 milliseconds = 1 hour
securityManager.sessionManager.globalSessionTimeout = 3600000
```

#### 设置单个Session过期时间

`globalSessionTimeout`是所有新创建的session默认值。您可以通过设置单个会话的[`timeout`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/session/Session.html#setTimeout-long-)值来控制每个会话的会话超时。与上面一样`globalSessionTimeout`，值是以**毫秒**（而不是秒）**为单位的**时间。

### 监听Session

Shiro 支持`SessionListener` 概念,  可以监听重要的session事件。您可以实现[`SessionListener`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/session/SessionListener.html)接口（或[`SessionListenerAdapter`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/session/SessionListenerAdapter.html)）并相应地对会话操作做出反应。

由于`SessionManager.sessionListeners`属性是一个集合，您可以 为`SessionManager` 配置一个或多个监听器，就像配置其他集合一样`shiro.ini`：

**shiro.ini中的SessionListener配置**

```ini
[main]
...
aSessionListener = com.foo.my.SessionListener
anotherSessionListener = com.foo.my.OtherSessionListener

securityManager.sessionManager.sessionListeners = $aSessionListener, $anotherSessionListener, etc.
```

 

> **All Session Events**
>
> `SessionListeners` 监听所有session事件- 而不仅仅是针对单个session。

### Session持久化

无论何时创建或更新会话，其数据都需要持久保存到某个位置(例如,  内存/数据库等)，以便应用程序稍后可以访问它。类似地，当会话无效且使用时间较长时，需要从存储中删除它，以防存储空间耗尽。在`SessionManager` 委托`SessionDAO` 组件 实现创建/读取/更新/删除（CRUD）操作，它反映了[数据访问对象（DAO）](https://en.wikipedia.org/wiki/Data_access_object)设计模式。

SessionDAO的强大之处在于只要实现了这个接口session数据可以存储到任何地方。这意味着您的会话数据可以存储在内存，文件系统，关系数据库或NoSQL数据存储或其他任何位置。您可以控制持久性行为。

您可以将`SessionDAO` 实例对象配置为`SessionManager`上的属性。例如，在shiro.ini中：

**在shiro.ini中配置SessionDAO**

```ini
[main]
...
sessionDAO = com.foo.my.SessionDAO
securityManager.sessionManager.sessionDAO = $sessionDAO
```

但是，正如您所料，Shiro已经有一些很好的`SessionDAO`实现，您可以根据自己的需要使用。

> **Web应用程序**
>
> `securityManager.sessionManager.sessionDAO = $sessionDAO` 仅在本机使用时生效。默认情况下，Web应用程序不使用本机会话管理器，而是使用Servlet容器默认的会话管理器，该管理器不支持SessionDAO。如果要在基于Web的应用程序中启用SessionDAO以进行自定义会话存储或会话群集，则必须首先配置本机Web会话管理器。例如：
>
> ```ini
> [main]
> ...
> sessionManager = org.apache.shiro.web.session.mgt.DefaultWebSessionManager
> securityManager.sessionManager = $sessionManager
> 
> # Configure a SessionDAO and then set it:
> securityManager.sessionManager.sessionDAO = $sessionDAO
> ```

 

> **配置SessionDAO**
>
> Shiro的默认配置原生SessionManagers使用**仅内存的**会话存储。这不适合大多数生产应用。大多数生产应用程序都希望配置提供的EHCache支持（见下文）或提供自己的`SessionDAO`实现。请注意，Web应用程序默认使用基于servlet容器的SessionManager，并且没有此问题。这只是使用Shiro原生SessionManager时的一个问题。

#### EHCache SessionDAO

EHCache默认是关闭的，如你不打算自定义`SessionDAO`，强烈推荐你打开EhCache支持。EHCache SessionDao支持session存储到内存中,  也支持存储到硬盘中。

 

> **使用EHCache作为默认SessionDAO**
>
> 如果您没有编写自定义`SessionDAO`，请务必在Shiro配置中启用EHCache。开启EHCache对Sessions，缓存认证和授权数据有诸多好处。有关更多信息，请参阅[缓存](http://shiro.apache.org/caching.html)文档。

 

> **容器无关的Session集群**
>
> 如果您需要快速的创建与容器无关的session群集，EHCache也是一个不错的选择。您可以在EHCache中使用TerraCotta，并拥有一个与容器无关的集群会话缓存。再也不用使用Tomcat，JBoss，Jetty，WebSphere或WebLogic特定的session集群了！



为session开启EHCache非常简单。首先，确保您`shiro-ehcache-<version>.jar`在classpath路径中包含该文件（请参阅[下载](http://shiro.apache.org/download.html)页面或使用Maven或Ant + Ivy）。

进入类路径后，第一个`shiro.ini`示例将向您展示如何使用EHCache满足Shiro的所有缓存需求（而不仅仅是Session支持）：

**在shiro.ini中为Shiro的所有缓存需求配置EHCache**

```ini
[main]

sessionDAO = org.apache.shiro.session.mgt.eis.EnterpriseCacheSessionDAO
securityManager.sessionManager.sessionDAO = $sessionDAO

cacheManager = org.apache.shiro.cache.ehcache.EhCacheManager
securityManager.cacheManager = $cacheManager
```

最后一行，`securityManager.cacheManager = $cacheManager`配置了`CacheManager`。这个`CacheManager`实例将自动传播到`SessionDAO`（通过`EnterpriseCacheSessionDAO`实现[`CacheManagerAware`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/cache/CacheManagerAware.html)接口的性质）。

然后，当`SessionManager`要求`EnterpriseCacheSessionDAO`持久化`Session` 时，它将使用EHCache支持的[`Cache`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/cache/Cache.html)实现来存储session数据。

> **Web应用程序**
>
> 只有在使用shiro本地SessionManager时,  SessionDAO在生效。Web应用程序默认使用基于Servlet容器的SessionManager，它不支持`SessionDAO`。如果要在Web应用程序中使用基于Ehcache的会话存储，请按[上述说明](http://shiro.apache.org/session-management.html#SessionManagement-websessionmanagersessiondao)配置本机Web SessionManager 。

##### [EHCache会话缓存配置](http://shiro.apache.org/session-management.html#ehcache-session-cache-configuration)

默认情况下，`EhCacheManager`使用特定于Shiro的[`ehcache.xml`](https://github.com/apache/shiro/blob/master/support/ehcache/src/main/resources/org/apache/shiro/cache/ehcache/ehcache.xml)文件来设置会话缓存区域和必要的设置，以确保正确存储和检索会话。

但是，如果您希望更改缓存设置或配置自己的`ehcache.xml`或EHCache `net.sf.ehcache.CacheManager`实例，则需要配置缓存区域以确保正确处理Sessions。

如果查看默认[`ehcache.xml`](https://github.com/apache/shiro/blob/master/support/ehcache/src/main/resources/org/apache/shiro/cache/ehcache/ehcache.xml)文件，您将看到以下`shiro-activeSessionCache`缓存配置：

```
<cache name="shiro-activeSessionCache"
       maxElementsInMemory="10000"
       overflowToDisk="true"
       eternal="true"
       timeToLiveSeconds="0"
       timeToIdleSeconds="0"
       diskPersistent="true"
       diskExpiryThreadIntervalSeconds="600"/>
```

如果您希望使用自己的`ehcache.xml`文件，请确保为Shiro的需要定义了类似的缓存条目。您最有可能更改`maxElementsInMemory`属性值以满足您的需求。但是，在您自己的配置中至少存在以下两个属性（并且不会更改）非常重要：

- `overflowToDisk="true"` - 这确保了如果用完进程内存，会话将不会丢失并可以序列化到磁盘
- `eternal="true"` - 确保缓存中的缓存条目（会话实例）永远不会过期或自动清除。这是必要的，因为Shiro基于预定的流程进行自己的验证（请参阅下面的“会话验证和调度”）。如果我们关闭它，缓存可能会在没有Shiro知道的情况下驱逐Sessions，这可能会导致问题。

##### [EHCache会话缓存名称](http://shiro.apache.org/session-management.html#ehcache-session-cache-name)

默认情况下，`EnterpriseCacheSessionDAO`询问`CacheManager`了`Cache`名为“ `shiro-activeSessionCache`”。`ehcache.xml`如上所述，期望配置该缓存名称/区域。

如果要使用其他名称而不是此默认名称，可以在其上配置该名称`EnterpriseCacheSessionDAO`，例如：

**配置在shiro.ini四郎的活动会话高速缓存名** <

```
...
sessionDAO = org.apache.shiro.session.mgt.eis.EnterpriseCacheSessionDAO
sessionDAO.activeSessionsCacheName = myname
...
```

只要确保在相应的条目`ehcache.xml`匹配的名字和你配置`overflowToDisk="true"`和`eternal="true"`如上所述。

#### [自定义会话ID](http://shiro.apache.org/session-management.html#custom-session-ids)

Shiro的`SessionDAO`实现使用内部[`SessionIdGenerator`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/session/mgt/eis/SessionIdGenerator.html)组件在每次创建新会话时生成新的会话ID。生成ID，分配给新创建的`Session`实例，然后`Session`通过`SessionDAO`。

默认`SessionIdGenerator`值为a [`JavaUuidSessionIdGenerator`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/session/mgt/eis/JavaUuidSessionIdGenerator.html)，它`String`基于Java 生成ID [`UUIDs`](http://download.oracle.com/javase/6/docs/api/java/util/UUID.html)。此实现适用于所有生产环境。

如果这不符合您的需求，您可以`SessionIdGenerator`在Shiro的`SessionDAO`实例上实现接口并配置实现。例如，在`shiro.ini`：

**在shiro.ini中配置SessionIdGenerator**

```
[main]
...
sessionIdGenerator = com.my.session.SessionIdGenerator
securityManager.sessionManager.sessionDAO.sessionIdGenerator = $sessionIdGenerator
```

### 会话验证和调度

必须验证会话，以便可以从会话数据存储中删除任何无效（已过期或已停止）的会话。这可确保数据存储不会随着时间的推移而填满永远不会再次使用的会话。

出于性能原因，`Sessions`仅验证它们在访问时是否已停止或过期（即`subject.getSession()`）。这意味着如果没有额外的定期定期验证，`Session`孤儿就会开始填满会话数据存储。

说明孤儿的常见示例是Web浏览器场景：假设用户登录到Web应用程序并创建会话以保留数据（身份验证状态，购物车等）。如果用户在应用程序不知情的情况下没有注销并关闭浏览器，那么他们的会话基本上只是在会话数据存储中“孤立”（孤立）。在`SessionManager`没有检测到用户不再使用其浏览器的方式，会话决不会再次访问（它是孤立的）。

会话孤儿，如果没有定期清除，将填满会话数据存储（这将是不好的）。因此，为了防止孤儿堆积，这些`SessionManager`实现支持a的概念[`SessionValidationScheduler`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/session/mgt/SessionValidationScheduler.html)。A `SessionValidationScheduler`负责定期验证会话，以确保在必要时对其进行清理。

#### [默认SessionValidationScheduler](http://shiro.apache.org/session-management.html#default-sessionvalidationscheduler)

`SessionValidationScheduler`在所有环境中都可使用的默认值是[`ExecutorServiceSessionValidationScheduler`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/session/mgt/ExecutorServiceSessionValidationScheduler.html)使用JDK [`ScheduledExecutorService`](http://docs.oracle.com/javase/6/docs/api/java/util/concurrent/ScheduledExecutorService.html)来控制验证发生的频率。

默认情况下，此实现将每小时执行一次验证。您可以通过指定**新**实例`ExecutorServiceSessionValidationScheduler`并指定不同的时间间隔（以毫秒为单位）来更改验证发生的速率：

**shiro.ini中的ExecutorServiceSessionValidationScheduler间隔**

```
[main]
...
sessionValidationScheduler = org.apache.shiro.session.mgt.ExecutorServiceSessionValidationScheduler
# Default is 3,600,000 millis = 1 hour:
sessionValidationScheduler.interval = 3600000

securityManager.sessionManager.sessionValidationScheduler = $sessionValidationScheduler
```

#### [自定义SessionValidationScheduler](http://shiro.apache.org/session-management.html#custom-sessionvalidationscheduler)

如果要提供自定义`SessionValidationScheduler`实现，可以将其指定为默认`SessionManager`实例的属性。例如，在`shiro.ini`：

**在shiro.ini中配置自定义SessionValidationScheduler**

```
[main]
...
sessionValidationScheduler = com.foo.my.SessionValidationScheduler
securityManager.sessionManager.sessionValidationScheduler = $sessionValidationScheduler
```

#### [禁用会话验证](http://shiro.apache.org/session-management.html#disabling-session-validation)

在某些情况下，您可能希望完全禁用会话验证，因为您已在Shiro的控件之外设置了一个进程来为您执行验证。例如，您可能正在使用企业缓存并依赖缓存的生存时间设置来自动清除旧会话。或者，您可能已设置了一个cron作业来自动清除自定义数据存储。在这些情况下，您可以关闭会话验证计划：

**在shiro.ini中禁用会话验证调度**

```
[main]
...
securityManager.sessionManager.sessionValidationSchedulerEnabled = false
```

从会话数据存储中检索会话时仍会验证会话，但这将禁用Shiro的定期验证。

 

在*某处*启用会话验证

------

如果您关闭Shiro的会话验证调度程序，您*必须*通过其他一些机制（cron作业等）执行定期会话验证。这是保证Session孤儿不填满数据存储的唯一方法。

#### [会话删除无效](http://shiro.apache.org/session-management.html#invalid-session-deletion)

如上所述，定期会话验证的目的主要是删除任何无效（已过期或已停止）的会话，以确保它们不会填满会话数据存储。

默认情况下，每当Shiro检测到无效会话时，它都会尝试通过该`SessionDAO.delete(session)`方法将其从基础会话数据存储中删除。对于大多数应用程序来说，这是一种很好的做法，可确保会话数据存储空间不会耗尽。

但是，某些应用程序可能不希望Shiro自动删除会话。例如，如果应用程序提供了`SessionDAO`支持可查询数据存储的应用程序，则应用程序团队可能希望旧的或无效的会话在一段时间内可用。这将允许团队针对数据存储运行查询，以查看用户在上周创建的会话数，或用户会话的平均持续时间，或类似的报告类型查询。

在这些情况下，您可以完全关闭无效会话删除。例如，在`shiro.ini`：

**在shiro.ini中禁用无效会话删除**

```
[main]
...
securityManager.sessionManager.deleteInvalidSessions = false
```

不过要小心！如果关闭此功能，则您有责任确保会话数据存储不会耗尽其空间。您必须自己从数据存储中删除无效会话！

另请注意，即使您阻止Shiro删除无效会话，您仍应以某种方式启用会话验证 - 通过Shiro现有的验证机制或您自己提供的自定义机制（请参阅上面的“禁用会话验证”部分以获取更多信息）。验证机制将更新您的会话记录以反映无效状态（例如，当它被无效时，上次访问时等），即使您将在其他时间自己手动删除它们。

 

警告

------

如果您配置Shiro以便它不会删除无效会话，则您有责任确保会话数据存储不会耗尽其空间。您必须自己从数据存储中删除无效会话！

另外请注意，禁用会话缺失是**不**一样的禁用会话验证调度。您应该几乎总是使用会话验证调度机制 - 直接由Shiro或您自己支持的机制。

## [会话群集](http://shiro.apache.org/session-management.html#session-clustering)

Apache Shiro会话功能的一个非常令人兴奋的事情是，您可以本地群集主题会话，而不必再担心如何根据容器环境群集会话。也就是说，如果您使用Shiro的本地会话并配置会话群集，您可以在开发中部署到Jetty或Tomcat，在生产中或任何其他环境中部署到JBoss或Geronimo - 同时从不担心容器/环境特定群集设置或配置。在Shiro中配置一次会话群集，无论您的部署环境如何，它都可以正常运行。

那么它是怎样工作的？

由于Shiro基于POJO的N层架构，启用会话群集就像在会话持久性级别启用群集机制一样简单。也就是说，如果您配置了支持群集的功能[`SessionDAO`](http://shiro.apache.org/session-management.html#SessionManagement-sessionstorage)，DAO可以与群集机制进行交互，而Shiro `SessionManager`从不需要了解群集问题。

**分布式缓存**

分布式缓存（如[Ehcache + TerraCotta](http://www.ehcache.org/documentation/2.7/configuration/distributed-cache-configuration.html)，[GigaSpaces ](http://www.gigaspaces.com/)[Oracle Coherence](http://www.oracle.com/technetwork/middleware/coherence/overview/index.html)和[Memcached](http://memcached.org/)（以及许多其他））已经解决了分布式数据在持久性级别的问题。因此，在Shiro中启用会话群集就像配置Shiro以使用分布式缓存一样简单。

这使您可以灵活地选择适合*您*环境的精确聚类机制。

 

高速缓存存储器

------

请注意，当启用分布式/企业高速缓存作为会话群集数据存储时，必须满足以下两种情况之一：

- 分布式缓存具有足够的群集范围内存以保留_all_活动/当前会话
- 如果分布式缓存没有足够的群集范围内存来保留所有活动会话，则它必须支持磁盘溢出，因此会话不会丢失。

缓存无法支持这两种情况中的任何一种都会导致会话随机丢失，这可能会让最终用户感到沮丧。

### [EnterpriseCacheSessionDAO](http://shiro.apache.org/session-management.html#enterprisecachesessiondao)

正如您所料，Shiro已经提供了一种`SessionDAO`将数据保存到企业/分布式缓存的实现。所述[EnterpriseCacheSessionDAO](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/session/mgt/eis/EnterpriseCacheSessionDAO.html)需要一个四郎`Cache`或`CacheManager`要在其上配置为使得它可以利用缓存机制。

例如，在`shiro.ini`：

```
#This implementation would use your preferred distributed caching product's APIs:
activeSessionsCache = my.org.apache.shiro.cache.CacheImplementation

sessionDAO = org.apache.shiro.session.mgt.eis.EnterpriseCacheSessionDAO
sessionDAO.activeSessionsCache = $activeSessionsCache

securityManager.sessionManager.sessionDAO = $sessionDAO
```

虽然您可以`Cache`直接将实例注入到`SessionDAO`如上所示，但通常配置一个通用`CacheManager`以用于所有Shiro的缓存需求（会话以及身份验证和授权数据）更为常见。在这种情况下，`Cache`您可以告诉应该用于存储活动会话`EnterpriseCacheSessionDAO`的缓存名称，而不是直接配置实例`CacheManager`。

例如：

```
# This implementation would use your caching product's APIs:
cacheManager = my.org.apache.shiro.cache.CacheManagerImplementation

# Now configure the EnterpriseCacheSessionDAO and tell it what
# cache in the CacheManager should be used to store active sessions:
sessionDAO = org.apache.shiro.session.mgt.eis.EnterpriseCacheSessionDAO
# This is the default value.  Change it if your CacheManager configured a different name:
sessionDAO.activeSessionsCacheName = shiro-activeSessionsCache
# Now have the native SessionManager use that DAO:
securityManager.sessionManager.sessionDAO = $sessionDAO

# Configure the above CacheManager on Shiro's SecurityManager
# to use it for all of Shiro's caching needs:
securityManager.cacheManager = $cacheManager
```

但是上面的配置有些奇怪。你注意到了吗？

关于这个配置的有趣的事情是在配置中没有任何地方我们实际上告诉`sessionDAO`实例使用`Cache`或`CacheManager`！那么如何`sessionDAO`使用分布式缓存呢？

当Shiro初始化时`SecurityManager`，它将检查是否`SessionDAO`实现了[`CacheManagerAware`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/cache/CacheManagerAware.html)接口。如果是，它将自动提供任何可用的全局配置`CacheManager`。

因此，当Shiro评估该`securityManager.cacheManager = $cacheManager`行时，它将发现`EnterpriseCacheSessionDAO`实现`CacheManagerAware`接口并`setCacheManager`使用您配置`CacheManager`的方法参数调用该方法。

然后在运行时，当`EnterpriseCacheSessionDAO`需要`activeSessionsCache`它时，它会要求`CacheManager`实例返回它，使用`activeSessionsCacheName`查找键来获取`Cache`实例。该`Cache`实例（由您的分布式/企业缓存产品的API支持）将用于存储和检索所有`SessionDAO`CRUD操作的会话。

### [Ehcache + Terracotta](http://shiro.apache.org/session-management.html#ehcache-terracotta)

人们在使用Shiro时取得成功的一种分布式缓存解决方案是Ehcache + Terracotta配对。有关如何使用Ehcache启用分布式缓存的完整详细信息，请参阅Ehcache托管的[带有Terracotta的分布式缓存](http://www.ehcache.org/documentation/get-started/about-distributed-cache)文档。

一旦你将Terracotta集群与Ehcache配合使用，Shiro特有的部件就非常简单了。阅读并遵循[Ehcache SessionDAO](http://shiro.apache.org/session-management.html#SessionManagement-ehcachesessiondao)文档，但我们需要进行一些更改

[之前引用](http://shiro.apache.org/session-management.html#SessionManagement-ehcachesessioncacheconfiguration)的Ehcache会话缓存配置不起作用 - 需要特定于Terracotta的配置。以下是经过测试可正常工作的示例配置。将其内容保存在文件中并将其保存在`ehcache.xml`文件中：

**TerraCotta会话聚类**

```
<ehcache>
    <terracottaConfig url="localhost:9510"/>
    <diskStore path="java.io.tmpdir/shiro-ehcache"/>
    <defaultCache
            maxElementsInMemory="10000"
            eternal="false"
            timeToIdleSeconds="120"
            timeToLiveSeconds="120"
            overflowToDisk="false"
            diskPersistent="false"
            diskExpiryThreadIntervalSeconds="120">
        <terracotta/>
    </defaultCache>
    <cache name="shiro-activeSessionCache"
           maxElementsInMemory="10000"
           eternal="true"
           timeToLiveSeconds="0"
           timeToIdleSeconds="0"
           diskPersistent="false"
           overflowToDisk="false"
           diskExpiryThreadIntervalSeconds="600">
        <terracotta/>
    </cache>
    <!-- Add more cache entries as desired, for example,
         Realm authc/authz caching: -->
</ehcache>
```

当然，您需要更改`<terracottaConfig url="localhost:9510"/>`条目以引用Terracotta服务器阵列的相应主机/端口。另请注意，与[以前的](http://shiro.apache.org/session-management.html#SessionManagement-ehcachesessioncacheconfiguration)配置不同，`ehcache-activeSessionCache`元素**不**设置`diskPersistent`或`overflowToDisk`属性`true`。它们应该都是`false`在群集配置中不支持的真值。

保存此`ehcache.xml`文件后，我们需要在Shiro的配置中引用它。假设您已经`ehcache.xml`在类路径的根目录中访问了特定于兵马俑的文件，这里是最终的Shiro配置，它可以为所有Shiro的需求（包括Sessions）启用Terracotta + Ehcache群集：

**shiro.ini用于Ehcache和Terracotta的会话聚类**

```
sessionDAO = org.apache.shiro.session.mgt.eis.EnterpriseCacheSessionDAO
# This name matches a cache name in ehcache.xml:
sessionDAO.activeSessionsCacheName = shiro-activeSessionsCache
securityManager.sessionManager.sessionDAO = $sessionDAO

# Configure The EhCacheManager:
cacheManager = org.apache.shiro.cache.ehcache.EhCacheManager
cacheManager.cacheManagerConfigFile = classpath:ehcache.xml

# Configure the above CacheManager on Shiro's SecurityManager
# to use it for all of Shiro's caching needs:
securityManager.cacheManager = $cacheManager
```

请记住，**订购事项**。通过配置`cacheManager`上的`securityManager`最后，我们保证的CacheManager可以传播到所有先前配置的`CacheManagerAware`组件（如`EnterpriseCachingSessionDAO`）。

### [动物园管理员](http://shiro.apache.org/session-management.html#zookeeper)

用户已经报告使用[Apache Zookeeper](http://zookeeper.apache.org/)来管理/协调分布式会话。如果您有任何关于这将如何工作的文档/评论，请将它们发布到Shiro [邮件列表](http://shiro.apache.org/mailing-lists.html)

## [会话和主题国家](http://shiro.apache.org/session-management.html#sessions-and-subject-state)

### 有状态申请（允许会议）

默认情况下，Shiro的SecurityManager实现将使用Subject的Session作为策略来存储Subject的identity（`PrincipalCollection`）和身份验证状态（`subject.isAuthenticated()`）以供继续引用。这通常发生在主题登录或通过RememberMe服务发现主题的身份之后。

这种默认方法有一些好处：

- 服务请求，调用或消息的任何应用程序都可以将会话ID与请求/调用/消息有效负载相关联，这是Shiro将用户与入站请求相关联所需的全部内容。例如，如果使用`Subject.Builder`，则只需获取相关主题即可：

  ```java
  Serializable sessionId = //get from the inbound request or remote method invocation payload Subject requestSubject = new Subject.Builder().sessionId(sessionId).buildSubject();
  ```

  对于大多数Web应用程序以及编写远程处理或消息传递框架的任何人来说，这都非常方便。（事实上，Shiro的Web支持如何在其自己的框架代码中将Subjects与ServletRequests相关联）。

- 初始请求中找到的任何“RememberMe”身份可以在首次访问时保留到会话中。这确保了可以跨请求保存主题的记住身份，而无需在*每个*请求上反序列化和解密。例如，在Web应用程序中，如果会话中已知该标识，则无需在每个请求上读取加密的RememberMe cookie。这可以是一个很好的性能增强。

虽然上述默认策略对于大多数应用程序来说很好（并且通常是可取的），但是在尽可能尝试无状态的应用程序中这是不可取的。许多无状态架构要求请求之间不存在持久状态，在这种情况下不允许Sessions（Session本身就代表持久状态）。

但是这个要求是以便利成本出现的 - 在请求之间不能保留主题状态。这意味着具有此要求的应用程序必须确保可以针对*每个*请求以其他方式表示主题状态。

这几乎总是通过验证应用程序处理的每个请求/调用/消息来实现。例如，大多数无状态Web应用程序通常通过强制执行HTTP Basic身份验证来支持此操作，允许浏览器代表最终用户对每个请求进行身份验证。远程处理或消息传递框架必须确保将主体主体和凭证附加到每个调用或消息有效负载，通常由框架代码执行。

#### [禁用主题状态会话存储](http://shiro.apache.org/session-management.html#disabling-subject-state-session-storage)

从Shiro 1.2及更高版本开始，希望禁用Shiro的持久主题状态到会话的内部实现策略的应用程序可以通过执行以下操作在*所有*主题中完全禁用它：

在`shiro.ini`，配置以下属性`securityManager`：

**shiro.ini**

```ini
[main]
...
securityManager.subjectDAO.sessionStorageEvaluator.sessionStorageEnabled = false
...
```

这将防止从四郎使用对象的会话存储跨请求/调用/邮件是主题的国家*所有科目*。请确保您对每个请求进行身份验证，以便Shiro知道主题对于任何给定的请求/调用/消息是谁。

 

Shiro的需求与您的需求相比

------

这将禁止Shiro自己的实现使用Sessions作为存储策略。它**不会**完全禁用Sessions。如果您自己的代码显式调用`subject.getSession()`或仍然会创建会话`subject.getSession(true)`。

### [混合方法](http://shiro.apache.org/session-management.html#a-hybrid-approach)

上面的`shiro.ini`配置行（`securityManager.subjectDAO.sessionStorageEvaluator.sessionStorageEnabled = false`）将禁止Shiro使用Session作为*所有*Subject 的实现策略。

但是，如果你想要一种混合方法呢？如果某些主题应该有会话而其他人不应该怎么办？这种混合方法对许多应用都是有益的。例如：

- 也许人类主体（例如网络浏览器用户）应该能够使用Sessions获得上面提供的好处。
- 也许非人物主体（如API客户或第三方应用程序）应该*不*创建会话，因为他们与软件的交互可能是间歇性的和/或不稳定。
- 也许某个类型的所有主题或从某个位置访问系统的主体应该在会话中保持状态，但所有其他主管不应该。

如果您需要这种混合方法，您可以实现一个`SessionStorageEvaluator`。

#### [SessionStorageEvaluator](http://shiro.apache.org/session-management.html#sessionstorageevaluator)

如果您想要确切地控制哪些主题的状态在其会话中保持不变，您可以实现该`org.apache.shiro.mgt.SessionStorageEvaluator`接口并告诉Shiro确切哪些主题应该支持会话存储。

该接口有一个方法：

**SessionStorageEvaluator**

```
public interface SessionStorageEvaluator {

    public boolean isSessionStorageEnabled(Subject subject);

}
```

有关更详细的API说明，请参阅[SessionStorageEvaluator JavaDoc](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/mgt/SessionStorageEvaluator.html)。

您可以实现此接口并检查主题以获取您做出此决定可能需要的任何信息。

##### [主题检查](http://shiro.apache.org/session-management.html#subject-inspection)

在实现`isSessionStorageEnabled(subject)`界面方法时，您始终可以查看`Subject`并获取您做出决定所需的任何内容。当然，所有预期的Subject方法都可以使用（`getPrincipals()`等），但环境特定的`Subject`实例也很有价值。

例如，在Web应用程序中，如果必须根据当前数据做出决策，则`ServletRequest`可以获取请求或响应，因为运行时`Subject`实例实际上是一个[`WebSubject`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/web/subject/WebSubject.html)实例：

```
...
    public boolean isSessionStorageEnabled(Subject subject) {
        boolean enabled = false;
        if (WebUtils.isWeb(Subject)) {
            HttpServletRequest request = WebUtils.getHttpRequest(subject);
            //set 'enabled' based on the current request.
        } else {
            //not a web request - maybe a RMI or daemon invocation?
            //set 'enabled' another way...
        }

        return enabled;
    }
```

**NB** Framework开发人员应牢记这种类型的访问，并确保通过特定于环境的`Subject`实现提供任何请求/调用/消息上下文对象。如果您想要为您的框架/环境设置一些帮助，请联系Shiro用户邮件列表。

#### [组态](http://shiro.apache.org/session-management.html#configuration)

实现`SessionStorageEvaluator`界面后，可以在`shiro.ini`以下位置进行配置：

**shiro.ini SessionStorageEvaluator配置**

```
[main]
...
sessionStorageEvaluator = com.mycompany.shiro.subject.mgt.MySessionStorageEvaluator
securityManager.subjectDAO.sessionStorageEvaluator = $sessionStorageEvaluator

...
```

### [Web应用程序](http://shiro.apache.org/session-management.html#web-applications)

Web应用程序通常希望在每个请求的基础上简单地启用或禁用会话创建，而不管哪个Subject正在执行请求。这通常用于支持REST和Messaging / RMI体系结构。例如，可能允许正常的最终用户（使用浏览器的人）创建和使用会话，但远程API客户端使用REST或SOAP，并且根本不应该有会话（因为它们在每个请求上进行身份验证，这在REST / SOAP架构）。

为了支持这种混合/按请求功能，`noSessionCreation`Shiro为Web应用程序启用了默认过滤器的“池”中添加了一个过滤器。此过滤器将阻止在请求期间创建新会话以保证无状态体验。在`shiro.ini` `[urls]`部分中，您通常会在所有其他过滤器之前定义此过滤器，以确保永远不会使用会话。

例如：

**shiro.ini - 根据请求禁用会话创建**

```
[urls]
...
/rest/** = noSessionCreation, authcBasic, ...
```

此筛选器允许任何*现有*会话使用会话，但不允许在筛选的请求期间创建新会话。也就是说*，对于尚未具有现有会话*的请求或主题的以下四个方法调用中的任何*一个*将自动触发`DisabledSessionException`：

- `httpServletRequest.getSession()`
- `httpServletRequest.getSession(true)`
- `subject.getSession()`
- `subject.getSession(true)`

如果`Subject`在访问noSessionCreation-protected-URL之前已有会话，则上述4个调用仍将按预期工作。

最后，在所有情况下始终允许以下调用：

- `httpServletRequest.getSession(false)`
- `subject.getSession(false)`