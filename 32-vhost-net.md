# 32 vhost-net 数据面

[返回索引](README.md)

> 阅读提纲：实际调用链、状态与不变量需在选定版本和配置下核对。

## 阅读基线

- Linux / QEMU / Guest 版本或 commit：待填写。
- 硬件后端、irqchip、内存或设备配置：待填写。
- 核心问题：与 QEMU 用户态后端相比，vhost 接管哪些工作，队列内存和 eventfd 如何配置？

## Trigger：触发事件

Guest 提交 virtio-net 队列操作或 TAP 收到数据。

## Call Path：事件路径

Guest Virtqueue ↔ ioeventfd/vhost Worker ↔ TAP；完成通知 → irqfd → Guest。

待补充实际执行的关键函数、源码位置和分支条件；标出用户态、内核态、Guest 与硬件之间的边界，以及异步通知与完成点。

## State Changes：状态变化

待记录路径开始、关键转折和结束时的状态，以及每项状态的存储位置、修改者与可见时机。

## Invariant：必须维持的条件

待从源码确认正确性条件，并解释依赖的锁、原子操作、内存屏障、IRQ/抢占约束或硬件规则；不适用的机制无需列入。

## Outcome：完成与去向

目标终点：选定 TX 或 RX 后，数据与队列缓冲区完成一个闭环。

待记录正常、重试与错误分支分别向谁交还控制权，并附可复现的实验或跟踪证据。
