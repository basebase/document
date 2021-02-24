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

