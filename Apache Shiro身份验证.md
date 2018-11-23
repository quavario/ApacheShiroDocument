# [Apache Shiro身份验证](http://shiro.apache.org/authentication.html#apache-shiro-authentication)

- [ `Subjects认证步骤`](http://shiro.apache.org/authentication.html#Authentication-Authenticating%7B%7BSubjects%7D%7D)
  - [第1步：收集subject的身份和凭证](http://shiro.apache.org/authentication.html#Authentication-Step1%3ACollecttheSubject%27sprincipalsandcredentials)
  - [第2步：提交身份和凭证](http://shiro.apache.org/authentication.html#Authentication-Step2%3ASubmittheprincipalsandcredentials)
  - [第3步：处理结果](http://shiro.apache.org/authentication.html#Authentication-Step3%3AHandlingSuccessorFailure)
- [记住与认证](http://shiro.apache.org/authentication.html#Authentication-Rememberedvs.Authenticated)
  - [为什么区别？](http://shiro.apache.org/authentication.html#Authentication-Whythedistinction%3F)
  - [一个说明的例子](http://shiro.apache.org/authentication.html#Authentication-Anillustratingexample)
- [注销](http://shiro.apache.org/authentication.html#Authentication-LoggingOut)
- 验证序列
  - [`Authenticator`](http://shiro.apache.org/authentication.html#Authentication-%7B%7BAuthenticator%7D%7D)
  - [`AuthenticationStrategy`](http://shiro.apache.org/authentication.html#Authentication-%7B%7BAuthenticationStrategy%7D%7D)
  - Realm认证顺序
    - [隐式排序](http://shiro.apache.org/authentication.html#Authentication-ImplicitOrdering)
    - [明确的订购](http://shiro.apache.org/authentication.html#Authentication-ExplicitOrdering)
- [Realm认证](http://shiro.apache.org/authentication.html#Authentication-RealmAuthentication)

![img](http://shiro.apache.org/assets/images/ShiroFeatures_Authentication.png)  

认证是指验证身份的过程 - 比如说, 你说你要进入一个小区,  你说你是这个小区的业主,  门卫不会让你进入,  如果你出示了门禁卡,  门卫才会让你进去 。 这里验证你是不是这个小区的业主这个过程就是认证.  认证的结果可能是也能不是.  业主就是身份,  而门禁卡就是凭证. 

这是通过向Shiro 提交用户的*身份*和*凭据*来确定它们是否有登陆的权限。

- **Principals**(这里我们叫做身份)是Subject的“识别属性”。Principals可以是标识主体(这里我们管Subject叫做主题)的任何内容，例如名字，姓氏，用户名，身份证号码等。当然，姓名之类的不一定能准确的表示主体，比如, 你们班上有很多叫张伟的,  你喊一声张伟不知道你喊得是哪个张伟呢  因此用于身份验证的最佳`身份`通常是唯一的 - 比如电子邮件地址

> **主要身份**
>
> 虽然Shiro可以代表任意数量的主体，但Shiro希望应用程序只有一个“主要”主体 - 一个唯一标识应用程序中“主题”的值。这通常是大多数应用程序中的用户名，电子邮件地址或全局唯一用户ID。

- **凭证**通常是加密的，只有`Subject`被用作支持证据才能证明他们实际上“拥有”所声称的身份。证书的一些常见示例是密码，指纹和视网膜扫描等生物识别数据以及X.509证书。

委托人/凭证配对的最常见示例是用户名和密码。用户名是声明的身份，密码是与声明的身份相匹配的证明。如果提交的密码与应用程序的预期密码匹配，则应用程序可以在很大程度上假设用户确实是他们所说的人，因为没有其他人应该知道相同的密码。

## 认证`Subjects`

验证`Subject`的过程可以有效地分解为三个不同的步骤：

1. 收集主体提交的身份和凭据
2. 提交身份和凭据以进行身份验证。
3. 如果提交成功，则允许访问，否则重新进行身份验证或阻止访问。

以下代码演示了Shiro的API如何反映这些步骤：

### 第1步：收集主体的身份和凭据

```
//Example using most common scenario of username/password pair:
UsernamePasswordToken token = new UsernamePasswordToken(username, password);

//"Remember Me" built-in: 
token.setRememberMe(true);
```

在这种特殊情况下，我们使用[UsernamePasswordToken](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authc/UsernamePasswordToken.html)，支持最常见的用户名/密码身份验证方法。这是Shiro的[org.apache.shiro.authc.AuthenticationToken](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authc/AuthenticationToken.html)接口的实现，该接口是Shiro的身份验证系统用来表示提交的主体和凭据的基本接口。

这里需要注意的是，Shiro并不关心如何获取这些信息：数据可能是由提交HTML表单的用户获取的，也可能是从HTTP头中检索的，或者可能是从Swing或Flex中读取的GUI密码表单，或者可能通过命令行参数。从应用程序最终用户收集信息的过程与Shiro的`AuthenticationToken`概念完全分离。

您可以根据`AuthenticationToken`自己的喜好构建和表示实例 - 它与协议无关。

这个例子还表明我们已经表示希望Shiro为身份验证尝试执行“记住我”服务。这可确保Shiro在以后返回应用程序时记住用户身份。我们将在后面的章节中介绍Remember Me服务。

### 第2步：提交身份和凭据

在收集了身份和凭据并为创建`AuthenticationToken`实例后，我们需要将令牌提交给Shiro以执行实际的身份验证：

```
Subject currentUser = SecurityUtils.getSubject();

currentUser.login(token);
```

获取当前执行的`Subject`后，我们进行一次[`login`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/subject/Subject.html#login-org.apache.shiro.authc.AuthenticationToken-)调用，传入我们之前创建的`AuthenticationToken`实例。

### 第3步：处理成功或失败

如果`login`方法执行成功,并别没有报错就说明认证通过了！在`Subject` 通过认证后, 在调用`getSubject()`方法,  返回的subject都是认证成功的主体，并且`subject. isAuthenticated()会 `调用 `true`。

但是如果登录尝试失败会发生什么？例如，如果用户提供了错误的密码，或者访问了系统太多次并且他们的帐户被锁定了怎么办？

Shiro具有丰富的运行时[`AuthenticationException`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authc/AuthenticationException.html)层次结构，可以准确指示尝试失败的原因。您可以`login`在一个`try/catch`块中包装并捕获您希望的任何异常并相应地对它们做出反应。例如：

```
try {
    currentUser.login(token);
} catch ( UnknownAccountException uae ) { ...
} catch ( IncorrectCredentialsException ice ) { ...
} catch ( LockedAccountException lae ) { ...
} catch ( ExcessiveAttemptsException eae ) { ...
} ... catch your own ...
} catch ( AuthenticationException ae ) {
    //unexpected error?
}

//No problems, continue on as expected...
```

如果现有异常类不能满足您的需求，则可以创建自定义`AuthenticationExceptions`以表示特定的错误。

 

> **登录失败时推荐的做法**
>
> 虽然您的代码可以根据需要对特定异常做出反应并执行逻辑，但最安全的做法是仅在发生故障时向最终用户显示通用失败消息，例如“用户名或密码不正确”。这可确保黑客无法获得可能正在尝试攻击媒介的特定信息。

## Remembered Me与认证

如上例所示，除了正常的登录过程外，Shiro还支持“记住我”的概念。值得指出的是，Shiro在*记住的*主题和实际*认证的*主题之间做了非常精确的区分：

- **记住**：记住`Subject`的不是匿名的，并且具有已知的身份（即`subject.getPrincipals()`非空）。但是，在**先前的**会话期间，先前的身份验证会记住此身份。如果`subject.`[`isRemembered()`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/subject/Subject.html#isRemembered--)返回，则认为主题被记住`true`。
- **经过身份验证**：经过身份验证的`Subject`是*在Subject的当前会话期间*已成功通过身份验证的`login`方法（即，在不抛出异常的情况下调用该方法）。如果返回，则认为主题被认证。`subject.`[`isAuthenticated()`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/subject/Subject.html#isAuthenticated--)`true`

 

> **互斥**
>
> 记住和认证的状态是互斥的 - 一个`true`值表示另一个`false`值，反之亦然。
>
> > 备注:  官方说的,  这里不太明白,  难道一个已经通过认证的用户,   不能是记住我状态吗,  就是isAuthenticated为true, isRemembered不能为true吗

### [区别？](http://shiro.apache.org/authentication.html#why-the-distinction-)

“身份认证”对认证的意义非凡。也就是说,  需要有个验证器来验证主体.

当用户仅在上一次的操作时， 勾选了记住我,  过了一段时间 ,  已认证状态不再存在：记住的身份使系统知道该用户可能是谁，但实际上，无法绝对*保证*记住的主体代表上一次的用户。一旦主体经过身份验证，就不再仅仅考虑它们，因为它们的身份将在当前会话期间得到验证。

因此，尽管应用程序的许多部分仍然可以基于记住的主体（例如自定义视图）执行特定于用户的逻辑，但是在用户通过执行成功的身份验证尝试合法地验证其身份之前，它通常不会执行高度敏感的操作。

例如，检查是否`Subject`可以访问财务信息应该几乎总是依赖于`isAuthenticated()`，而不是`isRemembered()`保证预期和验证的身份。

### [一个说明的例子](http://shiro.apache.org/authentication.html#an-illustrating-example)

以下是一个相当常见的场景，有助于说明为什么记住和验证之间的区别很重要。

假设您正在使用[Amazon.com](https://www.amazon.com/)。您已成功登录并已将几本书添加到购物车中。但你必须去参加一个会议，但忘记退出。会议结束时，是时候回家了，你离开了办公室。

第二天当你进来工作时，你意识到你没有完成购买，所以你回到amazon.com。这一次，亚马逊“记得”你是谁，通过名字问候你，并且仍然给你一些个性化的书籍推荐。对亚马逊来说，`subject.isRemembered()`会回来`true`。

但是，如果您尝试访问自己的帐户以更新信用卡信息以购买图书，会发生什么？虽然亚马逊记得'你（`isRemembered()`== `true`），但它不能保证你实际上是你（例如，可能是同事正在使用你的电脑）。

因此，在您执行更新信用卡信息等敏感操作之前，亚马逊会强制您登录，以便他们保证您的身份。登录后，您的身份已经过验证，`isAuthenticated()`现在已经验证了`true`。

对于许多类型的应用程序，这种情况经常发生，因此Shiro内置了该功能，因此您可以将其用于您自己的应用程序。现在，无论您使用`isRemembered()`或`isAuthenticated()`自定义您的视图和工作流程都取决于您，但Shiro将保持这种基本状态，以备您需要时使用。

## [注销](http://shiro.apache.org/authentication.html#logging-out)

与认证相反的是释放所有已知的登陆状态。当`Subject`完成与应用程序交互，你可以叫`subject.`[`logout()`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/subject/Subject.html#logout--)放弃所有的身份信息：

```
currentUser.logout(); //removes all identifying information and invalidates their session too.
```

当您嗲用时`logout`方法，任何现有的`Session`将被取消，并且任何身份将被取消关联（例如，在Web应用程序中，RememberMe cookie也将被删除）。

在`Subject`注销之后，该`Subject`实例再次被认为是匿名的，并且除了Web应用程序之外，`login`如果需要，可以再次重新使用该实例。

 

>  **Web Application提示**
>
> 由于Web应用程序中记住的身份通常与cookie保持一致，并且只能在提交响应主体之前删除cookie，因此强烈建议在调用后立即将用户重定向到新的视图或页面`subject.logout()`。这可以保证按预期删除任何与安全相关的cookie。这是HTTP cookie功能的限制，而不是Shiro的限制。





## [认证步骤](http://shiro.apache.org/authentication.html#authentication-sequence)

到目前为止，我们只研究了如何在应用程序代码中对`Subject`进行身份验证。现在我们将介绍在发生身份验证时Shiro内部发生的事情。

我们从[架构](http://shiro.apache.org/architecture.html)章节中获取了我们之前的架构图，并且只保留了与身份验证相关的组件。每个数字代表身份验证尝试期间的一个步骤：

![img](http://shiro.apache.org/assets/images/ShiroAuthenticationSequence.png)  

**步骤1**：执行`Subject.login`方法，传入`AuthenticationToken`。

**步骤2**：`Subject`实例(通常是[`DelegatingSubject`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/subject/support/DelegatingSubject.html)或子类）通过委托`SecurityManager`调用他的`securityManager.login(token)`，这是真正身份验证工作开始。

> 还记得我们之前说过吗,  我们一般通过subject的api进行认证和授权的操作,但实际上是securityManager进行的操作,  这里就有了体现,  官方文档说的还是很清楚的.  

**步骤3**：`SecurityManager`有个`Authenticator`对象,  接受传过来token后,  会把token传给`Authenticator` 然后调用他的`authenticate(token)`方法。这个`Authenticator` 对象是 直接new出来的`ModularRealmAuthenticator`  对象。`ModularRealmAuthenticator`  在认证期间可以支持一个或多个`Realm`调度。`ModularRealmAuthenticator`  其实是一个[可插拔认证模块](https://en.wikipedia.org/wiki/Pluggable_authentication_module)(PAM)。

> **可插拔认证模块**(PAM)
>
> PAM（Pluggable Authentication Modules ）是由Sun提出的一种认证机制。它通过提供一些[动态链接库](https://baike.baidu.com/item/%E5%8A%A8%E6%80%81%E9%93%BE%E6%8E%A5%E5%BA%93)和一套统一的API，将系统提供的服务 和该服务的认证方式分开，使得系统管理员可以灵活地根据需要给不同的服务配置不同的认证方式而无需更改服务程序，同时也便于向系 统中添加新的认证手段。



**步骤4**：如果配置了多个`Realm`，`ModularRealmAuthenticator`对象通过配置的[`AuthenticationStrategy`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authc/pam/AuthenticationStrategy.html) 开始多 `Realm`认证请求。在`Realms` 进行身份验证阶段，`AuthenticationStrategy` 可以获得每个Realm的认证结果。稍后会介绍到。

> **单Realm程序**
>
> 如果只配置了一个Realm会被直接调用, 而不会调用 `AuthenticationStrategy`



**步骤5**：然后会遍历出每个Realm,  通过调用`realm.supports(token)` 方法判断时候支持用户提交的token。如果是，[`getAuthenticationInfo(token)`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/realm/Realm.html#getAuthenticationInfo-org.apache.shiro.authc.AuthenticationToken-) 方法会执行。`getAuthenticationInfo`方法对根据这个Realm做认证。

### [Authenticator(认证器)](http://shiro.apache.org/authentication.html#authenticator)

如前所述，Shiro `SecurityManager`默认使用[`ModularRealmAuthenticator`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authc/pam/ModularRealmAuthenticator.html)。在`ModularRealmAuthenticator` 同时支持单Realm和多个Realm的应用。

在单Realm程序中，`ModularRealmAuthenticator`将直接调用单个`Realm`。如果配置了多个Realm，它将使用`AuthenticationStrategy`实例来协调请求的发送。我们将在下面介绍AuthenticationStrategies。

如果您希望`SecurityManager`使用自定义`Authenticator`实现进行配置，则可以执行`shiro.ini`以下操作：

```
[main]
...
authenticator = com.foo.bar.CustomAuthenticator

securityManager.authenticator = $authenticator
```

在实践中，`ModularRealmAuthenticator`基本可以满足大多数场合。

### [AuthenticationStrategy(认证策略)](http://shiro.apache.org/authentication.html#authenticationstrategy)

当配置了多个Realm时，`ModularRealmAuthenticator`依赖内部[`AuthenticationStrategy`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authc/pam/AuthenticationStrategy.html) 对象来确定登陆成功的条件。

例如，如果只有一个Realm `AuthenticationToken`成功验证通过，但所有其他验证失败，那么登录请求是否成功？或者所有Realm是否必须都成功通过验证才能被认为是登录成功？或者，如果一个Realm成功验证，是否有必要验证其他Realm？`AuthenticationStrategy` 就是为了解决这些问题的。

AuthenticationStrategy是一个无状态组件，在身份验证期间会被调用4次（这4次交互所需的任何必要状态将作为方法参数提供）：

1. 没有调用Realm前
2. 一个Realm的`getAuthenticationInfo` 方法调用前
3. 一个Realm的`getAuthenticationInfo` 方法调用后
4. 在所有Realm被调用之后

此外，`AuthenticationStrategy`还负责聚合每个成功Realm的结果并将它们“捆绑”为单个[`AuthenticationInfo`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authc/AuthenticationInfo.html)表示。`Authenticator`返回的`AuthenticationInfo`，也是Shiro用来表示`Subject`身份（即Principals）的实例。

 

> 主体身份'视图'

------

如果您在应用程序中使用多个Realm从多个数据源获取帐户数据，`AuthenticationStrategy`则最终负责应用程序看到的Subject的标识的最终“合并”视图。

Shiro有3个`AuthenticationStrategy`具体实现：

| `AuthenticationStrategy` 类                                  | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [`AtLeastOneSuccessfulStrategy`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authc/pam/AtLeastOneSuccessfulStrategy.html) | 如果一个（或多个）Realm通过认证，则认为整体认证成功。如果一个都没有通过认证，则失败。 |
| [`FirstSuccessfulStrategy`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authc/pam/FirstSuccessfulStrategy.html) | 只第一个通过认证的信息会被Realms使用。其他Realm会被忽略。如果一个都没有通过认证，则失败。 |
| [`AllSuccessfulStrategy`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authc/pam/AllSuccessfulStrategy.html) | 所有已配置的Realm必须成功进行身份认证才能视为登陆成功。如果任何一个未通过认证，则失败。 |

实现的`ModularRealmAuthenticator`默认值**AtLeastOneSuccessfulStrategy**，因为这是最常用的策略。但是，如果您需要，可以配置不同的策略：

```
[main]
...
authcStrategy = org.apache.shiro.authc.pam.FirstSuccessfulStrategy

securityManager.authenticator.authenticationStrategy = $authcStrategy

...
```

 

> 自定义认证策略

------

如果您想自己创建自己的`AuthenticationStrategy`实现，可以使用它[`org.apache.shiro.authc.pam.AbstractAuthenticationStrategy`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authc/pam/AbstractAuthenticationStrategy.html)作为起点。`AbstractAuthenticationStrategy`类自动实现合并来自各域的结果到一个单一的“捆绑” /聚集行为`AuthenticationInfo`实例。

### [Realm认证顺序](http://shiro.apache.org/authentication.html#realm-authentication-order)

`ModularRealmAuthenticator`会按顺序与每个Realm。

在`ModularRealmAuthenticator`先后获得了`Realm`所配置的实例`SecurityManager`。执行身份验证尝试时，它将遍历该集合，并且对于`Realm`支持提交的每个集合`AuthenticationToken`，调用Realm的`getAuthenticationInfo`方法。

#### [隐式排序](http://shiro.apache.org/authentication.html#implicit-ordering)

使用Shiro的INI配置格式时，应*按照希望它们处理的顺序AuthenticationToken*配置Realms 。例如，在`shiro.ini`Real中将按照它们在INI文件中定义的顺序进行查阅。也就是说，对于以下`shiro.ini`示例：

```
blahRealm = com.company.blah.Realm
...
fooRealm = com.company.foo.Realm
...
barRealm = com.company.another.Realm
```

所述`SecurityManager`将与这些三个境界被配置，以及认证尝试期间，`blahRealm`，`fooRealm`，和`barRealm`将被调用*的顺序*。

这与定义以下行的效果基本相同：

```
securityManager.realms = $blahRealm, $fooRealm, $barRealm
```

使用此方法，您无需设置`securityManager's` `realms`属性 - 每个定义的Realm将自动添加到`realms`属性中。

#### [明确的订购](http://shiro.apache.org/authentication.html#explicit-ordering)

如果要显式定义域与其交互的顺序，无论它们如何定义，都可以将securityManager的`realms`属性设置为显式集合属性。例如，如果使用上面的定义，但您希望`blahRealm`最后查询而不是第一个：

```
blahRealm = com.company.blah.Realm
...
fooRealm = com.company.foo.Realm
...
barRealm = com.company.another.Realm

securityManager.realms = $fooRealm, $barRealm, $blahRealm
...
```

 

明确的境界包含

------

显式配置`securityManager.realms`属性时，*只会*在上面配置引用的域`SecurityManager`。这意味着你可以在INI中定义5个Realm，但实际上只有3个`realms`属性引用3 。这与隐式域排序不同，其中将使用所有可用域。

## [Realm认证](http://shiro.apache.org/authentication.html#realm-authentication)

本章介绍了Shiro的主要工作流程，说明了如何进行身份验证。在身份认证期间（例如上面的“步骤5”），在单个Realm中发生的内部工作流程将在[Realm](http://shiro.apache.org/realm.html)章节的[Realm认证](http://shiro.apache.org/realm.html#Realm-authentication)部分中介绍。