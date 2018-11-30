# Apache Shiro的权限控制

Shiro将Permission(权限)定义为定义显式行为或操作的语句。它是应用程序中原始功能的声明，仅此而已。权限是安全策略中最低级别的构造，它明确定义了应用当前用户可以执行哪些操作。

他们不负责**谁**能执行操作.

一些权限控制的示例：

- 打开一个文件
- 查看“/ user / list”网页
- 打印文件
- 删除'jsmith'用户

定义“谁”（用户）被允许做“什么”（权限）就是给用户授权。这是由项目的用户数据完成,  并且可以根据需要改变。

例如，一个角色可以有很多权限，该角色可以与一个或多个用户关联。或者某些应用程序可以拥有一组用户，并且通过传递关联可以为一个用户组分配一个角色，那么所有组里的用户都有这个角色,  同样也就有角色用的被赋予的权限。

为用户授权有很多种方式。

## [通配符权限](http://shiro.apache.org/permissions.html#wildcard-permissions)

上述权限示例，“打开文件”，“查看'user/list'网页”等都是有效的权限声明。但是对计算机来说并不能理解。

因此，为了简便易懂的权限声明，Shiro提供了强大而直观的权限语法，我们将其称为WildcardPermission。

### [简单用法](http://shiro.apache.org/permissions.html#simple-usage)

假设您希望为公司的打印机设置使用和打印的权限

一种非常简单的方法是授予用户“queryPrinter”权限。然后，您可以通过调用以查看用户是否具有queryPrinter权限来查看：

```java
subject.isPermitted("queryPrinter")
```

这（几乎）相当于

```java
subject.isPermitted( new WildcardPermission("queryPrinter") )
```

简单权限字符串可能适用于简单的应用程序，但它要求您具有“printPrinter”，“queryPrinter”，“managePrinter”等权限。您还可以使用通配符`"*"`为用户赋予所有操作权限。

但是使用这种方法，没有办法说用户具有“所有打印机权限”。因此，通配符权限支持多*级*权限。

### [多级权限](http://shiro.apache.org/permissions.html#multiple-parts)

通配符权限支持多个*级别*或*部分*的概念。例如,  你可以通过如下示例向用户授权

```ini
printer:query
```

此示例中的冒号是一个特殊字符，用户分割权限的上一部分和下部分。

> 译者:  这就是说当你为一个用户授予printer:query (这是字符串,  是根据需要定义的) 的权限后,  这个用户就拥有了查询打印机的权限.  在举个简单的例子,  我们为admin用户赋予 `user:delete`权限,  admin就有了删除用户的权限.  在执行删除用户的操作时,  shiro就会检查用户是否有 `user:delete`权限

在此示例中，第一部分是在（`printer`）上操作的，第二部分是`query`正在执行的动作:

```ini
printer:print
printer:manage
```



#### [如果为用户授予多个权限](http://shiro.apache.org/permissions.html#multiple-values)

每个部分都可以包含多个值。因此，您不必向用户授予“printer：print”和“printer：query”权限，而只需授予他们一个权限：

```ini
printer:print,query
```

这使他们有可以`print`和`query`打印机。并且由于它们被授予这两个操作，您可以通过以下方法查看用户是否能够查询打印机：

```java
subject.isPermitted("printer:query")
```

如果有权限就会返回 `true`

#### [如何为用户赋予所有权限](http://shiro.apache.org/permissions.html#all-values)

如果您想为用户赋予所有权限，该怎么办？如果`printer`域有3个可能的权限（`query`，`print`，和`manage`），我们可以通过列出所有权限的方式.

```ini
printer:query,print,manage
```

也简单地变成这样：

```ini
printer:*
```

然后,  在调用`subject.isPermitted("printer:XXX")` 无论XXX是什么都会返回`true`了,   这样方便程序扩展,   如果添加了某种权限,  就无需在配置中添加该全新啊。

最后，你还可以在权限的任何部分使用同配置。例如，如果您想要授予所有域（不仅仅是打印机）的“查看”操作，您可以授予此权限：

```ini
*:view
```

然后将返回“XXX：view”的任何权限检查 `true`

> 译者: 也就是说当你调用`subject.isPermitted("user:view")` 或者`subject.isPermitted("printer:view")` 又获取任何都会返回true



### 实例级访问控制

通配符权限的另一个常见用法是为实例级访问控制列表建模。在这种情况下，您使用三个部分 - 第一部分是*域*，第二部分是*操作*，第三部分是被操作的实例。

所以例如你可以

```ini
printer:query:lp7200
printer:print:epsoncolor
```

第一个行定义了查询id为`lp7200`的打印机的操作。第二行定义了在`epsoncolor`打印的权限。如果您将这些权限授予用户，则他们可以在*特定实例*上执行特定行为。然后你可以检查代码：

```java
if ( SecurityUtils.getSubject().isPermitted("printer:query:lp7200") {
    // Return the current jobs on printer lp7200 }
}
```

这是一种表达权限的极其强大的方法。但同样，必须为所有打印机定义多个实例ID并不能很好地扩展，特别是在将新打印机添加到系统时。您可以改为使用通配符：

```ini
printer:print:*
```

这确实可以扩展，因为它也涵盖了所有新的打印机。您甚至可以允许访问所有打印机上的所有操作：

```ini
printer:*:*
```

或单个打印机上的所有操作：

```ini
printer:*:lp7200
```

甚至是具体行动：

```ini
printer:query,print:lp7200
```

'*'通配符和'，'子部分分隔符可用于权限的任何部分。

#### [没有配置的部分](http://shiro.apache.org/permissions.html#missing-parts)

最后要注意的是权限分配：缺少部分意味着用户可以访问与该部分对应的所有值。换一种说法，

```ini
printer:print
```

相当于

```ini
printer:print:*
```

和

```
printer
```

相当于

```ini
printer:*:*
```

但是，您只能在字符串*末尾*部分留空，因此：

```ini
printer:lp7200
```

不等于

```ini
printer:*:lp7200
```

## [检查权限](http://shiro.apache.org/permissions.html#checking-permissions)

虽然权限分配使用通配符简便移动,  而且便于扩展，但运行时的应*始终*基于明确的具体的权限字符串。

例如，如果用户有一个UI并且他们想要将文档打印到`lp7200`打印机，则**应该**通过执行以下代码来检查是否允许用户这样做：

```java
if ( SecurityUtils.getSubject().isPermitted("printer:print:lp7200") ) {
    //print the document to the lp7200 printer 
}
```

该检查非常具体，并明确反映了用户在该时刻尝试做的事情。

但是，对于运行时以下内容则不太理想：

```java
if ( SecurityUtils.getSubject().isPermitted("printer:print") ) {
    //print the document
}
```

为什么？因为第二个例子说“你必须能够打印到**任何**打印机才能执行以下代码块”。所以请记住，“printer：print”相当于“printer：print：*”！

因此，这是一个不正确的检查。如果当前用户无法打印到所有打印机，但他们**确实**能够打印`lp7200`和`epsoncolor`打印机，该**怎么办**？然后上面的第二个例子将永远不允许他们打印到`lp7200`打印机，即使他们已被授予该权限！

**因此，在执行权限检查时使用最具体的权限字符串。**

## [使用隐式逻辑，而不是相等](http://shiro.apache.org/permissions.html#implication-not-equality)

为什么运行时权限检查应尽可能具体，但权限分配可以更通用一些？这是因为权限检查由*隐含*逻辑评估- 而不是相等性检查。

也就是说，如果为用户分配了`user:*`权限，则这*意味着*用户可以执行该`user:view`操作。字符串`user：*`显然不等于`user：view`，但前者包含后者。`user：*`是`user：view`的超集。

为了支持隐含规则，所有权限都会转换为实现该`org.apache.shiro.authz.Permission`接口的实例对象。这样可以在运行时执行隐含逻辑，并且隐含逻辑通常比简单的字符串相等性检查更复杂。本文档中描述的所有通配符行为实际上都可以通过`org.apache.shiro.authz.permission.WildcardPermission`类实现实现。以下是一些通配符权限字符串，通过暗示显示访问权限：

```ini
user:*
```

*意味着*还能删除用户：

```ini
user:delete
```

同样的，

```ini
user:*:12345
```

*意味着*还能够更新ID为12345的用户帐户：

```ini
user:update:12345
```

和

```ini
printer
```

*意味着*能够打印到任何打印机

```ini
printer:print
```

## [性能注意事项](http://shiro.apache.org/permissions.html#performance-considerations)

权限检查比简单的等于比较更复杂，因此必须为每个分配的权限执行运行时隐含逻辑。当使用如上所示的权限字符串时，您隐式使用Shiro的默认值`WildcardPermission`来执行必要的隐含逻辑。

Shiro对Realm实现的默认行为是，对于每个权限检查（例如，对其进行调用`subject.isPermitted`），需要单独检查分配给该用户的*所有*权限（在其组，角色或直接分配给它们）中的*所有*权限。Shiro通过在第一次成功检查后立即返回来“短路”此过程以提高性能，但它不是一个灵丹妙药。

当使用适当的[CacheManager时](http://shiro.apache.org/cachemanager.html)，当用户，角色和权限缓存在内存中时，这通常非常快，Shiro确实支持Realm实现。只要知道使用此默认行为，随着分配给用户或其角色或组的权限数量的增加，执行检查的时间必然会增加。

如果Realm实现者有更有效的方法来检查权限并执行此隐含逻辑，特别是如果基于应用程序的数据模型，他们应该将其作为Realm isPermitted *方法实现的一部分来实现。默认的Realm / WildcardPermission支持占大多数用例的80-90％，但对于在运行时存储和/或检查的大量权限的应用程序，它可能不是最佳解决方案。