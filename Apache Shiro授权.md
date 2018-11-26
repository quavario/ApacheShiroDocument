# [Apache Shiro授权](http://shiro.apache.org/authorization.html#apache-shiro-authorization)

- [授权要素](http://shiro.apache.org/authorization.html#Authorization-ElementsofAuthorization)
  - [权限](http://shiro.apache.org/authorization.html#Authorization-Permissions)
    - [权限粒度](http://shiro.apache.org/authorization.html#Authorization-PermissionGranularity)
  - [角色](http://shiro.apache.org/authorization.html#Authorization-Roles)
  - [用户](http://shiro.apache.org/authorization.html#Authorization-Users)
- [授权主题](http://shiro.apache.org/authorization.html#Authorization-AuthorizingSubjects)
  - [程序化授权](http://shiro.apache.org/authorization.html#Authorization-ProgrammaticAuthorization)
    - [基于角色的授权](http://shiro.apache.org/authorization.html#Authorization-RoleBasedAuthorization)
      - [角色检查](http://shiro.apache.org/authorization.html#Authorization-RoleChecks)
      - [角色断言](http://shiro.apache.org/authorization.html#Authorization-RoleAssertions)
    - [基于权限的授权](http://shiro.apache.org/authorization.html#Authorization-PermissionBasedAuthorization)
      - [许可检查](http://shiro.apache.org/authorization.html#Authorization-PermissionChecks)
        - [基于对象的权限检查](http://shiro.apache.org/authorization.html#Authorization-ObjectbasedPermissionChecks)
        - [基于字符串的权限检查](http://shiro.apache.org/authorization.html#Authorization-Stringbasedpermissionchecks)
      - [权限断言](http://shiro.apache.org/authorization.html#Authorization-PermissionAssertions)
  - [基于注释的授权](http://shiro.apache.org/authorization.html#Authorization-AnnotationbasedAuthorization)
    - [组态](http://shiro.apache.org/authorization.html#Authorization-Configuration)
    - [该`RequiresAuthentication`注解](http://shiro.apache.org/authorization.html#Authorization-The%7B%7BRequiresAuthentication%7D%7Dannotation)
    - [该`RequiresGuest`注解](http://shiro.apache.org/authorization.html#Authorization-The%7B%7BRequiresGuest%7D%7Dannotation)
    - [该`RequiresPermissions`注解](http://shiro.apache.org/authorization.html#Authorization-The%7B%7BRequiresPermissions%7D%7Dannotation)
    - [该`RequiresRoles`许可](http://shiro.apache.org/authorization.html#Authorization-The%7B%7BRequiresRoles%7D%7Dpermission)
    - [该`RequiresUser`注解](http://shiro.apache.org/authorization.html#Authorization-The%7B%7BRequiresUser%7D%7Dannotation)
  - [JSP TagLib授权](http://shiro.apache.org/authorization.html#Authorization-JSPTagLibAuthorization)
- [授权序列](http://shiro.apache.org/authorization.html#Authorization-AuthorizationSequence)
  - [`ModularRealmAuthorizer`](http://shiro.apache.org/authorization.html#Authorization-%7B%7BModularRealmAuthorizer%7D%7D)
    - [领域授权命令](http://shiro.apache.org/authorization.html#Authorization-RealmAuthorizationOrder)
    - [配置全局 `PermissionResolver`](http://shiro.apache.org/authorization.html#Authorization-Configuringaglobal%7B%7BPermissionResolver%7D%7D)
    - [配置全局 `RolePermissionResolver`](http://shiro.apache.org/authorization.html#Authorization-Configuringaglobal%7B%7BRolePermissionResolver%7D%7D)
  - [自定义授权者](http://shiro.apache.org/authorization.html#Authorization-CustomAuthorizer)

授权（也称为*访问控制*）是管理资源访问的过程。换句话说，控制*谁*可以访问*什么*。

授权的示例有很多：是否允许用户查看此网页，编辑数据，查看此按钮或打印到此打印机？这些都决定了用户可以访问的内容。

## [授权核心要素](http://shiro.apache.org/authorization.html#elements-of-authorization)

授权有三个核心要素：权限，角色和用户。

### [权限](http://shiro.apache.org/authorization.html#permissions)

Apache Shiro中的权限代表安全策略中最原子的元素。它们基本上是关于行为的陈述，并明确表示可以在应用程序中完成的任务。格式良好的权限声明实质上描述了资源以及`Subject`与这些资源交互时可能采取的操作。

许可声明的一些示例：

- 打开一个文件
- 查看“/ user / list”网页
- 打印文件
- 删除'jsmith'用户

大多数资源都支持典型的CRUD（创建，读取，更新，删除）操作，但任何对特定资源类型有意义的操作都可以。基本思想是权限声明至少基于*资源*和*操作*。

在查看权限时，最重要的事情可能是权限语句没有表示*谁*可以执行所表示的行为。他们是唯一的语句*什么*可以在应用程序来完成。

 

>  **权限仅表示行为**
>
> 权限陈述反映的行为（具有资源类型相关联的动作）*只*。它们并不反映*谁*能够执行此类行为。

定义*谁*（用户）被允许做*什么*（权限）以某种方式分配权限给用户的锻炼。这始终由应用程序的数据模型完成，并且可以在不同应用程序之间有很大差异。

例如，权限可以在角色中分组，该角色可以与一个或多个用户对象相关联。或者某些应用程序可以拥有一组用户，并且可以为一个组分配一个角色，通过传递关联将意味着该组中的所有用户都被隐式授予角色中的权限。

如何为用户授予权限有许多变体 - 应用程序根据应用程序要求确定如何对其进行建模。

我们将介绍Shiro如何确定是否`Subject`允许以后做某事。

#### [权限粒度](http://shiro.apache.org/authorization.html#permission-granularity)

上面的权限示例都指定了对资源类型（门，文件，客户等）的操作（打开，读取，删除等）。在某些情况下，它们甚至可以指定非常精细的*实例级*行为 - 例如，使用用户名“jsmith”（实例标识符）'删除'（操作）'user'（资源类型）。在Shiro中，您可以准确定义这些语句的精确程度。

我们在Shiro的[权限文档中](http://shiro.apache.org/permissions.html)更详细地介绍了权限声明的权限粒度和“级别” 。

### [角色](http://shiro.apache.org/authorization.html#roles)

角色是一个命名实体，通常代表一组行为或责任。这些行为转化为您可以或不可以使用软件应用程序执行的操作。角色通常分配给用户帐户，因此通过关联，用户可以“执行”归因于各种角色的事物。

实际上有两种类型的角色，Shiro支持这两种概念：

- **隐式角色**：大多数人使用角色作为*隐式*构造：您的应用程序仅基于角色名称*隐含*一组行为（即权限）。对于隐式角色，软件级别没有任何内容表示“允许角色X执行行为A，B和C”。行为仅由名称暗示。

 

潜在的脆弱安全

------

虽然更简单和最常见的方法，隐式角色可能会带来许多软件维护和管理问题。

例如，如果您只想添加或删除角色，或稍后重新定义角色的行为，该怎么办？每次需要进行此类更改时，您都必须返回源代码并更改所有角色检查以反映安全模型的变化！更不用说这会产生的运营成本（重新测试，通过QA，关闭应用程序，使用新的角色检查升级软件，重新启动应用程序等）。

这对于非常简单的应用程序来说可能是好的（例如，可能存在'管理员'角色和'其他人'）。但对于更复杂或可配置的应用程序，这可能是应用程序整个生命周期中的主要问题，并为您的软件带来巨大的维护成本。

- **显式角色**：然而，显式角色本质上是实际权限语句的命名集合。在这种形式下，应用程序（和Shiro）*确切*知道具有特定角色的含义。因为已知可以执行与否的*确切*行为，所以没有猜测或暗示特定角色可以做什么或不可以做什么。

Shiro团队主张使用权限和显式角色而不是旧的隐式方法。您将更好地控制应用程序的安全体验。

 

基于资源的访问控制

------

请务必阅读Les Hazlewood的文章[“新RBAC：基于资源的访问控制”](https://stormpath.com/blog/new-rbac-resource-based-access-control)，该文章深入介绍了使用权限和显式角色（以及它们对源代码的积极影响）的好处，而不是旧的隐式角色方法。

### [用户](http://shiro.apache.org/authorization.html#users)

用户本质上是应用程序的“谁”。正如我们之前所述，这`Subject`确实是Shiro的“用户”概念。

允许用户（主题）通过与角色或直接权限的关联在您的应用程序中执行某些操作。您的应用程序的数据模型确切地定义了`Subject`允许执行某些操作的方式。

例如，在您的数据模型中，您可能拥有一个实际的`User`类，并且您可以直接为`User`实例分配权限。或者，您可能只是`Roles`直接分配权限，然后将角色分配给`Users`，因此通过关联，可以`Users`传递“拥有”分配给其角色的权限。或者你可以用“群体”概念来表达这些东西。这取决于您 - 使用对您的应用程序有意义的东西。

您的数据模型确切地定义了授权的功能。Shiro依靠[Realm](http://shiro.apache.org/realm.html)实现将您的数据模型关联细节转换为Shiro理解的格式。我们将介绍Realms如何做到这一点。

 

注意

------

最终，您的[Realm](http://shiro.apache.org/realm.html)实现与您的数据源（RDBMS，LDAP等）进行通信。因此，您的领域将告诉Shiro是否存在角色或权限。您可以完全控制授权模型的结构和定义方式。

## [授权主题](http://shiro.apache.org/authorization.html#authorizing-subjects)

在Shiro执行授权可以通过3种方式完成：

- 以编程方式 - 您可以使用结构`if`和`else`块来执行Java代码中的授权检查。
- JDK注释 - 您可以将授权注释附加到Java方法
- JSP / GSP TagLibs - 您可以根据角色和权限控制JSP或GSP页面输出

### [程序化授权](http://shiro.apache.org/authorization.html#programmatic-authorization)

执行授权的最简单和最常用的方法可能是以编程方式`Subject`直接与当前实例进行交互。

#### 基于角色的授权

如果要基于更简单/传统的隐式角色名称来控制访问，则可以执行角色检查：

##### [角色检查](http://shiro.apache.org/authorization.html#role-checks)

如果只想检查当前`Subject`是否有角色，可以`hasRole*`在`Subject`实例上调用variant 方法。

例如，要查看a `Subject`是否具有特定（单个）角色，您可以调用该`subject.` [`hasRole(roleName)`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/subject/Subject.html#hasRole-java.lang.String-)方法，并做出相应的反应：

```java
Subject currentUser = SecurityUtils.getSubject();

if (currentUser.hasRole("administrator")) {
    //show the admin button 
} else {
    //don't show the button?  Grey it out? 
}
```

`Subject`根据您的需要，您可以调用几种面向角色的方法：

| 主题方法                                                     | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [`hasRole(String roleName)`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/subject/Subject.html#hasRolejava.lang.String-) | 返回`true`如果`Subject`被分配指定的角色，`false`否则。       |
| [`hasRoles(List roleNames)`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/subject/Subject.html#hasRoles-java.util.List-) | 返回`hasRole`与方法参数中的索引对应的结果数组。如果需要执行许多角色检查（例如，在自定义复杂视图时），则可用作性能增强 |
| [`hasAllRoles(Collection roleNames)`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/subject/Subject.html#hasAllRoles-java.util.Collection-) | 返回`true`如果`Subject`被分配*所有*指定的角色，`false`否则。 |

##### [角色断言](http://shiro.apache.org/authorization.html#role-assertions)

作为检查a `boolean`是否`Subject`具有角色的替代方法，您可以在执行逻辑之前断言它们具有预期的角色。如果`Subject`没有预期的角色，[`AuthorizationException`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authz/AuthorizationException.html)则抛出一个。如果它们确实具有预期的角色，则断言将静默执行，逻辑将按预期继续。

例如：

```
Subject currentUser = SecurityUtils.getSubject();

//guarantee that the current user is a bank teller and 
//therefore allowed to open the account: 
currentUser.checkRole("bankTeller");
openBankAccount();
```

这种方法相对于这些`hasRole*`方法的一个好处是，代码可以更清晰，因为`AuthorizationExceptions`如果当前`Subject`不满足预期条件（如果您不想这样做），则不必构建自己的代码。

`Subject`根据您的需要，您可以调用几种面向角色的断言方法：

| 主题方法                                                     | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [`checkRole(String roleName)`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/subject/Subject.html#checkRole-java.lang.String-) | 如果`Subject`为指定的角色分配，则静默返回，否则抛出`AuthorizationException`。 |
| [`checkRoles(Collection roleNames)`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/subject/Subject.html#checkRoles-java.util.Collection-) | 如果`Subject`为*所有*指定的角色分配，则静默返回，否则抛出`AuthorizationException`。 |
| [`checkRoles(String... roleNames)`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/subject/Subject.html#checkRoles-java.lang.String...-) | 与上述`checkRoles`方法相同，但允许使用Java 5 var-args样式参数。 |

#### 基于权限的授权

如上面我们对角色的概述所述，通常更好的方法是执行访问控制是通过基于权限的授权。基于权限的授权，因为它与应用程序的原始功能（以及应用程序的核心资源上的行为）密切相关，基于权限的授权源代码会在您的功能更改时更改，而不是在安全策略更改时更改。这意味着代码比类似的基于角色的授权代码受到的影响要小得多。

##### [许可检查](http://shiro.apache.org/authorization.html#permission-checks)

如果要检查`Subject`是否允许某些操作，可以调用任何`isPermitted*`一种方法变体。检查权限有两种主要方法 - 使用基于对象的`Permission`实例或使用表示的字符串`Permissions`

###### 基于对象的权限检查

执行权限检查的一种可能方法是实例化Shiro [`org.apache.shiro.authz.Permission`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authz/Permission.html)接口的实例并将其传递给`*isPermitted`接受权限实例的方法。

例如，请考虑以下情形：`Printer`办公室中有一个具有唯一标识符的办公室`laserjet4400n`。我们的软件需要检查当前用户是否允许在该打印机上打印文档，然后才允许他们按下“打印”按钮。权限检查以确定是否可以这样表达：

```jaVA
Permission printPermission = new PrinterPermission("laserjet4400n", "print");

Subject currentUser = SecurityUtils.getSubject();

if (currentUser.isPermitted(printPermission)) {
    //show the Print button 
} else {
    //don't show the button?  Grey it out?
}
```

在此示例中，我们还看到了一个非常强大的*实例级*访问控制检查示例- 基于*单个数据实例*限制行为的能力。

基于对象在`Permissions`以下情况下很有用：

- 你想要编译时类型安全
- 您希望保证正确表示和使用权限
- 您希望显式控制权限解析逻辑（称为权限隐含逻辑，基于Permission接口的[`implies`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authz/Permission.html#implies-org.apache.shiro.authz.Permission-)方法）的执行方式。
- 您希望保证权限准确反映应用程序资源（例如，可以在项目的构建期间根据项目的域模型自动生成权限类）。

`Subject`根据您的需要，您可以调用几种面向对象权限的方法：

| 主题方法                                                     | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [`isPermitted(Permission p)`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/subject/Subject.html#isPermitted-org.apache.shiro.authz.Permission-) | 返回`true`是否`Subject`允许执行操作或访问由指定`Permission`实例汇总的资源，`false`否则返回。 |
| [`isPermitted(List perms)`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/subject/Subject.html#isPermitted-java.util.List-) | 返回`isPermitted`与方法参数中的索引对应的结果数组。如果需要执行许多权限检查（例如，在自定义复杂视图时），则可用作性能增强 |
| [`isPermittedAll(Collection perms)`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/subject/Subject.html#isPermittedAll-java.util.Collection-) | `true`如果`Subject`允许*所有*指定的权限，`false`则返回，否则返回。 |

###### 基于字符串的权限检查

虽然基于对象的权限可能很有用（编译时类型安全，保证行为，自定义隐含逻辑等），但对于许多应用程序来说，它们有时会感觉有点“苛刻”。另一种方法是使用normal `Strings`来表示权限实例。

例如，根据上面的打印权限示例，我们可以将相同的检查重新制定为`String`基于权限的检查：

```java
Subject currentUser = SecurityUtils.getSubject();

if (currentUser.isPermitted("printer:print:laserjet4400n")) {
    //show the Print button
} else {
    //don't show the button?  Grey it out? 
}
```

此示例仍显示相同的实例级权限检查，但权限的重要部分 - `printer`（资源类型），`print`（操作）和`laserjet4400n`（实例ID） - 都在String中表示。

这个特殊的例子展示了一个由Shiro的默认[`org.apache.shiro.authz.permission.WildcardPermission`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authz/permission/WildcardPermission.html)实现定义的特殊冒号分隔格式，大多数人都会认为这种格式是合适的。

也就是说，上面的代码块（大部分）是以下的快捷方式：

```
Subject currentUser = SecurityUtils.getSubject();

Permission p = new WildcardPermission("printer:print:laserjet4400n");

if (currentUser.isPermitted(p) {
    //show the Print button
} else {
    //don't show the button?  Grey it out?
}
```

该`WildcardPermission`令牌格式和地层选项将在深入四郎在[许可文件](http://shiro.apache.org/permissions.html)。

虽然上面的String默认为`WildcardPermission`格式，但您实际上可以创建自己的String格式并根据需要使用它。我们将在下面的领域授权部分介绍如何执行此操作。

基于字符串的权限是有益的，因为您不必强制实现接口，并且简单的字符串通常易于阅读。缺点是您没有类型安全性，如果您需要更复杂的行为，这些行为超出了Strings所代表的范围，您将需要基于权限接口实现您自己的权限对象。在实践中，大多数Shiro最终用户选择基于字符串的方法是为了简化，但最终您的应用程序的要求将决定哪个更好。

与基于对象的权限检查方法一样，String变体支持基于String的权限检查：

| 主题方法                                                     | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [`isPermitted(String perm)`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/subject/Subject.html#isPermitted-java.lang.String-) | 返回`true`是否`Subject`允许执行操作或访问由指定`String`权限汇总的资源，`false`否则返回。 |
| [`isPermitted(String... perms)`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/subject/Subject.html#isPermitted-java.util.List-) | 返回`isPermitted`与方法参数中的索引对应的结果数组。如果`String`需要执行许多权限检查（例如，在自定义复杂视图时），则可用作性能增强 |
| [`isPermittedAll(String... perms)`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/subject/Subject.html#isPermittedAll-java.lang.String...-) | `true`如果`Subject`允许*所有*指定的`String`权限，`false`则返回，否则返回。 |

##### [权限断言](http://shiro.apache.org/authorization.html#permission-assertions)

作为一种替代检查`boolean`，看看是否`Subject`被允许做某件事，或者没有，你可以简单地断言执行逻辑之前，他们有一个预期的许可。如果`Subject`不允许，[`AuthorizationException`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authz/AuthorizationException.html)将抛出一个。如果它们被允许按预期进行，则断言将静默执行，逻辑将按预期继续。

例如：

```
Subject currentUser = SecurityUtils.getSubject();

//guarantee that the current user is permitted 
//to open a bank account: 
Permission p = new AccountPermission("open");
currentUser.checkPermission(p);
openBankAccount();
```

或者，使用String权限进行相同的检查：

```
Subject currentUser = SecurityUtils.getSubject();

//guarantee that the current user is permitted 
//to open a bank account: 
currentUser.checkPermission("account:open");
openBankAccount();
```

这种方法相对于这些`isPermitted*`方法的一个好处是，代码可以更清晰，因为`AuthorizationExceptions`如果当前`Subject`不满足预期条件（如果您不想这样做），则不必构建自己的代码。

`Subject`根据您的需要，您可以调用几种面向权限的断言方法：

| 主题方法                                                     | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [`checkPermission(Permission p)`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/subject/Subject.html#checkPermission-org.apache.shiro.authz.Permission-) | 如果`Subject`允许执行操作或访问由指定`Permission`实例汇总的资源，则静默返回，否则抛出`AuthorizationException`if。 |
| [`checkPermission(String perm)`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/subject/Subject.html#checkPermission-java.lang.String-) | 如果`Subject`允许执行操作或访问由指定`String`权限汇总的资源，则静默返回，否则抛出`AuthorizationException`if。 |
| [`checkPermissions(Collection perms)`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/subject/Subject.html#checkPermissions-java.util.Collection-) | 如果`Subject`允许*所有*指定的权限，则安静地返回，如果不允许则返回`AuthorizationException`。 |
| [`checkPermissions(String... perms)`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/subject/Subject.html#checkPermissions-java.lang.String...-) | 与上述`checkPermissions`方法相同，但使用`String`基于权限的方法。 |

### 基于注释的授权

除了`Subject`API调用之外，如果您更喜欢基于元的授权控制，Shiro还提供了Java 5+注释的集合。

#### [组态](http://shiro.apache.org/authorization.html#configuration)

在使用Java注释之前，您需要在应用程序中启用AOP支持。有许多不同的AOP框架，遗憾的是，没有标准的方法可以在应用程序中启用AOP。

对于AspectJ，您可以查看我们的[AspectJ示例应用程序](https://github.com/apache/shiro/tree/master/samples/aspectj)。

对于Spring应用程序，您可以查看我们的[Spring Integration](http://shiro.apache.org/spring.html)文档。

对于Guice应用程序，您可以查看我们的[Guice Integration](http://shiro.apache.org/guice.html)文档。

#### 该`RequiresAuthentication`注解

该[RequiresAuthentication](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authz/annotation/RequiresAuthentication.html)注解要求当前`Subject`的本届会议期间已被认证被访问或调用被注解类/实例/方法。

例如：

```
@RequiresAuthentication
public void updateAccount(Account userAccount) {
    //this method will only be invoked by a
    //Subject that is guaranteed authenticated
    ...
}
```

这大致等同于以下基于主题的逻辑：

```
public void updateAccount(Account userAccount) {
    if (!SecurityUtils.getSubject().isAuthenticated()) {
        throw new AuthorizationException(...);
    }

    //Subject is guaranteed authenticated here
    ...
}
```

#### 该`RequiresGuest`注解

该[RequiresGuest](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authz/annotation/RequiresGuest.html)注解要求当前Subject是一个“客人”，也就是说，他们不认证，不从注解的类/实例/方法前一交易日记得访问或调用。

例如：

```
@RequiresGuest
public void signUp(User newUser) {
    //this method will only be invoked by a
    //Subject that is unknown/anonymous
    ...
}
```

这大致等同于以下基于主题的逻辑：

```
public void signUp(User newUser) {
    Subject currentUser = SecurityUtils.getSubject();
    PrincipalCollection principals = currentUser.getPrincipals();
    if (principals != null && !principals.isEmpty()) {
        //known identity - not a guest:
        throw new AuthorizationException(...);
    }

    //Subject is guaranteed to be a 'guest' here
    ...
}
```

#### 该`RequiresPermissions`注解

所述[RequiresPermissions](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authz/annotation/RequiresPermissions.html)注解要求当前Subject为了执行被注解的方法被允许一个或多个许可。

例如：

```
@RequiresPermissions("account:create")
public void createAccount(Account account) {
    //this method will only be invoked by a Subject
    //that is permitted to create an account
    ...
}
```

这大致等同于以下基于主题的逻辑：

```
public void createAccount(Account account) {
    Subject currentUser = SecurityUtils.getSubject();
    if (!subject.isPermitted("account:create")) {
        throw new AuthorizationException(...);
    }

    //Subject is guaranteed to be permitted here
    ...
}
```

#### 该`RequiresRoles`许可

该[RequiresRoles](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authz/annotation/RequiresRoles.html)注解要求当前Subject有所有指定的角色。如果他们没有角色，则不会执行该方法并抛出AuthorizationException。

例如：

```
@RequiresRoles("administrator")
public void deleteUser(User user) {
    //this method will only be invoked by an administrator
    ...
}
```

这大致等同于以下基于主题的逻辑：

```
public void deleteUser(User user) {
    Subject currentUser = SecurityUtils.getSubject();
    if (!subject.hasRole("administrator")) {
        throw new AuthorizationException(...);
    }

    //Subject is guaranteed to be an 'administrator' here
    ...
}
```

#### 该`RequiresUser`注解

所述[RequiresUser](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authz/annotation/RequiresUser.html) *注释需要当前主题是用于向被访问或调用的注解的类/实例/方法的应用程序的用户。“应用程序用户”被定义为`Subject`具有已知身份的已知身份，或者由于在当前会话期间进行身份验证或者从先前会话的“RememberMe”服务中记住而已知。

```
@RequiresUser
public void updateAccount(Account account) {
    //this method will only be invoked by a 'user'
    //i.e. a Subject with a known identity
    ...
}
```

这大致等同于以下基于主题的逻辑：

```
public void updateAccount(Account account) {
    Subject currentUser = SecurityUtils.getSubject();
    PrincipalCollection principals = currentUser.getPrincipals();
    if (principals == null || principals.isEmpty()) {
        //no identity - they're anonymous, not allowed:
        throw new AuthorizationException(...);
    }

    //Subject is guaranteed to have a known identity here
    ...
}
```

### [JSP TagLib授权](http://shiro.apache.org/authorization.html#jsp-taglib-authorization)

Shiro提供了一个标记库，用于根据`Subject`状态控制JSP / GSP页面输出。[Web](http://shiro.apache.org/web.html)章节的[JSP / GSP标记库](http://shiro.apache.org/web.html#Web-taglibrary)部分对此进行了介绍。

## [授权序列](http://shiro.apache.org/authorization.html#authorization-sequence)

现在我们已经看到了如何根据当前的情况执行授权`Subject`，让我们来看看每当进行授权调用时Shiro内部会发生什么。

我们从[架构](http://shiro.apache.org/architecture.html)章节中获取了以前的架构图，只留下了与授权相关的组件。每个数字代表授权操作中的一个步骤：

**步骤1**：应用程序或框架代码调用任何的`Subject` `hasRole*`，`checkRole*`，`isPermitted*`，或`checkPermission*`方法的变体，传递是必需的任何许可或角色表示。

**步骤2**：该`Subject`情况下，典型地是[`DelegatingSubject`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/subject/support/DelegatingSubject.html)（或子类）委托给应用程序的`SecurityManager`调用`securityManager`的几乎相同的各自`hasRole*`，`checkRole*`，`isPermitted*`，或`checkPermission*`方法变体（所述`securityManager`实现了[`org.apache.shiro.authz.Authorizer`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authz/Authorizer.html)接口，它定义了所有特定主题授权方法）。

**步骤3**：本`SecurityManager`，作为一个基本的“伞”分量，继电器/委托给其内部[`org.apache.shiro.authz.Authorizer`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authz/Authorizer.html)通过调用实例`authorizer`的各自`hasRole*`，`checkRole*`，`isPermitted*`，或`checkPermission*`方法。`authorizer`默认情况[`ModularRealmAuthorizer`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authz/ModularRealmAuthorizer.html)下，该实例是一个实例，它支持`Realm`在任何授权操作期间协调一个或多个实例。

**步骤4**：`Realm`检查每个配置以查看它是否实现相同的[`Authorizer`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authz/Authorizer.html)接口。如果是这样，该领域的各自的`hasRole*`，`checkRole*`，`isPermitted*`，或`checkPermission*`方法被调用。

### [ModularRealmAuthorizer](http://shiro.apache.org/authorization.html#modularrealmauthorizer)

如前所述，Shiro `SecurityManager`实现默认使用[`ModularRealmAuthorizer`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authz/ModularRealmAuthorizer.html)实例。在`ModularRealmAuthorizer`同样支持单领域，以及那些与多个领域的应用。

对于任何授权操作，`ModularRealmAuthorizer`将迭代其内部集合`Realms`并以迭代顺序与每个集合进行交互。每个`Realm`交互功能如下：

1. 如果`Realm`自身实现[`Authorizer`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authz/Authorizer.html)接口，其各自的`Authorizer`方法（`hasRole*`，`checkRole*`，`isPermitted*`，或`checkPermission*`）被调用。
   1. 如果Realm的方法导致异常，则异常将作为a传播[`AuthorizationException`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authc/AuthenticationException.html)给`Subject`调用者。这会使授权过程短路，并且不会为该授权操作查询任何剩余的域。
   2. 如果Realm的方法是返回布尔值且返回值为的变量`hasRole*`或`isPermitted*`变量，则立即返回`true`该`true`值，并且任何剩余的Realms都会被短路。此行为作为性能增强而存在，通常如果一个Realm允许，则暗示允许使用Subject。这有利于安全策略，默认情况下禁止所有内容，并且明确允许使用最安全的安全策略类型。
2. 如果Realm没有实现`Authorizer`接口，则忽略它。

#### [领域授权命令](http://shiro.apache.org/authorization.html#realm-authorization-order)

重要的是要指出，与身份验证完全一样，`ModularRealmAuthorizer`它将以*迭代*顺序与Realm实例交互。

在`ModularRealmAuthorizer`先后获得了`Realm`所配置的实例`SecurityManager`。当执行一个授权操作时，它会遍历该集合，并且对于每个`Realm`实现该`Authorizer`接口本身，调用Realm的相应`Authorizer`方法（例如`hasRole*`，`checkRole*`，`isPermitted*`，或`checkPermission*`）。

#### 配置全局 `PermissionResolver`

在执行`String`基于权限的权限检查时，Shiro的大多数默认`Realm`实现[`Permission`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authz/Permission.html)在执行权限*隐含*逻辑之前首先将此String转换为实际实例。

这是因为权限是基于隐含逻辑而不是直接相等性检查来评估的（有关隐含与相等性的更多信息，请参阅[权限](http://shiro.apache.org/permissions.html)文档）。隐含逻辑在代码中比通过字符串比较更好地表示。因此，大多数Realms需要将提交的权限字符串转换或*解析*为相应的代表`Permission`实例。

为了帮助这种转换，Shiro支持a的概念[`PermissionResolver`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authz/permission/PermissionResolver.html)。大多数`Shiro`Realm实现使用a `PermissionResolver`来支持它们基于`Authorizer`接口的`String`权限方法的实现：当在Realm上调用其中一个方法时，它将使用它将`PermissionResolver`字符串转换为Permission实例，并以这种方式执行检查。

所有Shiro `Realm`实现默认为内部[WildcardPermissionResolver](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authz/permission/WildcardPermissionResolver.html)，它采用Shiro的[`WildcardPermission`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authz/permission/WildcardPermission.html)String格式。

如果要创建自己的`PermissionResolver`实现，可能是为了支持自己的Permission字符串语法，并且希望所有已配置的`Realm`实例都支持该语法，则可以为所有可配置的实例设置`PermissionResolver`全局实现`Realms`。

例如，在`shiro.ini`：

**shiro.ini**

```ini
globalPermissionResolver = com.foo.bar.authz.MyPermissionResolver
...
securityManager.authorizer.permissionResolver = $globalPermissionResolver
...
```

 

PermissionResolverAware

------

如果要配置全局`PermissionResolver`，则每个`Realm`要接收已配置的`PermissionResolver` **必须**实现该[`PermisionResolverAware`](http://shiro.apache.org/static/current/apidocs/src-html/org/apache/shiro/authz/permission/PermissionResolverAware.html)接口。这可以保证配置的实例可以中继到`Realm`支持此类配置的每个实例。

如果您不想使用全局`PermissionResolver`或者您不希望被`PermissionResolverAware`接口所困扰，则可以始终`PermissionResolver`显式配置具有实例的域（假设存在与JavaBeans兼容的setPermissionResolver方法）：

```ini
permissionResolver = com.foo.bar.authz.MyPermissionResolver

realm = com.foo.bar.realm.MyCustomRealm
realm.permissionResolver = $permissionResolver
...
```

#### 配置全局 `RolePermissionResolver`

在概念上与a类似`PermissionResolver`，a [`RolePermissionResolver`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authz/permission/RolePermissionResolver.html)具有表示执行权限检查`Permission`所需的实例的能力`Realm`。

`RolePermissionResolver`然而，与a的关键区别在于输入`String`是*角色名称*，而*不是*权限字符串。

当需要将角色名称转换为具体的实例集时，内部`RolePermissionResolver`可以使用A.`Realm``Permission`

这是一个特别有用的功能，用于支持可能没有权限概念的遗留或不灵活的数据源。

例如，许多LDAP目录存储角色名称（或组名称），但不支持将角色名称与具体权限关联，因为它们没有“权限”概念。基于Shiro的应用程序可以使用存储在LDAP中的角色名称，但实现a `RolePermissionResolver`将LDAP名称转换为一组显式权限以执行首选显式访问控制。权限关联将存储在另一个数据存储中，可能是本地数据库。

因为将角色名转换为权限这一概念非常特定于应用程序，所以Shiro的默认`Realm`实现不使用它们。

但是，如果要创建自己的实现并且要使用它配置`RolePermissionResolver`多个`Realm`实现，则可`RolePermissionResolver`以为所有`Realms`可以配置的实现设置全局。

**shiro.ini**

```
globalRolePermissionResolver = com.foo.bar.authz.MyPermissionResolver
...
securityManager.authorizer.rolePermissionResolver = $globalRolePermissionResolver
...
```

 

RolePermissionResolverAware

------

如果要配置全局`RolePermissionResolver`，则每个`Realm`要接收已配置的`RolePermissionResolver` **必须**实现该[`RolePermisionResolverAware`](http://shiro.apache.org/static/current/apidocs/org/apache/shiro/authz/permission/RolePermissionResolverAware.html)接口。这可以保证配置的全局`RolePermissionResolver`实例可以中继到`Realm`支持此类配置的每个实例。

如果您不想使用全局`RolePermissionResolver`或者您不希望被`RolePermissionResolverAware`接口所困扰，则可以始终`RolePermissionResolver`显式配置具有实例的域（假设存在与JavaBeans兼容的setRolePermissionResolver方法）：

```ini
rolePermissionResolver = com.foo.bar.authz.MyRolePermissionResolver

realm = com.foo.bar.realm.MyCustomRealm
realm.rolePermissionResolver = $rolePermissionResolver
...
```

### [自定义授权者](http://shiro.apache.org/authorization.html#custom-authorizer)

如果您的应用程序使用多个领域执行授权，并且`ModularRealmAuthorizer`基于默认的简单迭代，短路授权行为不适合您的需求，您可能需要创建自定义`Authorizer`并相应地进行配置`SecurityManager`。

例如，在`shiro.ini`：

```ini
[main]
...
authorizer = com.foo.bar.authz.CustomAuthorizer

securityManager.authorizer = $authorizer
```