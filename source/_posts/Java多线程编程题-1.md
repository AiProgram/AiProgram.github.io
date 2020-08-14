---
title: Java多线程编程题-1
date: 2020-08-14 09:59:11
tags:
categories:
- Java
- 多线程
---

## 3个线程论文轮流打印数字（消费资源）

<!-- more -->

### 使用`synchronized、wait、notifyAll`解决

+ 首先我们需要为每个线程设定**id**，这主要是为了确定本线程是否能够消费，假定依次为$0,1,2$
+ 由于`synchronized`一次只能有一个锁对应`wait/notifyAll`指令，所以为了简洁和防止出错我们需要让三个线程公用一个锁对象，假定为**lock**。
+ 编程时，我们需要明确几点：
  1.  无论我们使用什么样的编码顺序，这三个线程最开始启动的顺序一定是无法保证的。（除非我们在确认线程1启动了后在线程1内部启动线程2）
  2. 共用锁对象让我们无法保证下一次唤醒的线程的顺序
  3. 因为1，2点，我们首先需要在线程被启动/唤醒时检测当前线程的运行条件，不能运行则自己wait放弃锁等待。如果可以运行则进行消费，然后notifyAll唤醒其他线程。检测运行条件可以使用**id,count**解决。
  4. 需要注意线程退出问题：如果规定消费的总数是N，则最后一个回合只有$id=N\%3$的线程能进入到消费阶段，一旦该线程退出则其他两个线程就阻塞在了$count=N+1$处。所以消费、阻塞检测两个过程都需要加入退出条件。
+ 示例代码如下：

```java
class PrintThread1 implements Runnable {
    private static final Object LOCK = new Object();

    private static int count = 0; // 计数，同时确定线程是否要加入等待队列，还是可以直接去资源队列里面去获取数据进行打印
    private LinkedList<Integer> queue;
    private Integer threadNo;

    public PrintThread1(LinkedList<Integer> queue, Integer threadNo) {
        this.queue = queue;
        this.threadNo = threadNo;
    }

    @Override
    public void run() {
        while (true) {
            synchronized (LOCK) {
                while (count % 3 != this.threadNo) {
                    if (count >= 101) {
                        break;
                    }
                    try {
                        LOCK.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }

                if (count >= 101) {
                    break;
                }

                Integer val = this.queue.poll();
                System.out.println("thread-" + this.threadNo + ":" + val);
                count++;

                LOCK.notifyAll();
            }
        }
    }
}
```

### 使用`ReentrantLock,Condition,await,signal`实现

+ 有了这几个，我们首先可以把它当作synchronized锁用，主要需要注意在finally块中释放锁的问题，这里不再赘述。
+ 但是有了多个`Condtion`，我们可以不再无序一股脑地唤醒所有线程来竞争锁，我们可以为每一个线程设置一个Condition。那么每一个线程等待属于自己的`Condition.await`，由上一个线程结束消费时负责调用`Condition.signal`唤醒。
+ 在最后一个消费完成的线程中，可以一次性唤醒其他的线程让其他线程同样退出等待，结束运行。
+ 范例代码：

```java
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;


public class PrintNumber extends Thread {
    /**
     * 多个线程共享这一个sequence数据
     */
    private static int sequence=0;

    private static final int SEQUENCE_END =75;

    private Integer id;
    private ReentrantLock lock;
    private Condition[] conditions;



    private PrintNumber(Integer id,  ReentrantLock lock, Condition[] conditions) {
        this.id = id;
        this.setName("thread" + id);
        this.lock = lock;
        this.conditions = conditions;
    }

    @Override
    public void run() {
        while (sequence >= 0 && sequence < SEQUENCE_END) {
            lock.lock();
            try {
                //对序号取模,如果不等于当前线程的id,则先唤醒其他线程,然后当前线程进入等待状态
                while (sequence % conditions.length != id) {
                    conditions[(id + 1) % conditions.length].signal();
                    conditions[id].await();
                }
                System.out.println(Thread.currentThread().getName() + " " + sequence);
                //序号加1
                sequence = sequence + 1;
                //唤醒当前线程的下一个线程
                conditions[(id + 1) % conditions.length].signal();
                //当前线程进入等待状态
                conditions[id].await();
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                //将释放锁的操作放到finally代码块中,保证锁一定会释放
                lock.unlock();
            }
        }
        //数字打印完毕,线程结束前唤醒其余的线程,让其他线程也可以结束
        end();
    }

    private void end() {
        lock.lock();
        conditions[(id + 1) % conditions.length].signal();
        conditions[(id + 2) % conditions.length].signal();
        lock.unlock();
    }

    public static void main(String[] args) {
        int threadCount = 3;
        ReentrantLock lock = new ReentrantLock();
        Condition[] conditions = new Condition[threadCount];
        for (int i = 0; i < threadCount; i++) {
            conditions[i] = lock.newCondition();
        }
        PrintNumber[] printNumbers = new PrintNumber[threadCount];
        for (int i = 0; i < threadCount; i++) {
            PrintNumber p = new PrintNumber(i, lock, conditions);
            printNumbers[i] = p;
        }
        for (PrintNumber printNumber : printNumbers) {
            printNumber.start();
        }
    }

}
```

+ 显然本方法也可以**高效**的实现N个线程轮流打印的问题。