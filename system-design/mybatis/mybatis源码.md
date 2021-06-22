# mybatis 源码分析

mybatis 也提供了对缓存的支持，分为：

- 一级缓存
- 二级缓存

![img](https://filescdn.proginn.com/2fb3b2f2085549b531582047ceb52567/402891529098f36d72a2520f93025baa.webp)

## 一级缓存

​		订单表与会员表是存在一对多的关系 为了尽可能减少join 查询，进行了分阶段查询，即先查询出订单表，在根据member_id 字段查询出会员表，最后进行数据整合 。如果订单表中存在重复的member_id，就会出现很多没必要的重复查询。

   	 针对这种情况myBatis 通过缓存来实现，在同一次查询会话中如果出现相同的语句及参数，就会从缓存中取出不在走数据库查询。一级缓存只能作用于查询会话中 所以也叫做会话缓存。
   	 一级缓存流程：
   	 
   	(1)对于某个查询，根据statementId,params,rowBounds来构建一个key值，根据这个key值去缓存Cache中取出对应的key值存储的缓存结果
   	(2)判断从Cache中根据特定的key值取的数据是否为空，即是否命中；
   	(3)如果命中，则直接将缓存结果返回；
   	(4)如果没命中, 去数据库中查询数据，得到查询结果；
   	                将key和查询到的结果分别作为key,value对存储到Cache中；
   	                将查询结果返回；

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

private  List queryFromDatabase(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler, CacheKey key, BoundSql boundSql) throws SQLException {
    List list;
    //1. 把key存入缓存，value放一个占位符
 localCache.putObject(key, EXECUTION_PLACEHOLDER);
    try {
      //2. 与数据库交互
      list = doQuery(ms, parameter, rowBounds, resultHandler, boundSql);
    } finally {
      //3. 如果第2步出了什么异常，把第1步存入的key删除
      localCache.removeObject(key);
    }
      //4. 把结果存入缓存
    localCache.putObject(key, list);
    if (ms.getStatementType() == StatementType.CALLABLE) {
      localOutputParameterCache.putObject(key, parameter);
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

==**一级缓存的CacheKey**==

```java
public class CacheKey implements Cloneable, Serializable {

    public static final CacheKey NULL_CACHE_KEY = new NullCacheKey();

    private static final int DEFAULT_MULTIPLYER = 37;
    private static final int DEFAULT_HASHCODE = 17;
    // 乘子，默认为37
    private final int multiplier;
    // CacheKey 的 hashCode，综合了各种影响因子
    private int hashcode;
    // 校验和
    private long checksum;
    // 影响因子个数
    private int count;
    // 影响因子集合
    private List<Object> updateList;
    
    public CacheKey() {
        this.hashcode = DEFAULT_HASHCODE;
        this.multiplier = DEFAULT_MULTIPLYER;
        this.count = 0;
        this.updateList = new ArrayList<Object>();
    }

    /*
     * 每当执行更新操作时，表示有新的影响因子参与计算 
     *  当不断有新的影响因子参与计算时，hashcode 和 checksum 将会变得愈发复杂和随机。这样可降低冲突率，使 CacheKey 可在缓存中更均匀的分布。
     */ 
    public void update(Object object) {
        int baseHashCode = object == null ? 1 : ArrayUtil.hashCode(object); 
        // 自增 count
        count++;
        // 计算校验和
        checksum += baseHashCode;
        // 更新 baseHashCode
        baseHashCode *= count;
        // 计算 hashCode
        hashcode = multiplier * hashcode + baseHashCode;
        // 保存影响因子
        updateList.add(object);
    }   

    /**
     *  CacheKey 最终要作为键存入 HashMap，因此它需要覆盖 equals 和 hashCode 方法
     */ 
    @Override
    public boolean equals(Object object) {
        // 检测是否为同一个对象
        if (this == object) {
            return true;
        }
        // 检测 object 是否为 CacheKey
        if (!(object instanceof CacheKey)) {
            return false;
        }
        
        final CacheKey cacheKey = (CacheKey) object;
        // 检测 hashCode 是否相等
        if (hashcode != cacheKey.hashcode) {
            return false;
        }
        // 检测校验和是否相同
        if (checksum != cacheKey.checksum) {
            return false;
        }
        // 检测 coutn 是否相同
        if (count != cacheKey.count) {
            return false;
        }
        // 如果上面的检测都通过了，下面分别对每个影响因子进行比较
        for (int i = 0; i < updateList.size(); i++) {
            Object thisObject = updateList.get(i);
            Object thatObject = cacheKey.updateList.get(i);
            if (!ArrayUtil.equals(thisObject, thatObject)) {
                return false;
            }
        }
        return true;
    }   
    
    @Override
    public int hashCode() {
        // 返回 hashcode 变量
        return hashcode;
    }   
}
```

```java
public CacheKey createCacheKey(MappedStatement ms, Object parameterObject, RowBounds rowBounds, BoundSql boundSql) {
        if (super.isClosed()) {
            throw new ExecutorException("Executor was closed.");
        }
    	 //初始化CacheKey
        CacheKey cacheKey = new CacheKey();
    	//存入statementId
        cacheKey.update(ms.getId());
    	//分别存入分页需要的Offset和Limit
        Object offset = rowBounds.getOffset();
        Object limit = rowBounds.getLimit();
    	 //把从BoundSql中封装的sql取出并存入到cacheKey对象中
        if (parameterObject instanceof Map) {
            Map<?, ?> parameterMap = (Map<?, ?>) parameterObject;
            Optional<? extends Map.Entry<?, ?>> optional = parameterMap.entrySet().stream().filter(entry -> entry.getValue() instanceof IPage).findFirst();
            if (optional.isPresent()) {
                IPage<?> page = (IPage) optional.get().getValue();
                offset = page.getCurrent();
                limit = page.getSize();
                //折磨人的小妖精,只能当缓存条件了,避免缓存错误命中.
                cacheKey.update(page.isSearchCount());
            }
        } else if (parameterObject instanceof IPage) {
            IPage<?> page = (IPage) parameterObject;
            offset = page.getCurrent();
            limit = page.getSize();
            cacheKey.update(page.isSearchCount());
        }
        cacheKey.update(offset);
        cacheKey.update(limit);
        cacheKey.update(boundSql.getSql());
    	 //下面这一块就是封装参数
        List<ParameterMapping> parameterMappings = boundSql.getParameterMappings();
        TypeHandlerRegistry typeHandlerRegistry = ms.getConfiguration().getTypeHandlerRegistry();
        // mimic DefaultParameterHandler logic
        for (ParameterMapping parameterMapping : parameterMappings) {
            if (parameterMapping.getMode() != ParameterMode.OUT) {
                Object value;
                String propertyName = parameterMapping.getProperty();
                if (boundSql.hasAdditionalParameter(propertyName)) {
                    value = boundSql.getAdditionalParameter(propertyName);
                } else if (parameterObject == null) {
                    value = null;
                } else if (typeHandlerRegistry.hasTypeHandler(parameterObject.getClass())) {
                    value = parameterObject;
                } else {
                    MetaObject metaObject = configuration.newMetaObject(parameterObject);
                    value = metaObject.getValue(propertyName);
                }
                cacheKey.update(value);
            }
        }
     	//从configuration对象中（也就是载入配置文件后存放的对象）把EnvironmentId存入
        /**
         *     "development">
         *         "development"> //就是这个id
         *             
         *             type="JDBC">
         *             
         *             type="POOLED">
         *                 "driver" value="${jdbc.driver}"/>
         *                 "url" value="${jdbc.url}"/>
         *                 "username" value="${jdbc.username}"/>
         *                 "password" value="${jdbc.password}"/>
         *             
         *         
         *     
         */
        if (configuration.getEnvironment() != null) {
            // issue #176
            cacheKey.update(configuration.getEnvironment().getId());
        }
        return cacheKey;
    }
```

### 总结

1. MyBatis一级缓存的生命周期和SqlSession一致。
2. MyBatis一级缓存内部设计简单，只是一个没有容量限定的HashMap，在缓存的功能性上有所欠缺。
3. MyBatis的一级缓存最大范围是SqlSession内部，有多个SqlSession或者分布式的环境下，数据库写操作会引起脏数据，建议设定缓存级别为Statement。

## 二级缓存：

一级缓存中，其最大的共享范围就是一个SqlSession内部，如果多个SqlSession之间需要共享缓存，则需要使用到二级缓存。开启二级缓存后，会使用CachingExecutor装饰Executor，进入一级缓存的查询流程前，先在CachingExecutor进行二级缓存的查询，具体的工作流程如下所示。

![img](https://awps-assets.meituan.net/mit-x/blog-images-bundle-2018a/28399eba.png)

注意二级缓存是从 MappedStatement 中获取的。由于 MappedStatement 存在于全局配置中，可以多个 CachingExecutor 获取到，这样就会出现线程安全问题。除此之外，若不加以控制，多个事务共用一个缓存实例，会导致脏读问题。至于脏读问题，需要借助其他类来处理，也就是上面代码中 tcm 变量对应的类型。

### TransactionalCacheManager

``` java

/*
 * 事务缓存管理器 
 */
public class TransactionalCacheManager {

    // Cache 与 TransactionalCache 的映射关系表
    private final Map<Cache, TransactionalCache> transactionalCaches = new HashMap<Cache, TransactionalCache>();
    
    public void clear(Cache cache) {
        // 获取 TransactionalCache 对象，并调用该对象的 clear 方法，下同
        getTransactionalCache(cache).clear();
    }
    
    public Object getObject(Cache cache, CacheKey key) {
        // 直接从TransactionalCache中获取缓存
        return getTransactionalCache(cache).getObject(key);
    }
    
    public void putObject(Cache cache, CacheKey key, Object value) {
        // 直接存入TransactionalCache的缓存中
        getTransactionalCache(cache).putObject(key, value);
    }
 
    public void commit() {
        for (TransactionalCache txCache : transactionalCaches.values()) {
            txCache.commit();
        }
    }

    public void rollback() {
        for (TransactionalCache txCache : transactionalCaches.values()) {
            txCache.rollback();
        }
    }
    
    private TransactionalCache getTransactionalCache(Cache cache) {
        // 从映射表中获取 TransactionalCache
        TransactionalCache txCache = transactionalCaches.get(cache);
        if (txCache == null) {
            // TransactionalCache 也是一种装饰类，为 Cache 增加事务功能
            // 创建一个新的TransactionalCache，并将真正的Cache对象存进去     
            txCache = new TransactionalCache(cache);
            transactionalCaches.put(cache, txCache);
        }
        return txCache;
    }

}
```

TransactionalCacheManager 内部维护了 Cache 实例与 TransactionalCache 实例间的映射关系，该类也仅负责维护两者的映射关系，真正做事的还是 TransactionalCache。TransactionalCache 是一种缓存装饰器，可以为 Cache 实例增加事务功能。我在之前提到的脏读问题正是由该类进行处理的。下面分析一下该类的逻辑。

### TransactionalCache

```java
public class TransactionalCache implements Cache {
    
    //真正的缓存对象，和上面的Map<Cache, TransactionalCache>中的Cache是同一个
    private final Cache delegate;
    private boolean clearOnCommit;
    // 在事务被提交前，所有从数据库中查询的结果将缓存在此集合中
    private final Map<Object, Object> entriesToAddOnCommit;
    // 在事务被提交前，当缓存未命中时，CacheKey 将会被存储在此集合中
    private final Set<Object> entriesMissedInCache;

    public TransactionalCache(Cache delegate) {
        this.delegate = delegate;
        this.clearOnCommit = false;
        this.entriesToAddOnCommit = new HashMap<Object, Object>();
        this.entriesMissedInCache = new HashSet<Object>();
    }
    
    @Override
    public Object getObject(Object key) {
        // 查询的时候是直接从delegate中去查询的，也就是从真正的缓存对象中查询
        Object object = delegate.getObject(key);
        if (object == null) {
            // 缓存未命中，则将 key 存入到 entriesMissedInCache 中
            entriesMissedInCache.add(key);
        }
        if (clearOnCommit) {
            return null;
        } else {
            return object;
        }
    }
    
    @Override
    public void putObject(Object key, Object object) {
        // 将键值对存入到 entriesToAddOnCommit 这个Map中中，而非真实的缓存对象 delegate 中
        entriesToAddOnCommit.put(key, object);
    }
    
    @Override
    public Object removeObject(Object key) {
        return null;
    }
    
    @Override
    public void clear() {
        clearOnCommit = true;
        // 清空 entriesToAddOnCommit，但不清空 delegate 缓存
        entriesToAddOnCommit.clear();
    }
    
    public void commit() {
        // 根据 clearOnCommit 的值决定是否清空 delegate
        if (clearOnCommit) {
            delegate.clear();
        }
        // 刷新未缓存的结果到 delegate 缓存中
        flushPendingEntries();
        // 重置 entriesToAddOnCommit 和 entriesMissedInCache
        reset();
    }

    public void rollback() {
        unlockMissedEntries();
        reset();
    }
    
    private void reset() {
        clearOnCommit = false;
        // 清空集合
        entriesToAddOnCommit.clear();
        entriesMissedInCache.clear();
    }
    
    private void flushPendingEntries() {
        for (Map.Entry<Object, Object> entry : entriesToAddOnCommit.entrySet()) {
            // 将 entriesToAddOnCommit 中的内容转存到 delegate 中
            delegate.putObject(entry.getKey(), entry.getValue());
        }
        for (Object entry : entriesMissedInCache) {
            if (!entriesToAddOnCommit.containsKey(entry)) {
                // 存入空值
                delegate.putObject(entry, null);
            }
        }
    }
    
    private void unlockMissedEntries() {
        for (Object entry : entriesMissedInCache) {
            try {
                // 调用 removeObject 进行解锁
                delegate.removeObject(entry);
            } catch (Exception e) {
                log.warn("Unexpected exception while notifiying a rollback to the cache adapter."
                    + "Consider upgrading your cache adapter to the latest version.  Cause: " + e);
            }
        }
    }

}
```

存储二级缓存对象的时候是放到了TransactionalCache.entriesToAddOnCommit这个map中，但是每次查询的时候是直接从TransactionalCache.delegate中去查询的，所以这个二级缓存查询数据库后，设置缓存值是没有立刻生效的，主要是因为直接存到 delegate 会导致脏数据问题。

### 为何只有SqlSession提交或关闭之后二级缓存才会生效？

###### SqlSession



```java
public class DefaultSqlSession implements SqlSession {

    private final Configuration configuration;
    private final Executor executor;

    @Override
    public void commit(boolean force) {
        try {
            // 主要是这句
            executor.commit(isCommitOrRollbackRequired(force));
            dirty = false;
        } catch (Exception e) {
            throw ExceptionFactory.wrapException("Error committing transaction.  Cause: " + e, e);
        } finally {
            ErrorContext.instance().reset();
        }
    }
}


public class CachingExecutor implements Executor {

    private final Executor delegate;
    private final TransactionalCacheManager tcm = new TransactionalCacheManager();
    
    @Override
    public void commit(boolean required) throws SQLException {
        delegate.commit(required);
        tcm.commit();// 在这里
    }
}

public class TransactionalCacheManager {

    private final Map<Cache, TransactionalCache> transactionalCaches = new HashMap<Cache, TransactionalCache>();
    
    public void commit() {
        for (TransactionalCache txCache : transactionalCaches.values()) {
            txCache.commit();// 在这里
        }
    }
}

public class TransactionalCache implements Cache {

    private final Cache delegate;

    public void commit() {
        if (clearOnCommit) {
            delegate.clear();
        }
        flushPendingEntries();//这一句
        reset();
    }
    
    private void flushPendingEntries() {
        for (Map.Entry<Object, Object> entry : entriesToAddOnCommit.entrySet()) {
            // 在这里真正的将entriesToAddOnCommit的对象逐个添加到delegate中，只有这时，二级缓存才真正的生效
            delegate.putObject(entry.getKey(), entry.getValue());
        }
        for (Object entry : entriesMissedInCache) {
            if (!entriesToAddOnCommit.containsKey(entry)) {
                delegate.putObject(entry, null);
            }
        }
    }
}
```

如果从数据库查询到的数据直接存到 delegate 会导致脏数据问题。下面通过一张图演示一下脏数据问题发生的过程，假设两个线程开启两个不同的事务，它们的执行过程如下：

![img](https:////upload-images.jianshu.io/upload_images/13587608-9d2dcaafca191ea4.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

如上图，时刻2，事务 A 对记录 A 进行了更新。时刻3，事务 A 从数据库查询记录 A，并将记录 A 写入缓存中。时刻4，事务 B 查询记录 A，由于缓存中存在记录 A，事务 B 直接从缓存中取数据。这个时候，脏数据问题就发生了。事务 B 在事务 A 未提交情况下，读取到了事务 A 所修改的记录。为了解决这个问题，我们可以为每个事务引入一个独立的缓存。查询数据时，仍从 delegate 缓存（以下统称为共享缓存）中查询。若缓存未命中，则查询数据库。存储查询结果时，并不直接存储查询结果到共享缓存中，而是先存储到事务缓存中，也就是 entriesToAddOnCommit 集合。当事务提交时，再将事务缓存中的缓存项转存到共享缓存中。这样，事务 B 只能在事务 A 提交后，才能读取到事务 A 所做的修改，解决了脏读问题。

### 二级缓存的刷新

我们来看看SqlSession的更新操作

```java
public class DefaultSqlSession implements SqlSession {

    private final Configuration configuration;
    private final Executor executor;

    @Override
    public int update(String statement, Object parameter) {
        try {
            dirty = true;
            MappedStatement ms = configuration.getMappedStatement(statement);
            return executor.update(ms, wrapCollection(parameter));
        } catch (Exception e) {
            throw ExceptionFactory.wrapException("Error updating database.  Cause: " + e, e);
        } finally {
            ErrorContext.instance().reset();
        }
    }
}

public class CachingExecutor implements Executor {

    private final Executor delegate;
    private final TransactionalCacheManager tcm = new TransactionalCacheManager();

    @Override
    public int update(MappedStatement ms, Object parameterObject) throws SQLException {
        flushCacheIfRequired(ms);
        return delegate.update(ms, parameterObject);
    }
    
    private void flushCacheIfRequired(MappedStatement ms) {
        //获取MappedStatement对应的Cache，进行清空
        Cache cache = ms.getCache();
        //SQL需设置flushCache="true" 才会执行清空
        if (cache != null && ms.isFlushCacheRequired()) {      
            tcm.clear(cache);
        }
    }
}
```

MyBatis二级缓存只适用于不常进行增、删、改的数据，比如国家行政区省市区街道数据。一但数据变更，MyBatis会清空缓存。因此二级缓存不适用于经常进行更新的数据。

### 总结

1. MyBatis的二级缓存相对于一级缓存来说，实现了`SqlSession`之间缓存数据的共享，同时粒度更加的细，能够到`namespace`级别，通过Cache接口实现类不同的组合，对Cache的可控性也更强。
2. MyBatis在多表查询时，极大可能会出现脏数据，有设计上的缺陷，安全使用二级缓存的条件比较苛刻。
3. 在分布式环境下，由于默认的MyBatis Cache实现都是基于本地的，分布式环境下必然会出现读取到脏数据，需要使用集中式缓存将MyBatis的Cache接口实现，有一定的开发成本，直接使用Redis、Memcached等分布式缓存可能成本更低，安全性也更高。