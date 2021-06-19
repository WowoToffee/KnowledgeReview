# mybatis 源码分析

mybatis 也提供了对缓存的支持，分为：

- 一级缓存
- 二级缓存

![img](https://filescdn.proginn.com/2fb3b2f2085549b531582047ceb52567/402891529098f36d72a2520f93025baa.webp)

## 一级缓存

​		订单表与会员表是存在一对多的关系 为了尽可能减少join 查询，进行了分阶段查询，即先查询出订单表，在根据member_id 字段查询出会员表，最后进行数据整合 。如果订单表中存在重复的member_id，就会出现很多没必要的重复查询。

   	 针对这种情况myBatis 通过缓存来实现，在同一次查询会话中如果出现相同的语句及参数，就会从缓存中取出不在走数据库查询。一级缓存只能作用于查询会话中 所以也叫做会话缓存。

**每个sqlSeesion对象都有一个一级缓存，我们在操作数据库时需要构造sqlSeesion对象，在对象中有一个HashMap用于存储缓存数据。不同的sqlSession之间的缓存数据区域（HashMap）是互不影响的。**

### 使用条件：

```
1.必须是相同的SQL和参数

2.必须是相同的会话

3.必须是相同的namespace 即同一个mapper

4.必须是相同的statement 即同一个mapper 接口中的同一个方法

5.查询语句中间没有执行session.clearCache() 方法
```

### 源码分析：

接着看一下MyBatis一级缓存工作流程。前面说了，MyBatis的一级缓存是SqlSession级别的缓存，当openSession()的方法运行完毕或者主动调用了SqlSession的close方法，SqlSession就被回收了，一级缓存与此同时也一起被回收掉了。

```java
>org.apache.ibatis.session.SqlSession#selectList(java.lang.String)
	>org.apache.ibatis.executor.BaseExecutor#query()
	
    public <E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql) throws SQLException {
    ErrorContext.instance().resource(ms.getResource()).activity("executing a query").object(ms.getId());
    if (closed) {
      throw new ExecutorException("Executor was closed.");
    }
    if (queryStack == 0 && ms.isFlushCacheRequired()) {
      clearLocalCache();
    }
    List<E> list;
    try {
      queryStack++;
        // query的时候会尝试从localCache中去获取查询结果，如果获取到的查询结果为null，那么执行对应的代码从DB中捞数据，捞完之后会把CacheKey作为key，把查询结果作为value放到localCache中。
      list = resultHandler == null ? (List<E>) localCache.getObject(key) : null;
      if (list != null) {
        handleLocallyCachedOutputParameters(ms, key, parameter, boundSql);
      } else {
        list = queryFromDatabase(ms, parameter, rowBounds, resultHandler, key, boundSql);
      }
    } finally {
      queryStack--;
    }
    if (queryStack == 0) {
      for (DeferredLoad deferredLoad : deferredLoads) {
        deferredLoad.load();
      }
      // issue #601
      deferredLoads.clear();
      if (configuration.getLocalCacheScope() == LocalCacheScope.STATEMENT) {
        // issue #482
        clearLocalCache();
      }
    }
    return list;
  }

// update 
public int update(MappedStatement ms, Object parameter) throws SQLException {
    ErrorContext.instance().resource(ms.getResource()).activity("executing an update").object(ms.getId());
    if (this.closed) {
        throw new ExecutorException("Executor was closed.");
    } else {
        // clearLocalCache()方法，这意味着所有的增、删、改都会清空本地缓存，这和是否配置了flushCache=true是无关的
        this.clearLocalCache();
        return this.doUpdate(ms, parameter);
    }
}
```

#### 结论：

1. MyBatis的一级缓存是SqlSession级别的，但是它并不定义在SqlSessio接口的实现类DefaultSqlSession中，而是定义在DefaultSqlSession的成员变量Executor中，Executor是在openSession的时候被实例化出来的，它的默认实现为SimpleExecutor
2. MyBatis中的一级缓存，与有没有配置无关，==只要SqlSession存在，MyBastis一级缓存就存在==，localCache的类型是PerpetualCache，它其实很简单，一个id属性+一个HashMap属性而已，id是一个名为"localCache"的字符串，HashMap用于存储数据，Key为CacheKey，Value为查询结果
3. MyBatis的一级缓存查询的时候默认都是会先尝试从一级缓存中获取数据的，但是我们看第6行的代码做了一个判断，ms.isFlushCacheRequired()，即想==每次查询都走DB也行，将<select>标签中的flushCache属性设置为true即可==，这意味着每次查询的时候都会清理一遍PerpetualCache，PerpetualCache中没数据，自然只能走DB

#### 怎么样的查询条件算和上一次查询是一样的查询，从而返回同样的结果回去？

