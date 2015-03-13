#Timer Example.

# Timer Example #
```
#include <stdio.h>
#include <time.h>
#include <sys/time.h>
#include <stdlib.h>
#include <signal.h>
int count = 0;
void set_timer()
{
        struct itimerval itv, oldtv;
        itv.it_interval.tv_sec = 5;
        itv.it_interval.tv_usec = 0;
        itv.it_value.tv_sec = 5;
        itv.it_value.tv_usec = 0;

        setitimer(ITIMER_REAL, &itv, &oldtv);
}

void sigalrm_handler(int sig)
{
        count++;
        printf("timer signal.. %d\n", count);
}

int main()
{
        signal(SIGALRM, sigalrm_handler);
        set_timer();
        while (count < 1000)
        {}
        exit(0);
}
```
## References ##
  * alarm和 setitimer系统调用
```

系统调用alarm的功能是设置一个定时器，当定时器计时到达时，将发出一个信号给进程。该调用的声明格式如下： 
unsigned int alarm(unsigned int seconds); 
在使用该调用的进程中加入以下头文件： 
#include <unistd.h> 

系 统调用alarm安排内核为调用进程在指定的seconds秒后发出一个SIGALRM的信号。如果指定的参数seconds为0，则不再发送 SIGALRM信号。后一次设定将取消前一次的设定。该调用返回值为上次定时调用到发送之间剩余的时间，或者因为没有前一次定时调用而返回0。 

注意，在使用时，alarm只设定为发送一次信号，如果要多次发送，就要多次使用alarm调用。 

对于alarm，这里不再举例。现在的系统中很多程序不再使用alarm调用，而是使用setitimer调用来设置定时器，用getitimer来得到定时器的状态，这两个调用的声明格式如下： 
int getitimer(int which, struct itimerval *value); 
int setitimer(int which, const struct itimerval *value, struct itimerval *ovalue); 
在使用这两个调用的进程中加入以下头文件： 
#include <sys/time.h> 

该系统调用给进程提供了三个定时器，它们各自有其独有的计时域，当其中任何一个到达，就发送一个相应的信号给进程，并使得计时器重新开始。三个计时器由参数which指定，如下所示： 
TIMER_REAL：按实际时间计时，计时到达将给进程发送SIGALRM信号。 
ITIMER_VIRTUAL：仅当进程执行时才进行计时。计时到达将发送SIGVTALRM信号给进程。 
ITIMER_PROF：当进程执行时和系统为该进程执行动作时都计时。与ITIMER_VIR-TUAL是一对，该定时器经常用来统计进程在用户态和内核态花费的时间。计时到达将发送SIGPROF信号给进程。 

定时器中的参数value用来指明定时器的时间，其结构如下： 
struct itimerval { 
struct timeval it_interval; /* 下一次的取值 */ 
struct timeval it_value; /* 本次的设定值 */ 
}; 

该结构中timeval结构定义如下： 
struct timeval { 
long tv_sec; /* 秒 */ 
long tv_usec; /* 微秒，1秒 = 1000000 微秒*/ 
}; 

在setitimer 调用中，参数ovalue如果不为空，则其中保留的是上次调用设定的值。定时器将it_value递减到0时，产生一个信号，并将it_value的值设 定为it_interval的值，然后重新开始计时，如此往复。当it_value设定为0时，计时器停止，或者当它计时到期，而it_interval 为0时停止。调用成功时，返回0；错误时，返回-1，并设置相应的错误代码errno： 
EFAULT：参数value或ovalue是无效的指针。 
EINVAL：参数which不是ITIMER_REAL、ITIMER_VIRT或ITIMER_PROF中的一个。 

下面是关于setitimer调用的一个简单示范，在该例子中，每隔一秒发出一个SIGALRM，每隔0.5秒发出一个SIGVTALRM信号： 

#include <signal.h> 
#include <unistd.h> 
#include <stdio.h> 
#include <sys/time.h> 
int sec; 

void sigroutine(int signo) { 
switch (signo) { 
case SIGALRM: 
printf("Catch a signal -- SIGALRM "); 
break; 
case SIGVTALRM: 
printf("Catch a signal -- SIGVTALRM "); 
break; 
} 
return; 
} 

int main() { 
struct itimerval value,ovalue,value2; 
sec = 5; 

printf("process id is %d ",getpid()); 
signal(SIGALRM, sigroutine); 
signal(SIGVTALRM, sigroutine); 

value.it_value.tv_sec = 1; 
value.it_value.tv_usec = 0; 
value.it_interval.tv_sec = 1; 
value.it_interval.tv_usec = 0; 
setitimer(ITIMER_REAL, &value, &ovalue); 

value2.it_value.tv_sec = 0; 
value2.it_value.tv_usec = 500000; 
value2.it_interval.tv_sec = 0; 
value2.it_interval.tv_usec = 500000; 
setitimer(ITIMER_VIRTUAL, &value2, &ovalue); 

for (;;) ; 
} 

该例子的屏幕拷贝如下： 

localhost:~$ ./timer_test 
process id is 579 
Catch a signal – SIGVTALRM 
Catch a signal – SIGALRM 
Catch a signal – SIGVTALRM 
Catch a signal – SIGVTALRM 
Catch a signal – SIGALRM 
Catch a signal –GVTALR
```