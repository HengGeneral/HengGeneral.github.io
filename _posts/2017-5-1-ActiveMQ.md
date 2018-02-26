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

常用到的ConnectionFactory有PooledConnectionFactory、CachingConnectionFactory等。

#### Connection
JMS Connection封装了客户端（producer/consumer）到消息服务提供者broker之间的网络连接，它代表消息客户端到消息服务器之间的网络通道。
connection的创建见PooledConnectionFactory/CachingConnectionFactory部分。

#### Session
JMS Session是生产和消费消息的一个单线程上下文，提供了以下功能：
*   用于创建producers、 consumers、destination；
*   用于消息消费过程中的管理，如消息ack前的保留、消息生产消费的序号等；
*   提供了一个事务性的上下文，在这个上下文中，一组发送和接收被组合到了一个原子操作中。

AMQ session的创建如下：
```
    // ActiveMQConnection.java
    public Session createSession(boolean transacted, int acknowledgeMode) throws JMSException {
        checkClosedOrFailed();
        ensureConnectionInfoSent();
        if (!transacted) {
            if (acknowledgeMode == Session.SESSION_TRANSACTED) {
                throw new JMSException("acknowledgeMode SESSION_TRANSACTED cannot be used for an non-transacted Session");
            } else if (acknowledgeMode < Session.SESSION_TRANSACTED || acknowledgeMode > ActiveMQSession.MAX_ACK_CONSTANT) {
                throw new JMSException("invalid acknowledgeMode: " + acknowledgeMode + ". Valid values are Session.AUTO_ACKNOWLEDGE (1), " +
                        "Session.CLIENT_ACKNOWLEDGE (2), Session.DUPS_OK_ACKNOWLEDGE (3), ActiveMQSession.INDIVIDUAL_ACKNOWLEDGE (4) or for transacted sessions Session.SESSION_TRANSACTED (0)");
            }
        }
        return new ActiveMQSession(this, getNextSessionId(), transacted ? Session.SESSION_TRANSACTED : acknowledgeMode, isDispatchAsync(), isAlwaysSessionAsync());
    }
    
    // ActiveMQSession.java
    protected ActiveMQSession(ActiveMQConnection connection, SessionId sessionId, int acknowledgeMode, boolean asyncDispatch, boolean sessionAsyncDispatch) throws JMSException {
            this.debug = LOG.isDebugEnabled();
            this.connection = connection;
            this.acknowledgementMode = acknowledgeMode;
            this.asyncDispatch = asyncDispatch;
            this.sessionAsyncDispatch = sessionAsyncDispatch;
            this.info = new SessionInfo(connection.getConnectionInfo(), sessionId.getValue());
            setTransactionContext(new TransactionContext(connection));
            stats = new JMSSessionStatsImpl(producers, consumers);
            this.connection.asyncSendPacket(info);
            setTransformer(connection.getTransformer());
            setBlobTransferPolicy(connection.getBlobTransferPolicy());
            this.connectionExecutor = connection.getExecutor();
            // ssionExecutor，执行器用于在有消息时异步分发消息给consumers
            this.executor = new ActiveMQSessionExecutor(this);
            connection.addSession(this);
            // 启动session(其实就是启动session的consumers进行消息处理的管理)
            if (connection.isStarted()) {
                start();
            }
        }
```

一个connection可以创建多个session，connection可以被producer/consumer/session共享，而session则不可以。

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

而使用session创建producer的源码如下：
```
    // ActiveMQMessageProducer.java
    protected ActiveMQMessageProducer(ActiveMQSession session, ProducerId producerId, ActiveMQDestination destination, int sendTimeout) throws JMSException {
        super(session);
        this.info = new ProducerInfo(producerId);
        this.info.setWindowSize(session.connection.getProducerWindowSize());
        // Allows the options on the destination to configure the producerInfo
        if (destination != null && destination.getOptions() != null) {
            Map<String, Object> options = IntrospectionSupport.extractProperties(
                new HashMap<String, Object>(destination.getOptions()), "producer.");
            IntrospectionSupport.setProperties(this.info, options);
            if (options.size() > 0) {
                String msg = "There are " + options.size()
                    + " producer options that couldn't be set on the producer."
                    + " Check the options are spelled correctly."
                    + " Unknown parameters=[" + options + "]."
                    + " This producer cannot be started.";
                LOG.warn(msg);
                throw new ConfigurationException(msg);
            }
        }

        this.info.setDestination(destination);

        // Enable producer window flow control if protocol >= 3 and the window size > 0
        if (session.connection.getProtocolVersion() >= 3 && this.info.getWindowSize() > 0) {
            producerWindow = new MemoryUsage("Producer Window: " + producerId);
            producerWindow.setExecutor(session.getConnectionExecutor());
            producerWindow.setLimit(this.info.getWindowSize());
            producerWindow.start();
        }

        this.defaultDeliveryMode = Message.DEFAULT_DELIVERY_MODE;
        this.defaultPriority = Message.DEFAULT_PRIORITY;
        this.defaultTimeToLive = Message.DEFAULT_TIME_TO_LIVE;
        this.startTime = System.currentTimeMillis();
        this.messageSequence = new AtomicLong(0);
        this.stats = new JMSProducerStatsImpl(session.getSessionStats(), destination);
        try {
            this.session.addProducer(this);
            this.session.syncSendPacket(info);
        } catch (JMSException e) {
            this.session.removeProducer(this);
            throw e;
        }
        this.setSendTimeout(sendTimeout);
        setTransformer(session.getTransformer());
    }
```

消息的发送源码如下：
```
    // ActiveMQMessageProducer.java
    public void send(Destination destination, Message message, int deliveryMode, int priority, long timeToLive, AsyncCallback onComplete) throws JMSException {
        checkClosed();
        if (destination == null) {
            if (info.getDestination() == null) {
                throw new UnsupportedOperationException("A destination must be specified.");
            }
            throw new InvalidDestinationException("Don't understand null destinations");
        }

        ActiveMQDestination dest;
        if (destination.equals(info.getDestination())) {
            dest = (ActiveMQDestination)destination;
        } else if (info.getDestination() == null) {
            dest = ActiveMQDestination.transform(destination);
        } else {
            throw new UnsupportedOperationException("This producer can only send messages to: " + this.info.getDestination().getPhysicalName());
        }
        if (dest == null) {
            throw new JMSException("No destination specified");
        }

        if (transformer != null) {
            Message transformedMessage = transformer.producerTransform(session, this, message);
            if (transformedMessage != null) {
                message = transformedMessage;
            }
        }

        if (producerWindow != null) {
            try {
                producerWindow.waitForSpace();
            } catch (InterruptedException e) {
                throw new JMSException("Send aborted due to thread interrupt.");
            }
        }

        // 使用session发送消息
        this.session.send(this, dest, message, deliveryMode, priority, timeToLive, producerWindow, sendTimeout, onComplete);

        stats.onMessage();
    }
    
    // ActiveMQSession.java
    protected void send(ActiveMQMessageProducer producer, ActiveMQDestination destination, Message message, int deliveryMode, int priority, long timeToLive,
                        MemoryUsage producerWindow, int sendTimeout, AsyncCallback onComplete) throws JMSException {

        checkClosed();
        if (destination.isTemporary() && connection.isDeleted(destination)) {
            throw new InvalidDestinationException("Cannot publish to a deleted Destination: " + destination);
        }
        synchronized (sendMutex) {
            // tell the Broker we are about to start a new transaction
            doStartTransaction();
            TransactionId txid = transactionContext.getTransactionId();
            long sequenceNumber = producer.getMessageSequence();

            //Set the "JMS" header fields on the original message, see 1.1 spec section 3.4.11
            message.setJMSDeliveryMode(deliveryMode);
            long expiration = 0L;
            if (!producer.getDisableMessageTimestamp()) {
                long timeStamp = System.currentTimeMillis();
                message.setJMSTimestamp(timeStamp);
                if (timeToLive > 0) {
                    expiration = timeToLive + timeStamp;
                }
            }
            message.setJMSExpiration(expiration);
            message.setJMSPriority(priority);
            message.setJMSRedelivered(false);

            // transform to our own message format here
            ActiveMQMessage msg = ActiveMQMessageTransformation.transformMessage(message, connection);
            msg.setDestination(destination);
            msg.setMessageId(new MessageId(producer.getProducerInfo().getProducerId(), sequenceNumber));

            // Set the message id.
            if (msg != message) {
                message.setJMSMessageID(msg.getMessageId().toString());
                // Make sure the JMS destination is set on the foreign messages too.
                message.setJMSDestination(destination);
            }
            //clear the brokerPath in case we are re-sending this message
            msg.setBrokerPath(null);

            msg.setTransactionId(txid);
            if (connection.isCopyMessageOnSend()) {
                msg = (ActiveMQMessage)msg.copy();
            }
            msg.setConnection(connection);
            msg.onSend();
            msg.setProducerId(msg.getMessageId().getProducerId());
            if (LOG.isTraceEnabled()) {
                LOG.trace(getSessionId() + " sending message: " + msg);
            }
            if (onComplete==null && sendTimeout <= 0 && !msg.isResponseRequired() && !connection.isAlwaysSyncSend() && (!msg.isPersistent() || connection.isUseAsyncSend() || txid != null)) {
                this.connection.asyncSendPacket(msg);
                if (producerWindow != null) {
                    // Since we defer lots of the marshaling till we hit the
                    // wire, this might not
                    // provide and accurate size. We may change over to doing
                    // more aggressive marshaling,
                    // to get more accurate sizes.. this is more important once
                    // users start using producer window
                    // flow control.
                    int size = msg.getSize();
                    producerWindow.increaseUsage(size);
                }
            } else {
                if (sendTimeout > 0 && onComplete==null) {
                    this.connection.syncSendPacket(msg,sendTimeout);
                }else {
                    // 开始使用connection进行消息的发送
                    this.connection.syncSendPacket(msg, onComplete);
                }
            }
        }
    }
    
    // 消息交由ActiveMQConnection发送，最终使用transport进行消息发送
    private void doAsyncSendPacket(Command command) throws JMSException {
        try {
            this.transport.oneway(command);
        } catch (IOException e) {
            throw JMSExceptionSupport.create(e);
        }
    }
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

消息发送的过程是ConnectionFactory->Connection->Session->MessageProducer->send，如下图所示（图片来源于何晓娟同学）：

![amqPubSub](/images/activemq/amq_sendProcess.png)

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

而使用session创建consumer的源码如下：
```
    // ActiveMQMessageConsumer.java
    public ActiveMQMessageConsumer(ActiveMQSession session, ConsumerId consumerId, ActiveMQDestination dest,
            String name, String selector, int prefetch,
            int maximumPendingMessageCount, boolean noLocal, boolean browser,
            boolean dispatchAsync, MessageListener messageListener) throws JMSException {
        if (dest == null) {
            throw new InvalidDestinationException("Don't understand null destinations");
        } else if (dest.getPhysicalName() == null) {
            throw new InvalidDestinationException("The destination object was not given a physical name.");
        } else if (dest.isTemporary()) {
            String physicalName = dest.getPhysicalName();

            if (physicalName == null) {
                throw new IllegalArgumentException("Physical name of Destination should be valid: " + dest);
            }

            String connectionID = session.connection.getConnectionInfo().getConnectionId().getValue();

            if (physicalName.indexOf(connectionID) < 0) {
                throw new InvalidDestinationException("Cannot use a Temporary destination from another Connection");
            }

            if (session.connection.isDeleted(dest)) {
                throw new InvalidDestinationException("Cannot use a Temporary destination that has been deleted");
            }
            if (prefetch < 0) {
                throw new JMSException("Cannot have a prefetch size less than zero");
            }
        }
        if (session.connection.isMessagePrioritySupported()) {
            this.unconsumedMessages = new SimplePriorityMessageDispatchChannel();
        }else {
            this.unconsumedMessages = new FifoMessageDispatchChannel();
        }

        this.session = session;
        this.redeliveryPolicy = session.connection.getRedeliveryPolicyMap().getEntryFor(dest);
        if (this.redeliveryPolicy == null) {
            this.redeliveryPolicy = new RedeliveryPolicy();
        }
        setTransformer(session.getTransformer());

        this.info = new ConsumerInfo(consumerId);
        this.info.setExclusive(this.session.connection.isExclusiveConsumer());
        this.info.setClientId(this.session.connection.getClientID());
        this.info.setSubscriptionName(name);
        this.info.setPrefetchSize(prefetch);
        this.info.setCurrentPrefetchSize(prefetch);
        this.info.setMaximumPendingMessageLimit(maximumPendingMessageCount);
        this.info.setNoLocal(noLocal);
        this.info.setDispatchAsync(dispatchAsync);
        this.info.setRetroactive(this.session.connection.isUseRetroactiveConsumer());
        this.info.setSelector(null);

        // Allows the options on the destination to configure the consumerInfo
        if (dest.getOptions() != null) {
            Map<String, Object> options = IntrospectionSupport.extractProperties(
                new HashMap<String, Object>(dest.getOptions()), "consumer.");
            IntrospectionSupport.setProperties(this.info, options);
            if (options.size() > 0) {
                String msg = "There are " + options.size()
                    + " consumer options that couldn't be set on the consumer."
                    + " Check the options are spelled correctly."
                    + " Unknown parameters=[" + options + "]."
                    + " This consumer cannot be started.";
                LOG.warn(msg);
                throw new ConfigurationException(msg);
            }
        }

        this.info.setDestination(dest);
        this.info.setBrowser(browser);
        if (selector != null && selector.trim().length() != 0) {
            // Validate the selector
            SelectorParser.parse(selector);
            this.info.setSelector(selector);
            this.selector = selector;
        } else if (info.getSelector() != null) {
            // Validate the selector
            SelectorParser.parse(this.info.getSelector());
            this.selector = this.info.getSelector();
        } else {
            this.selector = null;
        }

        this.stats = new JMSConsumerStatsImpl(session.getSessionStats(), dest);
        this.optimizeAcknowledge = session.connection.isOptimizeAcknowledge() && session.isAutoAcknowledge()
                                   && !info.isBrowser();
        if (this.optimizeAcknowledge) {
            this.optimizeAcknowledgeTimeOut = session.connection.getOptimizeAcknowledgeTimeOut();
            setOptimizedAckScheduledAckInterval(session.connection.getOptimizedAckScheduledAckInterval());
        }

        this.info.setOptimizedAcknowledge(this.optimizeAcknowledge);
        this.failoverRedeliveryWaitPeriod = session.connection.getConsumerFailoverRedeliveryWaitPeriod();
        this.nonBlockingRedelivery = session.connection.isNonBlockingRedelivery();
        this.transactedIndividualAck = session.connection.isTransactedIndividualAck()
                        || this.nonBlockingRedelivery
                        || session.connection.isMessagePrioritySupported();
        this.consumerExpiryCheckEnabled = session.connection.isConsumerExpiryCheckEnabled();
        if (messageListener != null) {
            setMessageListener(messageListener);
        }
        try {
            // consumer创建
            this.session.addConsumer(this);
            this.session.syncSendPacket(info);
        } catch (JMSException e) {
            this.session.removeConsumer(this);
            throw e;
        }

        if (session.connection.isStarted()) {
            start();
        }
    }
```

消息的消费可以采用以下两种方法之一：

*   同步消费, 通过调用MessageConsumer.receive()方法从目的地中显式提取消息。receive()方法可以一直阻塞，直到消息到达。
*   异步消费, 客户可以为消费者注册一个消息监听器（MessageListener），以定义在消息到达时所采取的动作, 这个接口主要实现了onMessage()方法。

消息的接收源码如下：
```
    //ActiveMQSession.java
    protected void start() throws JMSException {
        started.set(true);
        for (Iterator<ActiveMQMessageConsumer> iter = consumers.iterator(); iter.hasNext();) {
            ActiveMQMessageConsumer c = iter.next();
            c.start();
        }
        executor.start();
    }
    
    
    //ActiveMQSessionExecutor.java
    synchronized void start() {
        if (!messageQueue.isRunning()) {
            messageQueue.start();
            if (hasUncomsumedMessages()) {
                wakeup();
            }
        }
    }
    
    public void wakeup() {
        if (!dispatchedBySessionPool) {
            if (session.isSessionAsyncDispatch()) {
                try {
                    TaskRunner taskRunner = this.taskRunner;
                    if (taskRunner == null) {
                        synchronized (this) {
                            if (this.taskRunner == null) {
                                if (!isRunning()) {
                                    // stop has been called
                                    return;
                                }
                                this.taskRunner = session.connection.getSessionTaskRunner().createTaskRunner(this,
                                        "ActiveMQ Session: " + session.getSessionId());
                            }
                            taskRunner = this.taskRunner;
                        }
                    }
                    // 消息消费逻辑，最终也会调用iterate()方法
                    taskRunner.wakeup();
                } catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }
            } else {
                // 消息消费逻辑
                while (iterate()) {
                }
            }
        }
    }
    
    public boolean iterate() {

        // Deliver any messages queued on the consumer to their listeners.
        for (ActiveMQMessageConsumer consumer : this.session.consumers) {
            if (consumer.iterate()) {
                return true;
            }
        }

        // No messages left queued on the listeners.. so now dispatch messages
        // queued on the session
        MessageDispatch message = messageQueue.dequeueNoWait();
        if (message == null) {
            return false;
        } else {
            dispatch(message);
            return !messageQueue.isEmpty();
        }
    }
    
    void dispatch(MessageDispatch message) {
        // TODO - we should use a Map for this indexed by consumerId
        for (ActiveMQMessageConsumer consumer : this.session.consumers) {
            ConsumerId consumerId = message.getConsumerId();
            if (consumerId.equals(consumer.getConsumerId())) {
                consumer.dispatch(message);
                break;
            }
        }
    }
    
    
    // ActiveMQMessageConsumer.java
    public void dispatch(MessageDispatch md) {
        MessageListener listener = this.messageListener.get();
        try {
            clearMessagesInProgress();
            clearDeliveredList();
            synchronized (unconsumedMessages.getMutex()) {
                if (!unconsumedMessages.isClosed()) {
                    if (this.info.isBrowser() || !session.connection.isDuplicate(this, md.getMessage())) {
                        if (listener != null && unconsumedMessages.isRunning()) {
                            if (redeliveryExceeded(md)) {
                                posionAck(md, "listener dispatch[" + md.getRedeliveryCounter() + "] to " + getConsumerId() + " exceeds redelivery policy limit:" + redeliveryPolicy);
                                return;
                            }
                            ActiveMQMessage message = createActiveMQMessage(md);
                            beforeMessageIsConsumed(md);
                            try {
                                boolean expired = isConsumerExpiryCheckEnabled() && message.isExpired();
                                // 消息异步处理通过onMessage钩子方法进行处理，这个是最终的处理位置
                                if (!expired) {
                                    listener.onMessage(message);
                                }
                                afterMessageIsConsumed(md, expired);
                            } catch (RuntimeException e) {
                                LOG.error("{} Exception while processing message: {}", getConsumerId(), md.getMessage().getMessageId(), e);
                                md.setRollbackCause(e);
                                if (isAutoAcknowledgeBatch() || isAutoAcknowledgeEach() || session.isIndividualAcknowledge()) {
                                    // schedual redelivery and possible dlq processing
                                    rollback();
                                } else {
                                    // Transacted or Client ack: Deliver the next message.
                                    afterMessageIsConsumed(md, false);
                                }
                            }
                        } else {
                            if (!unconsumedMessages.isRunning()) {
                                // delayed redelivery, ensure it can be re delivered
                                session.connection.rollbackDuplicate(this, md.getMessage());
                            }

                            if (md.getMessage() == null) {
                                // End of browse or pull request timeout.
                                unconsumedMessages.enqueue(md);
                            } else {
                                if (!consumeExpiredMessage(md)) {
                                    unconsumedMessages.enqueue(md);
                                    if (availableListener != null) {
                                        availableListener.onMessageAvailable(this);
                                    }
                                } else {
                                    beforeMessageIsConsumed(md);
                                    afterMessageIsConsumed(md, true);

                                    // Pull consumer needs to check if pull timed out and send
                                    // a new pull command if not.
                                    if (info.getCurrentPrefetchSize() == 0) {
                                        unconsumedMessages.enqueue(null);
                                    }
                                }
                            }
                        }
                    } else {
                        // deal with duplicate delivery
                        ConsumerId consumerWithPendingTransaction;
                        if (redeliveryExpectedInCurrentTransaction(md, true)) {
                            LOG.debug("{} tracking transacted redelivery {}", getConsumerId(), md.getMessage());
                            if (transactedIndividualAck) {
                                immediateIndividualTransactedAck(md);
                            } else {
                                session.sendAck(new MessageAck(md, MessageAck.DELIVERED_ACK_TYPE, 1));
                            }
                        } else if ((consumerWithPendingTransaction = redeliveryPendingInCompetingTransaction(md)) != null) {
                            LOG.warn("{} delivering duplicate {}, pending transaction completion on {} will rollback", getConsumerId(), md.getMessage(), consumerWithPendingTransaction);
                            session.getConnection().rollbackDuplicate(this, md.getMessage());
                            dispatch(md);
                        } else {
                            LOG.warn("{} suppressing duplicate delivery on connection, poison acking: {}", getConsumerId(), md);
                            posionAck(md, "Suppressing duplicate delivery on connection, consumer " + getConsumerId());
                        }
                    }
                }
            }
            if (++dispatchedCount % 1000 == 0) {
                dispatchedCount = 0;
                Thread.yield();
            }
        } catch (Exception e) {
            session.connection.onClientInternalException(e);
        }
    }    
```

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

从上面的代码可以看出，创建的connection数量的限制会受限于PooledConnectionFactory.setMaxConnections()方法(默认是8)。

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
1. ActiveMQ in Action.pdf, Bruce Snyder、Dejan Bosanac and Rob Davies
2. ActiveMQ 源码分享-Producer, 何晓娟
3. java消息服务（第二版）, 闫怀志译