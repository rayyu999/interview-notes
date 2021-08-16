# 场景题



## AB线程交替打印

```java
public class ABThread {

    public static void main(String[] args) {
        int nThreads = Runtime.getRuntime().availableProcessors();
        SyncObject lock = new SyncObject();
        Thread t1 = new Thread(() -> {
                for (int i = 0; i < 10; ++i) {
                    lock.printA();
                }
            });
        Thread t2 = new Thread(() -> {
                for (int i = 0; i < 10; ++i) {
                    lock.printB();
                }
            });
        t1.start();
        t2.start();
    }

}

class SyncObject {

    private boolean flag = true;

    public void printA() {
        synchronized (this) {
            // 为true时才打印A
            while (!this.flag) {
                try {
                    this.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            // 打印并唤醒B
            System.out.println("A");
            flag = false;
            notifyAll();
        }
    }

    public synchronized void printB() {
        synchronized (this) {
            // 为false时才打印B
            while (this.flag) {
                try {
                    this.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
            // 打印并唤醒A
            System.out.println("B");
            flag = true;
            notifyAll();
        }
    }

}
```



## 用面向对象的思想设计停车场场景类

一个停车场要有查询剩余车位、停车计费功能，大概设计可以分为 6 个类：

* 停车场类：包括了停车场的各个组件以及信息；
* 车类：车辆实体；
* 相机类：对入场的车辆进行拍照，生成快照；
* 照片类：入场快照实体；
* 屏幕类：根据入场快照计算并显示车辆的停车费用；
* 停车位类：停车位实体。

### UML 类图

![](https://images.yingwai.top/picgo/202108162142210.png)

### 逻辑代码

```java
public class ParkingLot {
    private List<Place> places;
    private Camera camera;
    private Screen screen;
    private shotMap<String, Shot> shotMap;

}
```

