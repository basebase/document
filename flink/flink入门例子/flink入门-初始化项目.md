### Flink入门-项目初始化

#### 初始化项目
如何创建一个在idea中可以运行的flink工程呢, 我们可以通过下面几种方式进行:
1. 添加flink相关依赖
2. 利用maven命令构建flink工程
3. shell脚本初始化(其实就是把maven命令打包放在了shell中)

这里, 我推荐使用第二种方式用于创建flink项目, 其中已经将相关依赖添加完成, 只需要实现代码功能即可;

```shell
mvn archetype:generate                               \
  -DarchetypeGroupId=org.apache.flink              \
  -DarchetypeArtifactId=flink-quickstart-java      \
  -DarchetypeVersion=1.12.0
```

这里我们使用的flink版本是1.12.0, 如果不想使用该版本可以进行修改;

在运行命令后下载依赖jar包后要求我们手动输入一些信息, 这个根据自己所需填写即可;

![maven命令初始化项目](https://github.com/basebase/document/blob/master/flink/image/%E5%88%9D%E5%A7%8B%E5%8C%96%E9%A1%B9%E7%9B%AE/maven%E5%91%BD%E4%BB%A4%E5%88%9D%E5%A7%8B%E5%8C%96%E9%A1%B9%E7%9B%AE.png?raw=true)


项目导入IDEA后, 我们先打开pom.xml文件, 将下面内容进行注释
![项目部署](https://github.com/basebase/document/blob/master/flink/image/%E5%88%9D%E5%A7%8B%E5%8C%96%E9%A1%B9%E7%9B%AE/%E9%A1%B9%E7%9B%AE%E9%83%A8%E7%BD%B2.png?raw=true)

如果没把上面内容进行注释, 当我们运行项目会抛出下面异常信息
```java
Error: A JNI error has occurred, please check your installation and try again
```

但是, 如果我们运行编写的wordcount例子, 就会出现更多找不到类信息, 如下
```text
Error: A JNI error has occurred, please check your installation and try again
Exception in thread "main" java.lang.NoClassDefFoundError: org/apache/flink/api/common/functions/FlatMapFunction
	at java.lang.Class.getDeclaredMethods0(Native Method)
	at java.lang.Class.privateGetDeclaredMethods(Class.java:2701)
	at java.lang.Class.privateGetMethodRecursive(Class.java:3048)
	at java.lang.Class.getMethod0(Class.java:3018)
	at java.lang.Class.getMethod(Class.java:1784)
	at sun.launcher.LauncherHelper.validateMainClass(LauncherHelper.java:544)
	at sun.launcher.LauncherHelper.checkAndLoadMain(LauncherHelper.java:526)
Caused by: java.lang.ClassNotFoundException: org.apache.flink.api.common.functions.FlatMapFunction
	at java.net.URLClassLoader.findClass(URLClassLoader.java:381)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:424)
	at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:338)
	at java.lang.ClassLoader.loadClass(ClassLoader.java:357)
	... 7 more
```

#### 参考
[Flink官网项目配置](https://ci.apache.org/projects/flink/flink-docs-release-1.12/dev/project-configuration.html#flink-core-and-application-dependencies)