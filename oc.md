# objective C

## 代理和协议
协议是一组方法，遵守这个协议的就要按照要求实现这组方法

代理是让别的类帮你代做事情，如果某个参数拿不到，或者要给代理的那个类传一些参数，一般这么写

如果你的类有些能力自己做不到，需要别的类来帮忙
以oc为例，步骤如下：
1.声明一个协议
```objc
@protocol testADelegate <NSObject>
@required//required是必须要实现的方法
- (void) testSendParameters:(NSString *)parameter;
@optional//optional可以选择性的实现
- (void) testOptionalSendParameters:(NSString *)parameter;
@end
```
2.声明一个类，此类遵循上面的协议，类内变量声明一个weak id<xxxdelegate> delegate，用weak的原因是为了防止使用这个变量造成循环引用
```objc
@interface testA : NSObject
@property(nonatomic, readwrite, weak) id<testADelegate> delegate;
@end
```
3.在初始化此类的时候，对delegate这个变量赋值，复制给哪个对象，哪个对象就要遵循这个协议
```objc
@property(nonatomic, readwrite, strong) testA *test;

test.delegate = self;
//一般用self，指代实例化这个的类的对象，总之是代理方
```
4.代理方重写最开始声明的协议，此时函数的参数是被代理者传入，函数的返回值可以传回给被代理者

5.在代理者的内部写一个函数去触发协议，不然协议不触发，被代理者的重写的函数将一直没机会被执行
```objc
if ([weakSelf.delegate respondsToSelector:@selector(testSendParameters:)]) {
    [weakSelf.delegate testSendParameters:(NSString*)parameter];
    //parameter是传给被代理方在重写时候用到的参数
```

## 定时器

oc两种定时器，一种NSTimer，一种GCD定时器
使用哪个取决于原来项目用的什么
NSTimer一般配合runlooper使用，runlooper任务繁重会导致时间不准确
GCD设置的定时器相对于NSTimer时间更加精准，其依赖于系统内核
GCD定时器设置方法：
```objc
dispatch_source_t Executor = dispatch_source_create(DISPATCH_SOURCE_TYPE_TIMER, 0, 0, clientWorkingQueue);
//创建定时器：para1定时器类型，para2-3对定时器无用，para4在哪个队列里调用
dispatch_source_set_timer(Executor, DISPATCH_TIME_NOW, second * NSEC_PER_SEC, 0 * NSEC_PER_SEC);
//定时器配置：para1定时器，para2开始时间，para3任务的时间间隔，para4误差(0为没有误差)
dispatch_source_set_event_handler(self.bweResolutionsExecutor, ^{
//在block内写需要定时执行的内容
    [self function];
});
dispatch_resume(Executor);
//启动定时器
```
