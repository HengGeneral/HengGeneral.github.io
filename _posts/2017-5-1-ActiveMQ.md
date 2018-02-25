---
layout: post
title: ActiveMQ基本介绍
tags:  [ActiveMQ]
categories: [中间件]
author: liheng
excerpt: "ActiveMQ"
---
### 介绍

消息中间件(MOM, message-oriented middleware), 官方的解释是software infrastructure supporting sending and receiving messages between distributed systems.

### JMS的基本元素
#### ConnectionFactory
连接工厂是客户用来创建connection的对象，例如 ActiveMQ提供的ActiveMQConnectionFactory，通过这个对象可以创建消息客户端到消息服务提供者之间的连接。

#### Connection
JMS Connection封装了客户端（producer/consumer）到消息服务提供者之间的网络连接，它代表消息客户端到消息服务器之间的逻辑通道。

#### Session
JMS Session是生产和消费消息的一个单线程上下文，提供了以下功能：
*   用于创建producers、 consumers、destination；
*   用于消息消费过程中的管理，如消息ack前的保留、消息生产消费的序号等；
*   提供了一个事务性的上下文，在这个上下文中，一组发送和接收被组合到了一个原子操作中。

#### Destination
目的地是客户用来指定它生产的消息的目标和它消费的消息的来源的对象。
JMS规范中定义了两种消息传递模型：点对点消息传递模型（PTP）和发布/订阅消息传递模型(Publish/Subscribe)。
这两种模式对应于ActiveMQ则是，在点对点消息中，目的地被称为队列（queue）；在发布/订阅中，目的地被成为主题（topic）。

![amqPtp](/images/activemq/amq_ptp.png)

如上图所示，该消息传递模型使用的是点对点消息传递模型。
图中左边是消息生产者，右边是消息消费者，生产者发送消息到broker的queue中，broker会把这条消息分发给注册在这个队列上的某一个消费者，同一条消息只会被一个消费者所消费。
代码如下：
```
    Destination destination = session.createQueue(this.sAlias);
```

![amqPubSub](/images/activemq/amq_pubsub.png)

如上图所示，该消息传递模型使用的是发布/订阅消息传递模型。图中左边是消息生产者，右边是消息消费者，生产者发送消息到broker的topic中，
broker会把这条消息转发给这个topic上的所有消费者，同一条消息会被所有消费者消费。代码如下：
```
    Destination destination = session.createTopic(this.sAlias);
```

#### Producer
消息生产者是由session创建的一个对象，用于把消息发送到一个目的地。
客户端使用MessageProducer来向Destination 发送消息。通过session.createProducer(Destination destination)方法创建MessageProducer。  
JMS client使用MessageProducer类来向一个目的地址（destination）发送消息。通常来讲，一个producer的目的地址在producer被建立时已经确认，使用方式如下：
```
    MessageProducer producer = session.createProducer(Destination destination);


    connection = AMQConnectionFactory.getPooledConnectionFactory(this.sAlias).createConnection();
    session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
    Destination destination = session.createQueue(this.sAlias);
    MessageProducer producer = session.createProducer(destination);
    connection.start();
    producer.send(session.createTextMessage(message));
```

而对于每条消息，也提供了覆盖这个地址的方法，这个在spring中也运用较多，使用方式如下：
```
    public void send(final Destination destination, final MessageCreator messageCreator) throws JmsException；
    
    
    producer.send(queue, new MessageCreator() {
                    @Override
                    public Message createMessage(Session session)
                            throws JMSException {
                        TextMessage textMessage = session.createTextMessage();
                        textMessage.setText(convertJsonData(message));
                        return textMessage;
                    }
                });
```

#### Consumer
消息消费者也是由session创建的一个对象，它用于接收发送到destination的消息。客户端使用session.createConsumer(Destination)方法创建MessageConsumer, 创建方式如下：
```
    MessageConsumer createConsumer(Destination destination) throws JMSException;
    
    
    Connection conn = AMQConnectionFactory.getCachingConnectionFactorys(this.sAlias).createConnection();
    conn.start();
    Session session = conn.createSession(false, Session.AUTO_ACKNOWLEDGE);
    Destination destination = session.createQueue(this.sAlias);
    MessageConsumer consumer = session.createConsumer(destination);
```

消息的消费可以采用以下两种方法之一：

*   同步消费, 通过调用MessageConsumer.receive()方法从目的地中显式提取消息。receive()方法可以一直阻塞，直到消息到达。
*   异步消费, 客户可以为消费者注册一个消息监听器（MessageListener），以定义在消息到达时所采取的动作, 这个接口主要实现了onMessage()方法。


#### Message消息体
JMS消息体有以下五种类型：

*   TextMessage：包含了一个java.lang.String
*   MapMessage：包含了一些列的name-value对，name是String，value是Java primitive类型。消息体中的条目可以被enumerator按照顺序访问，也可以自由访问。
*   BytesMessage：包含一个不间断的字节流。
*   StreamMessage：包含了一个Java primitive流对象，这个流可以顺序读取和填充。
*   ObjectMessage：包含了一个可序列化的Java对象。

#### 对象间的关系
上面提到的这些JMS元素之间的关系如下图所示：

![amqInteract](/images/activemq/amq_interact.png)

### JMS高效使用
#### PooledConnectionFactory
该链接工厂可以在Connections, sessions and producers使用之后会缓存下来，之后这些资源就可以复用(它们的创建会消耗大量系统资源)。
```
    @Override
    public synchronized Connection createConnection(String userName, String password) throws JMSException {
        if (stopped.get()) {
            LOG.debug("PooledConnectionFactory is stopped, skip create new connection.");
            return null;
        }

        ConnectionPool connection = null;
        // userName和passpord都传的为null
        ConnectionKey key = new ConnectionKey(userName, password);

        // This will either return an existing non-expired ConnectionPool or it
        // will create a new one to meet the demand.
        if (getConnectionsPool().getNumIdle(key) < getMaxConnections()) {
            try {
                connectionsPool.addObject(key);
                connection = mostRecentlyCreated.getAndSet(null);
                connection.incrementReferenceCount();
            } catch (Exception e) {
                throw createJmsException("Error while attempting to add new Connection to the pool", e);
            }
        } else {
            try {
                // We can race against other threads returning the connection when there is an
                // expiration or idle timeout.  We keep pulling out ConnectionPool instances until
                // we win and get a non-closed instance and then increment the reference count
                // under lock to prevent another thread from triggering an expiration check and
                // pulling the rug out from under us.
                while (connection == null) {
                    connection = connectionsPool.borrowObject(key);
                    synchronized (connection) {
                        if (connection.getConnection() != null) {
                            connection.incrementReferenceCount();
                            break;
                        }

                        // Return the bad one to the pool and let if get destroyed as normal.
                        connectionsPool.returnObject(key, connection);
                        connection = null;
                    }
                }
            } catch (Exception e) {
                throw createJmsException("Error while attempting to retrieve a connection from the pool", e);
            }

            try {
                connectionsPool.returnObject(key, connection);
            } catch (Exception e) {
                throw createJmsException("Error when returning connection to the pool", e);
            }
        }

        return newPooledConnection(connection);
    }
```

从上面的代码可以看出，创建的connection数量的限制会受限于DEFAULT_MAX_IDLE_PER_KEY(默认是8)。

需要注意的是，PooledConnectionFactory并不会缓存consumers。原因是，consumer会批量的获取消息，
接收消息的多少可以根据prefetch的大小设置，因此在大量消息发送到消费者时，消费者使用类似统一获取、统一消费的方式处理，“池”没有存在的价值。

#### CachingConnectionFactory
CachingConnectionFactory继承了SingleConnectionFactory，并在所有的createConnection()调用中返回同一个connection对象。代码如下：
```
    //SingleConnectionFactory.java
    public Connection createConnection() throws JMSException {
            Object var1 = this.connectionMonitor;
            synchronized(this.connectionMonitor) {
                if(this.connection == null) {
                    this.initConnection();
                }
    
                return this.connection;
            }
    }
    
    public void initConnection() throws JMSException {
            if(this.getTargetConnectionFactory() == null) {
                throw new java.lang.IllegalStateException("\'targetConnectionFactory\' is required for lazily initializing a Connection");
            } else {
                Object var1 = this.connectionMonitor;
                synchronized(this.connectionMonitor) {
                    if(this.target != null) {
                        this.closeConnection(this.target);
                    }
    
                    this.target = this.doCreateConnection();
                    this.prepareConnection(this.target);
                    if(this.logger.isInfoEnabled()) {
                        this.logger.info("Established shared JMS Connection: " + this.target);
                    }
    
                    this.connection = this.getSharedConnectionProxy(this.target);
                }
            }
    }
    
    //ActiveMQConnectionFactory.java
    protected ActiveMQConnection createActiveMQConnection(String userName, String password) throws JMSException {
            if(this.brokerURL == null) {
                throw new ConfigurationException("brokerURL not set.");
            } else {
                ActiveMQConnection connection = null;
    
                try {
                    Transport e = this.createTransport();
                    connection = this.createActiveMQConnection(e, this.factoryStats);
                    connection.setUserName(userName);
                    connection.setPassword(password);
                    this.configureConnection(connection);
                    e.start();
                    if(this.clientID != null) {
                        connection.setDefaultClientID(this.clientID);
                    }
    
                    return connection;
                } catch (JMSException var8) {
                    try {
                        connection.close();
                    } catch (Throwable var7) {
                        ;
                    }
    
                    throw var8;
                } catch (Exception var9) {
                    try {
                        connection.close();
                    } catch (Throwable var6) {
                        ;
                    }
    
                    throw JMSExceptionSupport.create("Could not connect to broker URL: " + this.brokerURL + ". Reason: " + var9, var9);
                }
            }
        }
```

我们在生产环境遇到一个问题，生产环境中与AMQ Broker连接数过多造成部分生产者连接不上，同时又不停地连接再断开问题。
原因则是，最开始创建一个生产者就创建一个连接然后关闭。之前就发现这个问题会影响性能，但消息量并不大，所以就没有管。
之后使用了PooledConnectionFactory，情况有所好转但是仍然会有连接不上且不停地连接再断开的问题，后来使用了CachingConnectionFactory
就彻底地解决了问题，最终的结果是每台机器上对该broker只有一个连接（命令是：netstat -nt | grep 61616）。

### AMQ基本介绍

### Activemq Console
利用ActiveMQ控制台可以查看消息消费的实时情况，如下图所示：
![amqConsole](/images/activemq/amq_console.png)

Dequeues: 表示每个消费者消费的消息数量；
Dispatched: 表示AMQ 推送给消费者的消息数量，与Enqueues 这一列的数据是相同的；
Dispatched Queue: 表示消费者未ACK的消息数量；
ClientID: 可以看出消费者所部署的机器host; 
Prefetch: 1000代表的是消费者设置的Prefetch Size,消费者使用的默认的Prefetch Size设置, 业务并没有在代码中去设置自己的Prefetch Size. 

根据控制台如何判断消费者是否存在慢消费呢？

当再次刷新控制台界面时，发现Dequeues这一列的数据更新与上次刷新差距不大而Dispatched Queue 中的数据减少量也特别小（说明消费的消息很少），
同时队列不停有大量消息来时（说明生产的消息较多），就可以说明这时候消费比较慢。

慢消费造成的直接影响是，当消费者未ACK的消息一直持续不减少，ActiveMQ Broker会认为该消费者消费过慢，会不再推送新的消息给该消费者.

消费者一直持有未ACK的消息，只有断开该消费者，该消费者未ACK的消息才会被ActiveMQ Broker重新推送给其他消费者（这就是为什么队列堵了一些老消息，机器重启能解决的原因）。

## 参考文献:
1. ActiveMQ in Action.pdf