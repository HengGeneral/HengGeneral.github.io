---
layout: post
title: Bean的提取和注册
tags:  [JAVA]
categories: [JAVA]
author: liheng
excerpt: "Bean"
---
## Bean的提取和注册

当将XML转换为Document后, 接下来就是调用registerBeanDefinitions(Document, EncodedResource)提取和注册bean了。

#### XmlBeanDefinitionReader

该方法的代码如下文所示, 其中doc则是[之前][XML2DOC]生成的Document的实例, resource参数则是EncodedResource实例。
其中, 从[之前][XML2DOC]中代码可以知道, this.getRegistry()返回的则是之前创建的XmlBeanFactory实例。

```
   public int registerBeanDefinitions(Document doc, Resource resource)
    throws BeanDefinitionStoreException {
        BeanDefinitionDocumentReader documentReader = this.createBeanDefinitionDocumentReader();
        int countBefore = this.getRegistry().getBeanDefinitionCount();
        documentReader.registerBeanDefinitions(doc, this.createReaderContext(resource));
        return this.getRegistry().getBeanDefinitionCount() - countBefore;
    }
```    

```
    //位于XmlBeanFactory的父类DefaultListableBeanFactory中
    private final Map<String, BeanDefinition> beanDefinitionMap = new ConcurrentHashMap(256);
    
    public int getBeanDefinitionCount() {
        return this.beanDefinitionMap.size();
    }
```

```
    private ProblemReporter problemReporter = new FailFastProblemReporter();
    private ReaderEventListener eventListener = new EmptyReaderEventListener();
    private SourceExtractor sourceExtractor = new NullSourceExtractor();
    
    public XmlReaderContext createReaderContext(Resource resource) {
        return new XmlReaderContext(resource,
                                    this.problemReporter,
                                    this.eventListener,
                                    this.sourceExtractor,
                                    this,
                                    this.getNamespaceHandlerResolver());
    }
    
    public NamespaceHandlerResolver getNamespaceHandlerResolver() {
        if(this.namespaceHandlerResolver == null) {
            this.namespaceHandlerResolver = this.createDefaultNamespaceHandlerResolver();
        }

        return this.namespaceHandlerResolver;
    }    
```

#### DefaultBeanDefinitionDocumentReader

之后, 会进入registerBeanDefinitions(Document, XmlReaderContext)方法中对readerContext进行赋值, 
然后获取Document的根元素root, 基于root完成bean定义的注册, 代码如下:

```
    public void registerBeanDefinitions(Document doc, XmlReaderContext readerContext) {
        this.readerContext = readerContext;
        this.logger.debug("Loading bean definitions");
        Element root = doc.getDocumentElement();
        this.doRegisterBeanDefinitions(root);
    }
```

接下来,就真正地进入Bean的解析阶段了, 进入doRegisterBeanDefinitions(Element)方法,
其中 delegate 为代理BeanDefinitionParserDelegate实例,用于代理xml的解析;
readerContext则是上文创建的XmlReaderContext实例, parent为null。 代码如下:

```
protected void doRegisterBeanDefinitions(Element root) {
    BeanDefinitionParserDelegate parent = this.delegate;
    this.delegate = this.createDelegate(this.getReaderContext(), root, parent);
        
    //判断xmlns是否为空或者为"http://www.springframework.org/schema/beans"
    if(this.delegate.isDefaultNamespace(root)) { 
        String profileSpec = root.getAttribute("profile"); //beans属性profile
        if(StringUtils.hasText(profileSpec)) {
            String[] specifiedProfiles = StringUtils.tokenizeToStringArray(profileSpec, ",; ");
            if(!this.getReaderContext().getEnvironment().acceptsProfiles(specifiedProfiles)) {
                return;
            }
        }
    }

    this.preProcessXml(root);
    this.parseBeanDefinitions(root, this.delegate);
    this.postProcessXml(root);
    this.delegate = parent;
}

protected BeanDefinitionParserDelegate createDelegate(XmlReaderContext readerContext,
 Element root, BeanDefinitionParserDelegate parentDelegate) {
    BeanDefinitionParserDelegate delegate = new BeanDefinitionParserDelegate(readerContext);
    delegate.initDefaults(root, parentDelegate);
    return delegate;
}

protected final XmlReaderContext getReaderContext() {
    return this.readerContext;
}
```

其中, preProcessXml(Element)和postProcessXml(Element) 都是空方法, 它们的作用可以查看[官网][PrePostProcessXML]。
接下来, 进入parseBeanDefinitions(Element, BeanDefinitionParserDelegate)方法, 代码如下:

```
        if(delegate.isDefaultNamespace(root)) {
            NodeList nl = root.getChildNodes();

            for(int i = 0; i < nl.getLength(); ++i) {
                Node node = nl.item(i);
                if(node instanceof Element) {
                    Element ele = (Element)node;
                    if(delegate.isDefaultNamespace(ele)) {
                        this.parseDefaultElement(ele, delegate);//解析xml中4种默认标签
                    } else {
                        delegate.parseCustomElement(ele);
                    }
                }
            }
        } else {
            delegate.parseCustomElement(root);
        }
```

当遇到4种默认标签(import, alias, bean, beans)时, 则进入parseDefaultElement(Element, BeanDefinitionParserDelegate)方法, 其源码如下:

```
private void parseDefaultElement(Element ele, BeanDefinitionParserDelegate delegate) {
   if(delegate.nodeNameEquals(ele, "import")) {
         this.importBeanDefinitionResource(ele);
   } else if(delegate.nodeNameEquals(ele, "alias")) {
        this.processAliasRegistration(ele);
   } else if(delegate.nodeNameEquals(ele, "bean")) {
        this.processBeanDefinition(ele, delegate);
   } else if(delegate.nodeNameEquals(ele, "beans")) {
        this.doRegisterBeanDefinitions(ele);
   }
}
```

我们先以bean标签为例, 来看看processBeanDefinition(Element, BeanDefinitionParserDelegate),
 
代码如下:

```
protected void processBeanDefinition(Element ele, BeanDefinitionParserDelegate delegate) {
    BeanDefinitionHolder bdHolder = delegate.parseBeanDefinitionElement(ele);
    if(bdHolder != null) {
        bdHolder = delegate.decorateBeanDefinitionIfRequired(ele, bdHolder);

        try {
            BeanDefinitionReaderUtils.registerBeanDefinition(bdHolder,
                this.getReaderContext().getRegistry());
        } catch (BeanDefinitionStoreException var5) {
            this.getReaderContext()
                .error("Failed to register bean definition with name \'"
                    + bdHolder.getBeanName() + "\'", ele, var5);
        }

        this.getReaderContext().fireComponentRegistered(new BeanComponentDefinition(bdHolder));
    }

}
```

#### BeanDefinitionParserDelegate
我们先来看看parseBeanDefinitionElement(Element)方法, 代码如下:

```
public BeanDefinitionHolder parseBeanDefinitionElement(Element ele) {
    return this.parseBeanDefinitionElement(ele, (BeanDefinition)null);
}


public BeanDefinitionHolder parseBeanDefinitionElement(Element ele, BeanDefinition containingBean) {
        String id = ele.getAttribute("id");
        String nameAttr = ele.getAttribute("name");
        ArrayList aliases = new ArrayList();
        if(StringUtils.hasLength(nameAttr)) {
            String[] beanName = StringUtils.tokenizeToStringArray(nameAttr, ",; ");
            aliases.addAll(Arrays.asList(beanName));
        }

        String beanName1 = id;
        if(!StringUtils.hasText(id) && !aliases.isEmpty()) {
            beanName1 = (String)aliases.remove(0);
            if(this.logger.isDebugEnabled()) {
                this.logger.debug(
                    "No XML \'id\' specified - using \'" + beanName1 + "\' as bean name and " + aliases + " as aliases");
            }
        }

        if(containingBean == null) {
            this.checkNameUniqueness(beanName1, aliases, ele);
        }

        AbstractBeanDefinition beanDefinition = this.parseBeanDefinitionElement(ele, beanName1, containingBean);
        if(beanDefinition != null) {
            if(!StringUtils.hasText(beanName1)) {
                try {
                    if(containingBean != null) {
                        beanName1 = BeanDefinitionReaderUtils
                            .generateBeanName(beanDefinition, this.readerContext.getRegistry(), true);
                    } else {
                        beanName1 = this.readerContext.generateBeanName(beanDefinition);
                        String aliasesArray = beanDefinition.getBeanClassName();
                        if(aliasesArray != null 
                                && beanName1.startsWith(aliasesArray) 
                                && beanName1.length() > aliasesArray.length() 
                                && !this.readerContext.getRegistry().isBeanNameInUse(aliasesArray)) {
                            aliases.add(aliasesArray);
                        }
                    }

                    if(this.logger.isDebugEnabled()) {
                        this.logger.debug(
                            "Neither XML \'id\' nor \'name\' specified - using generated bean name [" + beanName1 + "]");
                    }
                } catch (Exception var9) {
                    this.error(var9.getMessage(), ele);
                    return null;
                }
            }

            String[] aliasesArray1 = StringUtils.toStringArray(aliases);
            return new BeanDefinitionHolder(beanDefinition, beanName1, aliasesArray1);
        } else {
            return null;
        }
    }
```

进入parseBeanDefinitionElement(Element, String, BeanDefinition)方法, 根据xml bean中
各个参数的配置来解析, 并封装到AbstractBeanDefinition中, 相应代码如下:

```
public AbstractBeanDefinition parseBeanDefinitionElement(Element ele,
                String beanName, BeanDefinition containingBean) {
    this.parseState.push(new BeanEntry(beanName));
    String className = null;
    if(ele.hasAttribute("class")) {
       className = ele.getAttribute("class").trim();
    }

    try {
        String ex = null;
        if(ele.hasAttribute("parent")) {
            ex = ele.getAttribute("parent");
        }

        AbstractBeanDefinition bd = this.createBeanDefinition(className, ex);
        this.parseBeanDefinitionAttributes(ele, beanName, containingBean, bd);
        bd.setDescription(DomUtils.getChildElementValueByTagName(ele, "description"));
        this.parseMetaElements(ele, bd);
        this.parseLookupOverrideSubElements(ele, bd.getMethodOverrides());
        this.parseReplacedMethodSubElements(ele, bd.getMethodOverrides());
        this.parseConstructorArgElements(ele, bd);
        this.parsePropertyElements(ele, bd);
        this.parseQualifierElements(ele, bd);
        bd.setResource(this.readerContext.getResource());
        bd.setSource(this.extractSource(ele));
        AbstractBeanDefinition var7 = bd;
        return var7;
    } catch (ClassNotFoundException var13) {
        this.error("Bean class [" + className + "] not found", ele, var13);
    } catch (NoClassDefFoundError var14) {
        this.error(
            "Class that bean class [" + className + "] depends on not found", ele, var14);
    } catch (Throwable var15) {
        this.error("Unexpected failure during bean definition parsing", ele, var15);
    } finally {
        this.parseState.pop();
    }

    return null;
}
```

#### BeanDefinitionReaderUtils

```
    public static void registerBeanDefinition(BeanDefinitionHolder definitionHolder,
            BeanDefinitionRegistry registry) throws BeanDefinitionStoreException {
        String beanName = definitionHolder.getBeanName();
        registry.registerBeanDefinition(beanName, definitionHolder.getBeanDefinition());
        String[] aliases = definitionHolder.getAliases();
        if(aliases != null) {
            String[] var4 = aliases;
            int var5 = aliases.length;

            for(int var6 = 0; var6 < var5; ++var6) {
                String alias = var4[var6];
                registry.registerAlias(beanName, alias);
            }
        }

    }
```

#### DefaultListableBeanFactory

```
    public void registerBeanDefinition(String beanName, BeanDefinition beanDefinition)
            throws BeanDefinitionStoreException {
        Assert.hasText(beanName, "Bean name must not be empty");
        Assert.notNull(beanDefinition, "BeanDefinition must not be null");
        if(beanDefinition instanceof AbstractBeanDefinition) {
            try {
                ((AbstractBeanDefinition)beanDefinition).validate();
            } catch (BeanDefinitionValidationException var9) {
                throw new BeanDefinitionStoreException(
                    beanDefinition.getResourceDescription(),
                    beanName,
                    "Validation of bean definition failed", var9);
            }
        }

        BeanDefinition oldBeanDefinition = (BeanDefinition)this.beanDefinitionMap.get(beanName);
        if(oldBeanDefinition != null) {
            if(!this.isAllowBeanDefinitionOverriding()) {
                throw new BeanDefinitionStoreException(
                    beanDefinition.getResourceDescription(),
                    beanName,
                    "Cannot register bean definition [" + beanDefinition + "] for bean \'"
                        + beanName + "\': There is already [" + oldBeanDefinition + "] bound.");
            }

            if(oldBeanDefinition.getRole() < beanDefinition.getRole()) {
                if(this.logger.isWarnEnabled()) {
                    this.logger.warn(
                    "Overriding user-defined bean definition for bean \'" 
                    + beanName + "\' with a framework-generated bean definition: replacing ["
                     + oldBeanDefinition + "] with [" + beanDefinition + "]");
                }
            } else if(!beanDefinition.equals(oldBeanDefinition)) {
                if(this.logger.isInfoEnabled()) {
                    this.logger.info(
                        "Overriding bean definition for bean \'" 
                        + beanName + "\' with a different definition: replacing ["
                         + oldBeanDefinition + "] with [" + beanDefinition + "]");
                }
            } else if(this.logger.isDebugEnabled()) {
                this.logger.debug("Overriding bean definition for bean \'"
                 + beanName + "\' with an equivalent definition: replacing ["
                  + oldBeanDefinition + "] with [" + beanDefinition + "]");
            }

            this.beanDefinitionMap.put(beanName, beanDefinition);
        } else {
            if(this.hasBeanCreationStarted()) {
                Map var4 = this.beanDefinitionMap;
                synchronized(this.beanDefinitionMap) {
                    this.beanDefinitionMap.put(beanName, beanDefinition);
                    ArrayList updatedDefinitions = new ArrayList(this.beanDefinitionNames.size() + 1);
                    updatedDefinitions.addAll(this.beanDefinitionNames);
                    updatedDefinitions.add(beanName);
                    this.beanDefinitionNames = updatedDefinitions;
                    if(this.manualSingletonNames.contains(beanName)) {
                        LinkedHashSet updatedSingletons = new LinkedHashSet(this.manualSingletonNames);
                        updatedSingletons.remove(beanName);
                        this.manualSingletonNames = updatedSingletons;
                    }
                }
            } else {
                this.beanDefinitionMap.put(beanName, beanDefinition);
                this.beanDefinitionNames.add(beanName);
                this.manualSingletonNames.remove(beanName);
            }

            this.frozenBeanDefinitionNames = null;
        }

        if(oldBeanDefinition != null || this.containsSingleton(beanName)) {
            this.resetBeanDefinition(beanName);
        }

    }
```    

#### ReaderContext

```
    public void fireComponentRegistered(ComponentDefinition componentDefinition) {
        this.eventListener.componentRegistered(componentDefinition);
    }
```    
    
#### EmptyReaderEventListener
    
```
    public void componentRegistered(ComponentDefinition componentDefinition) {
    }
```
    
[PrePostProcessXML]: http://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/xml/DefaultBeanDefinitionDocumentReader.html#postProcessXml-org.w3c.dom.Element-

## 参考文献
1. <<Spring源码深度解析>>



