# Java synchronize

## JVM启动

UseBiasedLocking是在JVM启动的时候，是否启动偏向锁的标识

1. 如果支持偏向锁,则执行 ObjectSynchronizer::fast_enter的逻辑
2. 如果不支持偏向锁,则执行 ObjectSynchronizer::slow_enter逻辑，绕过偏向锁，直接进入轻量级锁

fast_enter方法的主要流程做一个简单的解释

1. 再次检查偏向锁是否开启
2. 当处于不安全点时，通过 revoke_and_rebias尝试获取偏向锁，如果成功则直接返回，如果失败则进入轻量级锁获取过程
3. revoke_and_rebias这个偏向锁的获取逻辑在 biasedLocking.cpp中
4. 如果偏向锁未开启，则进入 slow_enter获取轻量级锁的流程

## 偏向锁

### 获取

```java
BiasedLocking::Condition BiasedLocking::revoke_and_rebias(Handle obj, bool attempt_rebias, TRAPS) {
  assert(!SafepointSynchronize::is_at_safepoint(), "must not be called while at safepoint");
  markOop mark = obj->mark(); //获取锁对象的对象头
  //判断mark是否为可偏向状态，即mark的偏向锁标志位为1，锁标志位为 01，线程id为null
  if (mark->is_biased_anonymously() && !attempt_rebias) {
    //这个分支是进行对象的hashCode计算时会进入，在一个非全局安全点进行偏向锁撤销
    markOop biased_value       = mark;
    //创建一个非偏向的markword
    markOop unbiased_prototype = markOopDesc::prototype()->set_age(mark->age());
    //Atomic:cmpxchg_ptr是CAS操作，通过cas重新设置偏向锁状态
    markOop res_mark = (markOop) Atomic::cmpxchg_ptr(unbiased_prototype, obj->mark_addr(), mark);
    if (res_mark == biased_value) {//如果CAS成功，返回偏向锁撤销状态
      return BIAS_REVOKED;
    }
  } else if (mark->has_bias_pattern()) {//如果锁对象为可偏向状态（biased_lock:1, lock:01，不管线程id是否为空）,尝试重新偏向
    Klass* k = obj->klass(); 
    markOop prototype_header = k->prototype_header();
    //如果已经有线程对锁对象进行了全局锁定，则取消偏向锁操作
    if (!prototype_header->has_bias_pattern()) {
      markOop biased_value       = mark;
      //CAS 更新对象头markword为非偏向锁
      markOop res_mark = (markOop) Atomic::cmpxchg_ptr(prototype_header, obj->mark_addr(), mark);
      assert(!(*(obj->mark_addr()))->has_bias_pattern(), "even if we raced, should still be revoked");
      return BIAS_REVOKED; //返回偏向锁撤销状态
    } else if (prototype_header->bias_epoch() != mark->bias_epoch()) {
      //如果偏向锁过期，则进入当前分支
      if (attempt_rebias) {//如果允许尝试获取偏向锁
        assert(THREAD->is_Java_thread(), "");
        markOop biased_value       = mark;
        markOop rebiased_prototype = markOopDesc::encode((JavaThread*) THREAD, mark->age(), prototype_header->bias_epoch());
        //通过CAS 操作， 将本线程的 ThreadID 、时间错、分代年龄尝试写入对象头中
        markOop res_mark = (markOop) Atomic::cmpxchg_ptr(rebiased_prototype, obj->mark_addr(), mark);
        if (res_mark == biased_value) { //CAS成功，则返回撤销和重新偏向状态
          return BIAS_REVOKED_AND_REBIASED;
        }
      } else {//不尝试获取偏向锁，则取消偏向锁
        //通过CAS操作更新分代年龄
        markOop biased_value       = mark;
        markOop unbiased_prototype = markOopDesc::prototype()->set_age(mark->age());
        markOop res_mark = (markOop) Atomic::cmpxchg_ptr(unbiased_prototype, obj->mark_addr(), mark);
        if (res_mark == biased_value) { //如果CAS操作成功，返回偏向锁撤销状态
          return BIAS_REVOKED;
        }
      }
    }
  }
  ...//省略
}

```

### 撤销

1. 偏向锁的释放，需要等待全局安全点（在这个时间点上没有正在执行的字节码）

2. 首先暂停拥有偏向锁的线程，然后检查持有偏向锁的线程是否还活着

    - 如果线程不处于活动状态，则将对象头设置成无锁状态。

    - 如果线程仍然活着，则会升级为轻量级锁，遍历偏向对象的锁记录。

3. 栈帧中的锁记录和对象头的Mark Word要么重新偏向其他线程，要么恢复到无锁，或者标记对象不适合作为偏向锁。

4. 最后唤醒暂停的线程。

> JVM内部为每个类维护了一个偏向锁revoke计数器，对偏向锁**撤销**进行计数
>
> 当这个值达到指定阈值时，JVM会认为这个类的偏向锁有问题，需要重新偏向(rebias),对所有属于这个类的对象进行重偏向的操作称为批量重偏向(bulk rebias)。
>
> 在做bulk rebias时，会对这个类的epoch的值做递增，这个epoch会存储在对象头中的epoch字段。
>
> 在判断这个对象是否获得偏向锁的条件是:markword的 biased_lock:1、lock:01、threadid和当前线程id相等、epoch字段和所属类的epoch值相同，
>
> - 如果epoch的值不一样
>     - 要么就是撤销偏向锁
>     - 要么就是rebias； 
>
> 如果这个类的revoke计数器的值继续增加到一个阈值，那么jvm会认为这个类不适合偏向锁，就需要进行**bulk revoke**操作

### 获取

1. mark->is_neutral()方法, is_neutral这个方法是在 markOop.hpp中定义，如果 biased_lock:0且lock:01表示无锁状态
2. 如果mark处于无锁状态，则进入步骤(3)，否则执行步骤(5)
3. 把mark保存到BasicLock对象的displacedheader字段
4. 通过CAS尝试将markword更新为指向BasicLock对象的指针，如果更新成功，表示竞争到锁，则执行同步代码，否则执行步骤(5)
5. 如果当前mark处于加锁状态，且mark中的ptr指针指向当前线程的栈帧，则执行同步代码，否则说明有多个线程竞争轻量级锁，轻量级锁需要膨胀升级为重量级锁

### 撤销

轻量级锁的释放也比较简单，就是将当前线程栈帧中锁记录空间中的Mark Word替换到锁对象的对象头中

- 如果成功表示锁释放成功
- 否则，锁膨胀成重量级锁，实现重量级锁的释放锁逻辑

锁膨胀

1. mark->has_monitor() 判断如果当前锁对象为重量级锁，也就是lock:10，则执行(2),否则执行(3)

2. 通过 mark->monitor获得重量级锁的对象监视器ObjectMonitor并返回，锁膨胀过程结束

3. 如果当前锁处于 INFLATING,说明有其他线程在执行锁膨胀，那么当前线程通过自旋等待其他线程锁膨胀完成

4. 如果当前是轻量级锁状态 mark->has_locker(),则进行锁膨胀。

    首先，通过omAlloc方法获得一个可用的ObjectMonitor，并设置初始数据

    然后通过CAS将对象头设置为`markOopDesc:INFLATING，表示当前锁正在膨胀

    如果CAS失败，继续自旋

5. 如果是无锁状态，逻辑类似第4步骤

## 重量级锁

### 获取

1. 通过CAS将monitor的 _owner字段设置为当前线程，如果设置成功，则直接返回
2. 如果之前的 _owner指向的是当前的线程，说明是重入，执行 _recursions++增加重入次数
3. 如果当前线程获取监视器锁成功，将 _recursions设置为1， _owner设置为当前线程
4. 如果获取锁失败，则等待锁释放

如果获取锁失败，则需要通过自旋的方式等待锁释放，自旋执行的方法是

1. 将当前线程封装成ObjectWaiter对象node，状态设置成TS_CXQ
2. 通过自旋操作将node节点push到_cxq队列
3. node节点添加到_cxq队列之后，继续通过自旋尝试获取锁，如果在指定的阈值范围内没有获得锁，则通过park将当前线程挂起，等待被唤醒

### 撤销

1. 判断当前锁对象中的owner没有指向当前线程，如果owner指向的BasicLock在当前线程栈上,那么将_owner指向当前线程_

2. 如果当前锁对象中的_owner指向当前线程，则判断当前线程重入锁的次数

    如果不为0，继续执行ObjectMonitor::exit()，直到重入锁次数为0为止

3. 释放当前锁，并根据QMode的模式判断，是否将_cxq中挂起的线程唤醒。还是其他操作