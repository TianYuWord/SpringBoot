Spring Boot 异步框架
### 一，课程目标
熟悉Spring的异步框架，学会使用异步@Async注解


### 二，为什么要使用异步框架，它解决了什么问题？

在SpringBoot的日常开发中，一般都是同步调用的，但是经常有特殊业务要做异步来处理，例如：
注册新的用户。送100个积分，或下单成功发送push消息等等。
就拿注册新用户为什么要异步处理？
- 第一个原因容错性，如果积分出现异常，不能应为送积分而导致用户注册失败；
应为用户注册是主要功能，送积分是次要功能，故送积分异常也要提示注册成功，然后后面就针对积分异常做补偿处理。
- 第二个原因：提升性能，例如用户花20秒，送积分花50秒，如果用同步的话，总耗时70秒，用异步的话，无需等待积分，故耗时20秒。
故异步能解决两个问题，性能和容错性

### 三，SpringBooot异步调用
在SpringBoot中使用异步调用是非常简单的，只需要用@Async注解即可实现方法的异步调用。

### 四，@Async异步调用例子

#### 步骤1：开启异步任务
采用@EnableAsync来开启异步任务支持，另外需要加入@Configuration来把当前类加入springIOC容器中。
'''
@Configuration
@EnableAsync
public class AsyncConfiguration {

}
'''

#### 步骤2：在方法上标记异步调用
位于服务成service，记得打上@Service注解
'''
@Async
public void addScore(){
	//模拟睡眠5秒，用于赠送积分
	try{
	    Thread.sleep(1000*5);
	    log.info("---------处理积分---------");
	}catch(InterruptedException e){
	    e.printStackTrace();
	}
}
'''

### 五，为什么要给Async自定义线程池？
@Async注解，在默认情况下用的是SimpleAsyncTaskExecutor线程池，该线程池不是真正意义上的线程池，
应为线程重用，每次调用都会新建一条线程。
可以通过控制台日志输出查看，每次打印的线程1都是Task-1，Task-2，Task-3，Task-4 ... 递增的。
@Async注解异步框架提供多种线程
SimpleAsyncTaskExecutor（默认）：不是真正的线程池，这个类不重用线程，每次调用都会创建一个新的线程。
SyncTaskExecutor：这个类没有实现异步调用，只是一个同步操作。只适用于不需要多线程的地方
ConcurrentTaskExecutor：Executor的适配类，不推荐使用。如果ThreadPoolTaskExecutor不满足要求时，才考虑使用这个类
ThreadPoolTaskScheduler：可以使用cron表达式
ThreadPoolTaskExecutor：最常使用，推荐。其实质是对java.util.concurrent.ThreadPoolExecutor的包装

### 六，为Async实现一个自定义线程池
#### 步骤1：配置线程池池
'''
@Configuration
@EnableAsync
public class AsyncConfiguration {
    @Bean(name="scorePoolTaskExecutor")
    public ThreadPoolTaskExecutor getScorePoolTaskExecutor(){
        ThreadPoolTaskExecutor taskExecutor = new ThreadPoolTaskExecutor();
        //核心线程数
        taskExecutor.setCorePoolSize(10);
        //线程池维护线程最大数量，只有在缓冲队列满了之后才会申请超过核心线程数的线程
        taskExecutor.setMaxPoolSize(100);
        //缓存队列
        taskExecutor.setQueueCapacity(50);
        //许的空闲时间，超过了核心线程数之外的线程在空闲时间达到后会被销毁
        taskExecutor.setKeepAliveSeconds(200);
        //异步方法内部线程名称
        taskExecutor.setThreadNamePrefix("score-");
        /**
         * 当线程的任务缓存队列已满并且并且线程池数目达到maximumPoolSize，如果还有任务到来
         * 就会采取仍无拒绝策略
         * 通常有一下几种策略：
         * ThreadPoolExecutor.AbortPolicy: 丢弃任务并抛出RejectedExecutionException异常
         * ThreadPoolExecutor.DiscardPolicy：也是丢弃任务，但是不抛出错误
         * ThreadPoolExecutor.DiscardOldestPolicy：丢弃队列最前面的任务，然后重新尝试执行任务（重复此过程）
         * ThreadPoolExecutor.CallerRunPolicy：重试添加当前的任务，自动重复调用 executor() 方法，直到成功
         */
        taskExecutor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        taskExecutor.initialize();

        return taskExecutor;
    }

}
'''

### 步骤2：为@Async指定线程池名字

### 七，课后练习
在显示1互联网项目开发中，针对高并发的请求，一般的做法是高并发接口单独线程池隔离处理
假设现在2个高并发接口，刷新用户redis缓存。
一个是下订单接口，发送app push信息。
请参考本课程内容，设计两个线程池，分别用于【刷新用户redis缓存】和【发送app push信息】