# Mybatis运行原理

首先来看这样一段代码

```java
public class TestMyBatis {
    // 1.需要引入mybatisjar包
    // 2.配置核心mybatis文件 数据源、mapper接口映射
    // 3.需要sqlmapper文件 sql数据 orm
    // 4.通过mybatis操作../
    // 疑问：你们在mybatis整合springboot之后需要在每个mapper 需要加入注入spring容器注解 这是为什么呢？
    // 疑问：Mapper如何调用的呢
    public static void main(String[] args) {
        try {
            // 基本mybatis环境
            // 1.定义mybatis_config文件地址
            String resources = "mybatis_config.xml";
            // 2.获取InputStreamReaderIo流
            Reader reader = Resources.getResourceAsReader(resources);
            // 3.获取SqlSessionFactory
            SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(reader);
            // 4.获取Session
            SqlSession sqlSession = sqlSessionFactory.openSession();
            // 5.操作Mapper接口
            UserMapper mapper = sqlSession.getMapper(UserMapper.class);
            UserEntity user = mapper.getUser(2);
            System.out.println(user.getName());
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

这段代码凡是学习过mybatis的人应该都很熟悉。

那我们就一步步学习下这段代码的底层运行原理

1. 创建SqlsessionFactory

`org.apache.ibatis.session.SqlSessionFactoryBuilder#build(java.io.Reader, java.lang.String, java.util.Properties)`

```java
  public SqlSessionFactory build(Reader reader, String environment, Properties properties) {
    try {
      XMLConfigBuilder parser = new XMLConfigBuilder(reader, environment, properties);
      return build(parser.parse());
    } catch (Exception e) {
      throw ExceptionFactory.wrapException("Error building SqlSession.", e);
    } finally {
      ErrorContext.instance().reset();
      try {
        reader.close();
      } catch (IOException e) {
        // Intentionally ignore. Prefer previous error.
      }
    }
  }
```

这里着重关注

**XMLConfigBuilder**的构造函数和parse方法

```java
  private XMLConfigBuilder(XPathParser parser, String environment, Properties props) {
    super(new Configuration());
    ErrorContext.instance().resource("SQL Mapper Configuration");
    this.configuration.setVariables(props);
    this.parsed = false;
    this.environment = environment;
    this.parser = parser;
  }
```

```java
  public Configuration parse() {
    if (parsed) {
      throw new BuilderException("Each XMLConfigBuilder can only be used once.");
    }
    parsed = true;
      //xml的根节点,步入进去
    parseConfiguration(parser.evalNode("/configuration"));
    return configuration;
  }
```



```java
  private void parseConfiguration(XNode root) {
    try {
      //issue #117 read properties first
      propertiesElement(root.evalNode("properties"));
      Properties settings = settingsAsProperties(root.evalNode("settings"));
      loadCustomVfs(settings);
      typeAliasesElement(root.evalNode("typeAliases"));
      pluginElement(root.evalNode("plugins"));
      objectFactoryElement(root.evalNode("objectFactory"));
      objectWrapperFactoryElement(root.evalNode("objectWrapperFactory"));
      reflectorFactoryElement(root.evalNode("reflectorFactory"));
      settingsElement(settings);
      // read it after objectFactory and objectWrapperFactory issue #631
      environmentsElement(root.evalNode("environments"));
      databaseIdProviderElement(root.evalNode("databaseIdProvider"));
      typeHandlerElement(root.evalNode("typeHandlers"));
      mapperElement(root.evalNode("mappers"));
    } catch (Exception e) {
      throw new BuilderException("Error parsing SQL Mapper Configuration. Cause: " + e, e);
    }
  }
```

我们在根节点Configuration下能写的标签这里依次都被解析。

最后一个是mapperElement是我们学习的重点。

```java
  private void mapperElement(XNode parent) throws Exception {
    if (parent != null) {
      for (XNode child : parent.getChildren()) {
        if ("package".equals(child.getName())) {
          String mapperPackage = child.getStringAttribute("name");
          configuration.addMappers(mapperPackage);
        } else {
          String resource = child.getStringAttribute("resource");
          String url = child.getStringAttribute("url");
          String mapperClass = child.getStringAttribute("class");
          if (resource != null && url == null && mapperClass == null) {
            ////进入该判断
            ErrorContext.instance().resource(resource);
            InputStream inputStream = Resources.getResourceAsStream(resource);
            XMLMapperBuilder mapperParser = new XMLMapperBuilder(inputStream, configuration, resource, configuration.getSqlFragments());
            mapperParser.parse();
          } else if (resource == null && url != null && mapperClass == null) {
            ErrorContext.instance().resource(url);
            InputStream inputStream = Resources.getUrlAsStream(url);
            XMLMapperBuilder mapperParser = new XMLMapperBuilder(inputStream, configuration, url, configuration.getSqlFragments());
            mapperParser.parse();
          } else if (resource == null && url == null && mapperClass != null) {
            Class<?> mapperInterface = Resources.classForName(mapperClass);
            configuration.addMapper(mapperInterface);
          } else {
            throw new BuilderException("A mapper element may only specify a url, resource or class, but not more than one.");
          }
        }
      }
    }
  }
```

可以发现这个mappers标签支持package、resource、url、class等方式去注册mapper。如果对Mybatis配置文件熟悉的话，应该明白。

