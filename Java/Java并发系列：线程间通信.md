#### Java并发基础：线程间通信

在Android中，我们有Handler，在子线程和主线程之间通信，只需要拿到主线程的Handler发送消息即可，但是子线程和子线程之间通信，我们却知道很少，毕竟很少用~那么今天我们来学习一下JDK中提供的线程通信的Api。

#### join加入

现在有2个线程，线程A和线程B，B需要等待A执行完后，才能执行，那么应该怎么做呢？我们可以使用**join**。使用**join**，即使B在A之前启动，也会让B在A执行完毕后再继续执行。

```
/**
 * B等A
 */
private static void joinTest() {
    Thread threadA = new Thread() {
        @Override
        public void run() {
            super.run();
            try {
                Thread.sleep(1500);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("我是A线程，我执行完了...");
        }
    };
    Thread threadB = new Thread() {
        @Override
        public void run() {
            super.run();
            System.out.println("我是B线程，我在等A执行完");
            try {
                threadA.join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("A线程执行完了，我可以执行了~");
        }
    };
    threadB.start();
    threadA.start();
}

//输出
我是B线程，我在等A执行完
我是A线程，我执行完了...
A线程执行完了，我可以执行了~
```

#### wait等待、notify唤醒

如果需要控制线程之间交替执行，并且可以按照我们指定的顺序执行呢？我们可以使用**synchronized**和**Lock锁**，加上**wait()**释放锁，**notify**唤醒其他线程并获得锁。

```
/**
 * B和A交叉执行
 */
private static void waitNotifyTest() {
    Object lock = new Object();
    Thread threadA = new Thread() {
        @Override
        public void run() {
            super.run();
            System.out.println("A：等待锁~");
            synchronized (lock) {
                System.out.println("A：拿到锁了~");
                try {
                    System.out.println("A：wait释放锁~");
                    lock.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("A：被唤醒了，拿到了锁~");
                System.out.println("A：啦啦啦~");
                System.out.println("A：嘿嘿~");
            }
        }
    };
    Thread threadB = new Thread() {
        @Override
        public void run() {
            super.run();
            System.out.println("B等待锁~");
            synchronized (lock) {
                System.out.println("B：得到了锁~");
                System.out.println("B：嗯呐~");
                System.out.println("B：啵啵~");
                System.out.println("B：好啦，完事~");
                //随机唤醒一个线程，但当前只有一个A线程，所以只有它
                lock.notify();
            }
        }
    };
    threadA.start();
    threadB.start();
}

//输出
A：等待锁~
A：拿到锁了~
A：wait释放锁~
B等待锁~
B：得到了锁~
B：嗯呐~
B：啵啵~
B：好啦，完事~
A：被唤醒了，拿到了锁~
A：啦啦啦~
A：嘿嘿~
```

#### CountDownLatch计数器

当需要一个线程D，在等待线程A、B、C3个线程都执行完后，再执行，有什么Api使用呢？我们可以使用**countDownLatch**计数器，线程A、B、C都执行完时让计数器-1，到0时，线程D就会被放行，继续完成它的任务。

```
/**
 * D等A\B\C都执行完毕后再开始执行
 */
private static void countDownLatchTest() {
    CountDownLatch countDownLatch = new CountDownLatch(3);
    Thread threadD = new Thread() {
        @Override
        public void run() {
            super.run();
            System.out.println("D：" + "开始等待其他线程执行~");
            System.out.println("--------------- 华丽的分割线 -----------------");
            try {
                //开始等待其他线程
                countDownLatch.await();
                System.out.println("--------------- 华丽的分割线 -----------------");
                System.out.println("D：小兔崽子们都执行完了~我可以跑了~");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    };
    threadD.start();
    for (char threadName = 'A'; threadName <= 'C'; threadName++) {
        String name = String.valueOf(threadName);
        new Thread() {
            @Override
            public void run() {
                super.run();
                System.out.println(name + "：开始运行~");
                try {
                    Thread.sleep(300);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(name + "：任务执行完啦~");
                //让通知计数器减数
                countDownLatch.countDown();
            }
        }.start();
    }
}

//输出
D：开始等待其他线程执行~
--------------- 华丽的分割线 -----------------
A：开始运行~
B：开始运行~
C：开始运行~
A：任务执行完啦~
B：任务执行完啦~
C：任务执行完啦~
--------------- 华丽的分割线 -----------------
D：小兔崽子们都执行完了~我可以跑了~
```

#### CyclicBarrier屏障

当需要3个线程都准备完毕后，再同时执行。就像体育运动员跑步，全部运动员都准备好了，再开始起跑。要实现这个需求，我们就可以**CyclicBarrier**屏障。**CyclicBarrier的构造函数**传入线程的数量，再在线程准备完毕时，调用**cyclicBarrier.await()**通知开始等待其他线程，当达到构造时传入的线程数量时，所有线程再继续前行~

```
/**
 * A\B\C3个线程全部准备完毕后，一起执行
 */
private static void cyclicBarrierTest() {
    int runner = 3;
    CyclicBarrier cyclicBarrier = new CyclicBarrier(runner);
    Random random = new Random();
    for (char runnerName = 'A'; runnerName <= 'C'; runnerName++) {
        String name = String.valueOf(runnerName);
        new Thread() {
            @Override
            public void run() {
                super.run();
                int prepareTime = random.nextInt(10000) + 100;
                System.out.println("运动员" + name + "：需要准备" + prepareTime + "毫秒");
                try {
                    Thread.sleep(prepareTime);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println("运动员" + name + "：准备完毕，等待其他运动员准备");
                try {
                    //通知准备好了，等待其他线程
                    cyclicBarrier.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                }
                System.out.println("所有的运动员都准备好了，开跑~");
            }
        }.start();
    }
}

//输出
运动员A：需要准备4308毫秒
运动员B：需要准备3584毫秒
运动员C：需要准备3396毫秒
运动员C：准备完毕，等待其他运动员准备
运动员B：准备完毕，等待其他运动员准备
运动员A：准备完毕，等待其他运动员准备
所有的运动员都准备好了，开跑~
所有的运动员都准备好了，开跑~
所有的运动员都准备好了，开跑~
```

#### 总结

本篇学习了JDK中提供的线程间通信的Api，在Android中，虽然大部分线程之间都是子线程做耗时操作，再切换到主线程进行UI更新，但是有些情况下还是需要用到线程之间的通信的，尤其是线程之间执行的顺序控制，在一些特殊场景，例如蓝牙扫描，位置扫描等，都需要维护一个子线程不断轮训，轮训期间将扫描到的信息，再组合，发送到外部，外部再更新UI，有些时候Handler是帮不了我们忙时，就需要Java JDK中提供的一些线程间Api。