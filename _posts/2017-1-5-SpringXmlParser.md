---
layout: post
title: XML解析
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
XmlBeanFactory xmlBeanFactory = new XmlBeanFactory(new ClassPathResource("service-context.xml"));
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

当xml文件完成ClassPathResource的封装后, 读取操作就交给XmlBeanDefinitionReader了, 代码如下(位于XmlBeanFactory.class中):

```
    this.reader = new XmlBeanDefinitionReader(this);
    this.reader.loadBeanDefinitions(resource);
```

进入loadBeanDefinitions(resource)后, 首先完成Resource到EncodedResource的封装以用于对资源文件的编码进行处理。
当指定了编码时, spring会使用相应的编码作为输入流的编码。两个代码片段如下:

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


[DomNodeType]: http://www.w3school.com.cn/xmldom/dom_nodetype.asp
[JavaWorld]: http://www.javaworld.com/article/2077344/core-java/find-a-way-out-of-the-classloader-maze.html


## 参考文献

1. http://www.cnblogs.com/davenkin/archive/2012/04/08/java-load-resources.html
2. http://docs.oracle.com/javase/7/docs/technotes/guides/lang/resources.html
3. http://blog.csdn.net/lhc1105/article/details/51354561

