#### Hadoop入门系列01-快速了解MR程序

##### 业务需求
现在老板张三丢给小王一个业务需求，说把今年最高分数找出来, 10分钟内可我, 小王想了想才一年数据量也不大, 于是写了一个单线程程序目的就是获取今年最高的分数, 非常轻松的就给了老板张三; 但是, 老板只看今年的不够, 我想看最近30年的最高分数，然后形成图表, 小王想着那我用多个线程去处理, 按照年份切分线程任务, 但不同年份数据量都不一样导致有的线程很快执行完有的线程还在执行, 最终还是要等待最大的文件执行完后才能汇总结果, 并且大量的数据受限于一台服务器的计算能力, 即使用上所有处理器也要花半个小时;

此时, 我们如何帮助小王解决这个需求问题呢?

##### 使用大数据框架Hadoop
Hadoop提供的MapReduce计算框架可以并行执行, 理解MR非常简单我们只需要实现一个map方法和一个reduce方法, 下面我们通过代码具体来展示MapReduce程序;

1. pom配置, 执行mr只需要添加一个hadoop-client依赖即可
```xml
<properties>
    <hadoop.version>2.7.1</hadoop.version>
</properties>
<dependencies>
    <dependency>
        <groupId>org.apache.hadoop</groupId>
            <artifactId>hadoop-client</artifactId>
            <version>${hadoop.version}</version>
        </dependency>
</dependencies>
```

2. 添加log4j.properties到resource文件夹中, 否则不会输出日志, 这里大家可以复制hadoop配置中的log4j.properties文件

3. 开始编写Mapper类

```java

/**
 *     获取最高分数Mapper类
 *      Mapper泛型类四个形参类型分别是: 输入KEY, 输入VALUE, 输出KEY, 输出VALUE
 *
 *      输入KEY: 偏移量, 从0~N直到文件结束的一个编号, 对于程序来说没有具体作用
 *      输入VALUE: 输入文件的具体内容, 也是我们要处理的数据
 *      输出KEY: 经过处理后得到的一个年份数据
 *      输出VALUE: 经过处理后得到的一个分数值
 *
 *      对于Text、LongWritable、IntWritable是Hadoop提供的可序列化及反序列化的类
 */
public class MaxScoreMapper extends Mapper<LongWritable, Text, Text, IntWritable> {
    @Override
    protected void map(LongWritable key, Text value, Context context) throws IOException, InterruptedException {
        String line = value.toString();
        String[] users = line.split(",");

        // 在mapper类中最一个简单逻辑判断, 如果缺失数据则丢弃掉, 真实开发中不允许
        if (users.length == 3) {
            // 通过Context写入数据, 其中输出KEY就是年份, 输出VALUE就是分数
            context.write(new Text(users[0]), new IntWritable(Integer.parseInt(users[2])));
        }
    }
}
```

4. 编写Reducer类

```java
/**
 * 最高分数Reducer类
 *      Reducer的KEYIN和VALUEIN就是我们Mapper类的输出类型, 所以必须和Mapper类的输出类型一致
 *      后面两个类型则是Reducer类型自己要输出的KEY和VALUE
 */
public class MaxScoreReducer extends Reducer<Text, IntWritable, Text, IntWritable> {
    @Override
    protected void reduce(Text key, Iterable<IntWritable> values, Context context) throws IOException, InterruptedException {
        int maxScore = Integer.MIN_VALUE;
        for (IntWritable value : values) {      //  map的数据，相同的key都在一起, 取出最大的分数值
            maxScore = Math.max(value.get(), maxScore);
        }
        context.write(key, new IntWritable(maxScore));      // 输出
    }
}
```

5. 编写运行的Job, 即程序执行入口

```java
public class MaxScoreJob {
    public static void main(String[] args) throws Exception {
        // 当不想手动输入时解开注释即可, 用于测试
        // File file = new File("src/main/resources/ch-01/in.csv");
        // String inPath = file.getAbsolutePath();
        // String outPath = inPath.substring(0, inPath.lastIndexOf("/") + 1) + "/output";

        args = new String[]{inPath, outPath};
        if (args.length != 2) {
            System.err.println("Usage: MaxScoreJob <input path> <output path>");
            System.exit(-1);
        }

        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf);        // 创建job实例

        job.setJarByClass(MaxScoreJob.class);
        job.setJobName("Max Score Job");            // 设置任务名称

        FileInputFormat.addInputPath(job, new Path(args[0]));       // 被处理数据路径
        FileOutputFormat.setOutputPath(job, new Path(args[1]));     // 处理后结果输出路径

        job.setMapperClass(MaxScoreMapper.class);               //  设置我们自定义的Mapper类
        job.setReducerClass(MaxScoreReducer.class);             // 设置Reducer类

        /***
         *
         *
         *     如果mapper和reducer的输出相同可以不用设置(本例中Mapper和Reducer输出都是相同的Text, IntWritable),
         *     但如果不相同就需要设置map函数输出类型
         *          job.setMapOutputKeyClass();
         *          job.setMapOutputValueClass();
         */

        job.setOutputKeyClass(Text.class);                  // 设置输出KEY类型, 即Reducer类的输出KEY类型
        job.setOutputValueClass(IntWritable.class);         // 设置输出VALUE类型, 即Reducer类的输出VALUE类型

        System.exit(job.waitForCompletion(true) ? 0 : 1);       // 提交任务并等待完成
    }
}
```


