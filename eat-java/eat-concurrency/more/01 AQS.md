https://www.bilibili.com/video/BV19J411Q7R5?t=6&p=2

Java的内置锁一直都是备受争议的，在JDK 1.6之前，synchronized这个重量级锁其性能一直都是较为低下，虽然在1.6后，进行大量的锁优化策略,但是与Lock相比synchronized还是存在
一些缺陷的。

虽然synchronized提供了便捷性的隐式获取锁释放锁机制（基于JVM机制），但是它却缺少了获取锁与释放锁的可操作性，可中断、超时获取锁，且它为独占式在高并发场景下性能大打折扣。

**自旋实现同步**

```java
volatile int status=0;//标识‐‐‐是否有线程在同步块‐‐‐‐‐是否有线程上锁成功

void lock(){
    while(!compareAndSet(0,1)){
    }
}

void unlock(){
    status=0;
}

boolean compareAndSet(int except,int newValue){
    //cas操作,修改status成功则返回true
}
```

缺点：耗费cpu资源。没有竞争到锁的线程会一直占用cpu资源进行cas操作，假如一个线程获得锁后要花费Ns处理业务逻辑，那另外一个线程就会白白的花费Ns的cpu资源

思路：让得不到锁的线程让出CPU

**yield + 自旋**

```java
volatile int status=0;
void lock(){
    while(!compareAndSet(0,1)){
        yield();//自己实现
    }
}

void unlock(){
    status=0;
}
```

yield()方法就能让出cpu资源，当线程竞争锁失败时，会调用yield方法让出cpu。

问题：当系统只有两个线程竞争锁时，yield 是有效的。需要注意的是该方法只是当前让出cpu，有可能操作系统下次还是选择运行该线程。