# 23 Virtio 设备初始化

[返回索引](README.md)

> 阅读提纲：实际调用链、状态与不变量需在选定版本和配置下核对。

## 阅读基线

- Linux / QEMU / Guest 版本或 commit：待填写。
- 硬件后端、irqchip、内存或设备配置：待填写。
- 核心问题：选定 transport 和 split/packed ring 后，特性、队列地址与设备状态如何达成一致？

## Trigger：触发事件

Guest 驱动探测到 Virtio 设备。

## Call Path：事件路径

Guest Virtio Driver → PCI/MMIO Transport → 特性协商 → Virtqueue 配置 → DRIVER_OK。

待补充实际执行的关键函数、源码位置和分支条件；标出用户态、内核态、Guest 与硬件之间的边界，以及异步通知与完成点。

## State Changes：状态变化

待记录路径开始、关键转折和结束时的状态，以及每项状态的存储位置、修改者与可见时机。

## Invariant：必须维持的条件

待从源码确认正确性条件，并解释依赖的锁、原子操作、内存屏障、IRQ/抢占约束或硬件规则；不适用的机制无需列入。

## Outcome：完成与去向

目标终点：设备与驱动就绪，可通过 virtqueue 提交请求。

待记录正常、重试与错误分支分别向谁交还控制权，并附可复现的实验或跟踪证据。
