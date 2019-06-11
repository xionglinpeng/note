# spring-cloud-netflix-hystrix





## 使用用详解



### 创建请求命令

#### HystrixCommand

通过继承的方式来实现，继承`com.netflix.hystrix.HystrixCommand<R>`，重载`run()`方法。

例如：

```java
import com.netflix.hystrix.HystrixCommand;
import com.netflix.hystrix.HystrixCommandGroupKey;

public class HelloCommand extends HystrixCommand<String>{

    public HelloCommand() {
        super(HystrixCommand.Setter
                .withGroupKey(HystrixCommandGroupKey.Factory.asKey("SeasonCommand")));
    }

    @Override
    protected String run() throws Exception {
        return "hello hystrix!";
    }
}
```



- 同步请求：`String execute = new HelloCommand().execute();`
- 异步请求：`Future<String> queue = new HelloCommand().queue();`



注解同步执行：

```java
@HystrixCommand
public String getHystrix(){
    return "hello hystrix!";
}
```

注解异步执行：

```java
@HystrixCommand
public Future<String> getHystrix(){
    return new AsyncResult<String>() {
        @Override
        public String invoke() {
            return "hello hystrix!";
        }
    };
}
```

响应式执行：

通过调用`observe()`和`toObservable()`方法可以返回`Observale`对象。

```java
Observable<String> observe = new HelloCommand().observe();
Observable<String> Observable = new HelloCommand().toObservable();
```

`observe()`和`toObservable()`区别：

- `observe()`：返回的是一个Hot Observable，该命令会在`observe()`调用的时候立即执行，当Observable每次被订阅的时候会重放它的行为；
- `toObservable()`：返回的是一个Cold Observable，`toObservable()`执行之后，命令不会被立即执行，只有当所有订阅者都订阅它之后才会执行。

#### HystrixObservableCommand

继承`com.netflix.hystrix.HystrixObservableCommand<R>`，重载`construct()`方法：

```java
import rx.Observable;
import rx.Subscriber;

public class HelloObservableCommand extends HystrixObservableCommand<String> {

    public HelloObservableCommand(){
        super(Setter
                .withGroupKey(HystrixCommandGroupKey.Factory.asKey("HelloObservableCommand")));
    }

    @Override
    protected Observable<String> construct() {
        return Observable.create(new Observable.OnSubscribe<String>() {
            @Override
            public void call(Subscriber<? super String> subscriber) {
                try {
                    if (!subscriber.isUnsubscribed()) {
                        subscriber.onNext("hello hystrix!");
                        subscriber.onCompleted();
                    }
                } catch (Exception e) {
                    subscriber.onError(e);
                }
            }
        });
    }
}
```

**注解实现：**

注解实现依然是使用`@HystrixCommand`，只是方法定义需要做一些变化，具体内容与`construct()`的实现类似。

在使用`@HystrixCommand`注解实现响应式命令时，可以通过`observableExecutionMode `参数来控制使用`observe()`还是`toObservable()`的执行方式：

- `@HystrixCommand(observableExecutionMode = ObservableExecutionMode.EAGER)`

  `EAGER`是该参数的模式值，表示使用`observe()`执行方法。

- `@HystrixCommand(observableExecutionMode = ObservableExecutionMode.LAZY)`

  表示使用`toObservable()`执行方法。

```java
@HystrixCommand
public Observable<String> getHystrix(){
    return Observable.create(new Observable.OnSubscribe<String>() {
        @Override
        public void call(Subscriber<? super String> subscriber) {
            try {
                if (!subscriber.isUnsubscribed()) {
                    subscriber.onNext("hello hystrix");
                    subscriber.onCompleted();
                }
            } catch (Exception e) {
                subscriber.onError(e);
            }
        }
    });
}
```

**区别：**

`HystrixCommand`：只能发射一次数据。

`HystrixObservableCommand`：可以发送多次数据。



### 定义服务降级



