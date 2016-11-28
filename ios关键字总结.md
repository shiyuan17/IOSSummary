# ios keywords关键字注意说明：
##strong、weak、assign、retain
1.**strong和weak**是在arc后引入的关键字，strong类似于retain,引用时候会产生引用计数+1，weak则不会改变引用计数。

2.**weak和assign**修饰的变量引用计数不会改变；weak和assign的区别在于，weak用于Object type也是就引用类型、指针类型，而assign用于简单的基本数据类型，如int bool等。

3.**assign和weak**大体一致，但是assign的变量在释放后并不设置为nil，指针的地址还是存在的，所以不能用assign修饰对象类型，如果用assign修饰对象类型当你释放后再去引用的时候就会发生错误崩溃，导致EXC_BAD_ACCESS.

##__block、__weak、__unsafe_unretained
1.Block可以访问局部变量，但是不能修改，如果修改局部变量，需要加__block（**注意：有这样一种情况我们可能很容易忽略，成员变量可以使用下划线_访问，然后我们就鬼使神差的在block里使用下划线去访问成员变量了，这样就造成了循环引用retain cycle**）。
例如：

    @property (nonatomic,strong)NSData          *data;
    [apiget downloadData:^(id responseData){
        _data = responseData;
    }];
self持有data,block使用了_data，则block copy了self。


2.__block修饰的变量在block块中会被retain(在arc下会，在mrc下不会)。
  __weak修饰的变量在block块中不会被retain。__unsafe_unretained与__weak一致。__unsafe_unretained是在arc环境下为了兼容4.x版本。
  在arc时要注意循环引用，可以使用__weak typedof(self) weakSelf = self;

3.如果局部变量是数组或者是指针的话，block只复制这个指针，两个指针指向同一个地址，**block只修改指针对应地址空间的内容**。
例如：

    NSMutableArray *mArray = [NSMutableArray arrayWithObjects:@"a",@"b",@"c",nil];
    NSMutableArray *mArrayCount = [NSMutableArray arrayWithCapacity:1];
    NSLog(@"mArrayCount--:%p",mArrayCount);
    [mArray enumerateObjectsWithOptions:NSEnumerationConcurrent 
    usingBlock: ^(id obj,NSUInteger idx, BOOL *stop){
        [mArrayCount addObject:@"12"];
    }];
    NSLog(@"myArrayData:%@",mArrayCount);
    NSLog(@"mArrayCountEND--%p",mArrayCount);
    
    //输出： %p %x是打印指针地址
    mArrayCount--:0x600000244e00
    myArrayData:(
        12,
        12,
        12
    )
    mArrayCountEND--0x600000244e00

##nonatomic、atomic
nonatomic是非线程安全的，不会给setter方法加锁，atomic是线程安全的，但是需要消耗大量的系统资源来为属性加锁，具体使用如：@synchronized(self){//code } 

**synchronized:**是为方法加锁，或是块加锁，不管处于哪个线程中，运行到该方法的时候，都会检查有没有其它线程在使用该方法。如果有则等待这个线程执行完再执行当前线程。这种机制确保了同一时刻对于每一个类，至多只有一个处于可执行状态，从而有效避免了类成员变量的访问冲突。


