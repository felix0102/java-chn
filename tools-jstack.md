## jstack简单使用，定位死循环、线程阻塞、死锁等问题

### 死循环
```java
package concurrency;

public class Test {

    public static void main(String[] args) throws InterruptedException {
        while (true) {

        }
    }
}
```

先运行以上程序，程序进入死循环；

打开cmd，输入jps命令，jps很简单可以直接显示java进程的pid，如下为7588：

![](https://images2015.cnblogs.com/blog/879896/201604/879896-20160411105910645-1266879991.jpg)

输入jstack 7588命令，找到跟我们自己代码相关的线程，如下为main线程，处于runnable状态，在main方法的第八行，也就是我们死循环的位置：

![](https://images2015.cnblogs.com/blog/879896/201604/879896-20160411095252504-2109868749.jpg)

### Object.wait()情况

写个小程序，调用wait使其中一线程等待，如下：

```java
package concurrency;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

class TestTask implements Runnable {
    @Override
    public void run() {

        synchronized (this) {
            try {
                //等待被唤醒
                wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

    }
}

public class Test {

    public static void main(String[] args) throws InterruptedException {

        ExecutorService ex = Executors.newFixedThreadPool(1);
        ex.execute(new TestTask());

    }
}
```

同样我们先找到javaw.exe的PID，再利用jstack分析该PID，很快我们就找到了一个线程处于WAITING状态，在Test.java文件13行处，正是我们调用wait方法的地方，说明该线程目前还没等到notify，如下：

![](https://images2015.cnblogs.com/blog/879896/201604/879896-20160411100501879-346452628.jpg)


### 死锁

写个简单的死锁例子，如下：

```java
package concurrency;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

class TestTask implements Runnable {
    private Object obj1;
    private Object obj2;
    private int order;

    public TestTask(int order, Object obj1, Object obj2) {
        this.order = order;
        this.obj1 = obj1;
        this.obj2 = obj2;
    }

    public void test1() throws InterruptedException {
        synchronized (obj1) {
            //建议线程调取器切换到其它线程运行
            Thread.yield();
            synchronized (obj2) {
                System.out.println("test。。。");
            }

        }
    }
    public void test2() throws InterruptedException {
        synchronized (obj2) {
            Thread.yield();
            synchronized (obj1) {
                System.out.println("test。。。");
            }

        }
    }

    @Override
    public void run() {

        while (true) {
            try {
                if(this.order == 1){
                    this.test1();
                }else{
                    this.test2();
                }
                
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

    }
}

public class Test {

    public static void main(String[] args) throws InterruptedException {
        Object obj1 = new Object();
        Object obj2 = new Object();

        ExecutorService ex = Executors.newFixedThreadPool(10);
        // 起10个线程
        for (int i = 0; i < 10; i++) {
            int order = i%2==0 ? 1 : 0;
            ex.execute(new TestTask(order, obj1, obj2));
        }

    }
}
```

![](https://images2015.cnblogs.com/blog/879896/201604/879896-20160411103220957-2000485826.jpg)


### 等待IO

```java
package concurrency;


import java.io.IOException;
import java.io.InputStream;

public class Test {

    public static void main(String[] args) throws InterruptedException, IOException {

        InputStream is = System.in;
        int i = is.read();
        System.out.println("exit。");

    }
}
```
同样我们先找到javaw.exe的PID，再利用jstack分析该PID，很快jstack就帮我们找到了位置，Test.java文件12行，如下所示

![](https://images2015.cnblogs.com/blog/879896/201604/879896-20160411104201988-719620672.jpg)
