# [命令行Hasher](http://shiro.apache.org/command-line-hasher.html#command-line-hasher)

- [用法](http://shiro.apache.org/command-line-hasher.html#CommandLineHasher-Usage)
- [常见情景](http://shiro.apache.org/command-line-hasher.html#CommandLineHasher-CommonScenarios)
- [`shiro.ini` 用户密码](http://shiro.apache.org/command-line-hasher.html#CommandLineHasher-%7B%7Bshiro.ini%7D%7DUserPasswords)
- [MD5校验和](http://shiro.apache.org/command-line-hasher.html#CommandLineHasher-MD5checksum)

Shiro 1.2.0及更高版本提供了一个命令行程序，可以散列几乎任何类型的字符串和资源（文件，URL，类路径条目）。要使用它，必须安装Java虚拟机，并且必须在`$PATH`环境变量中访问“java”命令。

## [用法](http://shiro.apache.org/command-line-hasher.html#usage)

确保您可以访问`shiro-tools-hasher-`_version_ `-cli.jar`文件。您可以在*buildroot*`/tools/hasher/target`目录中的源代码构建中找到它，也可以通过Maven下载。

```
# Use the following to download from Maven Central into
# ~/.m2/repository/org/apache/shiro/tools/shiro-tools-hasher/X.X.X/shiro-tools-hasher-X.X.X-cli.jar
$ mvn dependency:get -DgroupId=org.apache.shiro.tools -DartifactId=shiro-tools-hasher -Dclassifier=cli -Dversion=X.X.X
```

一旦有权访问jar，就可以运行以下命令：

```
$ java -jar shiro-tools-hasher-X.X.X-cli.jar
```

这将打印标准（MD5，SHA1）和更复杂的密码散列方案的所有可用选项。

## [常见情景](http://shiro.apache.org/command-line-hasher.html#common-scenarios)

请阅读上述命令的印刷说明。它将提供详尽的说明列表，以帮助您根据需要使用哈希。但是，为方便起见，我们在下面提供了一些快速参考用法/方案。

### `shiro.ini` 用户密码

最好将`shiro.ini` `[users]`部分中的用户密码保密。要添加新的用户帐户行，请使用带有`**-p**`（或`--password`）选项的上述命令：

```
$ java -jar shiro-tools-hasher-X.X.X-cli.jar -p
```

然后它会要求您输入密码然后确认：

```
Password to hash:
Password to hash (confirm):
```

执行此命令时，它将打印出安全盐渍迭代和散列的密码。例如：

```
$shiro1$SHA-256$500000$eWpVX2tGX7WCP2J+jMCNqw==$it/NRclMOHrfOvhAEFZ0mxIZRdbcfqIBdwdwdDXW2dM=
```

获取此值并将其作为密码放在[INI用户配置](http://shiro.apache.org/configuration.html#Configuration-%5Cusers%5C)文档中定义的用户定义行（后跟任何可选角色）中。例如：

```
[users]
...
user1 = $shiro1$SHA-256$500000$eWpVX2tGX7WCP2J+jMCNqw==$it/NRclMOHrfOvhAEFZ0mxIZRdbcfqIBdwdwdDXW2dM=
```

您还需要确保隐式`iniRealm`使用`CredentialsMatcher`知道如何执行安全散列密码比较的知识。所以在本`[main]`节中也要配置它：

```
[main]
...
passwordMatcher = org.apache.shiro.authc.credential.PasswordMatcher
iniRealm.credentialsMatcher = $passwordMatcher
...
```

### [MD5校验和](http://shiro.apache.org/command-line-hasher.html#md5-checksum)

虽然您可以使用JVM上支持的任何算法执行任何哈希，但默认的哈希算法是MD5，对于文件校验和是常见的。只需使用`**-r**`（或`--resource`）选项指示以下值是资源位置（而不是您希望散列的文本）：

```
$ java -jar shiro-tools-hasher-X.X.X-cli.jar -r RESOURCE_PATH
```

默认情况下`RESOURCE_PATH`，应该是文件路径，但您可以分别使用`classpath:`或`url:`前缀指定类路径或URL资源。

一些例子：

```
<command> -r fileInCurrentDirectory.txt
<command> -r ../../relativePathFile.xml
<command> -r ~/documents/myfile.pdf
<command> -r /usr/local/logs/absolutePathFile.log
<command> -r url:http://foo.com/page.html <command> -r classpath:/WEB-INF/lib/something.jar
```