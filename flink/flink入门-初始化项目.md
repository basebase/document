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

当看到SUCCESS的时候, 就可以将创建的项目导入到IDEA进行程序编写了;

#### 参考
[Flink官网项目配置](https://ci.apache.org/projects/flink/flink-docs-release-1.12/dev/project-configuration.html#flink-core-and-application-dependencies)