#### ActiveStandbyElector分析

##### 前言
Hadoop NameNode(包括YARN ResourceManager)的主备选举都是通过ActiveStandbyElector来完成的, ActiveStandbyElector主要是利用ZooKeeper的写一致性和临时节点机制, 具体的主备选举实现, 下面会通过源码的方式进行分析。


##### 构建ActiveStandbyElector实例
在分析ActiveStandbyElector具体如何选举, 如何防止脑裂等功能之前, 我们先来看看ActiveStandbyElector对象如何实例化, 都需要哪些参数, 以及在初始化都经过了哪些过程。


```java
// TestActiveStandbyElector.java

class ActiveStandbyElectorTester extends ActiveStandbyElector {
  private int sleptFor = 0;

  ActiveStandbyElectorTester(String hostPort, int timeout, String parent,
      List<ACL> acl, ActiveStandbyElectorCallback app) throws IOException,
      KeeperException {
    super(hostPort, timeout, parent, acl, Collections
        .<ZKAuthInfo> emptyList(), app,
        CommonConfigurationKeys.HA_FC_ELECTOR_ZK_OP_RETRIES_DEFAULT);
  }

  @Override
  public ZooKeeper getNewZooKeeper() {
    ++count;
    return mockZK;
  }

  @Override
  protected void sleepFor(int ms) {
    // don't sleep in unit tests! Instead, just record the amount of
    // time slept
    LOG.info("Would have slept for " + ms + "ms");
    sleptFor += ms;
  }
}
```

```java
// TestActiveStandbyElector.java

public void init() throws IOException, KeeperException {
  count = 0;
  mockZK = Mockito.mock(ZooKeeper.class);
  mockApp = Mockito.mock(ActiveStandbyElectorCallback.class);

  // 构建ActiveStandbyElector对象
  elector = new ActiveStandbyElectorTester("hostPort", 1000, ZK_PARENT_NAME,
      Ids.OPEN_ACL_UNSAFE, mockApp);
}
```

以上的代码都是hadoop源码中自带的测试类, 这里我们直接取出来使用, 这里可以看到开源框架对内部类的一个使用。

下面才是重头戏, 我们来看看ActiveStandbyElector构造参数。

```java

/****
  zookeeperHostPorts: ZooKeeper服务集群地址
  zookeeperSessionTimeout: ZooKeeper会话超时
  parentZnodeName: 在该目录下创建锁节点(znode)
  acl: ZooKeeper ACL's
  authInfo: a list of authentication credentials to add to the ZK connection
  app: 回调接口对象(ActiveStandbyElectorCallback)

*/
public ActiveStandbyElector(String zookeeperHostPorts,
    int zookeeperSessionTimeout, String parentZnodeName, List<ACL> acl,
    List<ZKAuthInfo> authInfo,
    ActiveStandbyElectorCallback app, int maxRetryNum) throws IOException,
    HadoopIllegalArgumentException, KeeperException {

  // 可以看到, 首先判断一些重要的参数值如果为空则直接抛出参数异常
  if (app == null || acl == null || parentZnodeName == null
      || zookeeperHostPorts == null || zookeeperSessionTimeout <= 0) {
    throw new HadoopIllegalArgumentException("Invalid argument");
  }
  zkHostPort = zookeeperHostPorts;
  zkSessionTimeout = zookeeperSessionTimeout;
  zkAcl = acl;
  zkAuthInfo = authInfo;
  appClient = app;
  znodeWorkingDir = parentZnodeName;
  // 锁节点路径
  // protected static final String LOCK_FILENAME = "ActiveStandbyElectorLock";
  zkLockFilePath = znodeWorkingDir + "/" + LOCK_FILENAME;

  // 暂时还没搞明白该字段的作用
  // protected static final String BREADCRUMB_FILENAME = "ActiveBreadCrumb";
  zkBreadCrumbPath = znodeWorkingDir + "/" + BREADCRUMB_FILENAME;
  // 最大重试次数
  this.maxRetryNum = maxRetryNum;

  // createConnection for future API calls
  createConnection();
}
```


```java

private void createConnection() throws IOException, KeeperException {

  // 如果ZK客户端还存在则先关闭。
  if (zkClient != null) {
    try {
      zkClient.close();
    } catch (InterruptedException e) {
      throw new IOException("Interrupted while closing ZK",
          e);
    }
    zkClient = null;
    watcher = null;
  }

  /**
    这里很有意思哦, 这里是不会走ActiveStandbyElector.java类的getNewZooKeeper方法的, 如果不清楚回头看一下。
    我们的子类重写了该方法, 所以会调用TestActiveStandbyElector.java的
    getNewZooKeeper方法, 客户端数量加1并返回ZK客户端对象实例。
  */
  zkClient = getNewZooKeeper();
  LOG.debug("Created new connection for " + this);
}
```
