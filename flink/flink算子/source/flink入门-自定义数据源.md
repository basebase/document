### flink入门-自定义数据源

#### 实战
生产环境中或许有一些特殊数据源渠道无法通过flink或者其它依赖包完成, 这个时候就需要我们自定义自己需要的数据源了, 要实现自己的数据源也非常简单, 我们只需要实现一个 ***SourceFunction***接口

SourceFunction接口中有两个方法需要我们实现
```java
void run(SourceContext<T> ctx) throws Exception;
void cancel();
```

我们将要传输的数据源通过run方法进行传递, 需要取消则通过cancel取消;


```java
public class UserDefinedSource {

    public static void main(String[] args) throws Exception {
        StreamExecutionEnvironment env =
                StreamExecutionEnvironment.getExecutionEnvironment();

        DataStreamSource<Student> studentSource = env.addSource(new StudentSource());
        studentSource.print();
        env.execute("User Defined Source Test Job");

    }

    private static class StudentSource implements SourceFunction<Student> {

        // 使用一个标志位, run方法判断当前标志位进行终止
        private volatile boolean isRunning = true;

        @Override
        public void run(SourceContext<Student> ctx) throws Exception {
            Random r = new Random();
            // 当调用canal方法后退出循环
            while (isRunning) {
                int id = r.nextInt();
                String name = "student_" + id;
                double score = r.nextDouble();
                // 通过SourceContext进行采集数据
                ctx.collect(new Student(id, name, score));
                // 避免发送过快, 暂停3s
                Thread.sleep(3000);
            }
        }

        @Override
        public void cancel() {
            isRunning = false;
            System.out.println("终止发送");
        }
    }
}
```