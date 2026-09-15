<callout icon="😀" color="gray_bg">
	线程池技术正是为解决这一痛点而生 —— 它以 “池化思想” 对线程资源进行统一管理，通过复用线程、动态调度任务队列，在性能、资源利用率与系统稳定性之间取得平衡。无论是后端服务应对海量请求，还是嵌入式设备在资源受限场景下实现高效并发，线程池都是并发编程领域绕不开的核心技术。
</callout>
# **线程池基础** {toggle="true"}
	## **认识线程池** {toggle="true"}
		首先，线程池是一个什么东西，简单从低层次来讲，线程本身是实现并发的一种机制，可以做到多任务同时进行或者说多线程去完成一个任务。但是线程去实现高并发的操作过程过于复杂，那么**线程池**的出现，就是为了用简单、快捷、高效率的方式去实现高并发。那原理很简单，顾名思义，线程池可以理解为多个线程的集合，就是将多个线程伴随服务一起启动，然后待命。当有任务到来时，它会自动分配去执行；当线程池线程不够用时，也会自动添加线程；并且当任务完成或者说并发量低时，线程池也会自动去删除一些不需要或者说是空闲的线程来减少内存的占用，来增加工作效率。
		简单理解一下线程池的定义以后，我们一起来往下看，来研究一下线程池是怎么做的吧！
	## **线程池的构造** {toggle="true"}
		### **线程池概念** {toggle="true"}
			线程池并不是一个操作系统或者说哪个协会提出的一些标准的机制，线程池完全脱离了这两个理念，**是一个完全由程序员写出来的一个小功能**，也就是说代码并不唯一而且并没有现成的供你调用的接口，所以说我们需要一步一步的去将它实现。
			首先，我们来看一下它的工作原理和内部结构，来研究一下到底是怎么个线程池！
		### **线程池的内部构造** {toggle="true"}
			线程池的内部有 **三个部分 **构成
			- **第一部分：任务队列**
				任务队列本身是一个队列，具备先进先出的原则，需要自己去实现。那任务队列的作用就是去对外接受任务，对内提供任务。
			- **第二部分：管理者线程**
				管理者线程本身也是一个线程，但是与线程池其他线程干的活是不一样的。**管理者线程的作用**就是自动去控制、分配、安排线程去干哪些活，然后适当去管理线程的创建和销毁，比较智能化。
			- **第三部分：一堆线程**
				一堆线程就可以理解为 “小兵”，全权听从管理者的安排，干活就干活，创建就创建，销毁就销毁！
			三者概念清楚了以后咱们来看一下线程池的工作框图（图示：以服务场景说明线程池工作流程，展示并发服务器接收请求、任务队列记录任务、线程池（任务队列、管理者线程、工作线程）分配执行任务的流程）。
			![](https://cdn.jsdmirror.com/gh/LZJ-I/picx-images-hosting@main/images/%E8%AF%AD%E8%A8%80%E4%B8%8E%E7%AE%97%E6%B3%95/Post-thread-pool/225d64f1-b512-4c4a-8a43-281a1b870c03.webp)
			通过上图，可以看出一个问题：线程是如何知道任务到来的？当线程池运行起来但任务还未到来时，全体线程（除了管理者线程之外）都会调用条件变量进行**挂起等待**；当任务队列中有任务到来时，任务队列会发送一个 “任务到来” 的 signal 信号（也就是生产者和消费者的关系），然后唤醒其中一个线程去**处理这个任务**。
	## **线程池实现大纲** {toggle="true"}
		### **构建一个线程池**
		首先我们要构建一个线程池，听着是不是有点面向对象的感觉？没错，我们需要用面向对象的思想，但我们需要用 C 语言实现，所以一般我们选用一个**结构体**来描述一个线程池。
		### **任务队列：**
		1. 其实任务对于我们而言就是一个**执行函数**。
		2. 执行函数需要**传递参数**。
		3. 任务队列中最多可以存储多少个任务（最大限制）。
		4. 任务队列中的当前任务数量。
		5. 任务队列中的头部。
		6. 任务队列中的尾部。
		### **管理者线程：**
		1. 因为我们需要创建一个管理者线程，那么这个线程在线程池中应该有一个**管理者线程 ID 号**作为它的体现。
		### **一堆线程：**
		1. 同样，存放线程池中 “干活的线程”，也需要一个**存放线程号的数组**。
		2. 线程池启动时最少 “一起来” 的线程数 —— 最小的线程个数。
		3. 线程池最大能支持多少个线程一起去工作 —— 最大线程的个数。
		4. 线程池中正在工作的线程个数 —— 忙碌的线程个数。
		5. 线程池中目前可以正常工作的线程个数 —— 存活的线程个数。
		6. 当线程池发现任务很少时，需要自动销毁线程到 “最小的线程个数”，所以还需要一个**销毁线程的个数**（后面一看代码就明白了）。
		### **其他辅助部分：**
		1. 三个条件变量：
			- 用于任务队列满时使用。
			- 用于任务队列空时使用。
			- 用于长时间未使用消息队列时使用（根据个人需求）。
		2. 二个互斥锁：
			- 需要一个互斥锁锁住**整个线程池的结构体**，因为它属于临界资源。
			- 需要一个互斥锁锁住**忙碌线程个数的结构体**，因为这是经常被调用的临界资源。
		3. 线程池状态：给线程池设定一个 “标注位”，来标识线程池的**运行**还是**销毁**状态。
		> 注：有些东西可能不是很理解 “为什么有它的存在”，只因为线程池本身就是程序员完全手写出来的。
# 源码参考 {toggle="true"}
	## pthreadpool.c {toggle="true"}
		```c
/*************************************************************************
																		
	#	 文件名称	: pthreadpool.c
	#	 作者		: 朱双剑 
																		
 ************************************************************************/
#include "pthreadpool.h"
#include <stdlib.h>			
#include <unistd.h>
#include <stdio.h>
#include <string.h>
#define COUNT_NUM 2


// 创建线程池
pthreadpool * init_pthreadPoolCreat(int minnum,int maxnum,int queueMaxSize)
{
	//创建一个指向线程池的指针,并且开辟空间
	pthreadpool * pool = (pthreadpool *)malloc(sizeof(pthreadpool));
	do 	//使用do-while方便管理return 函数最后有体现
	{
		if (pool == NULL)
		{
			printf("创建线程池失败,第1次\n");
			break;
		}

		//给线程池存放线程号的头指针开辟空间
		pool->thread_tID = (pthread_t *)malloc(sizeof(pthread_t) * maxnum);
		if (pool->thread_tID == NULL)
		{
				printf("创建任务队列失败,第2次\n");
				break;
		}

		//初始化申请到的内存
		memset(pool->thread_tID,0,sizeof(pthread_t) * maxnum);//初始化为0 用于后续线程号存放判断使用
		//初始化线程池信息
		pool->minnum = minnum; 	//线程池最小个数
		pool->maxnum = maxnum;  //最大个数
		pool->busynum = 0; 		//忙碌线程
		pool->livenum = minnum;	//存货线程数
		pool->exitnum = 0; 		//销毁线程数


		//初始化锁和条件变量
		if (pthread_mutex_init(&pool->poolmutex,NULL) != 0 || pthread_mutex_init(&pool->busymutex,NULL) != 0 
		||	pthread_cond_init(&pool->notfull,NULL) != 0 || pthread_cond_init(&pool->notempty,NULL) != 0)
		{
			printf("初始化锁或者初始化条件变量失败\n");
			break;
		}
		
		//任务队列初始化
		pool->taskQ = (task *)malloc(sizeof(task) * queueMaxSize);  //为队列开辟内存
		pool->queueMaxSize = queueMaxSize;  //容量固定
		pool->queueSize = 0; 	//任务个数
		pool->queueFront = 0; 	//头
		pool->queueRear = 0; 	//尾
		pool->shutdown = 0;		//标识线程时被创建

		//创建管理者线程
		if(pthread_create(&pool->managerID,NULL,manager,pool)!= 0 )
		{
			printf("创建管理者线程失败\n");
			break;
		}
		//创建线程池工作线程	
		int i = 0;
		for (i = 0;  i < minnum ; i++) //用最小的个数去创建线程池
		{ 
			if (pthread_create(&pool->thread_tID[i],NULL,myfunc,pool) != 0)
			{
				printf("创建线程 %d  发生错误\n",i);
				break;
			}
		}
		return pool;
	} while (0);

	//如果发生错误会跳出while循环 开始释放资源
	if (pool->thread_tID)
	{
		free(pool->thread_tID);
		pool->thread_tID = NULL;
	}
	if (pool->taskQ)
	{
		free(pool->taskQ);
		pool->taskQ = NULL;
	}
	if (pool)
	{
		free(pool);
		pool = NULL;
	}

	return NULL;
}



//工作线程

void * myfunc(void * arg) 
{
	pthreadpool * pool = (pthreadpool *) arg;	//获取线程池指针
	while (1)
	{
		pthread_mutex_lock(&pool->poolmutex);//操作线程池的临界区就必须加锁
		//当任务队列中没有任务时工作线程阻塞等待任务的到来
		while(pool->queueSize == 0 && !pool->shutdown) //判断线程池有没有被关闭 !pool->shutdown
		{
			//如果没有任务或者线程池关闭了,我需要阻塞线程
			pthread_cond_wait(&pool->notempty,&pool->poolmutex);
			if (pool->exitnum > 0) //判断是否在执行销毁线程
			{
				pool->exitnum--; //目标销毁线程数-1
				pool->livenum--;//存活的线程数-1			
				pthread_mutex_unlock(&pool->poolmutex);//解除锁				
				pthreadExit(pool);
			}
			
		}

		//唤醒后判断线程池是否被关闭
		if (pool->shutdown == 1) 
		{
			pthread_mutex_unlock(&pool->poolmutex);//解除锁
			pthreadExit(pool); //关闭线程
			
		}
		
		//开始工作,从工作队列中取出任务

		task mytask;
		mytask.fun = pool->taskQ[pool->queueFront].fun;//从头部开始取任务
		mytask.arg = pool->taskQ[pool->queueFront].arg;//从尾部开始取任务
		//将任务数组维护成环形队列-移动头节点
		//将移动后对队列最大数进行取于操作
		pool->queueFront = ( pool->queueFront + 1 ) % pool->queueMaxSize;
		//取出了一个任务,需要将任务个数减一
		pool->queueSize --;

		//向任务队列中放任务的线程有可能因为任务满在被阻塞
		//所以取出一个任务之后要告知任务生产者 任务队列中有一个位置可以存放任务
		pthread_cond_signal(&pool->notfull);

		pthread_mutex_unlock(&pool->poolmutex);//操作线程池的临界区就必须加锁
		//开始调用函数
		
		//开始工作时忙碌线程数+1
		pthread_mutex_lock(&pool->busymutex);
		pool->busynum ++; //忙碌线程数+1
		pthread_mutex_unlock(&pool->busymutex);

		mytask.fun(mytask.arg);  //开始工作
		//工作完成后释放空间
		//free(mytask.arg);
		//mytask.arg=NULL;
		
		printf("工作完成%ld\n",pthread_self());
		//工作完成后忙碌线程数-1；
		pthread_mutex_lock(&pool->busymutex);
		pool->busynum --; //忙碌线程数-1
		pthread_mutex_unlock(&pool->busymutex);



	}

	return "线程执行完毕";
}

//管理者线程
 void * manager (void * arg)
 {
	pthreadpool * pool = (pthreadpool *) arg;
	int j = 0;
	int count =40;
	while (pool->shutdown == 0) //工作条件,当线程池没有被销毁时要工作
	{
		
		//每三秒检测一次
		sleep(1);
		//监控线程池任务数和当前线程数量
		pthread_mutex_lock(&pool->poolmutex);	//锁
		int queuesize = pool->queueSize;
		int livenum = pool->livenum;
		printf("管理者第%d次检测,当前线程中任务队列任务的个数为%d,当前存活的线程个数为%d\n",j++,queuesize,livenum);
		//检测任务队列中任务,如果40秒之内未进入稳步则关闭线程池
		if (pool->queueSize  == 0 && pool->livenum == 5) 
		{
			printf("倒计时%d秒关闭线程池\n",count);
			count --;
			if (count == 0)
			{
				pthread_cond_signal(&pool->pooldestory);//关闭线程池
			}
			
		}
		
		pthread_mutex_unlock(&pool->poolmutex);
		//监控忙碌线程个数
		pthread_mutex_lock(&pool->busymutex);
		int busynum = pool->busynum;
		printf("当前忙碌的线程为%d\n",busynum);
		pthread_mutex_unlock(&pool->busymutex);

		//添加线程   ----指定一个规则来规定啥时候添加线程
		//任务个数 > 当前的可用线程个数 && 可用的线程数 < 最大线程数

		if ((queuesize > livenum) && (livenum < pool->maxnum))
		{
			printf("进入自动创建线程\n");
			pthread_mutex_lock(&pool->poolmutex);
			//添加线程 ,每次添加两个
			int counter = 0; //计数 
			int i = 0;
			//判断条件 循环i不能大于线程最大个数 并且 计数不能超过目标添加个数 并且 存活个数不能超过最大个数
			for ( i = 0; (i < pool->maxnum) && (counter < COUNT_NUM )&& (livenum < pool->maxnum); i++)
			{
				
				//创建线程
				if (pool->thread_tID[i] == 0) //判断存放线程号的数组中哪个可以存放线程号,如果为0可以存放
				{
						pthread_create(&pool->thread_tID[i],NULL,myfunc,pool);
						counter ++;
						pool->livenum++;
						printf("自动创建线程成功\n");	
				}
			}
			pthread_mutex_unlock(&pool->poolmutex);
			
		}
		


		//销毁线程  同样需要指定规则
		//忙的线程*2 《 存货的线程数 && 存活的线程数 > 最小线程数
		if (busynum * 2 < livenum && livenum > pool->minnum)
		{
			pthread_mutex_lock(&pool->poolmutex);
			pool->exitnum = COUNT_NUM;//每次销毁两个
			pthread_mutex_unlock(&pool->poolmutex);

			//让工作的线程自杀

			int i = 0;
			for ( i = 0; i < COUNT_NUM; i++)
			{
				pthread_cond_signal(&pool->notempty);
			}
			
		}
		
	}
		
	return NULL;

 }

 //退出线程函数
void pthreadExit(pthreadpool * pool){

	pthread_t pid = pthread_self();//获取当前线程的线程号
	int i = 0;
	for ( i = 0; i < pool->maxnum; i++)
	{
		if (pid == pool->thread_tID[i])
		{
			pool->thread_tID[i] = 0;
			printf("线程结束,该线程的线程号存放在数组中的位置已被清0\n");
			break;
		}
		
	}
	pthread_exit(NULL);
}

//给线程池添加任务函数

void ThreadPoolAdd(pthreadpool * pool,void(*func)(void * ),void * arg){
		//判断任务队列是否满了
		pthread_mutex_lock(&pool->poolmutex);	// 加互斥锁
		while (pool->queueSize == pool->queueMaxSize && !pool->shutdown)	//如果任务队列满了 并且 线程池正在运行
		{
			//阻塞生产者线程
			pthread_cond_wait(&pool->notfull,&pool->poolmutex);	//阻塞挂起等待，让出互斥锁
			//那这个阻塞由谁来发消息呢？因该由工作线程在拿走一个任务后告知一下该线程,所以
			//signal应该在工作线程拿走任务之后

		}
		
		if (pool->shutdown) //判断一下上个循环是否因为线程池被销毁而退出while循环
		{
			printf("线程池关闭,线程结束\n");
			pthread_mutex_unlock(&pool->poolmutex);
			return;
		}
		//添加任务,将任务添加的任务队列的队尾
		pool->taskQ[pool->queueRear].fun = func;
		pool->taskQ[pool->queueRear].arg = arg;
	
		//添加完任务后,队列指针后移
		pool->queueRear = (pool->queueRear + 1) % pool->queueMaxSize;//环形队列
		pool->queueSize ++; //任务队列任务数加1
		
		pthread_cond_signal(&pool->notempty);//唤醒消费者   唤醒任意挂起等待条件变量cond的至少一个线程。
		
		pthread_mutex_unlock(&pool->poolmutex);		//解互斥锁

}


int GetBusyNum(pthreadpool * pool)
{
	pthread_mutex_lock(&pool->busymutex);
	int busynum = pool->busynum;
	pthread_mutex_unlock(&pool->busymutex);
	return busynum;
}

int GetLiveNum(pthreadpool * pool)
{
	pthread_mutex_lock(&pool->poolmutex);
	int livenum = pool->livenum;
	pthread_mutex_unlock(&pool->poolmutex);
	return livenum;
}


int pthreadPoolDestory(pthreadpool * pool) //销毁线程池
{
	pthread_mutex_lock(&pool->poolmutex);	
	pthread_cond_wait(&pool->pooldestory,&pool->poolmutex);
	pthread_mutex_unlock(&pool->poolmutex);
	printf("开始释放资源\n");
	if (pool == NULL)
	{
		return -1;
	}
	//关闭线程池
	pool->shutdown = 1;
	//销毁管理者线程,因为管理者线程在判断pool->shutdown == 1时会自动退出
	pthread_join(pool->managerID,NULL);
	//唤醒阻塞的消费者线程
	printf("结束原有的线程池的线程\n");
	int i = 0;
	for ( i = 0; i < pool->livenum; i++)
	{
			pthread_cond_signal(&pool->notempty);
	}
		//释放互斥锁或者条件变量
		pthread_mutex_destroy(&pool->poolmutex);
		pthread_mutex_destroy(&pool->busymutex);
		pthread_cond_destroy(&pool->notempty);
		pthread_cond_destroy(&pool->notfull);
	//释放内存
	
		if (pool->thread_tID)
		{
			free(pool->thread_tID);
			pool->thread_tID = NULL;
		}
		if (pool->taskQ)
		{
			free(pool->taskQ);
			pool->taskQ = NULL;
		}
		
		if (pool)
		{
			free(pool);
			pool = NULL;
		}
	
	return 0;
}

		```
	## pthreadpool.h {toggle="true"}
		```c
/*************************************************************************
																		
	#	 文件名称	: pthreadpool.h
	#	 作者		: 朱双剑 
	#	 邮箱		: 328800461@qq.com
	#	 创建时间	: 2021年08月07日 星期六 08时04分45秒
																		
 ************************************************************************/

#ifndef __PTHREADPOOL_H__
#define __PTHREADPOOL_H__
#include <pthread.h>		// 线程库

//任务结构体
typedef struct task {
	void (* fun)(void * arg);	// 任务函数
	void * arg;					// 任务函数的参数
}task; //定义一个新的类型 task
//  需要线程执行的工作单元，包含一个函数指针 和 函数参数。

//线程池结构体
typedef struct pthreadpool{
//任务队列
	task* taskQ;  		//任务列--数组
	int queueMaxSize;	//任务队列的容量
	int queueSize;		//当前任务的个数
	int queueFront;		//队头
	int queueRear;		//队 尾
	
//线程池相关信息
	//管理者ID 
	pthread_t managerID;
	//工作线程ID 
	pthread_t * thread_tID; //因为多个所以说直接用一片地址

	//线程池的线程范围
	int minnum;	//最小的线程个数
	int maxnum;	//最大的线程个数
	int busynum;//忙碌的线程个数
	int livenum;//现在线程池可用的线程--存活的线程个数
	int exitnum;//销毁的线程个数
	 
	//保护临界资源的机制--互斥锁
	pthread_mutex_t poolmutex; 	//锁住整个线程池
	pthread_mutex_t busymutex; 	//锁住busynum 变量
	//线程同步唤醒机制--条件变量 来监控任务队列状态
	pthread_cond_t notfull;		//判断任务队列是不是满了
	pthread_cond_t notempty;	//判断任务队列是不是空了
	pthread_cond_t pooldestory; //线程池工作空闲关闭判断条件变量

	//线程池的状态1.运行 2.销毁
	int shutdown;//1代表被销毁，0代表正在运行
} pthreadpool;



/*函数介绍---------------- init_pthreadPoolCreat---------------
	作者：朱双剑
    功能：创建线程池并且初始化
	参数：
		minnum :线程池最小个数
		maxnum :线程池最大个数
		queueMaxSize ：任务队列中最大的任务个数
	返回值：会将创建好的线程池的地址返回 失败返回NULL 并打印错误信息
*/
pthreadpool * init_pthreadPoolCreat(int minnum,int maxnum,int queueMaxSize);


/*函数介绍---------------- pthreadPoolDestory  ---------------
	作者：朱双剑
    功能：销毁线程池
	参数：
		pool:线程池的首地址
	返回值：销毁成功返回0 失败返回-1 并打印错误信息
*/
int pthreadPoolDestory(pthreadpool * pool);


/*函数介绍----------------	ThreadPoolAdd ---------------
	作者：朱双剑
    功能：向线程池中添加任务
	参数： 
		pool：线程池首地址
		fun ：任务函数指针
		arg ：任务函数参数
	返回值：无
*/
void ThreadPoolAdd(pthreadpool * pool,void(*fun)(void *),void * arg);

/*函数介绍----------------	GetBusyNum ---------------
	作者：朱双剑
    功能：获取线程池正在工作的线程个数
	参数： 
		pool：线程池首地址
	返回值：这个函数永远时成功的 成功返回获取到的正在工作的线程个数
*/
int GetBusyNum(pthreadpool * pool);

/*函数介绍----------------	GetBusyNum ---------------
	作者：朱双剑
    功能：获取线程池中活着的线程个数
	参数： 
		pool：线程池首地址
	返回值：这个函数永远时成功的 成功返回获取到的存活的线程个数
*/
int GetLiveNum(pthreadpool * pool);

/*函数介绍----------------	myfunc ---------------
	作者：朱双剑
    功能：正常工作线程函数--不会被用户主动调用
	参数： 
		arg :工作函数的参数
	返回值：函数执行失败线程直接结束，如果执行成功会返回 "线程执行完毕" 字符串
*/
void * myfunc (void * arg);


/*函数介绍----------------	manager ---------------
	作者：朱双剑
    功能：管理者线程函数--不会被用户主动调用 --主要负责监控线程池自动调节
	参数： 
		arg :工作函数的参数
	返回值：该函数不会失败，退出时返回NULL
*/
void * manager(void * arg);




/*函数介绍----------------	manager ---------------
	作者：朱双剑
    功能：线程结束释放资源
	参数： 
		pool :工作函数的参数
	返回值：无
*/

void pthreadExit(pthreadpool * pool);

#endif

		```
	## main.c {toggle="true"}
		```c
/*************************************************************************
																		
	#	 文件名称	: 01_pthreads.c
	#	 作者		: 朱双剑
																		
 ************************************************************************/

#include <stdio.h>			
#include "pthreadpool.h"	
#include <unistd.h>			
#include <stdlib.h>			
void func(void * arg)
{
	int num = *(int *)arg;
	printf("线程ID为%ld 正在工作, number = %d\n",pthread_self(),num);
	sleep(3);
}

int main(int argc, const char *argv[])
{
	//创建出一个线程池
	pthreadpool * pool = init_pthreadPoolCreat(5,200,100);
	if (pool == NULL)
	{
		printf("创建线程池失败\n");
		return -1;
	}
	printf("线程池创建完毕\n");

	//添加任务
	int i = 0;
	for ( i = 0; i < 200; i++)
	{
		int * num = (int *)malloc(4);
		*num = i;
		ThreadPoolAdd(pool,func,num);
	}
	
	pthreadPoolDestory(pool);
    return 0;
}

		```
# 源码解释 {toggle="true"}
	要理解这个 C 语言实现的线程池，我们可以先从**线程池的基本概念**入手，再逐步拆解代码中的核心组件和工作流程 —— 就像理解 “一个工厂怎么运转”：先知道工厂里有 “工人（工作线程）”“调度员（管理者线程）”“任务仓库（任务队列）”，再看他们怎么配合完成工作、怎么应对任务多少调整工人数量，最后怎么关闭工厂。
	## **一、先搞懂：什么是线程池？为什么需要它？**
	线程池是**提前创建一批线程（“工人”）存起来，有任务来了就分配线程执行，执行完线程不销毁，回到池里等下一个任务**的机制。
	### **为什么不用 “有任务就创建线程”？**
	- 线程的 “创建 / 销毁” 是有开销的（比如向操作系统申请资源、回收资源），频繁创建销毁会浪费 CPU 和内存。
	- 线程池能控制线程总数（避免线程太多导致系统资源耗尽），还能通过 “复用线程” 提高效率。
	## **二、先看 “线程池的骨架”：两个核心结构体**
	代码里定义了两个关键结构体，它们是线程池的 “数据容器”—— 就像工厂的 “仓库布局图” 和 “工人信息表”。
	### **1. 任务结构体（****`task`****）：描述一个 “任务”**
	一个任务本质是 “要执行的函数”+“函数的参数”，所以结构体里只有两个成员：
	**c**
	运行
	```c
typedef struct task {
    void (*fun)(void *arg);  // 任务对应的函数（比如要执行的计算、打印等）
    void *arg;               // 传给这个函数的参数（比如计算用的数值、打印的内容）
} task;
	```
	比如：如果要执行 “打印数字 10”，`fun`就是打印函数，`arg`就是`&10`。
	### **2. 线程池结构体（****`pthreadpool`****）：描述整个 “线程池”**
	这个结构体是核心，里面包含了**任务队列、线程信息、同步互斥工具、线程池状态**四大块，我们按模块拆着看：
	<table header-row="true">
<tr>
<td>**模块**</td>
<td>**成员变量**</td>
<td>**作用说明**</td>
</tr>
<tr>
<td>**任务队列（仓库）**</td>
<td>`task* taskQ`</td>
<td>用数组实现的 “环形队列”，存所有待执行的任务</td>
</tr>
<tr>
<td></td>
<td>`queueMaxSize`</td>
<td>队列最大容量（最多能存多少个任务）</td>
</tr>
<tr>
<td></td>
<td>`queueSize`</td>
<td>当前队列里的任务数量</td>
</tr>
<tr>
<td></td>
<td>`queueFront`/`queueRear`</td>
<td>队列的 “头指针”（取任务的位置）和 “尾指针”（加任务的位置），实现环形复用</td>
</tr>
<tr>
<td>**线程信息（工人）**</td>
<td>`pthread_t managerID`</td>
<td>管理者线程的 ID（只有 1 个，负责监控和调节线程数）</td>
</tr>
<tr>
<td></td>
<td>`pthread_t *thread_tID`</td>
<td>工作线程的 ID 数组（存所有 “工人” 的 ID，最多`maxnum`个）</td>
</tr>
<tr>
<td></td>
<td>`minnum`/`maxnum`</td>
<td>线程池的 “最小线程数” 和 “最大线程数”（工人数量的范围）</td>
</tr>
<tr>
<td></td>
<td>`busynum`/`livenum`/`exitnum`</td>
<td>忙碌的线程数、存活的线程数、要销毁的线程数（实时状态）</td>
</tr>
<tr>
<td>**同步互斥（规则）**</td>
<td>`pthread_mutex_t poolmutex`</td>
<td>互斥锁 1：保护整个线程池的临界资源（比如任务队列、线程数），防止 “抢资源”</td>
</tr>
<tr>
<td></td>
<td>`pthread_mutex_t busymutex`</td>
<td>互斥锁 2：单独保护`busynum`（忙碌线程数），减少锁竞争</td>
</tr>
<tr>
<td></td>
<td>`pthread_cond_t notfull`</td>
<td>条件变量 1：判断任务队列 “是否满了”（生产者满了就等，有空位再加任务）</td>
</tr>
<tr>
<td></td>
<td>`pthread_cond_t notempty`</td>
<td>条件变量 2：判断任务队列 “是否空了”（消费者空了就等，有任务再执行）</td>
</tr>
<tr>
<td></td>
<td>`pthread_cond_t pooldestory`</td>
<td>条件变量 3：触发线程池销毁（空闲太久时用）</td>
</tr>
<tr>
<td>**线程池状态**</td>
<td>`int shutdown`</td>
<td>标识线程池是否销毁：0 = 运行中，1 = 要销毁（所有线程要退出）</td>
</tr>
	</table>
	## **三、线程池的 “完整生命周期”：从创建到销毁**
	我们按 “创建→加任务→执行任务→动态调节→销毁” 的流程，拆解核心函数的逻辑，这是理解线程池的关键。
	### **1. 第一步：创建线程池（****`init_pthreadPoolCreat`****）**
	作用：像 “建工厂” 一样，初始化所有资源（分配内存、创建初始工人和调度员）。核心步骤（按代码顺序）：
	1. **分配内存**：给线程池结构体、工作线程 ID 数组、任务队列数组分配内存（如果分配失败，后面要释放已分配的，避免内存泄漏）。
	2. **初始化参数**：
		- 线程数：初始存活线程数 =`minnum`，忙碌 / 销毁线程数 = 0，`shutdown=0`（运行中）。
		- 任务队列：队列大小 =`queueMaxSize`，初始任务数 = 0，头尾指针 = 0（空队列）。
	3. **初始化同步工具**：创建 2 个互斥锁（`poolmutex`/`busymutex`）和 3 个条件变量（`notfull`/`notempty`/`pooldestory`）—— 这是线程安全的关键，必须成功。
	4. **创建线程**：
		- 1 个**管理者线程**（跑`manager`函数，负责监控调节）。
		- `minnum`个**工作线程**（跑`myfunc`函数，负责执行任务）。
	5. **返回结果**：如果所有步骤成功，返回线程池指针；失败则释放所有已分配资源，返回`NULL`。
	### **2. 第二步：添加任务（****`ThreadPoolAdd`****）**
	作用：“给工厂送任务”（比如用户代码调用这个函数，把要执行的任务放进队列），属于 “生产者” 操作。核心逻辑（避免多线程抢队列）：
	1. **加锁**：先锁`poolmutex`，因为要修改任务队列（临界资源），防止多个线程同时加任务。
	2. **判断队列是否满**：如果队列满了且线程池没销毁，就调用`pthread_cond_wait(&notfull, &poolmutex)`—— 让当前线程 “阻塞等待”（相当于 “送货的人等仓库有空位”），此时会暂时释放`poolmutex`，避免占着锁不让别人用。
	3. **判断线程池是否销毁**：如果`shutdown=1`（要销毁了），解锁后直接返回（不接新任务）。
	4. **添加任务**：把任务（函数 + 参数）放到队列的 “尾指针” 位置，然后尾指针后移（环形队列，所以要对`queueMaxSize`取模，避免越界），任务数`queueSize++`。
	5. **唤醒消费者**：调用`pthread_cond_signal(&notempty)`—— 唤醒一个正在等任务的工作线程（告诉 “工人”：有新任务了，来拿！）。
	6. **解锁**：释放`poolmutex`，让其他线程能操作队列。
	### **3. 第三步：工作线程执行任务（****`myfunc`****）**
	作用：“工人干活”，每个工作线程都循环跑这个函数，属于 “消费者” 操作。核心逻辑（无限循环，直到线程池销毁）：
	1. **加锁**：锁`poolmutex`，准备检查任务队列。
	2. **等待任务**：如果队列空且线程池没销毁，调用`pthread_cond_wait(&notempty, &poolmutex)`—— 阻塞等待（“工人等任务”），直到被`ThreadPoolAdd`唤醒。
	3. **判断是否要销毁线程**：唤醒后先看`exitnum>0`（管理者要求销毁线程）：
		- 如果是：`exitnum--`（要销毁的线程少一个）、`livenum--`（存活线程少一个），解锁后调用`pthreadExit`退出线程（退出前把自己的 ID 在数组里清 0，方便后续复用位置）。
	4. **判断线程池是否销毁**：如果`shutdown=1`，解锁后退出线程。
	5. **取任务**：从队列 “头指针” 取任务（函数 + 参数），头指针后移（取模），任务数`queueSize--`。
	6. **唤醒生产者**：调用`pthread_cond_signal(&notfull)`—— 告诉 “送货的人”：仓库空出一个位置了，可以加新任务。
	7. **解锁**：释放`poolmutex`，准备执行任务。
	8. **执行任务**：
		- 加`busymutex`锁，`busynum++`（标记自己为忙碌），解锁。
		- 调用任务函数：`mytask.fun(mytask.arg)`（真正干活）。
		- 执行完后，加`busymutex`锁，`busynum--`（标记自己为空闲），解锁。
	9. **循环等待下一个任务**：回到步骤 1，继续等新任务。
	### **4. 第四步：管理者线程监控调节（****`manager`****）**
	作用：“工厂调度员”，每 1 秒检查一次，自动增减工作线程（应对任务多少），还判断是否要销毁线程池。核心逻辑（循环到`shutdown=1`）：
	1. **获取当前状态**：加锁`poolmutex`获取队列大小（`queueSize`）、存活线程数（`livenum`）；加锁`busymutex`获取忙碌线程数（`busynum`）—— 这些都是临界资源，必须加锁读。
	2. **判断是否销毁线程池**：如果队列空且存活线程数 = 5（代码硬编码，可修改），开始 40 秒倒计时，倒计时结束后发`pooldestory`信号（触发销毁）。
	3. **自动添加线程（任务多，工人不够）**：
		- 条件：任务数 \> 存活线程数（任务没人做），且存活线程数 \< 最大线程数（没到上限）。
		- 操作：每次创建 2 个线程（`COUNT_NUM=2`），遍历工作线程 ID 数组，找 ID 为 0 的位置（空位置），创建新工作线程，`livenum++`（存活数增加）。
	4. **自动销毁线程（任务少，工人空闲）**：
		- 条件：忙碌线程数 ×2 \<存活线程数（大部分工人没事干），且存活线程数\> 最小线程数（没到下限）。
		- 操作：设置`exitnum=2`（要销毁 2 个），然后调用`pthread_cond_signal(&notempty)`—— 唤醒 2 个空闲的工作线程（它们唤醒后会检查`exitnum`，然后退出）。
	5. **循环监控**：sleep (1)，1 秒后再检查一次。
	### **5. 第五步：销毁线程池（****`pthreadPoolDestory`****）**
	作用：“关闭工厂”，释放所有资源（线程、锁、内存）。核心步骤：
	1. **等待销毁信号**：加锁`poolmutex`，等待`pooldestory`信号（管理者倒计时结束后发出）。
	2. **标记销毁状态**：设置`shutdown=1`（告诉所有线程：要关了，别干活了）。
	3. **等待管理者线程退出**：调用`pthread_join(managerID, NULL)`—— 等管理者线程执行完循环（看到`shutdown=1`会退出）。
	4. **唤醒所有工作线程**：循环调用`pthread_cond_signal(&notempty)`—— 让所有空闲的工作线程唤醒，它们会检查`shutdown=1`，然后退出。
	5. **释放同步工具**：销毁 2 个互斥锁和 3 个条件变量（系统资源，必须销毁）。
	6. **释放内存**：依次释放工作线程 ID 数组、任务队列数组、线程池结构体的内存，避免泄漏。
	## **四、整体流程：用一个例子串起来**
	假设我们创建一个 “最小 2 个、最大 5 个工作线程，任务队列最大 10 个” 的线程池，整个过程如下：
	1. **创建池**：`init_pthreadPoolCreat(2,5,10)` → 分配内存、初始化锁和条件变量、创建 1 个管理者线程 + 2 个工作线程。
	2. **加任务**：调用`ThreadPoolAdd(pool, 打印函数, "任务1")` → 队列加任务，唤醒 1 个工作线程。
	3. **执行任务**：被唤醒的工作线程取 “任务 1”，执行打印，然后回到空闲状态等下一个任务。
	4. **动态调节**：
		- 如果一下子加了 8 个任务（队列有 8 个），管理者检测到 “任务数（8）\> 存活线程数（2）”，就创建 2 个新工作线程（现在存活 4 个）。
		- 如果任务都执行完了（队列空），管理者检测到 “忙碌线程数（0）×2 \< 存活线程数（4）”，就设置`exitnum=2`，唤醒 2 个空闲线程，它们退出（存活线程回到 2 个，符合最小数）。
	5. **销毁池**：如果 40 秒没任务（队列空 + 存活 2 个），管理者发`pooldestory`信号 → 销毁函数唤醒所有线程，释放锁和内存，线程池关闭。
	## **五、新手必懂的关键概念（避免困惑）**
	1. **临界资源**：多个线程都能访问的资源（比如任务队列、`livenum`、`busynum`）。如果不保护，会出现 “数据竞争”（比如两个线程同时改`queueSize`，导致计数错误）。
	2. **互斥锁（mutex）**：保护临界资源的 “锁”，同一时间只有一个线程能拿到锁，其他线程只能等（保证 “谁先拿到谁用”，不抢）。
	3. **条件变量（cond）**：配合互斥锁，实现 “线程等待特定条件”（比如队列空 / 满），避免线程 “忙等”（比如一直循环判断队列是否有任务，浪费 CPU）。
	4. **环形队列**：任务队列用 “数组 + 头尾指针取模” 实现，比如队列满了之后，头指针后移，前面的空位能复用（像绕圈子一样），比普通数组更节省空间。
	## **六、代码里的小细节（新手注意）**
	- **`COUNT_NUM=2`**：每次增减线程都操作 2 个（可修改，比如改成 1 个）。
	- **任务参数的内存释放**：代码里注释了`//free(mytask.arg)`，如果你的任务参数是动态分配的（比如`malloc`的），要在任务函数里或执行完后释放，否则会内存泄漏。
	- **硬编码的 “5 个存活线程”**：管理者判断是否倒计时销毁时，用了`pool->livenum ==5`（代码里写死的），实际用的时候要改成和`minnum`匹配（比如`pool->livenum == pool->minnum`）。
	通过以上步骤，你应该能理解这个线程池 “从建到用再到关” 的完整逻辑了 —— 核心就是 “提前建线程、复用线程、安全调度、按需调节、资源回收”。
	<empty-block/>
<callout icon="💡" color="gray_bg">
	有关其他Linux、嵌入式问题欢迎在底部评论或者在留言板留言\~
	<empty-block/>
</callout>