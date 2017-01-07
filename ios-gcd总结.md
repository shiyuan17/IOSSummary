# GCD

## 目录结构：

* [线程概念](/threadConcept)
* [线程常用操作](#threadCommonOpt)
* [多线程中的常见问题](#threadquestion)

### 1.线程概念

**同步：**阻塞当前线程，走到任务执行完毕后。

**异步：**当前线程任务执行，后续任务也可以不中断执行。

**串行队列：**安任务的顺序，一个接一个的执行。FIFO（先进先出），执行完一个，取下一个继续执行。

**并行队列：**放到并行队列的任务，GCD 也会 FIFO的取出来，但不同的是，它取出来一个就会放到别的线程，然后再取出来一个又放到另一个的线程。这样由于取的动作很快，忽略不计，看起来，所有的任务都是一起执行的。不过需要注意，GCD 会根据系统资源控制并行的数量，所以如果任务很多，它并不会让所有任务同时执行。

**gcd三种队列类型：Serial\(串行\)，Concurrent\(并行\)，Main dispatch queue\(主线程串行\)**

1.主线程串行队列，通过**dispatch\_get\_main\_queue\(\)**来获取，UI操作相关必须要在主线程队列运行，减少避免耗时操作在主线程运行。

2.全局并发队列，通过**dispatch\_get\_global\_queue**来获取，可以设置优先级，系统提供的并发队列，一般并行任务都可以添加到这里。

3.自定义队列，通过**dispatch\_queue\_create**来获取，可以为串行、并行。并行queue:**DISPATCH\_QUEUE\_CONCURRENT**,串行queue:**DISPATCH\_QUEUE\_SERIAL**。

 4.队列组，通过**dispatch\_group\_create**来获取，将多线程进行分组，获取线程队列的完成情况，通过**dispatch\_group\_notify**来监听队列组所有线程的完成情况。

|  | 同步执行 | 异步执行 |
| :--- | :--- | :--- |
| 串行队列 | 当前线程，一个一个执行 | 其它线程一个一个执行 |
| 并行队列 | 当前线程，一个一个执行 | 开很多线程，一个一个执行。 |

### [2.线程常用操作](/threadCommonOpt)

\*\*1.常用耗时GCD操作\*\*&lt;a name="async\_gcd"&gt;&lt;/a&gt;

```
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{  
            // 耗时的操作  
            dispatch_async(dispatch_get_main_queue(), ^{  
                // 更新界面  
            });  
        });  
```

**2.dispatch\_once 常用来设置单例，保证在程序运行中只执行一次。** &lt;a name="dispatch\_once"&gt;&lt;/a&gt;

```
+ (Instance *)sharedInstance  
    {  
            static AccountManager *instance = nil;  
            static dispatch_once_t predicate;  
            dispatch_once(&predicate, ^{  
                    //只初始化一次
                    instance  = [[self alloc] init];   
            });  
        return instance;  
    } 
```

* **&lt;a name="taskqueue"&gt;&lt;/a&gt;3.一组任务的执行情况可以使用dispatch\_group\_async进行监听，所有任务完成后，使用dispatch\_group\_notify进行监听完成。**

```
        //获取全局并行队列
        dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
        //创建队列组
        dispatch_group_t group = dispatch_group_create();
        //设置异步线程所属的分组和所属的队列
        dispatch_group_async(group, queue, ^{
            NSLog(@"task1");
        });
        dispatch_group_async(group, queue, ^{
            NSLog(@"task2");
        });
        dispatch_group_async(group, queue, ^{
            NSLog(@"task3");
        });
        //监听group组中的队列完成情况
        dispatch_group_notify(group, queue, ^{
            NSLog(@"task success");
        });
        
        //输出结果：
        task3
        task1
        task2
        task success
```

**&lt;a name="customerqueue"&gt;&lt;/a&gt;4.创建自定义的队列dispatch\_queue\_create\(,\)两个参数，第一个为队列标识，第二个为队列的类型\(串行、并行\)，当为NULL的时候表示串行队列**

```
        //创建自定义并行队列，标识为demo.queue
        dispatch_queue_t queue = dispatch_queue_create("demo.queue", DISPATCH_QUEUE_CONCURRENT);
        dispatch_async(queue, ^{
            NSLog(@"task1");
        });
        dispatch_async(queue, ^{
            NSLog(@"task2");
        });
        //等待队列前所有任务线程完成后才执行，并且dispatch_barrier_async后面的任务要等待当前dispatch_barrier_async执行完成后才执行。
        dispatch_barrier_async(queue, ^{
            NSLog(@"dispatch_barrier_async");
        });
        //end task要等待dispatch_barrier_async完成后才会运行。
        dispatch_async(queue, ^{
            NSLog(@"end task");
        });
```

## 3.[多线程中的常见问题](/threadquestion)

**1.临界访问：Critical Section**，在多线程数据共享中，不能同时被两个线程访问的代码段，比如一个变量，被并发进程访问后可能会改变变量值，造成数据污染。

**2.资源竞争：Race Condition**，多个线程同时访问共享数据时，会发生争用的情形，例如，线程A，和线程B 同时操作变量C，哪么就会发生竞争关系，变量C的值取决于最后操作的线程。

**3.线程死锁：Thread Lock**两个或多个线程间需要等待另一方才能进行下一步，这时就会改时线程死锁。

**4.异常UI绘制: **当绘制UI界面发生在子线程时，系统会产生莫名难追踪的错误。

**5.优先级反转:** 修改线程优先级要多注意低优先级的线程影响到高优级线程的运行。当不同优先级的许多线程共享对同样的锁和资源的访问权时，可能会发生优先级反转，即较低优先级的线程实际无限期地阻止较高优先级线程的进度。

**6.互斥锁:** ios中有**atomic**作为原子属性，他会为**setter**方法进行加锁，哪么该属性变量就支持互斥锁了。atomic作为解决多线程资源抢夺中会消耗大量的资源，并且一个线程在连续多次读取某个属性值的过程中有别的线程在同时改写该值，那么即便将属性声明为atomic，也还是会读取到不同的属性值。若要实现线程安全操作还需要采用更为深层的锁机制处理才行。

