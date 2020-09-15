---
title: "Java Thread"
date: 2020-09-08T13:34:30+08:00
draft: false
tags: [多线程]
categories: [Java]
comment: true
---
Java多线程
<!--more-->

# Java多线程

- 继承Thread类

```
public class Threadtest extends Thread {

    /**
     *
     * 多线程学习
     * **/
    @Override
    public void run(){
        for (int i = 0; i < 2000; i++) {
            System.out.println("我在看代码--");
        }
    }
    public static void main(String[] args) {

        Threadtest test = new Threadtest();

        test.start();
        for (int i = 0; i < 2000; i++) {
            System.out.println("我在学习多线程--");
        }
    }
```

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200907110727.png)

- 实现Runnable接口

```
public class Threadtest implements Runnable {

    /**
     *
     * 多线程学习
     * **/
    @Override
    public void run(){
        for (int i = 0; i < 2000; i++) {
            System.out.println("我在看代码--");
        }
    }
    public static void main(String[] args) {

        Threadtest test = new Threadtest();

        new Thread(test).start();

        for (int i = 0; i < 2000; i++) {
            System.out.println("我在学习多线程--");
        }
    }


}

```

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200907113426.png)

- 龟兔赛跑

```
public class Threadtest implements Runnable {

    /**
     *
     * 龟兔赛跑
     * **/

    private static String winner;

    @Override
    public void run(){
        for (int i = 0; i <= 100; i++) {
            if(Thread.currentThread().getName().equals("兔子")&& 1%10 ==0){
                try{
                    Thread.sleep(10);
                }catch (InterruptedException e){
                    e.printStackTrace();
                }
            }

            boolean flag = gameover(i);

            if ((flag)) {
                break;
            }

            System.out.println(Thread.currentThread().getName()+"跑了"+i+"步");
        }
    }

    private boolean gameover(int steps){
        if(winner!=null){
            return true;
        }{
            if(steps>=100){
                winner = Thread.currentThread().getName();
                System.out.println("winner is"+winner);
            }
        }

        return false;
    }
    public static void main(String[] args) {

        Threadtest test = new Threadtest();
        new Thread(test,"兔子").start();
        new Thread(test,"乌龟").start();
    }


}
```

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200907160554.png)

- 线程停止（建议使用标志位）

```
public class Threadtest implements Runnable {

    /**
     *
     * 龟兔赛跑
     * **/

    private boolean flag = true;

    @Override
    public void run(){
        int i =0;
        while (flag){
            System.out.println("run...Thread"+i++);
        }
    }

    public void stop(){
        this.flag = false;
    }


    public static void main(String[] args) {
        Threadtest test = new Threadtest();
        new Thread(test).start();

        for (int i = 0; i < 1000; i++) {
            System.out.println("main"+i);

            if(i==900) {
                test.stop();
                System.out.println("子线程停止");
            }
        }
    }

}
```

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200907171634.png)

- 线程休眠

```
Thread.sleep(1000)
```

- 线程礼让（yield）

```
public class Threadtest implements Runnable {


    private boolean flag = true;

    @Override
    public void run(){
        System.out.println(Thread.currentThread().getName()+"线程开始执行");
        Thread.yield();
        System.out.println(Thread.currentThread().getName()+"线程停止执行");
    }

    public static void main(String[] args) {
        Threadtest test = new Threadtest();

        new Thread(test,"a").start();
        new Thread(test,"b").start();
    }

}
```

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200907174748.png)

- 线程强制执行(join)

```
public class Threadtest implements Runnable {


    @Override
    public void run(){
        for (int i = 0; i < 1000; i++) {
            System.out.println("线程vipl来了"+i);
        }
    }

    public static void main(String[] args) throws InterruptedException{

        Threadtest test = new Threadtest();
        Thread thread = new Thread(test);
        thread.start();

        for (int q = 0; q < 500; q++) {
            if(q==200){
                thread.join();
            }
            System.out.println("main"+q);
        }
    }

}

```

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200907191302.png)

- 线程优先级

**优先级只是意味着获得调度的概率低，并不是优先级低就不会被调用，顺序也是概率高，顺序可能也是不同的，全看CPU的调度**

```

public class Threadtest {


    public static void main(String[] args) throws InterruptedException{

        //主线程默认优先级
        System.out.println(Thread.currentThread().getName()+"---->"+Thread.currentThread().getPriority());

        MyPriority MyPriority = new MyPriority();

        Thread t1= new Thread(MyPriority);
        Thread t2= new Thread(MyPriority);
        Thread t3= new Thread(MyPriority);

        //设置优先级

        t1.start();
        t2.setPriority(6);
        t2.start();
        t3.setPriority(Thread.MAX_PRIORITY);
        t3.start();
    }

}

class MyPriority implements Runnable{

    @Override
    public void run(){
        System.out.println(Thread.currentThread().getName()+"---->"+Thread.currentThread().getPriority());
    }
}
```

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200907194216.png)

- 守护线程

```
public class TestDaemon {

    /**
     * 测试守护线程
     * **/

    public static void main(String[] args) throws InterruptedException{

        god god = new god();
        Y4er Y4er = new Y4er();

         Thread thread = new Thread(god);
        thread.setDaemon(true);
        thread.start();

        new Thread(Y4er).start();

    }

}


class god implements Runnable{

    @Override
    public void run(){
        while (true){
            System.out.println("上帝保佑着你");
        }
    }
}
class Y4er implements Runnable{

    @Override
    public void run(){
        for (int i = 0; i < 36500; i++) {
            System.out.println("开心的活着");
        }

        System.out.println("Googbyg world");
    }
}
```

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200907195704.png)

- 线程同步

**锁机制**

```
synchronized(放置于资源竞争处)
```

**同步方法**

```
private synchronized void buy(){xx}
```

**同步块**

```
synchronized(Obj){}
```

- 死锁

```
public class DeadLock {

    public static void main(String[] args) {

        Makeup g1= new Makeup(0,"灰姑娘");
        Makeup g2= new Makeup(1,"白雪公主");
        g1.start();
        g2.start();
    }
}

//口红
class LipStick{}

//镜子

class Mirror{}

class Makeup extends Thread{

    static LipStick lipStick = new LipStick();
    static Mirror mirror = new Mirror();

    int choice;
    String girlName;

    Makeup(int choice,String girlName){
        this.choice = choice;
        this.girlName = girlName;
    }

    @Override
    public void run() {
        try {
            makeup();
        }catch (InterruptedException e){
            e.printStackTrace();
        }
    }

    private void makeup() throws InterruptedException{
        if(choice==0){
            synchronized (lipStick){
                System.out.println(this.girlName+"获得口红的🔒");
                Thread.sleep(1000);
                synchronized(mirror){
                    System.out.println(this.girlName+"获得镜子的🔒");
                }

            }
        }else {
            synchronized (mirror){
                System.out.println(this.girlName+"获得口红的🔒");
                Thread.sleep(1000);
                synchronized(lipStick){
                    System.out.println(this.girlName+"获得镜子的🔒");
                }

            }
        }
    }
}
```

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200908104512.png)

- Lock锁

```
import java.util.concurrent.locks.ReentrantLock;

public class TestLock {

    public static void main(String[] args) {
        TestLock2 test = new TestLock2();
        new Thread(test).start();
        new Thread(test).start();
        new Thread(test).start();

    }
}


class TestLock2 implements Runnable{


    int ticketNums = 10;

    private final ReentrantLock lock = new ReentrantLock();

    @Override
    public void run() {
        while(true){
            try{
                lock.lock();

                if(ticketNums>0){
                    try{
                        Thread.sleep(1000);
                    }catch (InterruptedException e){
                        e.printStackTrace();
                    }
                    System.out.println(ticketNums--);
                }else {
                    break;
                }
            }finally {
                lock.unlock();
            }
        }
    }
}
```

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200908113804.png)

## 线程协作（生产者与消费者问题）

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200908114642.png)

### 管程法

```

/**
 *
 *测试生产者消费者模型-》利用缓冲区解决：管程法
**/

//生产者，消费者，产品，缓冲区
public class TestPC {

    public static void main(String[] args) {
        SynContainer container = new SynContainer();

        new Productor(container).start();
        new Consumer(container).start();
    }
}


//生产者

class Productor extends Thread{

    SynContainer container;
    public Productor(SynContainer container){
        this.container = container;
    }

    //生产


    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            System.out.println("生产了"+i+"只鸡");
            container.push(new Chicken(i));
        }
    }
}


//消费者
class Consumer extends Thread{
    SynContainer container;

    public Consumer(SynContainer container){
        this.container = container;
    }

    //消费


    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            System.out.println("消费了第"+container.pop().id+"只鸡");
        }
    }
}

//产品

class Chicken{
    int id; //产品编号

    public Chicken(int id) {
        this.id = id;
    }
}

//缓冲区

class SynContainer{

    //需要一个容器大小

    Chicken[] chickens = new Chicken[10];

    //容器计数器

    int count = 0;
    //生产者放入产品

    public synchronized void push(Chicken chicken){

        //如果容器满了，就需要等待消费者消费
        if(count==chickens.length){
            //通知消费者消费，生产等待

            try{
                this.wait();
            }catch (InterruptedException e ){
                e.printStackTrace();
            }
        }
        //荣国没有满，就需要丢入产品
        chickens[count]=chicken;
        count++;

        //可以通知消费者消费了

        this.notifyAll();
    }

    //消费者消费产品

    public synchronized Chicken pop(){
        //判断能否消费

        if(count==0){
            //等待生产者生产，消费者等待
            try{
                this.wait();
            }catch (InterruptedException e){
                e.printStackTrace();
            }

        }

        //如果可以消费
        count--;
        Chicken chicken = chickens[count];

        //吃完了，通知生产者生产
        this.notifyAll();
        return chicken;
    }
    
}
```
![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200908123759.png)

### 信号灯法(设置标志位)

```
public class TestPc2 {

    public static void main(String[] args) {

        TV tv = new TV();

        new Player(tv).start();
        new Watcher(tv).start();
    }
}


//生产者-〉演员

class Player extends Thread{
    TV tv;
    public Player(TV tv){
        this.tv = tv;
    }

    @Override
    public void run() {
        for (int i = 0; i < 20; i++) {
            if(i%2==0){
                this.tv.play("快乐大本营");
            }else {
                this.tv.play("抖音，记录美好生活");

            }        }
    }
}

//消费者-〉观看

class Watcher extends Thread{
    TV tv;
    public Watcher(TV tv){
        this.tv = tv;
    }

    @Override
    public void run() {
        for (int i = 0; i < 20; i++) {
            tv.watch();
        }
    }
}

//产品->节目

class TV{
    //演员表演，观众等待
    //观众观看，演员等待

    String voice;
    boolean flag = true;

    //表演

    public synchronized void play(String voice){

        if(!flag){
            try{
                this.wait();
            }catch (InterruptedException e){
                e.printStackTrace();

            }
        }        System.out.println("演员表演了:"+voice);

        this.notifyAll();
        this.voice = voice;
        this.flag = !this.flag;
    }

    //观看


    public synchronized void watch(){

        if(flag){
            try{
                this.wait();
            }catch (InterruptedException e){
                e.printStackTrace();
            }
        }

        System.out.println("观看了"+voice);

        //通知演员表演

        this.notifyAll();
        this.flag = !this.flag;

    }
}
```

![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200908131618.png)

## 线程池

```
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

/**
 * 测试线程池
 * **/
public class TestPool {
    public static void main(String[] args) {
        ExecutorService service = Executors.newFixedThreadPool(10);

        service.execute(new MyThread());
        service.execute(new MyThread());
        service.execute(new MyThread());
        service.execute(new MyThread());

    }

}


class MyThread implements Runnable{

    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            System.out.println(Thread.currentThread().getName()+i);
        }
    }
}
```
![](https://maekdown-1300474679.cos.ap-beijing.myqcloud.com/20200908133136.png)

