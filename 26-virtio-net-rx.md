# 26 Virtio-net 接收数据

[返回索引](README.md)

> 阅读提纲：实际调用链、状态与不变量需在选定版本和配置下核对。

## 阅读基线

- Linux / QEMU / Guest 版本或 commit：待填写。
- 硬件后端、irqchip、内存或设备配置：待填写。
- 核心问题：Guest 预先提供的缓冲区怎样被填充，通知抑制与 NAPI 如何配合？

## Trigger：触发事件

Host 网络收到发往 Guest 的数据。

## Call Path：事件路径

Host 网络 → TAP/网络后端 → Virtio-net RX → Guest IRQ/NAPI → Guest Socket。

待补充实际执行的关键函数、源码位置和分支条件；标出用户态、内核态、Guest 与硬件之间的边界，以及异步通知与完成点。

## State Changes：状态变化

待记录路径开始、关键转折和结束时的状态，以及每项状态的存储位置、修改者与可见时机。

## Invariant：必须维持的条件

待从源码确认正确性条件，并解释依赖的锁、原子操作、内存屏障、IRQ/抢占约束或硬件规则；不适用的机制无需列入。

## Outcome：完成与去向

目标终点：数据交给 Guest socket，接收缓冲区生命周期闭合。

待记录正常、重试与错误分支分别向谁交还控制权，并附可复现的实验或跟踪证据。
