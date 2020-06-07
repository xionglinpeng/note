# Mybatis缓存

## Mybatis缓存的使用





```xml
    <settings>
        <setting name="cacheEnabled" value="true"/>
    </settings>
```





## Mybatis缓存实现类



*Cache接口源码如下：*

```java
public interface Cache {

  String getId();

  void putObject(Object key, Object value);

  Object getObject(Object key);

  Object removeObject(Object key);

  void clear();

  int getSize();

  default ReadWriteLock getReadWriteLock() {
    return null;
  }

}
```

