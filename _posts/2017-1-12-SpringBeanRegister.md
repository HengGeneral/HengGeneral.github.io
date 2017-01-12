---
layout: post
title: Bean
tags:  [JAVA]
categories: [JAVA]
author: liheng
excerpt: "Bean"
---
## Bean

当将XML转换为Document后, 解析来就是调用registerBeanDefinitions(Document, EncodedResource)提取和注册bean了。


#### DefaultBeanDefinitionDocumentReader

最终执行到DefaultBeanDefinitionDocumentReader类的registerBeanDefinitions(Document, XmlReaderContext)方法,
代码如下:

```
    public void registerBeanDefinitions(Document doc, XmlReaderContext readerContext) {
        this.readerContext = readerContext;
        this.logger.debug("Loading bean definitions");
        Element root = doc.getDocumentElement();
        this.doRegisterBeanDefinitions(root);
    }
```

接下来,就真正地进入Bean的解析阶段了, 进入doRegisterBeanDefinitions(Element)方法, 代码如下:

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
                        //解析beans下的子元素
                        this.parseDefaultElement(ele, delegate);
                    } else {
                        delegate.parseCustomElement(ele);
                    }
                }
            }
        } else {
            delegate.parseCustomElement(root);
        }
```

当遇到<bean/>等默认标签时, 则进入parseDefaultElement(Element, BeanDefinitionParserDelegate)方法, 其源码如下:

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

我们先以<bean />标签为例, 来看看processBeanDefinition(Element, BeanDefinitionParserDelegate), 代码如下:

```
    protected void processBeanDefinition(Element ele, BeanDefinitionParserDelegate delegate) {
        BeanDefinitionHolder bdHolder = delegate.parseBeanDefinitionElement(ele);
        if(bdHolder != null) {
            bdHolder = delegate.decorateBeanDefinitionIfRequired(ele, bdHolder);

            try {
                BeanDefinitionReaderUtils.registerBeanDefinition(bdHolder, this.getReaderContext().getRegistry());
            } catch (BeanDefinitionStoreException var5) {
                this.getReaderContext().error("Failed to register bean definition with name \'" + bdHolder.getBeanName() + "\'", ele, var5);
            }

            this.getReaderContext().fireComponentRegistered(new BeanComponentDefinition(bdHolder));
        }

    }
```

#### BeanDefinitionParserDelegate
我们先来看看parseBeanDefinitionElement(Element)方法, 代码如下:

```
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
                this.logger.debug("No XML \'id\' specified - using \'" + beanName1 + "\' as bean name and " + aliases + " as aliases");
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
                        beanName1 = BeanDefinitionReaderUtils.generateBeanName(beanDefinition, this.readerContext.getRegistry(), true);
                    } else {
                        beanName1 = this.readerContext.generateBeanName(beanDefinition);
                        String aliasesArray = beanDefinition.getBeanClassName();
                        if(aliasesArray != null && beanName1.startsWith(aliasesArray) && beanName1.length() > aliasesArray.length() && !this.readerContext.getRegistry().isBeanNameInUse(aliasesArray)) {
                            aliases.add(aliasesArray);
                        }
                    }

                    if(this.logger.isDebugEnabled()) {
                        this.logger.debug("Neither XML \'id\' nor \'name\' specified - using generated bean name [" + beanName1 + "]");
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

[PrePostProcessXML]: http://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/beans/factory/xml/DefaultBeanDefinitionDocumentReader.html#postProcessXml-org.w3c.dom.Element-

## 参考文献



