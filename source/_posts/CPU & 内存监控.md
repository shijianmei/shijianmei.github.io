---
title: CPU & 内存监控
date: 2021-03-24 20:12:32
categories: 
- 性能
tags:
---

## CPU

CPU 是移动设备最重要的计算资源，设计糟糕的应用可能会造成 CPU 持续以高负载运行，一方面会导致用户使用过程遭遇卡顿；另一方面也会导致手机发热发烫，电量被快速消耗完，严重影响用户体验。

### 相关系统原理

iOS 是基于 Apple Darwin 内核，由 kernel、XNU 和 Runtime 组成，XNU（X is not UNIX） 是 Darwin 的内核，一个混合内核，由 Mach 微内核和 BSD 组成。Mach 内核是轻量级的平台，只能完成操作系统最基本的职责，如：进程和线程、虚拟内存管理、任务调度、进程通信和消息传递机制。其他的工作，如文件操作和设备访问，都是由 BSD 层实现。

事实上，Mach 并不能识别 UNIX 中的所有进程，而是采用一种稍微不同的方式，使用了比进程更轻量级的概念：**任务（Task）**。经典的 UNIX 采用了自上而下的方式：最基本的对象是进程，然后进一步划分为一个或多个线程；Mach 则采用了自底向上的方式：最基本的单元是线程，一个或多个线程包含在一个任务中。

>**线程：**线程定义了 Mach 中最小的执行单元。线程表示的是底层的机器寄存器状态以及各种调度统计数据，其从设计上提供了调度所需要的大量信息。
>**任务：**任务是一种容器对象，虚拟内存空间和其他资源都是通过这个容器对象管理的。这些资源包括设备和其他句柄。资源进一步被抽象为端口。因此，资源的共享实际上相当于允许对对应端口进行访问。
严格来说，Mach 的任务并不是其它操作系统中所谓的进程，因为 Mach 作为一个微内核的操作系统，并没有提供“进程”的逻辑，而只提供了最基本的实现。在 BSD 模型中，这两个概念有一对一的简单映射，每个 BSD 进程（即 OS X 进程）都在底层关联了一个 Mach 任务对象。实现这种映射的方法是指定一个透明的指针 `bsd_info`，Mach 对 `bsd_info` 完全无知。Mach 将内核也用任务表示（全局范围称为 `kernel_task`），尽管该任务没有对应的 PID，但可以想象 PID 为 0。

下图所示为权威著作《OS X Internal: A System Approach》中提供的 Mach OS X 中进程子系统组成的概念图。与 Mac OS X 类似，iOS 的线程技术也是基于 Mach 线程技术实现的。
![](https://raw.githubusercontent.com/shijianmei/blog_Images/main/CPU%26%E5%86%85%E5%AD%98%E7%9B%91%E6%8E%A7/image.png)

 Mach 层中的 `thread_basic_info` 结构体的定义：

```c++
 struct thread_basic_info {
        time_value_t    user_time;      // 用户运行时长
        time_value_t    system_time;    // 系统运行时长
        integer_t       cpu_usage;      // CPU 使用率
        policy_t        policy;         // 调度策略
        integer_t       run_state;      // 运行状态
        integer_t       flags;          // 各种标记
        integer_t       suspend_count;  // 暂停线程的计数
        integer_t       sleep_time;     // 休眠时间
};
```
   
### APP 的 CPU 占用率

对于app使用率的获取，网上的方案还是比较统一的。

1. 使用task_threads函数，获取当前App行程中所有的线程列表。
2. 对于第一步中获取的线程列表进行遍历，通过thread_info函数获取每一个非闲置线程的cpu使用率，进行相加。
3. 使用vm_deallocate函数释放资源。
如下所示为 **CPU 占用率** 的代码实现：

```c++
+ (CGFloat)cpuUsageForApp {
    kern_return_t kr;
    thread_array_t         thread_list;
    mach_msg_type_number_t thread_count;
    thread_info_data_t     thinfo;
    mach_msg_type_number_t thread_info_count;
    thread_basic_info_t basic_info_th;
    
    // get threads in the task
    //  获取当前进程中 线程列表
    kr = task_threads(mach_task_self(), &thread_list, &thread_count);
    if (kr != KERN_SUCCESS)
        return -1;
 
    float tot_cpu = 0;
    
    for (int j = 0; j < thread_count; j++) {
        thread_info_count = THREAD_INFO_MAX;
        //获取每一个线程信息
        kr = thread_info(thread_list[j], THREAD_BASIC_INFO,
                         (thread_info_t)thinfo, &thread_info_count);
        if (kr != KERN_SUCCESS)
            return -1;
        
        basic_info_th = (thread_basic_info_t)thinfo;
        if (!(basic_info_th->flags & TH_FLAGS_IDLE)) {
            // cpu_usage : Scaled cpu usage percentage. The scale factor is TH_USAGE_SCALE.
            //宏定义TH_USAGE_SCALE返回CPU处理总频率：
            tot_cpu += basic_info_th->cpu_usage / (float)TH_USAGE_SCALE;
        }
        
    } // for each thread
    
    // 注意方法最后要调用 vm_deallocate，防止出现内存泄漏
    kr = vm_deallocate(mach_task_self(), (vm_offset_t)thread_list, thread_count * sizeof(thread_t));
    assert(kr == KERN_SUCCESS);
    
    return tot_cpu;
}
```

## Memory

### 相关系统原理

内存是有限且系统共享的资源，一个程序占用越多，系统和其他程序所能用的就越少。程序启动前都需要先加载到内存中，并且在程序运行过程中的数据操作也会占用一定的内存资源。减少内存占用也能同时减少其对 CPU 时间维度上的消耗，从而使不仅使 App 以及整个系统也都能表现的更好。

MacOS 和 iOS 都采用了虚拟内存技术来突破物理内存的大小限制，每个进程都有一段由多个大小相同的页（Page）所构成的逻辑地址空间。处理器和内存管理单元（MMU，Memory Management Unit）维护着由逻辑地址到物理地址的 **页面映射表**（简称 **页表**），当程序访问逻辑内存地址时，由 MMU 根据页表将逻辑地址转换为真实的物理地址。在早期的苹果设备中，每个页的大小为 4KB；基于 A7 和 A8 处理器的系统为 64 位程序提供了 16KB 的虚拟内存分页和 4KB 的物理内存分页；在 A9 之后，虚拟内存和物理内存的分页大小都达到了 16KB。

虚拟内存分页（Virtual Page，VP）有两种类型：

1. Clean：指能够被系统清理出内存且在需要时能重新加载的数据，包括：
    1. 内存映射文件
    2. Frameworks 中的 __DATA_CONST 部分
    3. 应用的二进制可执行文件
2. Dirty：指不能被系统回收的内存占用，包括：
    1. 所有堆上的对象
    2. 图片解码缓冲数据
    3. Framework 中的 **DATA 和**DATA_DIRTY 部分
由于内存容量和读写寿命的限制，iOS 上没有 Disk Swap 机制，取而代之使用 **Compressed Memory** 技术。 Disk Swap 是指在 macOS 以及一些其他桌面操作系统中，当内存可用资源紧张时，系统将内存中的内容写入磁盘中的backing store（Swapping out），并且在需要访问时从磁盘中再读入 RAM（Swapping in）。与大多数 UNIX 系统不同的是，macOS 没有预先分配磁盘中的一部分作为 backing store，而是利用引导分区所有可用的磁盘空间。

苹果最初只是公开了从 OS X Mavericks 开始使用 Compressed Memory 技术，但 iOS 系统也从 iOS 7 开始悄悄地使用。

Compressed Memory 技术在内存紧张时能够将最近使用过的内存占用压缩至原有大小的一半以下，并且能够在需要时解压复用。它在节省内存的同时提高了系统的响应速度，其特点可以归结为：

* 减少了不活跃内存占用
* 改善电源效率，通过压缩减少磁盘 IO 带来的损耗
* 压缩/解压非常快，能够尽可能减少 CPU 的时间开销
* 支持多核操作
本质上，Compressed Memory 也是 Dirty Memory。因此，**memory footprint = dirty size + compressed size**，这也是我们需要并且能够尝试去减少的内存占用。

### App 使用的内存

mach_task_basic_info 结构体存储了 Mach task 的内存使用信息，其中 resident_size 就是应用使用的物理内存大小，virtual_size 是虚拟内存大小。

```c++
#define MACH_TASK_BASIC_INFO     20         /* always 64-bit basic info */
struct mach_task_basic_info {
        mach_vm_size_t  virtual_size;       /* virtual memory size (bytes) */
        mach_vm_size_t  resident_size;      /* resident memory size (bytes) */
        mach_vm_size_t  resident_size_max;  /* maximum resident memory size (bytes) */
        time_value_t    user_time;          /* total user run time for
                                               terminated threads */
        time_value_t    system_time;        /* total system run time for
                                               terminated threads */
        policy_t        policy;             /* default policy for new threads */
        integer_t       suspend_count;      /* suspend count for task */
};
```
task_info API 根据指定的 flavor 类型返回 target_task 的信息。
```c++
kern_return_t task_info
(
    task_name_t target_task,
    task_flavor_t flavor,
    task_info_t task_info_out,
    mach_msg_type_number_t *task_info_outCnt
);
```
于是得到获取当前 App Memory 的使用情况
```c++
- (NSUInteger)getResidentMemory
{
    struct mach_task_basic_info info;
    mach_msg_type_number_t count = MACH_TASK_BASIC_INFO_COUNT;
    int r = task_info(mach_task_self(), MACH_TASK_BASIC_INFO, (task_info_t)& info, & count);
    if (r == KERN_SUCCESS)
    {
        return info.resident_size;
    }
    else
    {
        return -1;
    }
}
```
将上述代码采集到的 App RAM 的使用值与 Xcode 的 Debug Gauges 的 memory 对比，会发现代码会与 Debug Gauges 显示的值存在差异，有时甚至会差几百 MB，那么究竟怎样才能获取到应用使用的真实内存值呢？
我们先来看看 WebKit 源码中是怎样使用的，在 [MemoryFootprintCocoa.cpp](https://github.com/WebKit/webkit/blob/52bc6f0a96a062cb0eb76e9a81497183dc87c268/Source/WTF/wtf/cocoa/MemoryFootprintCocoa.cpp) 文件中，代码如下：

```c++
size_t memoryFootprint()
{
    task_vm_info_data_t vmInfo;
    mach_msg_type_number_t count = TASK_VM_INFO_COUNT;
    kern_return_t result = task_info(mach_task_self(), TASK_VM_INFO, (task_info_t) &vmInfo, &count);
    if (result != KERN_SUCCESS)
        return 0;
    return static_cast<size_t>(vmInfo.phys_footprint);
}
```
可以看到代码使用的不是 resident_size，而是 phys_footprint，phys_footprint 同样是 task_info 的成员变量。
当我们将获取内存的实现从 resident_size 换成 phys_footprint 时，于是代码获取的内存值就和 Xcode Debug Gauges 一致了。

```c++
- (NSUInteger)getApplicationUsedMemory
{
    struct mach_task_basic_info info;
    mach_msg_type_number_t count = MACH_TASK_BASIC_INFO_COUNT;
    int r = task_info(mach_task_self(), MACH_TASK_BASIC_INFO, (task_info_t)& info, & count);
    if (r == KERN_SUCCESS)
    {
        return info.phys_footprint;
    }
    else
    {
        return -1;
    }
}
```
### 获取设备总内存：

```c++

+ (NSUInteger)totalMemoryForDevice{
    return [NSProcessInfo processInfo].physicalMemory/1024/1024;
}
```

[DoKit](https://github.com/didi/DoKit)
[GT](https://github.com/Tencent/GT)
[iOS-System-Services](https://github.com/Shmoopi/iOS-System-Services)
[kern_memorystatus](https://github.com/apple/darwin-xnu/blob/0a798f6738bc1db01281fc08ae024145e84df927/bsd/kern/kern_memorystatus.c)
[NeteaseAPM iOS SDK](https://www.infoq.cn/article/netease-ios-sdk-neteaseapm-technology-share/)[CPU-Dasher-for-iOS](https://github.com/zenny-chen/CPU-Dasher-for-iOS)
[HookCppInitilizers](https://github.com/everettjf/Yolo/blob/master/HookCppInitilizers/hook_cpp_init.mm)
[a-method-of-hook-objective-c-load](https://everettjf.github.io/2017/01/06/a-method-of-hook-objective-c-load/)
[TimeProfiler](https://github.com/maniackk/TimeProfiler/tree/master)
[objc-file.mm](https://github.com/tripleCC/Laboratory/blob/5c084263d79973805649b89d166b50751045e937/AppleSources/objc4-750/runtime/objc-file.mm#L32-L47)


