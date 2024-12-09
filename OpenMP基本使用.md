# OpenMP基本使用

参考链接：
https://blog.csdn.net/weixin_42819452/article/details/102816640
参考链接：
https://blog.csdn.net/xyqqwer/article/details/136845617

### 常用的功能指令

| 功能指令          | 解析                                                         |
| ----------------- | ------------------------------------------------------------ |
| parallel          | 用在一个结构块之前，表示这段代码将被多个线程并行执行         |
| for               | 用于for循环语句之前，表示将循环计算任务分配到多个线程中并行执行，以实现任务分担，必须由编程人员自己保证每次循环之间无数据相关性 |
| sections          | 用在可被并行执行的代码段之前，用于实现多个结构块语句的任务分担，可并行执行的代码段各自用section指令标出（注意区分sections和section） |
| parallel sections | parallel和sections两个语句的结合，类似于parallel for         |
| single            | 用在并行域内，表示一段只被单个线程执行的代码                 |
| critical          | 用在一段代码临界区之前，保证每次只有一个OpenMP线程进入       |
| flush             | 保证各个OpenMP线程的数据影像的一致性                         |
| barrier           | 用于并行域内代码的线程同步，线程执行到barrier时要停下等待，直到所有线程都执行到barrier时才继续往下执行 |
| atomic            | 用于指定一个数据操作需要原子性地完成                         |
| master            | 用于指定一段代码由主线程执行                                 |
| threadprivate     | 用于指定一个或多个变量是线程专用，后面会解释线程专有和私有的区别 |

### 相应的OpenMP子句

| OpenMP子句   | 解析                                                         |
| ------------ | ------------------------------------------------------------ |
| private      | 指定一个或多个变量在每个线程中都有它自己的私有副本           |
| firstprivate | 指定一个或多个变量在每个线程都有它自己的私有副本，并且私有变量要在进入并行域或任务分担域时，继承主线程中的同名变量的值作为初值 |
| lastprivate  | 是用来指定将线程中的一个或多个私有变量的值在并行处理结束后复制到主线程中的同名变量中，负责拷贝的线程是for或sections任务分担中的最后一个线程 |
| reduction    | 用来指定一个或多个变量是私有的，并且在并行处理结束后这些变量要执行指定的归约运算，并将结果返回给主线程同名变量 |
| nowait       | 指出并发线程可以忽略其他制导指令暗含的路障同步               |
| num_threads  | 指定并行域内的线程的数目                                     |
| schedule     | 指定for任务分担中的任务分配调度类型                          |
| shared       | 指定一个或多个变量为多个线程间的共享变量                     |
| ordered      | 用来指定for任务分担域内指定代码段需要按照串行循环次序执行    |
| copyprivate  | 配合single指令，将指定线程的专有变量广播到并行域内其他线程的同名变量中 |
| copyinn      | 用来指定一个threadprivate类型的变量需要用主线程同名变量进行初始化 |
| default      | 用来指定并行域内的变量的使用方式，缺省是shared               |

### API函数·

| API                 | 解析                         |
| ------------------- | ---------------------------- |
| 函数名 作用         |                              |
| omp_in_parallel     | 判断当前是否在并行域中       |
| omp_get_thread_num  | 返回线程号                   |
| omp_set_num_thread  | 设置后续并行域中的线程格式   |
| omp_get_num_threads | 返回当前并行域中的线程数     |
| omp_get_max_threads | 返回并行域可用的最大线程数目 |
| omp_get_num_prpces  | 返回系统中处理器的数目       |
| omp_get_dynamic     | 判断是否支持动态改变线程数目 |
| omp_set_dynamic     | 启用或关闭线程数目的动态改变 |
| omp_get_nested      | 判断系统是否支持并行嵌套     |
| omp_set_nested      | 启用或关闭并行嵌套           |

## 1

设置线程
export OMP_NUM_THREADS=4

```C
#include <stdio.h>
#include <omp.h>

int main() {
    #pragma omp parallel
    {
        int thread_id = omp_get_thread_num();
        printf("Hello from thread %d\n", thread_id);
    }
    return 0;
}
```

编译：gcc -fopenmp -o my_program my_program.c
运行：./my_program
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/f8ee307762b646eea51bc2792fdff570.png)

## 2 使用parallel for

```
#include <stdio.h>
#include <stdlib.h>
#include <omp.h>
int main(int argc,char** argv){
    #pragma omp parallel for num_threads(6)
    for(int i=1;i<12;i++){
        printf("openmp test,线程编号：%d\n",omp_get_thread_num());
    }
    return 0;
}
```

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/75616c729b914419b77aa9399bff5d42.png)

## 3 OpenMP效率提升以及不同线程数效率对比

```
#include <stdio.h>
#include <stdlib.h>
#include <omp.h>
void test(){
    int sum=0;
    for(int i=0;i<100;i++){
        sum += i;
    }
}
int main(int argc,char** argv){
    float start_time=omp_get_wtime();
    #pragma omp parallel for num_threads(2)
    for(int i=0;i<80000;i++){
        test();
    }
    float end_time=omp_get_wtime();
    printf("指定两个线程，时间为：%f\n",end_time-start_time);
    start_time=end_time;
    #pragma omp parallel for num_threads(4)
    for(int i=0;i<80000;i++){
        test();
    }
    end_time=omp_get_wtime();
    printf("指定四个线程，时间为：%f\n",end_time-start_time);
    start_time=end_time;
    #pragma omp parallel for num_threads(8)
    for(int i=0;i<80000;i++){
        test();
    }
    end_time=omp_get_wtime();
    printf("指定八个线程，时间为:%f\n",end_time-start_time);
    start_time=end_time;
    #pragma omp parallel for num_threads(12)
    for(int i=0;i<80000;i++){
        test();
    }
    end_time=omp_get_wtime();
    printf("指定12个线程，时间为：%f\n",end_time-start_time);
    start_time=end_time;
    for (int i = 0; i < 80000; i++){
        test();
    }
    end_time = omp_get_wtime();
    printf("不使用OpenMP多线程，执行时间: %f\n", end_time - start_time);
    return 0;

}
```

编译：gcc -fopenmp -std=c99 -o test_diff_threads test_diff_threads.c
运行：./test_diff_threads
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/0973ff64f3354cdbbb0b2cde772a1248.png)
上面出现线程越多处理时间却更长了，原因是我设置的处理器以及核数均为1，多线程出现负载现象，所以我又重新设置了核数为4（因为我的电脑的物理核数为4）
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/278897dd85a84cf6a27a33d1d3eb01d8.png)

因为核数是4，所以线程多了还是会负载。

## 4 API使用

```
#include <omp.h>
#include <stdio.h>
#include <stdlib.h>
int main(int argc, char *argv[])
{
	printf("ID: %d, Max threads: %d, Num threads: %d \n", omp_get_thread_num(), omp_get_max_threads(), omp_get_num_threads());
	omp_set_num_threads(5);
	printf("ID: %d, Max threads: %d, Num threads: %d \n", omp_get_thread_num(), omp_get_max_threads(), omp_get_num_threads());

#pragma omp parallel num_threads(5)
	{
		printf("ID: %d, Max threads: %d, Num threads: %d \n", omp_get_thread_num(), 
		omp_get_max_threads(), omp_get_num_threads());
	}

	printf("ID: %d, Max threads: %d, Num threads: %d \n", omp_get_thread_num(), omp_get_max_threads(), omp_get_num_threads());

	omp_set_num_threads(6);
	printf("ID: %d, Max threads: %d, Num threads: %d \n", omp_get_thread_num(), omp_get_max_threads(), omp_get_num_threads());

	return 0;
}
```

编译：gcc -fopenmp -o API API.c
运行：./API

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/c1b37be838d844c499fb2a3fa7d993d8.png)

## 5 使用Sections并行执行不同的任务

每个 section 会在不同的线程中并行执行。sections 是一种 OpenMP 的并行结构，适用于你希望将不同的任务分配给不同线程的情况。

```
#include <omp.h>
#include <stdio.h>
 
int main() {
    #pragma omp parallel sections
    {
        #pragma omp section
        {
            printf("Task 1, Thread %d\n", omp_get_thread_num());
        }
 
        #pragma omp section
        {
            printf("Task 2, Thread %d\n", omp_get_thread_num());
        }
    }
    return 0;
}
```

编译：gcc -fopenmp -o section section.c
运行： ./section
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/e57a17fb34444bdc9909bcb5c9eaac1c.png)