# Shiro入门（二）

## 一、添加依赖

```xml
<dependencies>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.9</version>
    </dependency>
    <dependency>
        <groupId>commons-logging</groupId>
        <artifactId>commons-logging</artifactId>
        <version>1.1.3</version>
    </dependency>
    <dependency>
        <groupId>org.apache.shiro</groupId>
        <artifactId>shiro-core</artifactId>
        <version>1.2.2</version>
    </dependency>
</dependencies>
```

## 二、配置文件shiro.ini

```ini
[users]
zhang=123
wang=123
```

这里添加了两个用户（也就是Subject主体），生产环境中，当然都是存储在数据库中的。

## 三、测试

```java

```

