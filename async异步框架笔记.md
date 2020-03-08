Spring Boot �첽���
### һ���γ�Ŀ��
��ϤSpring���첽��ܣ�ѧ��ʹ���첽@Asyncע��


### ����ΪʲôҪʹ���첽��ܣ��������ʲô���⣿

��SpringBoot���ճ������У�һ�㶼��ͬ�����õģ����Ǿ���������ҵ��Ҫ���첽���������磺
ע���µ��û�����100�����֣����µ��ɹ�����push��Ϣ�ȵȡ�
����ע�����û�ΪʲôҪ�첽����
- ��һ��ԭ���ݴ��ԣ�������ֳ����쳣������ӦΪ�ͻ��ֶ������û�ע��ʧ�ܣ�
ӦΪ�û�ע������Ҫ���ܣ��ͻ����Ǵ�Ҫ���ܣ����ͻ����쳣ҲҪ��ʾע��ɹ���Ȼ��������Ի����쳣����������
- �ڶ���ԭ���������ܣ������û���20�룬�ͻ��ֻ�50�룬�����ͬ���Ļ����ܺ�ʱ70�룬���첽�Ļ�������ȴ����֣��ʺ�ʱ20�롣
���첽�ܽ���������⣬���ܺ��ݴ���

### ����SpringBooot�첽����
��SpringBoot��ʹ���첽�����Ƿǳ��򵥵ģ�ֻ��Ҫ��@Asyncע�⼴��ʵ�ַ������첽���á�

### �ģ�@Async�첽��������

#### ����1�������첽����
����@EnableAsync�������첽����֧�֣�������Ҫ����@Configuration���ѵ�ǰ�����springIOC�����С�
'''
@Configuration
@EnableAsync
public class AsyncConfiguration {

}
'''

#### ����2���ڷ����ϱ���첽����
λ�ڷ����service���ǵô���@Serviceע��
'''
@Async
public void addScore(){
	//ģ��˯��5�룬�������ͻ���
	try{
	    Thread.sleep(1000*5);
	    log.info("---------�������---------");
	}catch(InterruptedException e){
	    e.printStackTrace();
	}
}
'''

### �壬ΪʲôҪ��Async�Զ����̳߳أ�
@Asyncע�⣬��Ĭ��������õ���SimpleAsyncTaskExecutor�̳߳أ����̳߳ز������������ϵ��̳߳أ�
ӦΪ�߳����ã�ÿ�ε��ö����½�һ���̡߳�
����ͨ������̨��־����鿴��ÿ�δ�ӡ���߳�1����Task-1��Task-2��Task-3��Task-4 ... �����ġ�
@Asyncע���첽����ṩ�����߳�
SimpleAsyncTaskExecutor��Ĭ�ϣ��������������̳߳أ�����಻�����̣߳�ÿ�ε��ö��ᴴ��һ���µ��̡߳�
SyncTaskExecutor�������û��ʵ���첽���ã�ֻ��һ��ͬ��������ֻ�����ڲ���Ҫ���̵߳ĵط�
ConcurrentTaskExecutor��Executor�������࣬���Ƽ�ʹ�á����ThreadPoolTaskExecutor������Ҫ��ʱ���ſ���ʹ�������
ThreadPoolTaskScheduler������ʹ��cron���ʽ
ThreadPoolTaskExecutor���ʹ�ã��Ƽ�����ʵ���Ƕ�java.util.concurrent.ThreadPoolExecutor�İ�װ

### ����ΪAsyncʵ��һ���Զ����̳߳�
#### ����1�������̳߳س�
'''
@Configuration
@EnableAsync
public class AsyncConfiguration {
    @Bean(name="scorePoolTaskExecutor")
    public ThreadPoolTaskExecutor getScorePoolTaskExecutor(){
        ThreadPoolTaskExecutor taskExecutor = new ThreadPoolTaskExecutor();
        //�����߳���
        taskExecutor.setCorePoolSize(10);
        //�̳߳�ά���߳����������ֻ���ڻ����������֮��Ż����볬�������߳������߳�
        taskExecutor.setMaxPoolSize(100);
        //�������
        taskExecutor.setQueueCapacity(50);
        //��Ŀ���ʱ�䣬�����˺����߳���֮����߳��ڿ���ʱ��ﵽ��ᱻ����
        taskExecutor.setKeepAliveSeconds(200);
        //�첽�����ڲ��߳�����
        taskExecutor.setThreadNamePrefix("score-");
        /**
         * ���̵߳����񻺴�����������Ҳ����̳߳���Ŀ�ﵽmaximumPoolSize���������������
         * �ͻ��ȡ���޾ܾ�����
         * ͨ����һ�¼��ֲ��ԣ�
         * ThreadPoolExecutor.AbortPolicy: ���������׳�RejectedExecutionException�쳣
         * ThreadPoolExecutor.DiscardPolicy��Ҳ�Ƕ������񣬵��ǲ��׳�����
         * ThreadPoolExecutor.DiscardOldestPolicy������������ǰ�������Ȼ�����³���ִ�������ظ��˹��̣�
         * ThreadPoolExecutor.CallerRunPolicy��������ӵ�ǰ�������Զ��ظ����� executor() ������ֱ���ɹ�
         */
        taskExecutor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        taskExecutor.initialize();

        return taskExecutor;
    }

}
'''

### ����2��Ϊ@Asyncָ���̳߳�����

### �ߣ��κ���ϰ
����ʾ1��������Ŀ�����У���Ը߲���������һ��������Ǹ߲����ӿڵ����̳߳ظ��봦��
��������2���߲����ӿڣ�ˢ���û�redis���档
һ�����¶����ӿڣ�����app push��Ϣ��
��ο����γ����ݣ���������̳߳أ��ֱ����ڡ�ˢ���û�redis���桿�͡�����app push��Ϣ��