---
layout: post
title: XML解析成Document
tags:  [JAVA]
categories: [JAVA]
author: liheng
excerpt: "XML解析"
---
## 文档对象模型(Document Object Model)

DOM(abbr. DOM)是一个标准的树结构, 其中的每个结点包含一个XML文档中的一个组件。
其中, 最广泛使用的[结点类型][DomNodeType]为 元素结点 和 文本结点。 
使用DOM相关的函数可以创建、删除、遍历结点 和改变结点的内容等。

### XML解析为DOM

xml文件加载代码如下:

```
XmlBeanFactory xmlBeanFactory=new XmlBeanFactory(new ClassPathResource("service-context.xml"));
```

#### ClassPathResource

Spring 配置文件的读取是通过ClassPathResource封装的。
利用ClassPathResource读取xml配置就是通过构造函数传入的文件路径，接着交给clazz或者classLoader，调用getResourceAsStream获取到InputStream。
构造函数如下:

```
public class ClassPathResource extends AbstractFileResolvingResource {
    private final String path;
    private ClassLoader classLoader;
    private Class<?> clazz;

    public ClassPathResource(String path) {
        this(path, (ClassLoader)null);
    }

    public ClassPathResource(String path, ClassLoader classLoader) {
        Assert.notNull(path, "Path must not be null");
        String pathToUse = StringUtils.cleanPath(path);
        if(pathToUse.startsWith("/")) {
            pathToUse = pathToUse.substring(1);
        }

        this.path = pathToUse;
        this.classLoader = classLoader != null?classLoader:ClassUtils.getDefaultClassLoader();
    }
    
    ... 
```    

classLoader则是解析resource名称加载资源文件的。
另外, resource名称, 以"/"开头则为绝对名称(这类文件名称, 不需要任何修改, 直接使用ClassLoader下的方法来定位resource文件),
否则为相对名称(该类文件名称需要经过适当地修改,之后调用ClassLoader下的方法)。
resource名称的解析具体涉及到两种方法:

1.使用java.lang.Class中的方法 - 
使用getResource()和getResourceAsStream()来根据resource名称来定位资源。
若没有根据指定的名称定位到资源, 则返回null。当完成resource名称的转换后(规则:
若resource名称以"/"开头, 则什么都不做; 否则在resource名称前添加该类所在包的名称,然后将所有的"."转换为"/"),
Class类中的方法作为ClassLoader类中方法的代理, 将会得到执行。

```
     public InputStream getResourceAsStream(String name) {
        name = resolveName(name);
        ClassLoader cl = getClassLoader0();
        if (cl==null) {
            // A system class.
            return ClassLoader.getSystemResourceAsStream(name);
        }
        return cl.getResourceAsStream(name);
    }
```

2.使用java.lang.ClassLoader中的方法 - 
ClassLoader中的方法直接使用resource名称来定位资源, 同时不会对resource名称进行绝对/相对转换(absolute/relative transformation)。
而且, resource名称不能以"/"开头。

```
    public InputStream getResourceAsStream(String name) {
        URL url = getResource(name);
        try {
            return url != null ? url.openStream() : null;
        } catch (IOException e) {
            return null;
        }
    }
```

回到ClassPathResource(String, ClassLoader)构造函数中, 此时默认的是使用ClassLoader来定位resource名称。
获取classLoader的方法如下:

```    
    public static ClassLoader getDefaultClassLoader() {
        ClassLoader cl = null;

        try {
            //①当前线程的classLoader
            cl = Thread.currentThread().getContextClassLoader();
        } catch (Throwable var3) {
            ;
        }

        if(cl == null) {
            //② 加载此类的classLoader
            cl = ClassUtils.class.getClassLoader();
            if(cl == null) {
                try {
                    //③系统使用的classLoader, Application ClassLoader
                    cl = ClassLoader.getSystemClassLoader();
                } catch (Throwable var2) {
                    ;
                }
            }
        }

        return cl;
    }
```    

此处获取classLoader的地方有①②③三处, 它们的区别参考[JavaWorld][JavaWorld].

#### XmlBeanDefinitionReader

当xml文件完成ClassPathResource的封装后, 读取操作就交给XmlBeanDefinitionReader了。
然后创建XmlBeanFactory的实例并初始化一些变量, 
然后作为给XmlBeanDefinitionReader的BeanDefinitionRegistry registry参数赋值构造XmlBeanDefinitionReader实例。
代码如下(位于XmlBeanFactory.class中):

```
    private final BeanDefinitionRegistry registry;

    protected AbstractBeanDefinitionReader(BeanDefinitionRegistry registry) {
        Assert.notNull(registry, "BeanDefinitionRegistry must not be null");
        this.registry = registry;
        if(this.registry instanceof ResourceLoader) {
            this.resourceLoader = (ResourceLoader)this.registry;
        } else {
            this.resourceLoader = new PathMatchingResourcePatternResolver();
        }

        if(this.registry instanceof EnvironmentCapable) {
            this.environment = ((EnvironmentCapable)this.registry).getEnvironment();
        } else {
            this.environment = new StandardEnvironment();
        }

    }
```

```
    this.reader = new XmlBeanDefinitionReader(this);
    this.reader.loadBeanDefinitions(resource);
```

进入loadBeanDefinitions(resource)后, 首先完成Resource到EncodedResource的封装以用于对资源文件的编码进行处理。
当指定了编码时, 之后spring将会使用相应的编码作为输入流的编码。两个代码片段如下:

```
    return this.loadBeanDefinitions(new EncodedResource(resource));
```

```
    public Reader getReader() throws IOException {
        return this.charset != null?
        new InputStreamReader(this.resource.getInputStream(), this.charset):
        (this.encoding != null?
            new InputStreamReader(this.resource.getInputStream(), this.encoding)
                :new InputStreamReader(this.resource.getInputStream()));
    }
```

```
    public InputStream getInputStream() throws IOException {
        InputStream is;
        if(this.clazz != null) {
            is = this.clazz.getResourceAsStream(this.path);
        } else if(this.classLoader != null) {
            is = this.classLoader.getResourceAsStream(this.path);
        } else {
            is = ClassLoader.getSystemResourceAsStream(this.path);
        }

        if(is == null) {
            throw new FileNotFoundException(this.getDescription() + " cannot be opened because it does not exist");
        } else {
            return is;
        }
    }
```

之后进入loadBeanDefinitions(EncodedResource)方法, 
然后将EncodedResource封装到[InputSource类][InputSource], 以便之后使用SAX方式来解析XML文件。
另外该方法的核心代码如下:

```
    InputStream ex = encodedResource.getResource().getInputStream();

    try {
        InputSource inputSource = new InputSource(ex);
        if(encodedResource.getEncoding() != null) {
            inputSource.setEncoding(encodedResource.getEncoding());
        }

            var5 = this.doLoadBeanDefinitions(inputSource, encodedResource.getResource());
        } finally {
            ex.close();
        }
```

之后, 进入doLoadBeanDefinitions(InputSource, Resource)方法, 将xml解析成DOM文件以及注册相应的Bean, 其核心代码片段如下:

```
    Document ex = this.doLoadDocument(InputSource, Resource);
    return this.registerBeanDefinitions(ex, resource);
```

本文暂且关注XML解析成DOM的情况, 调用doLoadDocument(InputSource, Resource)则进入:

```
    protected Document doLoadDocument(InputSource inputSource, Resource resource)
      throws Exception {
        return this.documentLoader.loadDocument(inputSource, this.getEntityResolver(),
                this.errorHandler, this.getValidationModeForResource(resource), this.isNamespaceAware());
    }
```


#### DefaultDocumentLoader

接下来进入DefaultDocumentLoader的loadDocument(InputSource, EntityResolver, ErrorHandler, int, boolean)方法。
首先, 创建DocumentBuilderFactory实例, 查看创建DocumentBuilderFactory实例的源码会发现DocumentBuilderFactory为abstract,
所以对于这样一个抽象类, 其实最后返回的是默认的com.sun.org.apache.xerces.internal.jaxp.DocumentBuilderFactoryImpl实例(DocumentBuilderFactory类中)。
然后, 使用DocumentBuilder来将xml解析为Document。相应的代码片段如下:

```
    return FactoryFinder.find(
                /* The default property name according to the JAXP spec */
                DocumentBuilderFactory.class, // "javax.xml.parsers.DocumentBuilderFactory"
                /* The fallback implementation class name */
                "com.sun.org.apache.xerces.internal.jaxp.DocumentBuilderFactoryImpl");
```

```
    public Document loadDocument(InputSource inputSource, EntityResolver entityResolver,
     ErrorHandler errorHandler, int validationMode, boolean namespaceAware) throws Exception {
        DocumentBuilderFactory factory 
                    = this.createDocumentBuilderFactory(validationMode, namespaceAware);
        if(logger.isDebugEnabled()) {
            logger.debug("Using JAXP provider [" + factory.getClass().getName() + "]");
        }

        DocumentBuilder builder = this.createDocumentBuilder(factory, entityResolver, errorHandler);
        return builder.parse(inputSource);
    }
```


[DomNodeType]: http://www.w3school.com.cn/xmldom/dom_nodetype.asp
[JavaWorld]: http://www.javaworld.com/article/2077344/core-java/find-a-way-out-of-the-classloader-maze.html
[InputSource]: http://docs.oracle.com/javase/7/docs/api/org/xml/sax/InputSource.html

## 参考文献

1. http://www.cnblogs.com/davenkin/archive/2012/04/08/java-load-resources.html
2. http://docs.oracle.com/javase/7/docs/technotes/guides/lang/resources.html
3. http://blog.csdn.net/lhc1105/article/details/51354561

