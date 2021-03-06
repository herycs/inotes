# 工具

## CountDownLatch

> 允许一个或多个线程等待，但线程计数器只能使用一次

CountDownLatch是一个同步工具类，它允许一个或多个线程一直等待，直到其他线程执行完后再执行。

count == 0，打开门

## CyclicBarrier

> 同步屏障，线程计数器可重置（也就是说可以重复使用）

可以让一组线程达到一个屏障时被阻塞，直到最后一个线程达到屏障时，所以被阻塞的线程才能继续执行。

基于ReentrantLock

## Semaphore

> 信号量，控制对资源访问的线程数量

Semaphore也叫信号量，在JDK1.5被引入，可以用来控制同时访问特定资源的线程数量，通过协调各个线程，以保证合理的使用资源。

获取许可有公平策略和非公平许可策略，默认情况下使用非公平策略。

## Exchanger

> 线程间交换数据

简单说就是一个线程在完成一定的事务后想与另一个线程交换数据，则第一个先拿出数据的线程会一直等待第二个线程，直到第二个线程拿着数据到来时才能彼此交换对应数据。