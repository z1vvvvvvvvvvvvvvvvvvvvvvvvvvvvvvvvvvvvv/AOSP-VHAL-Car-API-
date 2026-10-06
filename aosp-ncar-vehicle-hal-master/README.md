# AOSP VHAL 车辆属性模拟与 Car API 数据流验证

---

## 为什么做这个项目？

我原本是计算机科学背景，对车载 OS 和手机 OS 的区别很好奇。

手机 App 可以直接访问传感器、摄像头、GPS；
但车机应用不能直接读车速、空调温度、车门状态——中间必须经过 **VHAL（Vehicle Hardware Abstraction Layer）**。

我想搞清楚一件事：

> **车辆信号到底是怎么从 CAN 总线一路走到车机屏幕上的？**

---

## 这个项目是什么

### 1. 理解 VHAL 在 AAOS 中的定位

VHAL 是 Android Automotive OS 中车辆硬件与 Car Service 之间的抽象层。
所有车辆属性，例如车速、空调温度、车门状态，都必须经过 VHAL 才能被上层应用安全读取。

### 2. 区分 SYSTEM 属性与 VENDOR 属性

- **SYSTEM 属性**：Google 定义的标准车辆属性，项目中仅做配置，不实际处理。
- **VENDOR 属性**：车厂可自定义扩展的属性，本项目中对部分 VENDOR 属性进行了模拟。

即车厂如何在不破坏标准的前提下，扩展自己的车辆功能。

### 3. 模拟 VENDOR 属性变化

项目中部分 VENDOR 属性通过不同方式模拟：

- 定时器模拟：
  - `VENDOR_TEST_1S_COUNTER`
  - `VENDOR_TEST_500MS_COUNTER`

- 系统属性动态修改：
  - `VENDOR_TEST_SYS_PROP`
  - 可通过 ADB 命令动态改值：

```bash
adb shell setprop debug.vendor.nkh-lab.VENDOR_TEST_SYS_PROP 6789
```

### 4. 在 Car API 客户端验证数据更新

配合开源项目 [Car API Hello World](https://github.com/nkh-lab/car-api-hello-world)，
观察 VHAL 属性变化后，Car API 客户端如何实时刷新。

完整链路：

```text
车辆信号 / CAN Bus
        ↓
VHAL（Vehicle Hardware Abstraction Layer）
        ↓
Car Service
        ↓
CarPropertyManager
        ↓
应用层 UI
```

### 5. 了解 AOSP 源码集成方式

通过 `NCAR manifest` 和 `NCAR device` 项目，
理解了 VHAL 在 AOSP 源码树中的位置、依赖关系和编译配置方式。

---


## 数据流示意图

```text
+-----------------------------+
|  车辆硬件 / CAN Bus          |
+--------------+--------------+
               |
               v
+-----------------------------+
|  VHAL                       |
|  Vehicle Hardware HAL       |
|  负责车辆属性抽象与模拟       |
+--------------+--------------+
               |
               v
+-----------------------------+
|  Car Service                |
|  管理车辆属性、权限与订阅     |
+--------------+--------------+
               |
               v
+-----------------------------+
|  CarPropertyManager         |
|  应用层读取车辆属性的入口     |
+--------------+--------------+
               |
               v
+-----------------------------+
|  应用层 UI                   |
|  仪表、中控、座舱应用         |
+-----------------------------+
```

---

## 关键命令记录

动态修改 VENDOR 属性：

```bash
adb shell setprop debug.vendor.nkh-lab.VENDOR_TEST_SYS_PROP 6789
```

观察 Car API 客户端界面更新，验证 VHAL 属性变化是否成功传递到应用层。

---

## 如何运行

本项目依赖 AOSP 源码环境，完整编译需要下载 AOSP 源码树。

如果你只想理解逻辑，可以直接阅读：

- VHAL 属性定义
- Car API 客户端代码
- Car Service 与 CarPropertyManager 的调用关系


---

## 参考

- Car API 示例：[nkh-lab/car-api-hello-world](https://github.com/nkh-lab/car-api-hello-world)
- AOSP VHAL 类型定义：[types.hal](https://android.googlesource.com/platform/hardware/interfaces/+/refs/tags/android-11.0.0_r48/automotive/vehicle/2.0/types.hal)


---
