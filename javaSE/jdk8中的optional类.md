# JDK8中的optional类

## 一、 案例



```java
@Data
public class User {
    private String name;
    private int age;
}
```

这里service层的业务逻辑可能返回Null

```java
public class UserService {
    public User getUserByName(String name) {
        return null;
    }
}
```

使用`Optional`类进行优化

```java
public class OptionalTest {
    public static void main(String[] args) {
        UserService userService = new UserService();
        User user = userService.getUserByName("张三");
        /**
         * jdk8以前的做法
         */
        if (user != null) {
            System.out.println("user = " + user);
        } else {
            System.out.println("为空的业务逻辑");
        }
        /**
         * 使用Optional
         * 与if ...else差别不大
         */
        Optional<User> optional = Optional.ofNullable(user);
        if (optional.isPresent()) {
            System.out.println("user不为null");
        } else {
            System.out.println("user为Null的业务逻辑");
        }

        /**
         * 如果optional中的user不为空，则会执行，否则不执行
         * 某些情况下，这样的逻辑刚好可用
         */
        optional.ifPresent((t -> {
            System.out.println("t = " + t);
        }));

        /**
         * 使用map方法，进行业务逻辑处理，
         */
        optional.map(t -> {
            // 业务逻辑处理
            return "user不为空";
        }).orElse(null);
    }
}
```

Java8中Optional的引入，使得开发避免了大量Null的出现，借助相关方法避免了if...else这种繁琐的逻辑代码编写，对于其应用，在处理空的场景下应用较多，对于ifelse的逻辑场景，同样使用Optional让程序更加简洁，同时使用Optional可以实现代码的链式处理。

