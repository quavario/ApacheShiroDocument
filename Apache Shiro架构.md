#  [Apache Shiro架构](http://shiro.apache.org/architecture.html#apache-shiro-architecture)

Apache Shiro的设计目标是通过直观和易用来简化应用程序安全性。Shiro的核心设计模拟了大多数人对应用程序安全性的看法 - 在某人（或某事）与应用程序交互的环境中。

软件应用程序通常基于用户故事设计。也就是说，您通常会根据用户（或应该）与软件交互的方式设计用户界面或服务API。例如，您可能会说，“如果用户与我的应用程序交互，则会向他们显示一个按钮，他们可以单击该按钮查看其帐户信息。如果他们没有登录，我会显示一个注册按钮。“

此示例语句指示应用程序主要是为满足用户要求和需求而编写的。即使“用户”是另一个软件系统而不是人类，您仍然会编写代码以反映基于当前与您的软件交互的人（或什么）的行为。

Shiro在自己的设计中反映了这些概念。通过匹配软件开发人员已经很直观的内容，Apache Shiro在任何应用程序中都保持直观且易于使用。

## 详细概述

在顶层架构，Shiro的架构有三个主要概念：和`Subject`，`SecurityManager`和`Realms`。下图是这些组件如何交互的高级概述，我们将在下面介绍每个概念：

![img](http://shiro.apache.org/assets/images/ShiroBasicArchitecture.png)  

- **Subject**：正如我们在[教程中](http://shiro.apache.org/tutorial.html)提到的，`Subject`它本质上是指正在执行操作的当前用户。虽然“用户”这个词通常意味着一个人，但是`Subject`它可以是一个人，但它也可以代表其他程序，守护进程，定时任务或者任何其他可能于程序产生交互的对象。

  `Subject ` 实例需要并且必须只能绑定到一个`SecurityManager` 对象上。当需要于`Subject`进行交互时，这些交互会转换为特定于这个`Subject` 的交互`SecurityManager`。

- **SecurityManager**：它`SecurityManager`是Shiro架构的核心，充当一种“伞形”对象，协调其内部安全组件，共同形成一个对象图。但是，一旦为应用程序配置了SecurityManager及其内部对象图，通常将其设为单例的，而程序开发人员主要工作几乎都花在`Subject`API上。

  我们稍后将`SecurityManager`详细讨论，但重要的是，当与`Subject`交互时，所有的对`Subject`的操作实际上是`SecurityManager`操作完成的。我们可以看上面的基本流程图。

- **Realms**：Realms充当Shiro与应用程序安全数据之间的“桥梁”或“连接器”。当实际与安全相关的数据（如用户帐户）进行交互以执行身份验证（登录）和授权（访问控制）时，Shiro会从配置的一个或多个Realm中中查找需要的信息。

  从这个意义上讲，Realm本质上是一个用于安全性相关的[DAO](https://en.wikipedia.org/wiki/Data_access_object)：它封装了数据源的连接细节，并获取需要使安全数据并应用于Shiro(就是说从数据库中获取身份权限等信息并告诉Shiro)。配置Shiro时，必须至少指定一个Realm用于身份验证和/或授权。`SecurityManager`可与多个境界被配置，但至少有一个是必需的。

  Shiro提供了可以直接使用的Realm，可以连接到许多安全数据源，如LDAP，关系数据库（JDBC），文本配置源（如INI和属性文件等）。如果默认Realm不符合您的需要，您可以自定义Realm来表示数据源。

  与其他内部组件一样，Shiro的 `SecurityManager`管理如何使用Realms获取要表示为`Subject`实例的安全性和身份数据。

## [架构细节](http://shiro.apache.org/architecture.html#detailed-architecture)

下图显示了Shiro的核心架构概念，后面是每个概述的摘要：

![img](http://shiro.apache.org/assets/images/ShiroArchitecture.png)  

- **Subject**（[`org.apache.shiro.subject.Subject`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/subject/Subject.html)）
  当前与程序交互的实体（用户，第三方程序，定时任务等）的特定于安全性的“视图”。
- **SecurityManager**（[org.apache.shiro.mgt.SecurityManager](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/mgt/SecurityManager.html)）
  如上所述，`SecurityManager`是Shiro架构的核心。它主要是一个“伞形”对象，它协调其托管组件以确保它们一起稳定运行。它还管理Shiro对每个用户的视图，因此它知道如何对每个用户执行安全操作。
- **Authenticator**(认证)（[org.apache.shiro.authc.Authenticator](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authc/Authenticator.html)）
  `Authenticator`(认证器)负责执行和验证（注册）用户登陆请求的组件。当用户发送登录请求时，逻辑代码由`Authenticator`执行。而这个`Authenticator`知道如何协调一个或多个`Realms`中存储的用户信息。`Authenticator`会从这些`Realm` 中获取的用户信息，然后跟用户发送的登陆数据做对比,  来验证用户是否具有登陆权限. 
  - **Authentication Strategy**(认证策略)（[org.apache.shiro.authc.pam.AuthenticationStrategy](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authc/pam/AuthenticationStrategy.html)）
    如果配置了多个`Realm` (例如,支付宝可以通过手机号来登陆,也可以通过邮箱登陆,  这时就有了多个Realm)，`Authentication Strategy`将协调`Realm` 来确定登陆成功的条件（例如，如果一个`Realm`成功但其他`Realm`失败,那么能成功登陆吗？还是只有所有的Realm都成功登陆呢？又或者只有第一个登陆成功就可以了？）。
- **Atuthorizer**(授权)（[org.apache.shiro.authz.Authorizer](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authz/Authorizer.html)）
  `Authorizer`是负责对用户进行资源的访问控制的组件。这种机制用于确定用户可以进行哪些操作(例如,管理员可以进行增删改查操作,  而员工只能进行查询操作,  不能进行删除操作)。与此`Authenticator`类似，`Authorizer`也知道如何协调多个后端数据源以访问角色和权限信息。在`Authorizer`使用该信息来确定到底是否允许用户执行特定的操作。
- **SessionManager**（[org.apache.shiro.session.mgt.SessionManager](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/session/mgt/SessionManager.html)）
  将`SessionManager`知道如何创建和管理用户`Session`的生命周期。这是安全框架领域的一项独特功能 - 即使没有可用的Web / Servlet或EJB容器，Shiro也能够在任何环境中本地管理用户会话。默认情况下，Shiro将使用现有的会话机制（例如Servlet Container），但如果没有，例如在独立应用程序或非Web环境中，它将使用其内置的会话管理。`SessionDAO`可以使用所有的数据库来持久化session. 
  - **SessionDAO**（[org.apache.shiro.session.mgt.eis.SessionDAO](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/session/mgt/eis/SessionDAO.html)）
    `SessionDAO`可以对`Session` 进行持久性（CRUD）操作。这允许将任何数据存储插入会话管理基础结构。
- **CacheManager的**（[org.apache.shiro.cache.CacheManager](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/cache/CacheManager.html)）
  `CacheManager` 可以创建`Cache ` 并管理`Cache`生命周期 。由于Shiro可以访问许多后端数据库以进行身份验证，授权和会话管理，因此`Cache`一直是框架中的顶层架构功能，可在使用这些数据源时提高性能。任何现代开源和/或企业缓存产品都可以插入Shiro，以提供快速有效的用户体验。
- **Cryptography**（[org.apache.shiro.crypto.*](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/crypto/package-summary.html)）
  Cryptograph是企业安全框架的自然补充。Shiro的`crypto`软件包包含易于使用和理解的密码密码，hash（也叫做digests）和不同编解码器实现的表示。该软件包中的所有类都经过精心设计，易于使用且易于理解。使用过Java内置的加密机制的程序要都知道有多么难用。Shiro的加密API简化了复杂的Java机制，任何人都能使用。
- **Realms**（[org.apache.shiro.realm.Realm](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/realm/Realm.html)）
  如上所述，Realms充当Shiro和应用程序安全数据之间的“桥接”或“连接器”。当实际与安全相关的数据（如用户帐户）进行交互以执行身份验证（登录）和授权（访问控制）时，Shiro会从为应用程序配置的一个或多个Realm中查找许多这些内容。您可以根据`Realms`需要配置任意数量（通常每个数据源一个），Shiro将根据需要进行身份验证和授权协调。

##  `SecurityManager`

因为Shiro的API鼓励采用`Subject`中心编程方法，所以大多数应用程序开发人员很少（如果有的话）`SecurityManager`直接与之交互（框架开发人员有时可能会发现它很有用）。即便如此，了解`SecurityManager`功能仍然很重要，尤其是在为应用程序配置功能时。

## [设计](http://shiro.apache.org/architecture.html#design)

如前所述，`SecurityManager`执行安全操作并管理*所有*应用程序用户的状态。在Shiro的默认`SecurityManager`实现如下这下功能：

- 认证
- 授权
- 会话管理
- 缓存管理
- 协调Realm
- 事件传播
- “记住我”服务
- 创建主题
- 登出等等。

但是，尝试在单个组件中管理这是很多功能。并且，如果将所有内容集中到单个实现类中，那么使这些内容变得灵活且可自定义将非常困难。

为了简化配置并实现灵活的配置/可插拔性，Shiro的实现都是高度模块化的设计 - 实际上是模块化的，SecurityManager实现（及其类层次结构）根本不起作用。相反，这些`SecurityManager`实现主要充当轻量级“容器”组件，几乎将所有行为委托给嵌套/包装组件。这种“包装”设计反映在上面的详细架构图中。

当组件实际执行逻辑时，`SecurityManager`实现知道如何以及何时协调组件以获得正确的行为。

的`SecurityManager`实现方式和组件也JavaBeans的兼容，它允许（或配置机构）能够容易地通过标准的JavaBeans存取/ mutator方法定制可插组件（获取* /组*）。这意味着Shiro的架构模块化可以转化为非常容易的自定义行为配置。