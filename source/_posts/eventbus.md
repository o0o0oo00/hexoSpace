---
title: Eventbus浅析
date: 2018-10-25 15:53:05
tag: technology
categories: 技术
---

### 注册EventBus
`EventBus.getDefault().register(this);`  

我们来看看**`getDefault()`**中有什么

```
/** Convenience singleton for apps using a process-wide EventBus instance. */
public static EventBus getDefault() {
    EventBus instance = defaultInstance;
    if (instance == null) {
        synchronized (EventBus.class) {
            instance = EventBus.defaultInstance;
            if (instance == null) {
                instance = EventBus.defaultInstance = new EventBus();
            }
        }
    }
    return instance;
}

```
很明显是一个**`DoubleCheck`**单例模式，具体的EventBus构造方法没贴，后面会介绍他其中的一些属性。
  
<!--more-->
获取完了对象紧接着对当前activity或fragment进行了注册**register**

```
public void register(Object subscriber) {
    Class<?> subscriberClass = subscriber.getClass();
    List<SubscriberMethod> subscriberMethods = subscriberMethodFinder.findSubscriberMethods(subscriberClass);
    synchronized (this) {
        for (SubscriberMethod subscriberMethod : subscriberMethods) {
            subscribe(subscriber, subscriberMethod);
        }
    }
}

```

我们可以看到，传入的`this`可以是Activity，进而得到了其class，通过**`findSubscriberMethods`**找到所有在这个class下的**Subscriber**们存放在一个List集合中，然后**逐个进行事件订阅**。

关键方法 **findSubscriberMethods** 

subscriberClass = 订阅类的class 

```
List<SubscriberMethod> findSubscriberMethods(Class<?> subscriberClass) {
    List<SubscriberMethod> subscriberMethods = METHOD_CACHE.get(subscriberClass);
    if (subscriberMethods != null) {
        return subscriberMethods;
    }
	 //是否忽略注解器生成的MyEventBusIndex类 默认false
    if (ignoreGeneratedIndex) {
        subscriberMethods = findUsingReflection(subscriberClass);
    } else {
        subscriberMethods = findUsingInfo(subscriberClass);
    }
    if (subscriberMethods.isEmpty()) {
        throw new EventBusException("Subscriber " + subscriberClass
                + " and its super classes have no public methods with the @Subscribe annotation");
    } else {
        METHOD_CACHE.put(subscriberClass, subscriberMethods);
        return subscriberMethods;
    }
}
```
其中第一行的`METHOD_CACHE`作为缓存Map，以订阅类的class为key，订阅的方法集合作为value。
`private static final Map<Class<?>, List<SubscriberMethod>> METHOD_CACHE = new ConcurrentHashMap<>();` 

如果没有缓存的时候，会判断是否**忽略生成index**，默认是false，那么我们来看一下`findUsingInfo(subscriberClass);`这个真正去查找该class下订阅方法的方法。

先看一下FindState是什么东西。他里面存储了一些订阅者和订阅方法信息

```
static class FindState {
	//订阅方法集合，
    final List<SubscriberMethod> subscriberMethods = new ArrayList<>();
    //以event为key，以method为value
    final Map<Class, Object> anyMethodByEventType = new HashMap<>();
    //以method的名字生成一个methodKey为key，该method的类(订阅者)为value
    final Map<String, Class> subscriberClassByMethodKey = new HashMap<>();
    final StringBuilder methodKeyBuilder = new StringBuilder(128);

    Class<?> subscriberClass;
    Class<?> clazz;
    boolean skipSuperClasses;
    SubscriberInfo subscriberInfo;

```
这个方法返回的是我们最终需要的`List<SubscriberMethod>`

```
private List<SubscriberMethod> findUsingInfo(Class<?> subscriberClass) {
    FindState findState = prepareFindState();
    // 利用findState辅助来获取订阅方法
    findState.initForSubscriber(subscriberClass);
    while (findState.clazz != null) {
        // 获取class对应的subscriberInfo
        findState.subscriberInfo = getSubscriberInfo(findState);
        if (findState.subscriberInfo != null) {
            // 获取订阅方法数组
            SubscriberMethod[] array = findState.subscriberInfo.getSubscriberMethods();
            for (SubscriberMethod subscriberMethod : array) {
                if (findState.checkAdd(subscriberMethod.method, subscriberMethod.eventType)) {
                    findState.subscriberMethods.add(subscriberMethod);
                }
            }
        } else {
            // subscriberInfo为null 则通过反射获取
            findUsingReflectionInSingleClass(findState);
        }
        //接着获取父类的订阅方法
        findState.moveToSuperclass();
    }
    return getMethodsAndRelease(findState);
}
```


### 方法订阅（注册方法）

其实上面这些就是为了获取到当前class 的所有订阅方法。然后遍历这些个订阅方法来完成订阅。接下来看一下**逐个订阅**  

```
for (SubscriberMethod subscriberMethod : subscriberMethods) {
    subscribe(subscriber, subscriberMethod);
}
```
首先**映入眼帘**的是两个参数，一个**订阅Class**一个是订阅类中的要**订阅的方法**。

上面把要订阅的方法封装成了`SubscriberMethod `。并且这个有一个事件类型（订阅方法的参数）`eventType`需要使用

```
public class SubscriberMethod {
    final Method method;
    final ThreadMode threadMode;
    final Class<?> eventType;
    final int priority;
    final boolean sticky;
    /** Used for efficient comparison */
    String methodString;

```
看订阅方法

```
// Must be called in synchronized block
private void subscribe(Object subscriber, SubscriberMethod subscriberMethod) {
    
	//订阅方法的参数类型，也是事件类型
	Class<?> eventType = subscriberMethod.eventType;
    // 封装订阅者和订阅方法
    Subscription newSubscription = new Subscription(subscriber, subscriberMethod);
    // 根据事件类型key，获取subscriptions （Map<Class<?>, CopyOnWriteArrayList<Subscription>>） 
    CopyOnWriteArrayList<Subscription> subscriptions = subscriptionsByEventType.get(eventType);
    // 如果subscriptions集合为空，说明该事件类型没有注册，则创建一个新的集合。并把事件类型`eventType`作为key 和订阅方法集合作为value加入其中
    if (subscriptions == null) {
        subscriptions = new CopyOnWriteArrayList<>();
        subscriptionsByEventType.put(eventType, subscriptions);
    } else {
        if (subscriptions.contains(newSubscription)) {
            throw new EventBusException("Subscriber " + subscriber.getClass() + " already registered to event "
                    + eventType);
        }
    }

    int size = subscriptions.size();
    // 按照subscriberMethod的优先级插入newSubscription到正确的位置
    for (int i = 0; i <= size; i++) {
        // 最后一个直接插入，或者是要插入的method优先级大于 i 位置 的优先级
        if (i == size || subscriberMethod.priority > subscriptions.get(i).subscriberMethod.priority) {
            subscriptions.add(i, newSubscription);
            break;
        }
    }

    // 存放订阅Method的事件类型
    List<Class<?>> subscribedEvents = typesBySubscriber.get(subscriber);
    if (subscribedEvents == null) {
        subscribedEvents = new ArrayList<>();
        typesBySubscriber.put(subscriber, subscribedEvents);
    }
    subscribedEvents.add(eventType);

    // 黏性事件
    if (subscriberMethod.sticky) {
        if (eventInheritance) {
            // Existing sticky events of all subclasses of eventType have to be considered.
            // Note: Iterating over all events may be inefficient with lots of sticky events,
            // thus data structure should be changed to allow a more efficient lookup
            // (e.g. an additional map storing sub classes of super classes: Class -> List<Class>).
            Set<Map.Entry<Class<?>, Object>> entries = stickyEvents.entrySet();
            for (Map.Entry<Class<?>, Object> entry : entries) {
                Class<?> candidateEventType = entry.getKey();
                if (eventType.isAssignableFrom(candidateEventType)) {
                    Object stickyEvent = entry.getValue();
                    checkPostStickyEventToSubscription(newSubscription, stickyEvent);
                }
            }
        } else {
            Object stickyEvent = stickyEvents.get(eventType);
            checkPostStickyEventToSubscription(newSubscription, stickyEvent);
        }
    }
}
```
**方法说明**
>
* 首先获取了方法参数类型 ： `eventType`
* 根据事件类型`eventType`，获取其所有订阅方法 ：`subscriptionsByEventType.get(eventType);`
* subscriptions如果已经被初始化了，那么按照优先级将新的method插入其中，如果没有初始化，那么就`new CopyOnWriteArrayList<>();`
* 获取订阅者所有的订阅事件类型 ： `typesBySubscriber.get(subscriber);`
* 如果初始化了，那么就直接添加这个`eventType `到对应的集合中，如果没有那么初始化并将**`Subscriber`（还记得参数传入的就是这个吗）**作为key，**`eventType `集合**作为value


**不太明白没关系，我们再接着往下看。。。**

### 事件发布
`EventBus.getDefault().post(new UpdateUIEvent());`

post：

```
/** Posts the given event to the event bus. */
public void post(Object event) {
    PostingThreadState postingState = currentPostingThreadState.get();
    List<Object> eventQueue = postingState.eventQueue;
    // 将事件添加进当前线程的事件队列
    eventQueue.add(event);
	 // 判断当前线程是否正在发布事件
    if (!postingState.isPosting) {
        postingState.isMainThread = isMainThread();
        postingState.isPosting = true;
        if (postingState.canceled) {
            throw new EventBusException("Internal error. Abort state was not reset");
        }
        try {
            while (!eventQueue.isEmpty()) {
                postSingleEvent(eventQueue.remove(0), postingState);
            }
        } finally {
            postingState.isPosting = false;
            postingState.isMainThread = false;
        }
    }
}

```

首先`currentPostingThreadState`是一个`ThreadLocal<PostingThreadState>`

ThreadLocal 是一个线程内部的数据存储类，通过它可以在指定的线程中存储数据，而这段数据是不会与其他线程共享的。可以看出currentPostingThreadState的实现是一个包含了PostingThreadState的ThreadLocal对象,这样可以保证取到的都是自己线程对应的数据。

```
/** For ThreadLocal, much faster to set (and get multiple values). */
final static class PostingThreadState {
    final List<Object> eventQueue = new ArrayList<>();//当前线程的事件队列
    boolean isPosting; //是否有事件正在分发
    boolean isMainThread;//post的线程是否是主线程
    Subscription subscription;
    Object event;
    boolean canceled;
}
```
PostingThreadState中包含了当前线程的事件队列，就是当前线程所有分发的事件都保存在eventQueue事件队列中以及订阅者订阅事件等信息，有了这些信息我们就可以从事件队列中取出事件分发给对应的订阅者。


我们看到了把当前的事件加入到了事件队列尾部，如果事件队列不为empty，那么就一直发送里面的事件直到为empty。我们看一下关键代码`postSingleEvent(eventQueue.remove(0), postingState);`  

```
private void postSingleEvent(Object event, PostingThreadState postingState) throws Error {
    Class<?> eventClass = event.getClass();
    boolean subscriptionFound = false;
    if (eventInheritance) {
        List<Class<?>> eventTypes = lookupAllEventTypes(eventClass);
        int countTypes = eventTypes.size();
        for (int h = 0; h < countTypes; h++) {
            Class<?> clazz = eventTypes.get(h);
            subscriptionFound |= postSingleEventForEventType(event, postingState, clazz);
        }
    } else {
        subscriptionFound = postSingleEventForEventType(event, postingState, eventClass);
    }
    if (!subscriptionFound) {
        if (logNoSubscriberMessages) {
            logger.log(Level.FINE, "No subscribers registered for event " + eventClass);
        }
        if (sendNoSubscriberEvent && eventClass != NoSubscriberEvent.class &&
                eventClass != SubscriberExceptionEvent.class) {
            post(new NoSubscriberEvent(this, event));
        }
    }
}
```
首先判断的是`eventInheritance `是否开启了继承，由于EventBus创建的是默认的Builder，所以默认值为初始值true。那么会找到它所有的父类，然后依次发送事件。关键在于方法**`postSingleEventForEventType(event, postingState, eventClass);`**  

```
private boolean postSingleEventForEventType(Object event, PostingThreadState postingState, Class<?> eventClass) {
    CopyOnWriteArrayList<Subscription> subscriptions;
    synchronized (this) {
        // 根据事件类型找出相关的订阅信息
        subscriptions = subscriptionsByEventType.get(eventClass);
    }
    if (subscriptions != null && !subscriptions.isEmpty()) {
        for (Subscription subscription : subscriptions) {
            postingState.event = event;
            postingState.subscription = subscription;
            boolean aborted = false;
            try {
                // 发布到具体的订阅者
                postToSubscription(subscription, event, postingState.isMainThread);
                aborted = postingState.canceled;
            } finally {
                postingState.event = null;
                postingState.subscription = null;
                postingState.canceled = false;
            }
            if (aborted) {
                break;
            }
        }
        return true;
    }
    return false;
}
```

我们可以看到，如果根据事件类型找不到订阅方法，或者订阅方法为空，那么会返回false，上面抛出异常`No subscribers registered for event`。那如果找到了订阅的方法呢。

```
private void postToSubscription(Subscription subscription, Object event, boolean isMainThread) {
    switch (subscription.subscriberMethod.threadMode) {
        // 订阅线程跟随发布线程
        case POSTING:
            // 订阅线程和发布线程相同，直接订阅
            invokeSubscriber(subscription, event);
            break;
        // 订阅线程为主线程
        case MAIN:
            // 发布线程和订阅线程都是主线程，直接订阅
            if (isMainThread) {
                invokeSubscriber(subscription, event);
            }
            // 发布线程不是主线程，订阅线程切换到主线程订阅
            else {
                mainThreadPoster.enqueue(subscription, event);
            }
            break;
        case MAIN_ORDERED:
            if (mainThreadPoster != null) {
                mainThreadPoster.enqueue(subscription, event);
            }
            else {
                // temporary: technically not correct as poster not decoupled from subscriber
                invokeSubscriber(subscription, event);
            }
            break;
        // 订阅线程为后台线程
        case BACKGROUND:
            // 发布线程为主线程，切换到后台线程订阅
            if (isMainThread) {
                backgroundPoster.enqueue(subscription, event);
            }
            // 发布线程不为主线程，直接订阅
            else {
                invokeSubscriber(subscription, event);
            }
            break;
        // 订阅线程为异步线程
        case ASYNC:
            // 使用线程池线程订阅
            asyncPoster.enqueue(subscription, event);
            break;
        default:
            throw new IllegalStateException("Unknown thread mode: " + subscription.subscriberMethod.threadMode);
    }
}
```
* POSTING：事件发布在什么线程，就在什么线程订阅。
* MAIN：无论事件在什么线程发布，都在主线程订阅。
* BACKGROUND：如果发布的线程不是主线程，则在该线程订阅，如果是主线程，则使用一个单独的后台线程订阅。
* ASYNC：在非主线程和发布线程中订阅。

我们知道了根据不同类型进行区分。那么关键是如何调用Subscribe方法的呢？
`invokeSubscriber(subscription, event);`

```
void invokeSubscriber(Subscription subscription, Object event) {
    try {
        subscription.subscriberMethod.method.invoke(subscription.subscriber, event);
    } catch (InvocationTargetException e) {
        handleSubscriberException(subscription, event, e.getCause());
    } catch (IllegalAccessException e) {
        throw new IllegalStateException("Unexpected exception", e);
    }
}

```
这样订阅者收到了时间，调用了订阅方法。event参数也成功传入。

### 反注册

```
/** Unregisters the given subscriber from all event classes. */
public synchronized void unregister(Object subscriber) {
    List<Class<?>> subscribedTypes = typesBySubscriber.get(subscriber);
    if (subscribedTypes != null) {
        for (Class<?> eventType : subscribedTypes) {
            unsubscribeByEventType(subscriber, eventType);
        }
        typesBySubscriber.remove(subscriber);
    } else {
        logger.log(Level.WARNING, "Subscriber to unregister was not registered before: " + subscriber.getClass());
    }
}

```

我们在注册的时候知道 **`typesBySubscriber`**是一个存储订阅者，订阅事件的Map  
`private final Map<Object, List<Class<?>>> typesBySubscriber;`。  
先解绑list中的元素再讲Subscribe也从map中移除。我们看关键代码`unsubscribeByEventType(subscriber, eventType);` 

```
/** Only updates subscriptionsByEventType, not typesBySubscriber! Caller must update typesBySubscriber. */
private void unsubscribeByEventType(Object subscriber, Class<?> eventType) {
    List<Subscription> subscriptions = subscriptionsByEventType.get(eventType);
    if (subscriptions != null) {
        int size = subscriptions.size();
        for (int i = 0; i < size; i++) {
            Subscription subscription = subscriptions.get(i);
            if (subscription.subscriber == subscriber) {
                subscription.active = false;
                subscriptions.remove(i);
                i--;
                size--;
            }
        }
    }
}

```

多看几遍源码就OK了。 (●ﾟωﾟ●)，接下来尝试着自己写一个简版的EventBus  
[观察者模式](https://o0o0oo00.github.io/2018/10/09/Observer/#more)

参考链接：


* https://juejin.im/post/5a3e19c26fb9a0452207b6b5
* https://juejin.im/post/58f5c86a8d6d810057c12975