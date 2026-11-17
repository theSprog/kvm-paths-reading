# KVM 执行路径阅读地图

以一次真实事件为阅读单位，沿着触发、执行、状态变化和完成条件追踪，而不是按源码模块划分。范围是 Linux x86 KVM + QEMU + Linux Guest；前 30 篇覆盖普通虚拟机的主要运行闭环，后 4 篇是进阶扩展，不代表 KVM 全部功能。

来源：[KVM 路径阅读方法](https://chatgpt.com/share/6afb1a71-c548-83ec-a6f1-b1fd61db5ea5)。

## 使用方式

每篇目前是阅读提纲，不是已经核对源码的分析结论。开始前记录 Linux、QEMU、Guest 的 tag 或 commit、VMX/SVM 后端和相关配置；硬件辅助、irqchip、内存与设备后端会改变实际路径，不能把不同配置的分支拼成一条调用链。

沿五个问题补全笔记：Trigger、Call Path、State Changes、Invariant、Outcome。函数定位应附固定版本的源码路径和行号；普通内存访问等硬件路径应说明硬件行为，而不是虚构软件调用。异步路径需要注明生产者、消费者和完成通知。

完成标准：能解释事件从哪里来、跨越哪些执行上下文、修改哪些状态、由谁负责完成，以及最终把控制权交还给谁；再用 tracepoint、ftrace 或最小 VMM 实验核对关键分支。

## 主题索引

复选框表示对应路径是否已完成源码核对与关键分支验证。

### 创建与启动

- [ ] [01 创建虚拟机](01-create-vm.md)
- [ ] [02 分配 Guest RAM](02-guest-ram.md)
- [ ] [03 创建并初始化 vCPU](03-create-vcpu.md)
- [ ] [04 启动第一条 Guest 指令](04-first-instruction.md)

### CPU 执行

- [ ] [05 一次完整的 VMEXIT](05-vmentry-vmexit.md)
- [ ] [06 执行 CPUID](06-cpuid-exit.md)
- [ ] [07 访问虚拟 MSR](07-msr-access.md)
- [ ] [08 Guest 执行 HLT](08-vcpu-halt.md)
- [ ] [09 Host 抢占或 Kick vCPU](09-vcpu-preempt-kick.md)

### 内存访问

- [ ] [10 正常内存访问](10-memory-access.md)
- [ ] [11 二阶段地址转换缺页](11-ept-npt-fault.md)
- [ ] [12 Guest 自己的缺页异常](12-guest-page-fault.md)
- [ ] [13 Host 后备内存缺页](13-host-backing-fault.md)
- [ ] [14 Guest 修改页表与 TLB 失效](14-page-table-tlb.md)

### 中断与时间

- [ ] [15 外部设备中断](15-external-irq.md)
- [ ] [16 Guest EOI](16-guest-eoi.md)
- [ ] [17 虚拟定时器到期](17-lapic-timer.md)
- [ ] [18 vCPU 间 IPI](18-vcpu-ipi.md)
- [ ] [19 中断唤醒 Halted vCPU](19-irq-wakeup.md)

### 设备与 I/O

- [ ] [20 Legacy PIO](20-pio-exit.md)
- [ ] [21 MMIO 设备访问](21-mmio-exit.md)
- [ ] [22 PCI 设备发现](22-pci-discovery.md)
- [ ] [23 Virtio 设备初始化](23-virtio-init.md)
- [ ] [24 Virtio-blk 读磁盘](24-virtio-blk-read.md)
- [ ] [25 Virtio-net 发送数据](25-virtio-net-tx.md)
- [ ] [26 Virtio-net 接收数据](26-virtio-net-rx.md)
- [ ] [27 I/O 加速路径](27-ioeventfd-irqfd.md)

### 运行与生命周期

- [ ] [28 暂停与恢复](28-pause-resume.md)
- [ ] [29 Live Migration](29-live-migration.md)
- [ ] [30 Guest 关机与 VM 销毁](30-shutdown-destroy.md)

### 进阶路径

- [ ] [31 VFIO 设备直通](31-vfio-passthrough.md)
- [ ] [32 vhost-net 数据面](32-vhost-net.md)
- [ ] [33 Nested Virtualization](33-nested-virtualization.md)
- [ ] [34 vCPU 与内存热插拔](34-cpu-memory-hotplug.md)

## 建议阅读顺序

1. 最小执行闭环：01 → 02 → 03 → 04 → 05 → 06 → 08。目标是解释 Guest 如何运行、退出和恢复。
2. 内存与异常：10 → 11 → 12 → 13 → 14 → 07。目标是区分 Guest 缺页、二阶段缺页和 Host 后备页缺页。
3. 中断与调度：09 → 15 → 16 → 17 → 18 → 19。目标是理解运行、等待、唤醒与中断递送。
4. 设备 I/O：20 → 21 → 22 → 23 → 24 → 25 → 26 → 27。目标是追踪完整的磁盘或网络请求。
5. 生命周期：28 → 29 → 30。目标是解释暂停、迁移和销毁中的状态一致性。
6. 进阶扩展：31 → 32 → 33 → 34。按设备直通、网络后端、嵌套虚拟化和热插拔的需要选择。

## 容易混淆的边界

- 硬件 VMEXIT 不等于 KVM_RUN 返回用户态，KVM_EXIT_* 是用户态接口的退出原因。
- memslot 注册建立 GPA 与后备内存的关系，不代表二阶段页表已经全部建立。
- Guest #PF、EPT Violation/NPT Fault 和 Host 后备内存缺页属于不同层次，可能出现在同一次访问的不同阶段。
- ioeventfd 可以避免部分用户态退出，但不等于消除硬件 VMEXIT；是否经过 QEMU 线程取决于设备后端。
- 磁盘与网络路径要追到请求完成和缓冲区回收，不能在队列通知处停止。
