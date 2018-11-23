# Apache Shiro 入门教程

##  第一个Apache Shiro应用程序

如果你是第一次接触Apache Shiro，这个入门教程将向你展示如何设置由Apache Shiro保护的非常简单的应用程序。我们将在此讨论Shiro的核心概念，以帮助你熟悉Shiro的设计和API。

如果你不想自己手写代码，你可以在下面选择一项下载到你的电脑本地上：

- 在Apache Shiro的Git存储库中：[https](https://github.com/apache/shiro/tree/master/samples/quickstart)：[//github.com/apache/shiro/tree/master/samples/quickstart](https://github.com/apache/shiro/tree/master/samples/quickstart)
- 在Apache Shiro的源代码发布`samples/quickstart`目录中。源分发可从“ [下载”](http://shiro.apache.org/download.html)页面获得。

### Setup 步骤

在这个简单的例子中，我们将创建一个非常简单的命令行应用程序.

>  **Shiro的通用性**

Apache Shiro旨在适用于任何工作环境 - 从最小的命令行应用程序到庞大的集群Web项目。即使我们现在创建的是一个简单的应用程序，但要知道无论您的应用程序如何创建或部署位置，这个教程都是适用的。

本教程需要Java 1.5或更高版本。我们将使用Apache [Maven](http://maven.apache.org/)作为构建工具

对于本教程，请确保您使用的是Maven 2.2.1或更高版本。可通过`mvn --version` 查看当前版本：

**测试Maven安装**

```
hazlewood:~/shiro-tutorial$ mvn --version
Apache Maven 2.2.1 (r801777; 2009-08-06 12:16:01-0700)
Java version: 1.6.0_24
Java home: /System/Library/Java/JavaVirtualMachines/1.6.0.jdk/Contents/Home
Default locale: en_US, platform encoding: MacRoman
OS name: "mac os x" version: "10.6.7" arch: "x86_64" Family: "mac"
```

接下来创建一个名为**shiro-tutorial**的maven项目,  pom文件如下：

**pom.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
    <project xmlns="http://maven.apache.org/POM/4.0.0"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">

    <modelVersion>4.0.0</modelVersion>
    <groupId>org.apache.shiro.tutorials</groupId>
    <artifactId>shiro-tutorial</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <name>First Apache Shiro Application</name>
    <packaging>jar</packaging>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>2.0.2</version>
                <configuration>
                    <source>1.5</source>
                    <target>1.5</target>
                    <encoding>${project.build.sourceEncoding}</encoding>
                </configuration>
            </plugin>

        <!-- This plugin is only to test run our little application.  It is not
             needed in most Shiro-enabled applications: -->
            <plugin>
                <groupId>org.codehaus.mojo</groupId>
                <artifactId>exec-maven-plugin</artifactId>
                <version>1.1</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>java</goal>
                        </goals>
                    </execution>
                </executions>
                <configuration>
                    <classpathScope>test</classpathScope>
                    <mainClass>Tutorial</mainClass>
                </configuration>
            </plugin>
        </plugins>
    </build>

    <dependencies>
        <dependency>
            <groupId>org.apache.shiro</groupId>
            <artifactId>shiro-core</artifactId>
            <version>1.1.0</version>
        </dependency>
        <!-- Shiro uses SLF4J for logging.  We'll use the 'simple' binding
             in this example app.  See http://www.slf4j.org for more info. -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-simple</artifactId>
            <version>1.6.1</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

</project>
```

#### Tutorial类

因为我们只是简单的运行一个命令行程序, 所以我们在`src/main/java`目录下创建一个Tutorial类,  然后创建main方法。

**src /main/java/ Tutorial.java**

```
import org.apache.shiro.SecurityUtils;
import org.apache.shiro.authc.*;
import org.apache.shiro.config.IniSecurityManagerFactory;
import org.apache.shiro.mgt.SecurityManager;
import org.apache.shiro.session.Session;
import org.apache.shiro.subject.Subject;
import org.apache.shiro.util.Factory;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class Tutorial {

    private static final transient Logger log = LoggerFactory.getLogger(Tutorial.class);

    public static void main(String[] args) {
        log.info("My First Apache Shiro Application");
        System.exit(0);
    }
}
```

暂时不要担心import语句 - 我们很快就会找到它们。但就目前而言，我们有一个典型的命令行程序。这个程序会打印出文本“My First Apache Shiro Application”并退出。

### 测试运行

要运行我们的Tutorial应用程序，请在项目的根目录（`shiro-tutorial`）中的命令提示符中执行以下命令：

`mvn compile exec:java`

你会看到“应用程序”运行并退出。您应该看到如下内容：

**运行应用程序**

```
lhazlewood:~/projects/shiro-tutorial$ mvn compile exec:java

... a bunch of Maven output ...

1 [Tutorial.main()] INFO Tutorial - My First Apache Shiro Application
lhazlewood:~/projects/shiro-tutorial\$
```

我们已经验证了应用程序运行成功 - 现在让我们启用Apache Shiro。在我们继续本教程时，您可以`mvn compile exec:java`在每次添加更多代码后运行，以查看更改的结果。



### 开启Shiro

Shiro所有的功能都和`SecurityManager` 息息相关，不过这和 java.lang.SecurityManager 不是一回事。

我们将在架构章节详细描述 Shiro 的设计，现在知道只要了解 Shrio SecurityManager 是程序中 Shiro 的核心，每一个程序都必定会存在一个 SecurityManager，所以，在我们这个示例程序中必须做的第一件事情是建立一个 SecurityManager 对象。

#### 配置

虽然我们可以直接对 SecurityManager 实例化，但在 Java 代码中对 SecurityManager 进行配置会很麻烦,  而使用配置文件就简单多了。

为此，Shiro 提供了一个基本的 INI 配置文件的解决方案，人们已经对庞大的 XML 文件有些厌倦了，而一个 INI 文件通俗易懂，而且所依赖的组件很少.

> **多种配置方式**
>
> SecurityManager 的实现和其所依赖的组件都是 JavaBean，所以可以用多种形式对 Shiro 进行配置，比如XML（Spring, JBoss, Guice, 等等），YAML, JSON, Groovy Builder markup等，INI 只是 Shiro 一种最基本的配置方式。

##### shiro.ini

在这个示例中我们使用一个 INI 文件来配置Shiro SecurityManager，首先，在`src/main/resources`中创建一个 shiro.ini 文件，内容如下：

src/main/resources/shiro.ini

```
# =============================================================================
# Tutorial INI configuration
#
# Usernames/passwords are based on the classic Mel Brooks' film "Spaceballs" :)
# =============================================================================

# -----------------------------------------------------------------------------
# Users and their (optional) assigned roles
# username = password, role1, role2, ..., roleN
# -----------------------------------------------------------------------------
[users]
root = secret, admin
guest = guest, guest
presidentskroob = 12345, president
darkhelmet = ludicrousspeed, darklord, schwartz
lonestarr = vespa, goodguy, schwartz

# -----------------------------------------------------------------------------
# Roles with assigned permissions
# roleName = perm1, perm2, ..., permN
# -----------------------------------------------------------------------------
[roles]
admin = *
schwartz = lightsaber:*
goodguy = winnebago:drive:eagle5
```

可以看到，在该配置文件中最基础地配置了几个静态的帐户，对我们这一个程序已经足够了，在以后的章节中，将会看到如何使用更复杂的用户数据比如数据库、LDAP 和活动目录等。



#### 引用配置

现在我们已经定义了一个 INI 文件，我们可以在我们的示例程序中创建SecurityManager 实例了，将 main 函数中的代码进行如下调整：

```
public static void main(String[] args) {

    log.info("My First Apache Shiro Application");

    //1.
    Factory<SecurityManager> factory = new IniSecurityManagerFactory("classpath:shiro.ini");

    //2.
    SecurityManager securityManager = factory.getInstance();

    //3.
    SecurityUtils.setSecurityManager(securityManager);

    System.exit(0);
}
```

这就是我们要做的--仅仅使用三行代码就把Shiro加进了我们的程序，就是这么简单。

执行`mvn compile exec:java` 可以看到程序成功的运行（由于 Shiro 默认在 debug 或更底层才记录日志，所以你不会看到任何 Shiro 的日志输出--只要运行时没有错误提示，你就可以知道已经成功了）。

上面所加入的代码做了下面的事情：

1. 使用 Shiro 的` IniSecurityManagerFactory` 加载shiro.ini 文件，该文件存在于 classpath 根目录里。这个执行动作反映出 shiro 支持 [Factory Method Design Pattern（工厂模式）](https://en.wikipedia.org/wiki/Factory_method_pattern)。classpath：资源的指示前缀，告诉 shiro 从哪里加载 ini 文件（其它前缀，如 url:和 file: 也被支持）。

2.factory.getInstance() 方法被调用，该方法分析 INI 文件并根据配置文件返回一个 SecurityManager 实例。

3.在这个简单示例中，我们将 SecurityManager 设置成了static (memory) singleton，可以通过 JVM 访问，注意如果你在一个 JVM 中加载多个使用 shiro 的程序时不要这样做，在这个简单示例中，这是可以的，但在其它成熟的应用环境中，通常会将 SecurityManager 放在程序指定的存储中（如在 web 中的 ServletContexct 或者 Spring、Guice、 JBoss DI 容器实例）中。

#### 使用Shiro

现在 SecurityManager 已经准备好了，我们可以开始进行我们真正关心的事情--执行安全操作了。

当我们为项目设置安全策略是，我们考虑最多的问题就是“当前用户是谁？”,  “当前用户是否有权限做某些操作？” 程序通常建立在用户基础上，程序功能展示（和安全）也基于每一个用户。所以，通常我们考虑我们程序安全的方法也建立在当前用户的基础上，Shiro 的 API 提供了'the current user'概念，即 Subject(以后我们叫主体)。

在几乎所有的环境中，你可以通过如下语句得到当前用户的信息：

```
Subject currentUser = SecurityUtils.getSubject();
```

通过 `SecurityUtils.getSubject()`，				我们可以获取当前执行的Subject，Subject就是当前运行用户的指定安全视图,   这里并不称之为“User”,  因为“User”这个词通常和一个人相关，但在安全认证中，`Subject`可以认为是一个人，也可以认为是第三方进程、定时任务、守护进程帐户或等等。它可简单描述为“和当前程序交互的主体”，在大多数情况下，你可以认为它是一个“人（User）”。

在一个独立的程序中调用 getSubject() 会在程序指定位置返回一个基于用户数据的 Subject，在服务器环境（如 web 程序）中，它将获取一个和当前线程或请求相关的基于用户数据的 Subject。

现在你得到了Subject，你可以利用它做什么呢？

如果你针对该用户希望一些事情在程序当前会话期内可行，你可以获取他们的 session：

```
Session session = currentUser.getSession();
session.setAttribute( "someKey", "aValue" );
```

Session 是 shiro 指定的一个实例，提供基本上所有 HttpSession 的功能，但具备额外的好处和不同：它不需要一个 HTTP 环境！

如果发布到一个 web 程序中，默认情况下 Session 将会使用HttpSession 作为基础，但是，在一个非 web 程序中，比如该简单示例程序中，Shiro 将自动默认使用它的 Enterprise Session Management，这意味着你可以在任何程序中使用相同的 API，而根本不需要考虑发布环境！这打开了一个全新的世界，从此任何需要 session 的程序不再需要强制使用 HttpSession 或者 EJB Stateful Session,并且，终端可以共享 session 数据。

现在你可以获取一个 Subject 和它们的 Session，真正填充有用的代码如检测其是否被允许做某些事情如何？比如检查其角色和权限？

我们只能对一个已知用户做这些检测，如上我们获取 Subject 实例表示当前用户，但是当前用户是认证，嗯，他们是任何人--直到他们至少登录一次，我们现在就做这件事情：

```
if ( !currentUser.isAuthenticated() ) {
    //收集用户的主要信息和凭据，来自GUI中的特定的方式
    //如包含用户名/密码的HTML表格，X509证书，OpenID，等。
    //我们将使用用户名/密码的例子因为它是最常见的。.
    UsernamePasswordToken token = new UsernamePasswordToken("lonestarr", "vespa");

    //支持'remember me' (无需配置，内建的!):
    token.setRememberMe(true);

    currentUser.login(token);
}
```

就是这样，不能再简单了。

但如果登录失败了呢，你可以捕获所有异常然后按你期望的方式去处理：

```
try {
    currentUser.login( token );
    //无异常，说明就是我们想要的!
} catch ( UnknownAccountException uae ) {
    //username 不存在，给个错误提示?
} catch ( IncorrectCredentialsException ice ) {
    //password 不匹配，再输入?
} catch ( LockedAccountException lae ) {
    //账号锁住了，不能登入。给个提示?
} 
    ... 更多类型异常 ...
} catch ( AuthenticationException ae ) {
    //未考虑到的问题 - 错误?
}
```

这里有许多不同类别的异常你可以检测到，也可以抛出你自己异常。详见 [AuthenticationException JavaDoc](https://shiro.apache.org/static/current/apidocs/org/apache/shiro/authc/AuthenticationException.html)

*小贴士：*

*最好的方式是将普通的失败信息反馈给用户，你总不会希望帮助黑客来攻击你的系统吧。*

好，到现在为止，我们有了一个登录用户，接下来我们还可以做什么？

让我们显示他们是谁

```
//打印主要信息 (本例子是 username):
log.info( "User [" + currentUser.getPrincipal() + "] logged in successfully." );
```

我们也可以判断他们是否拥有某个角色：

```
if ( currentUser.hasRole( "schwartz" ) ) {
    log.info("May the Schwartz be with you!" );
} else {
    log.info( "Hello, mere mortal." );
}
```

我们也可以判断他们是否拥有某个特定动作或入口的权限：

```
if ( currentUser.isPermitted( "lightsaber:weild" ) ) {
    log.info("You may use a lightsaber ring.  Use it wisely.");
} else {
    log.info("Sorry, lightsaber rings are for schwartz masters only.");
}
```

同样，我们还可以执行非常强大的 instance-level （实例级别）的权限检测，检测用户是否具备访问某个类型特定实例的权限：

```
if ( currentUser.isPermitted( "winnebago:drive:eagle5" ) ) {
    log.info("You are permitted to 'drive' the 'winnebago' with license plate (id) 'eagle5'.  " +
                "Here are the keys - have fun!");
} else {
    log.info("Sorry, you aren't allowed to drive the 'eagle5' winnebago!");
}
```

轻而易举，是吧！

最后，当用记不再使用系统，可以退出登录：

```
currentUser.logout(); //清楚识别信息，设置 session 失效.
```

#### Final Tutorial class 最终的 class

在加入上述代码后，下面的就是我们完整的文件，你可以自由编辑和运行它，可以尝试改变安全检测（以及INI配置）：

src/main/java/Tutorial.java

```
import org.apache.shiro.SecurityUtils;
import org.apache.shiro.authc.*;
import org.apache.shiro.config.IniSecurityManagerFactory;
import org.apache.shiro.mgt.SecurityManager;
import org.apache.shiro.session.Session;
import org.apache.shiro.subject.Subject;
import org.apache.shiro.util.Factory;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class Tutorial {

    private static final transient Logger log = LoggerFactory.getLogger(Tutorial.class);


    public static void main(String[] args) {
        log.info("My First Apache Shiro Application");

        Factory<SecurityManager> factory = new IniSecurityManagerFactory("classpath:shiro.ini");
        SecurityManager securityManager = factory.getInstance();
        SecurityUtils.setSecurityManager(securityManager);


        // 获取当前执行用户:
        Subject currentUser = SecurityUtils.getSubject();

        // 做点跟 Session 相关的事
        Session session = currentUser.getSession();
        session.setAttribute("someKey", "aValue");
        String value = (String) session.getAttribute("someKey");
        if (value.equals("aValue")) {
            log.info("Retrieved the correct value! [" + value + "]");
        }

        // 登录当前用户检验角色和权限
        if (!currentUser.isAuthenticated()) {
            UsernamePasswordToken token = new UsernamePasswordToken("lonestarr", "vespa");
            token.setRememberMe(true);
            try {
                currentUser.login(token);
            } catch (UnknownAccountException uae) {
                log.info("There is no user with username of " + token.getPrincipal());
            } catch (IncorrectCredentialsException ice) {
                log.info("Password for account " + token.getPrincipal() + " was incorrect!");
            } catch (LockedAccountException lae) {
                log.info("The account for username " + token.getPrincipal() + " is locked.  " +
                        "Please contact your administrator to unlock it.");
            }
            // ... 捕获更多异常
            catch (AuthenticationException ae) {
                //无定义?错误?
            }
        }

        //说出他们是谁:
        //打印主要识别信息 (本例是 username):
        log.info("User [" + currentUser.getPrincipal() + "] logged in successfully.");

        //测试角色:
        if (currentUser.hasRole("schwartz")) {
            log.info("May the Schwartz be with you!");
        } else {
            log.info("Hello, mere mortal.");
        }

        //测试一个权限 (非（instance-level）实例级别)
        if (currentUser.isPermitted("lightsaber:weild")) {
            log.info("You may use a lightsaber ring.  Use it wisely.");
        } else {
            log.info("Sorry, lightsaber rings are for schwartz masters only.");
        }

        //一个(非常强大)的实例级别的权限:
        if (currentUser.isPermitted("winnebago:drive:eagle5")) {
            log.info("You are permitted to 'drive' the winnebago with license plate (id) 'eagle5'.  " +
                    "Here are the keys - have fun!");
        } else {
            log.info("Sorry, you aren't allowed to drive the 'eagle5' winnebago!");
        }

        //完成 - 退出t!
        currentUser.logout();

        System.exit(0);
    }
}
```



### 总结

非常希望这示例介绍能帮助你理解如何在基础程序中加入 Shiro,并理解Shiro 的设计理念，Subject 和 SecurityManager。

但这个程序太简单了，你可能会问自己，“如果我不想使用 INI 用户帐号，而希望连接更为复杂的用户数据源呢？”

解决这些问题需要更深入地了解 并理解Shiro 的架构和配置机制，我们将在下一节 [Architecture](https://github.com/waylau/apache-shiro-1.2.x-reference/blob/master/I.%20Overview%20%E6%80%BB%E8%A7%88/3.%20Architecture%20%E6%9E%B6%E6%9E%84.md) 中介绍。