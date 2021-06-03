### flink入门-mysql目标源

#### 实战
flink将计算结果写入到mysql中方便后续程序进行展示, 不过在编写之前, 我们需要将依赖加入到pom文件中
```xml
<!-- mysql驱动依赖, 我这里使用的是8.0的版本了 -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>${mysql.version}</version>
</dependency>
```

添加依赖之后, 如果不使用数据库连接池, 则和普通的jdbc程序没有任何区别, 但唯一要注意的是, 我们使用的是***RichSinkFunction***类而不是***SinkFunction***接口, SinkFunction接口只有invoke()方法, 而每进入一条数据就会调用一次invoke()方法, 我们不可能在这里初始化连接, 所以, 需要使用RichSinkFunction类, 并在open()方法中进行初始化

编写之前, 我们先创建一张要写入的mysql表(注意: 下面建表方式只是为了方便测试)
```sql
create table student (
id int,
name varchar(50),
score double
);
```

```java
DataStreamSource<String> socketStream =
                env.socketTextStream("localhost", 8888);

SingleOutputStreamOperator<Student> studentStream = socketStream
        .filter(value -> value != null && value.length() > 0)
        .map(value -> {
            String[] fields = value.split(",");
            return new Student(Integer.parseInt(fields[0]), fields[1], Double.parseDouble(fields[2]));
        }).returns(Types.POJO(Student.class));
```

这里, 我们将socket数据进行过滤和转换为Student对象, 下面开发编写sink


```java
private static class StudentMySQLSink extends RichSinkFunction<Student> {
    private Connection connection;
    private PreparedStatement insertStatement;
    private PreparedStatement updateStatement;

    @Override
    public void open(Configuration parameters) throws Exception {
        connection = getConnection();
        insertStatement = connection.prepareStatement("insert into student(id, name, score) values(?,?,?)");
        updateStatement = connection.prepareStatement("update student set score = ? where id = ?");
    }

    @Override
    public void invoke(Student value, Context context) throws SQLException {

        updateStatement.setDouble(1, value.getScore());
        updateStatement.setInt(2, value.getId());
        updateStatement.execute();

        // 如果没更新成功, 表示mysql不存在数据, 进行插入
        if (updateStatement.getUpdateCount() == 0) {
            insertStatement.setInt(1, value.getId());
            insertStatement.setString(2, value.getName());
            insertStatement.setDouble(3, value.getScore());
            insertStatement.execute();
        }
    }

    @Override
    public void close() throws Exception {

        if (insertStatement != null)
            insertStatement.close();

        if (updateStatement != null)
            updateStatement.close();

        if (connection != null)
            connection.close();
    }

    private Connection getConnection() {
        try {
            String className = "com.mysql.cj.jdbc.Driver";
            Class.forName(className);
            String url = "jdbc:mysql://localhost:3306/test?useSSL=false&serverTimezone=Asia/Shanghai&characterEncoding=utf-8&autoReconnect=true";
            String user = "root";
            String pass = "123456";
            Connection connection = DriverManager.getConnection(url, user, pass);
            return connection;
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (SQLException e) {
            e.printStackTrace();
        }

        return null;
    }
}
```

最后, 我们只需要在刚才的数据源上进行addSink()添加对应的数据源就可以啦
```java
studentStream.addSink(new StudentMySQLSink());
```

#### 写在最后
当然, 大家可以将jdbc的方式该成连接池的方式, 当做一个小练习