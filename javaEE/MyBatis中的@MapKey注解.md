# MyBatis中的@MapKey注解

有时我们的一条查询语句返回了多个实体对象或Map集合

比如这样：

```java
List<User> users = abcDao.getNamesByIds(idList);
```

但我们在sql中这样让它返回

```java
Map<id, User> m = abcDao.getNamesByIds(idList);
```

那`ResultType`属性可以指定为`User`

并且在方法上加上注解

```java
@MapKey("id")
Map<id, User> m = abcDao.getNamesByIds(idList);
```

---

Mybatis官方文档的对该注解的解释