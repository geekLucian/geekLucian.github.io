---
title: 性能分析
date: 2021-09-30 14:59:53
comment: true
---

```bash
time cmd
```

- *real* - 从程序开始到结束流失掉的真实时间，包括其他进程的执行时间以及阻塞消耗的时间（例如等待 I/O或网络）；
- *User* - CPU 执行用户代码所花费的时间；
- *Sys* - CPU 执行系统内核代码所花费的时间。

```bash
python -m cProfile xxx.py
```

### 逐行时间分析

```bash
sudo apt install python3-line-profiler
```

```python
#!/usr/bin/env python3
#coding=utf8
import requests
from bs4 import BeautifulSoup

# 这个装饰器会告诉行分析器 
# 我们想要分析这个函数
@profile
def get_urls():
    response = requests.get('https://missing.csail.mit.edu')
    s = BeautifulSoup(response.content, 'lxml')
    urls = []
    for url in s.find_all('a'):
        urls.append(url['href'])

if __name__ == '__main__':
    get_urls()
```

```bash
kernprof -l -v foo.py
```

### 内存分析

```bash
sudo apt install python3-memory-profiler
```

```python
@profile
def my_func():
    a = [1] * (10 ** 6)
    b = [2] * (2 * 10 ** 7)
    del b
    return a

if __name__ == '__main__':
    my_func()
```

```bash
$ python3 -m memory_profiler bar.py       
Filename: bar.py

Line #    Mem usage    Increment   Line Contents
================================================
     1   38.105 MiB   38.105 MiB   @profile
     2                             def my_func():
     3   45.770 MiB    7.664 MiB       a = [1] * (10 ** 6)
     4  198.395 MiB  152.625 MiB       b = [2] * (2 * 10 ** 7)
     5   45.848 MiB -152.547 MiB       del b
     6   45.848 MiB    0.000 MiB       return a

```

### 事件分析

`perf` 可以报告不佳的缓存局部性（poor cache locality）、大量的页错误（page faults）或活锁（livelocks）

```bash
sudo apt-get install linux-tools-5.11.0-34-generic
perf list
```

### 可视化

 [火焰图](http://www.brendangregg.com/flamegraphs.html)

调用图[`pycallgraph`](http://pycallgraph.slowchop.com/en/master/) 

htop是top的改进版

iotop查看IO信息

df查看磁盘使用情况 du当前目录 -h友好显示

free查看空闲内存

lsof查看被进程打开的文件

ss监控网络包的收发情况



## About Perf

```bash
 usage: perf [--version] [--help] [OPTIONS] COMMAND [ARGS]

 The most commonly used perf commands are:
   annotate        Read perf.data (created by perf record) and display annotated code
   archive         Create archive with object files with build-ids found in perf.data file
   bench           General framework for benchmark suites
   buildid-cache   Manage build-id cache.
   buildid-list    List the buildids in a perf.data file
   c2c             Shared Data C2C/HITM Analyzer.
   config          Get and set variables in a configuration file.
   data            Data file related processing
   diff            Read perf.data files and display the differential profile
   evlist          List the event names in a perf.data file
   ftrace          simple wrapper for kernel's ftrace functionality
   inject          Filter to augment the events stream with additional information
   kallsyms        Searches running kernel for symbols
   kmem            Tool to trace/measure kernel memory properties
   kvm             Tool to trace/measure kvm guest os
   list            List all symbolic event types
   lock            Analyze lock events
   mem             Profile memory accesses
   record          Run a command and record its profile into perf.data
   report          Read perf.data (created by perf record) and display the profile
   sched           Tool to trace/measure scheduler properties (latencies)
   script          Read perf.data (created by perf record) and display trace output
   stat            Run a command and gather performance counter statistics
   test            Runs sanity tests.
   timechart       Tool to visualize total system behavior during a workload
   top             System profiling tool.
   probe           Define new dynamic tracepoints
   trace           strace inspired tool
```

- 事件

	- 软事件：内核计数，如context-switches、minor-faults
	- PMU硬件事件：来自处理器本身及其性能监视单元PMU，如周期数、失效指令、L1Cache miss，由CPU供应商提供文档
	- tracepoint事件：由内核ftrace基础实现

- perf stat收集常见事件

	```bash
	$ perf stat -B dd if=/dev/zero of=/dev/null count=1000000
	1000000+0 records in
	1000000+0 records out
	512000000 bytes (512 MB, 488 MiB) copied, 0.460126 s, 1.1 GB/s
	
	 Performance counter stats for 'dd if=/dev/zero of=/dev/null count=1000000':
	
	        459.490970      task-clock (msec)         #    0.992 CPUs utilized         
	                11      context-switches          #    0.024 K/sec                 
	                 0      cpu-migrations            #    0.000 K/sec                 
	                62      page-faults               #    0.135 K/sec                 
	   <not supported>      cycles                                                     
	   <not supported>      instructions                                               
	   <not supported>      branches                                                   
	   <not supported>      branch-misses                                             
	
	       0.463188434 seconds time elapsed
	```

	-e task-clock指定事件，-e task-clock:u指定用户级别 -e task-clock:uk指定用户和内核

- perf annotate指令级分析，编译程序时加 -ggdb可以逐行看

- perf top现场分析

- perf bench基准测试

  - sched调度器基准测试测量：多个任务之间的pipe(2)和socketpair(2)操作。允许测量线程与进程上下文切换的性能。
  - mem:内存访问基准测试
  - numa: numa调度和MM基准测试
  - futex: futex压力基准测试：处理futex内核实现的细粒度方面。它对于内核黑客非常有用。它目前支持唤醒和重新排队/等待操作，并强调私有和共享futexes的哈希方案。








































