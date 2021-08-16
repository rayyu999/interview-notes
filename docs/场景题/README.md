# 场景题



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

