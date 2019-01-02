## 关于RxJava

> RxJava — Reactive Extensions for the JVM — a library for comosing asynchronous and event-based programs using observable sequences for the Java VM. ——github#RxJava

RxJava是一个使用可观测序列来构成异步的和基于事件的程序的库，其主要的是实现异步操作。

## 主要概念

使用RxJava首先了解这几个概念：

- Observable 被观察者
- Observer 观察者
- subscribe 订阅

举个可能不太恰当的例子，类似于间谍这么一种场景。正派需要知道反派的一举一动，根据反派的作为而采取相应的行动，如何知道反派的行为可以安插间谍。

在以上的场景中，正派就相当于`Observer`，而反派就相当于`Observable`，安插间谍这个行为就相当于`subscribe`。看到这，回到RxJava所介绍那里，可以想到，正派和反派所做的事可以是不一样的，你不干涉我我不干涉你，只有当反派做出了正派所期望的某些行为后，正派才会做出相应的措施，所以这个异步性可以体现在这里。

将这个场景再具体化为这样子：反派会经营一些黑色产业，比如赌博和贩毒，为了打击反派，正派安插了间谍，目的是当发现反派在贩毒的时候将其一网打击。

将这些信息用RxJava的形式表示的话为：

- 创建一个Observable（反派），**可以经营赌博、贩毒**
- 创建一个Observer（正派），间接地通过间谍观察到反派作为，然后在反派贩毒的时候**将其抓获**。
- subscribe（间谍动作），在反派贩毒时**通知正派**。

~~~java

Observable<String> villain = new Observable<String>() {

            /**
             *
             * @param observer 这个相当于正派安插在反派当中的间谍
             */
            @Override
            protected void subscribeActual(Observer<? super String> observer) {
                // 反派可以执行的动作
                gamble();

                if (trafficking()) {
                    observer.onNext("我是间谍，反派正在贩毒");
                }
                
                // 二次贩毒
                trafficking();
                observer.onNext("反派再次贩毒！");
                // 完成
                observer.onComplete();
                // ……
                observer.onNext("再次报告情况，但由于前面调用了onComplete，正派不再接收消息");
            }

            private boolean trafficking() {
                // 贩毒
                return true;
            }

            private void gamble() {
                // 赌博
            }
        };

        Observer<String> police = new Observer<String>() {
            @Override
            public void onSubscribe(Disposable d) {
                // 在安插间谍之前做一些准备工作
            }

            @Override
            public void onNext(String s) {
                // 收到间谍带回来的消息,根据这个消息可以进行相关的逻辑操作
            }

            @Override
            public void onError(Throwable e) {
                // 中间环节出错，例如间谍可能被识破
            }

            @Override
            public void onComplete() {
                // 整个行动已经结束
            }
        };
        
        // 完成安插间谍动作
        villain.subscribe(police);
        
~~~

通过这个例子可以更容易理解RxJava中观察者和被观察者的概念。这里需要注意的是，间谍调用`onComplete`方法后，正派将不再接受间谍带回来的消息，但间谍还是可以继续发送消息。

## 简单使用

[官方文档](https://github.com/ReactiveX/RxJava/wiki/How-To-Use-RxJava)将其使用情况分为三个步骤：创建、转换及出错处理。

### 创建

可以使用`Observable.create()`方法创建，也可以使用`RxJava`提供的一些[操作符](https://github.com/ReactiveX/RxJava/wiki/Creating-Observables)创建带有修饰性的`Observable`。上面的例子采用`create()`可以替代`new Observable`写为：

~~~java
Observable.create(new ObservableOnSubscribe<String>() {
            @Override
            public void subscribe(ObservableEmitter<String> emitter) throws Exception {
                // 反派可以执行的动作
                // ……
                emitter.onNext("间谍通知消息");
            }
        });
~~~

这是默认线程同步的，也可以采用异步：

~~~java
Observable<String> observable = new Observable<String>() {
            @Override
            protected void subscribeActual(Observer<? super String> observer) {
                new Thread(new Runnable() {
                    @Override
                    public void run() {
                        // 模拟下载图片
                        observer.onNext("图片下载完成");
                    }
                }).start();
            }
        };
~~~

看了这么些例子，可以知道`Observable`是需要提供数据源给`Observer`使用的。所以，如果已经存在了数据源，可以使用`just()`或`from`来创建，创建出来的`Observable`会同步调用`onNext()`和`onComplete()`方法依次发射给订阅了的`Observer`。

~~~java
String[] source = new String[]{"A", "B", "C"};
        Observable.just("A", "B", "C").subscribe(observer);
        Observable.fromArray(source).subscribe(observer);
        // from方法有fromArray、fromIterable、fromCallable等方法
~~~

### 转换

指的是转换`Observable`。RxJava支持链式调用，可以很方便转化和构建`Observable`。下面是来自官方的例子：

~~~java
Integer[] list = new Integer[20];
        for (int i = 0; i < list.length; i++) {
            list[i] = i;
        }
        Observable.fromArray(list)
                .skip(10)
                .take(5)
                .map(new Function<Integer, String>() {
                    @Override
                    public String apply(Integer integer) throws Exception {
                        return integer + "_xfrom";
                    }
                })
                .subscribe(s -> Log.d(TAG, "onNext => " + s));
                
                // Output:
                // onNext => 10_xform
                // onNext => 11_xform
                // onNext => 12_xform
                // onNext => 13_xform
                // onNext => 14_xform
~~~

其转换过程如下：

![transform](BCF2FF51298A42628F1044B66F07FE6A)

这个转换过程为跳过开头的十个数据，取接下来的5个数据，将`Integer`类型转化为`String`类型。

### 错误处理

这里的处理和调用`onNext`方法是一样的，可以在`Observer`中写处理出错时的逻辑，然后在`Observable`中在发生错误的地方调用`onError`

## RxJava一些常用方法

为了方便用户使用，RxJava写了很多方法以供使用，比如创建、转化等操作。

在线程的切换上，在前面的例子中是在`Observable`创建中`new`了一个线程，也可以使用RxJava提供的线程转换方法。

- observeOn()
- subscribeOn()

这两个方法都可以转换线程，`observeOn()`表示观察者消费事件所在的线程，而`subscribeOn()`表示被观察者生产事件源所在的线程。在android中，UI更新在主线程中处理，以下是更新图片的例子：

~~~java
Observable.create(new ObservableOnSubscribe<Drawable>() {
            @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
            @Override
            public void subscribe(ObservableEmitter<Drawable> emitter) throws Exception {
                Drawable drawable = getResources().getDrawable(R.drawable.transform, null);
                emitter.onNext(drawable);
            }
        }).subscribeOn(Schedulers.io())
                .observeOn(AndroidSchedulers.mainThread)
                .subscribe(drawable -> imageView.setImageDrawable(drawable));
~~~

`Schedulers`是RxJava中基于`Executor`而创建`Scheduler`实例的类，用于线程服务。利用`Schedulers`可以生成5种实例：

- single 可共享的单线程
- computation 用于计算的线程
- io 用于IO操作的
- trampoline 用于延迟工作任务，它的工作队列会按照FIFO的方式处理
- newThread 创建一个新的线程

`subscribe`有多个重载方法，三个参数的分别表示`onNext`、`onComplete`和`onError`方法。

~~~java
Observable.just("Hello", "Hi")
                .subscribe(s -> Log.d(TAG, "onNext: " + s),
                        e -> Log.d(TAG, "onError: " + e.getMessage()),
                        () -> Log.d(TAG, "onComplete "));
~~~

除了上面使用的`Observable`，RxJava还有其他几种被观察者：

![Publisher-Subscriber-class-relation](124667CF9EF7490489885E6BA2725B94)

> [图片来源](https://zouzhberk.github.io/rxjava-study/)

- Flowable

RxJava2.0出现，与`Observable`相比支持背压。背压情况就是观察者的处理速度跟不上被观察发送事件的速度，从而导致事件丢失、缓冲区溢出等情况的发生，建议使用`Flowable`替代`Observable`。

![Flowable创建](AB005BB41D14447BBD138EA246BD91E0)

创建`Flowable`需要两个参数，第2个表示当背压情况发生时采取的策略。跟进代码发现`BackpressureStrategy`是个枚举类，其策略有`MISSING`、`BUFFER`和`DROP`等几种。比如`BUFFER`表示当观察者处理不过来时，后面的数据会用缓存保存起来。

- Single

只能发射一个数据或者一个异常，其发射器为`SingleObserver`，其中并没有`onComplete`方法。

- Completable

只能发射完成一个通知或者异常，其发射器为`CompletableEmitter`。

- Maybe

可以发射0条或1条数据和一个异常通或者完成通知，可看成是Single和Completable的结合体。

这么划分是因为有时候并不需要处理很多数据流，有可能只是一个通知或者一条数据。