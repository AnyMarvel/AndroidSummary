#初探RxJava以及结合Retrofit的使用
前言
android当前最主流的网络请求框架莫过于RxJava2+Retrofit2+OkHttp3，之前的项目中也曾经使用过Retrofit进行网络请求，二次封装后使用非常方便。但为解决一些嵌套网络请求存在的问题，这次的新项目打算在网络请求中使用RxJava，体验一下响应式编程的感觉。
首先强力推荐扔物线的经典博客 给 [Android 开发者的 RxJava 详解](http://gank.io/post/560e15be2dca930e00da1083)

##观察者模式

---
既然要使用RxJava，就不得不简单介绍一下观察者模式，因为RxJava作为一个工具库，使用的就是拓展形式的观察者模式。

- 观察者模式：简单来说，观察者模式就是一个对象A (观察者) 和一个对象B (被观察者) 达成了一种 订阅 关系，当对象B触发了某个事件时，比如一些数据的变化，就会立刻通知对象A，对象A接到通知后作出反应。这样做的好处就是对象A不用实时监控对象B的状态，只需在对象B发生特定事件时再作出响应即可。



- Android中的观察者模式：举一个在开发中最常见的观察者模式的例子OnClickListener

```java
button.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        //点击事件发生后的操作
    }
});

```
在这个例子中button是被观察者,View.OnClickListener是观察者，而setOnClickListener()方法即让二者达成了一种订阅关系，这样的效果就是：当button触发了被点击的事件后，会通知观察者,而OnClickListener接收到通知后，就会调用自身接口中的onClick()作为对点击事件的响应。


##介绍
首先我们来看一下官方在github上对RxJava的介绍：

>RxJava is a Java VM implementation of Reactive Extensions: a library for composing asynchronous and
event-based programs by using observable sequences.(一个通过使用可观察序列来组成异步的、基于事件的程序的库。)

从介绍中我们可以提取出一个关键词：异步,但安卓中已经有很多解决异步操作的方法了，比如Handler和AsyncTask等,为什么还选择RxJava呢，其实目的就是为了让代码更简洁，而且它的简洁是与众不同的，因为RxJava的使用方式是基于事件流的链式调用，这就保证了随着程序复杂性的提高，RxJava依然能保持代码的简洁和优雅。具体的例子将在后面结合Retrofit展示。

因为RxJava是基于一种拓展的观察者模式，所以也有观察者、被观察者、订阅、事件几个基本概念,这里借用大神Carson_Ho博客中的例子解释一下概念和基本原理。



|角色|作用|类比|
|-|-|-|
|Observable(被观察者)|事件的产生者|顾客|
|Observer(观察者)|事件消费者，接收事件后作出响应|厨师|
|Subscribe(订阅)|将Observable和Observer连接在一起|服务员|
|Event(事件)|Observable通知Observer的载体|菜品|

![](https://user-gold-cdn.xitu.io/2018/2/3/1615bebf81cda7f9?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

##基本实现
- 依赖添加：开始实践之前记得在gradle中添加依赖(本文发布时的RxJava2.0的最新版本)。

```shell
compile "io.reactivex.rxjava2:rxjava:2.1.1"
compile 'io.reactivex.rxjava2:rxandroid:2.0.1'


```
- Observable(被观察者) 的创建：

```Java
Observable<String> observable = Observable.create(new ObservableOnSubscribe<String>() {
    @Override
    public void subscribe(ObservableEmitter<String> e) throws Exception {

        e.onNext("事件1");
        e.onNext("事件2");
        e.onNext("事件3");
        e.onComplete();
    }
});

```

- Observable.create(): 创建一个Observable的最基本方法，也可以通过just()、from()等方法来简化操作。
- ObservableOnSubscribe<>(): 一个接口，在复写的subscribe（）里定义需要发送的事件。
- ObservableEmitter: 这是RxJava2中新推出的类，可以理解为发射器，用于发射数据onNext()和通知onComplete()/onError()。

Observer(观察者) 的创建：

```Java

Observer<String> observer = new Observer<String>() {

    @Override
    public void onSubscribe(Disposable d) {

        Log.d(TAG, "onSubscribe: 达成订阅");
    }

    @Override
    public void onNext(String s) {

        Log.d(TAG, "onNext: 响应了"+s);
    }

    @Override
    public void onError(Throwable e) {

        Log.d(TAG, "onError: 执行错误");
    }

    @Override
    public void onComplete() {

        Log.d(TAG, "onComplete: 执行完成");
    }
};


```
- onNext():普通事件，通过重写进行响应即可。
- onError():错误事件，当队列中事件处理出现异常时，就会调用该方法，此后不再有事件发出。
- onComplete():完成事件，当队列中的事件全部处理完成后触发。
- 在一个正常的序列中，onError()和onComplete()应该只有一个并且处于事件队列的末尾，而且调用了一个就不应该再调用另一个。

除了Observer之外，RxJava 还内置了一个实现了Observer的抽象类：Subscriber,两者的使用方式基本一样，主要区别在于Subscriber中新增了两个方法，下面引用一下扔物线对这两个方法的解释：

>1、onStart(): 这是 Subscriber 增加的方法。它会在 subscribe 刚开始，而事件还未发送之前被调用，可以用于做一些准备工作，例如数据的清零或重置。这是一个可选方法，默认情况下它的实现为空。需要注意的是，如果对准备工作的线程有要求（例如弹出一个显示进度的对话框，这必须在主线程执行）， onStart() 就不适用了，因为它总是在 subscribe 所发生的线程被调用，而不能指定线程。要在指定的线程来做准备工作，可以使用 doOnSubscribe() 方法。


>2、unsubscribe(): 这是 Subscriber 所实现的另一个接口 Subscription 的方法，用于取消订阅。在这个方法被调用后，Subscriber 将不再接收事件。一般在这个方法调用前，可以使用 isUnsubscribed() 先判断一下状态。 unsubscribe() 这个方法很重要，因为在 subscribe() 之后， Observable 会持有 Subscriber 的引用，这个引用如果不能及时被释放，将有内存泄露的风险。所以最好保持一个原则：要在不再使用的时候尽快在合适的地方（例如 onPause() onStop() 等方法中）调用 unsubscribe() 来解除引用关系，以避免内存泄露的发生。


- 二者达成 subscribe(订阅) 关系：
```Java
observable.subscribe(observer);


```
- 链式调用
上面的三步过程可以通过一条链式结构直接调用，使得代码变得更简洁。

```Java
Observable.create(new ObservableOnSubscribe<String>() {
    @Override
    public void subscribe(ObservableEmitter<String> e) throws Exception {

        e.onNext("事件1");
        e.onNext("事件2");
        e.onNext("事件3");
        e.onComplete();
    }
}).subscribe(new Observer<String>() {
    @Override
    public void onSubscribe(Disposable d) {

        Log.d(TAG, "onSubscribe: 达成订阅");
    }

    @Override
    public void onNext(String s) {

        Log.d(TAG, "onNext: 响应了"+s);
    }

    @Override
    public void onError(Throwable e) {

        Log.d(TAG, "onError: 执行错误");
    }

    @Override
    public void onComplete() {

        Log.d(TAG, "onComplete: 执行完成");
    }
});


```
在logcat中可以查看打印结果

![](https://user-gold-cdn.xitu.io/2018/2/3/1615bebf81de86d7?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

##线程调度
---
在RxJava中，我们可以自行指定事件产生和事件消费的线程，可以通过RxJava中的Scheduler来实现。
Scheduler
- RxJava内置的5个Scheduler
  - Schedulers.immediate(): 直接在当前线程运行，相当于不指定线程。这是默认的 Scheduler，但是为了防止被错误使用，在RxJava2中已经被移除了。


  - Schedulers.newThread(): 开启新线程，并在新线程执行操作。


  - Schedulers.io(): I/O 操作（读写文件、读写数据库、网络信息交互等）所使用的 Scheduler。行为模式和 newThread() 差不多，区别在于 io() 的内部实现是是用一个无数量上限的线程池，可以重用空闲的线程，因此多数情况下 io() 比 newThread() 更有效率。不要把计算工作放在 io() 中，可以避免创建不必要的线程。


  - Schedulers.computation(): 计算所使用的 Scheduler，例如图形的计算。这个 Scheduler 使用的固定的线程池，大小为 CPU 核数。不要把 I/O 操作放在 computation() 中，否则 I/O 操作的等待时间会浪费 CPU。


  - Schedulers.trampoline():主要用于延迟工作任务的执行。当我们想在当前线程执行一个任务时，但并不是立即，我们可以用.trampoline()将它入队，trampoline将会处理它的队列并且按序运行队列中每一个任务。

- Android特有的Scheduler

  - AndroidSchedulers.mainThread():指定的操作将在Android的主线程中进行，如UI界面的更新操作。

###线程的控制
- subscribeOn():指定事件产生的线程，例如subscribeOn(Schedulers.io())可以指定被观察者的网络请求、文件读写等操作放置在io线程。
- observeOn():指定事件消费的线程，例如observeOn(AndroidSchedulers.mainThread())指定Subscriber中的方法在主线程中运行。

在subscribe()之前写上两句subscribeOn(Scheduler.io())和observeOn(AndroidSchedulers.mainThread())的使用方式非常常见，它适用于多数的 <后台线程取数据，主线程显示> 的程序策略。


##结合Retrofit

---
在很多时候RxJava都会结合Retrofit一起使用，而且随着程序的复杂度提高，RxJava简洁的优越性就会渐渐得到体现。下面就举几个例子来具体感受一下RxJava操作符的神奇。

**1、单独使用Retrofit**
- 首先创建Service接口，这里拿一个登录接口来举例

```Java
public interface LoginService {

    @POST("login")
    Call<ApiResponse> login(@Query("phone") String username, @Query("password") String password);
}


```
其中ApiResponse是自己定义的统一格式的返回体

- 构造Retrofit并发送请求
```Java
public void login() {

    Retrofit retrofit = new Retrofit.Builder()
            .addConverterFactory(GsonConverterFactory.create())
            .baseUrl(Config.APP_SERVER_BASE_URL)
            .build();

    retrofit.create(LoginService.class)
            .login(phone,password)
            .enqueue(new Callback<ApiResponse>() {
                @Override
                public void onResponse(Call<ApiResponse> call, Response<ApiResponse> response) {

                    //登录成功后的操作
                }

                @Override
                public void onFailure(Call<ApiResponse> call, Throwable t) {

                    //登录失败后的操作
                }
            });
}		


```
**2、结合RxJava**
- 引入依赖：记得给Retrofit添加对RxJava的适配

```shell
compile 'com.squareup.retrofit2:adapter-rxjava2:2.3.0'


```
- 创建Service接口
```Java
public interface LoginService {

    @POST("login")
    Observable<ApiResponse> rxLogin(@Query("phone") String phone, @Query("password") String password);
}


```
构造Retrofit并发送请求

```Java
public void rxLogin() {

    Retrofit retrofit = new Retrofit.Builder()
            .addConverterFactory(GsonConverterFactory.create())
            //添加对RxJava的支持
            .addCallAdapterFactory(RxJava2CallAdapterFactory.create())
            .baseUrl(Config.APP_SERVER_BASE_URL)
            .build();

    retrofit.create(LoginService.class)
            .rxLogin(phone,password)
            .subscribeOn(Schedulers.io()) //io线程获取网络请求
            .observeOn(AndroidSchedulers.mainThread()) //主线程进行数据更新
            .subscribe(new Observer<ApiResponse>() {
                @Override
                public void onSubscribe(Disposable d) {

                }

                @Override
                public void onNext(ApiResponse apiResponse) {

                    //登录成功后的操作
                }

                @Override
                public void onError(Throwable e) {

                    //登录失败后的操作
                }

                @Override
                public void onComplete() {

                }
            });

}

```


可能大家看了之后会感觉并没有什么卵用，不但没有简洁，反而增加了代码，但老哥们别着急，这只是最简单的情况，随着情况变得复杂，你就会感受到RxJava的强大之处。

**3、flatMap()解决嵌套请求**
平时在网络请求的过程中，可能会出现这样的情况，注册成功之后直接调用登录接口，如果不用RxJava，正常的想法肯定是在retrofit的onResponse回调里再嵌套一层网络请求，就像这样：

```Java
//注册相关方法
private void register() {

    retrofit.create(RetrofitService.class)
            .register(phone,password)
            .enqueue(new Callback<ApiResponse>() {
                @Override
                public void onResponse(Call<ApiResponse> call, Response<ApiResponse> response) {

                    Log.d(TAG, "onResponse: 注册成功");
                    //注册成功开始登录请求
                    login();
                }

                @Override
                public void onFailure(Call<ApiResponse> call, Throwable t) {
                    Log.d(TAG, "onFailure: 注册失败");
                }
            });


}

//登录相关方法
private void login() {

    retrofit.create(RetrofitService.class)
            .login(phone,password)
            .enqueue(new Callback<ApiResponse>() {
                @Override
                public void onResponse(Call<ApiResponse> call, Response<ApiResponse> response) {

                    Log.d(TAG, "onResponse: 登录成功");
                }

                @Override
                public void onFailure(Call<ApiResponse> call, Throwable t) {

                    Log.d(TAG, "onFailure: 登录失败");
                }
            });
}


```

实际上这种很low的嵌套网络请求正是我们需要避免的，而在RxJava中，通过flatMap()操作符可以避免这种嵌套，flatMap的作用是对传入的上一个数据流中的对象进行处理，返回下一级所要的对象的Observable包装，相当于将二维的嵌套过程线性化了，先举个最原始的例子：

```Java
private void loginAndRegister() {


	//获得被观察者实体
    RetrofitService service = retrofit.create(RetrofitService.class);
    Observable<ApiResponse> register = service.register(phone, password);
    final Observable<ApiResponse> login = service.login(phone, password);


    register.subscribeOn(Schedulers.io()) //(注册被观察者)切换到IO线程进行注册网络请求
            .observeOn(AndroidSchedulers.mainThread()) //(注册观察者)切换到主线程 处理注册网络请求的结果
            .doOnEach(new Observer<ApiResponse>() {
                @Override
                public void onSubscribe(Disposable d) {

                }

                @Override
                public void onNext(ApiResponse apiResponse) {

                    // 对第注册成功网络请求返回的结果进行操作
                    Log.d(TAG, "注册成功");
                }

                @Override
                public void onError(Throwable e) {

                }

                @Override
                public void onComplete() {

                }
            })
            //切换到IO线程去发起登录网络请求(登录被观察者经过flatMap变换后也变成了相对于注册的观察者，所以用observeOn切换线程)
            .observeOn(Schedulers.io())
            .flatMap(new Function<ApiResponse, ObservableSource<ApiResponse>>() {
                @Override
                public ObservableSource<ApiResponse> apply(ApiResponse apiResponse) throws Exception {
                    return login;
                }
            })
            //(登录观察者)切回主线程处理回调
            .observeOn(AndroidSchedulers.mainThread())
            .subscribe(new Observer<ApiResponse>() {
                @Override
                public void onSubscribe(Disposable d) {

                }

                @Override
                public void onNext(ApiResponse apiResponse) {

                    Log.d(TAG, "onNext: 登录成功");
                }

                @Override
                public void onError(Throwable e) {

                }

                @Override
                public void onComplete() {

                }
            });
}


```

可能有的老哥觉得，这样写虽然采用了链式调用，没有了嵌套，但是这代码也太长了，而且重写了很多无用的方法，别着急，下面来个优雅版的

```Java
private void loginAndRegister() {

    final RetrofitService service = retrofit.create(RetrofitService.class);

    service.register(phone, password)
            .subscribeOn(Schedulers.io())
            .observeOn(AndroidSchedulers.mainThread())
            .doOnNext(new Consumer<ApiResponse>() {
                @Override
                public void accept(ApiResponse apiResponse) throws Exception {

                    Log.d(TAG, "注册成功");
                }
            })
            .observeOn(Schedulers.io())
            .flatMap(new Function<ApiResponse, ObservableSource<ApiResponse>>() {
                @Override
                public ObservableSource<ApiResponse> apply(ApiResponse apiResponse) throws Exception {
                    return service.login(phone, password);
                }
            })
            .observeOn(AndroidSchedulers.mainThread())
            .subscribe(new Consumer<ApiResponse>() {
                @Override
                public void accept(ApiResponse apiResponse) throws Exception {

                    Log.d(TAG, "登录成功");
                }
            });
}


```

Consumer是简易版的Observer，他有多重重载，可以自定义你需要处理的信息，这里调用的是只接受onNext消息的方法，他只提供一个回调接口accept，由于没有onError和onCompete，无法在接受到onError或者onCompete之后，实现函数回调，虽然无法回调，但不代表不接收，他还是会接收到onCompete和onError之后做出默认操作，这里我还是建议大家自己对Observer再进行一下封装，使用起来会更方便。

在logcat中可以看到请求成功的打印的日志

![](https://user-gold-cdn.xitu.io/2018/2/3/1615bebf81e52b6e?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

参考文章：
- [给 Android 开发者的 RxJava 详解](http://gank.io/post/560e15be2dca930e00da1083)
- [RxJava 从入门到放弃再到不离不弃](https://www.daidingkang.cc/2017/05/19/Rxjava/)
- [RxJava + Retrofit完成网络请求](https://www.jianshu.com/p/1fb294ec7e3b)
- [手把手带你入门神秘的 Rxjava](http://blog.csdn.net/carson_ho/article/details/78179340)
- [优雅实现 网络请求嵌套回调](http://blog.csdn.net/carson_ho/article/details/78315696)
