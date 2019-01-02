# 关于EventBus

> Event bus for Android and Java that simplifies communication between Activities,Fragments,Threads,Services,etc.Less code,better quality.

`EventBus`是提供给Android和Java用以简化activity、fragment和线程间等模块间的通信的开源库。它基于发布/订阅的模式。

![EventBus结构图](71F9524F1D204BB4874B9F439EE7E0BC)

# 使用

可分为三个步骤走：

1. 定义事件

~~~java
class MessageEvent {
        String name;

        public MessageEvent(String name) {
            this.name = name;
        }
    }
~~~

在定义的类中准备所要使用的一些字段，比如这里的name。

2. 准备订阅

要注册以及在合适的地方注销订阅者。比如在安卓中可以这么做：

~~~java
@Override
 public void onStart() {
     super.onStart();
     EventBus.getDefault().register(this);
 }

 @Override
 public void onStop() {
     super.onStop();
     EventBus.getDefault().unregister(this);
 }
~~~

然后声明订阅方法：

~~~java
    @Subscribe(threadMode = ThreadMode.MAIN)
    void handleMessage(MessageEvent event) {
        Toast.makeText(this, event.name, Toast.LENGTH_SHORT).show();
    }
~~~

在订阅的方法中执行相应的逻辑，可以用注解指定这个订阅的方法在什么样的线程中执行。

查看代码发现`ThreadMode`有5种模式：

- POSTING

默认值，订阅方法被调用时的线程和发布时所在的线程一样。因此，事件处理不应太长，避免影响后面发布的事件。

- MAIN

订阅时间会在`Android`的主线程中调用。如果发布线程是主线程会被阻塞发布，订阅事件会直接调用，如果不是则会排队分发。处理事件也不宜过长，避免阻塞主线程。

- MAIN_ORDERED

不同于MAIN的地方在于发布时都会排队，避免阻塞发布线程。

- BACKGROUND

订阅事件会在后台线程处理。如果发布线程不是主线程，则会直接调用订阅方法。如果发布在主线程，则使用同一个线程按照事件顺序进行分发。事件处理不宜太慢，不在android中使用的话则经常使用该模式。

- ASYNC

事件处理会被分配到单独一个线程，每个事件开启一个，但要避免数量过多。

3. 发布事件

~~~java
    EventBus.getDefault().post(new MessageEvent("LDG"));
~~~

当发布了之后，订阅方法就会被调用。所在在信息交换方面可以方便，切换线程的话可以直接借助注解的`threadMode`，不必借助广播或者`Handler`来完成。