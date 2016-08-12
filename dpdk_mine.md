##### <Author> : Terry.Wei

##### <Email> : wjm.terry@gmail.com

##### <status>: To be continue ...



#### dpdk eal option选项

- 默认: -c COREMASK -n NUM, 其中, COREMASK 为16进制掩码表示逻辑内核数量, NUM为10进制整数表示内存通道数量. 

  ```shell
  ./rte-app -c COREMASK [-n NUM] [-b <domain:bus:devid.func>] \
            [--socket-mem=MB,...] [-m MB] [-r NUM] [-v] [--file-prefix] \
            [--proc-type <primary|secondary|auto>] [-- xen-dom0]
  ```

  - **-c COREMASK**: 16进制逻辑内核数量;
  - **-n NUM**: 10进制内存通道数量;
  - **-b <domain:bus:devid.func>**: port网口黑名单, 不使用指定PCI设备. 支持配置多个;
  - **--use-device**: 使用指定PCI设备, **[domain:]bus:devid.func**格式, 用逗号隔开. 不能用**-b**选项;
  - **--socket-num**: 在特定的socket上分配内存, 分配自hugepages;
  - **-m MB**: 从hugepages分配的内存大小, 不考虑socket, 整体的大小. 一般建议使用 **--socket-num**选项代替;
  - **-r NUM**: 内存队列数量;
  - **-v**: 在启动时展示version信息;
  - **--huge-dir**: 指定hugetlbfs挂载的目录;
  - **--file-prefix**: hugepages文件前缀;
  - **--proc-type**: 进程类型, 如master, primary, secondary;
  - **--xen-dom0**: 没有hugetlbfs之下, 支持应用在Xen Domain0运行;
  - **--vmware-tsc-map**:
  - **--base-virtaddr**:
  - **--vfio-intr**:



#### NUMA

- 查看系统numa节点信息:  

  ```shell
  numactl --hardware
  ```

- **同一个numa的逻辑cpu共享L3缓存**. 这个可以通过 /sys/devices/system/cpu/cpu0/cache/index3/shared_cpu_map 查看到. 而同一个numa的逻辑cpu可由上条命令查出.

- 在NUMA架构中, 使用numactl命令将应用程序绑定在一个socket上的core上运行, 可以提高内存的访问效率. 

  ```shell
  numactl -m 0 --physcpubind=2,3 ./test
  -m 0: 在node0 上分配内存;
  --physcpubind=2,3: 在cpu2和3上运行程序, 每个核上一个线程.
  ```

- 如果是dpdk的应用程序app, 通过numactl 启动时, 只需要-m指定node, 也即指定了socket id; 然后, ./app需要通过-c 指定该socket id绑定的逻辑内核. 

  ```shell
  //this is command result;
  numactl --hardware
  available: 2 nodes (0-1)
  node 0 cpus: 0 2 4 6 8 10 12 14 16 18 20 22
  node 0 size: 32768 MB
  node 0 free: 27475 MB
  node 1 cpus: 1 3 5 7 9 11 13 15 17 19 21 23
  node 1 size: 32755 MB
  node 1 free: 27201 MB
  node distances:
  node   0   1
    0:  10  20
    1:  20  10
  ```

  - 从中可以看到, node0绑定的逻辑内核是0,2,4,…,22, node1绑定的逻辑内核是1,3,5,…,23. 

  ```shell
  numactl -m 0 ./app -c 0x555555;
  numactl -m 1 ./app -c 0xaaaaaa;
  ```

  - 可以通过这两条命令, 来启动app应用程序. 



