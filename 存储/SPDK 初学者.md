# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [Getting Started](https://spdk.io/doc/getting_started.html)

# SPDK编译运行
```shell
j2@j2-pc:~/spdk-18.10.1$ sudo scripts/pkgdep.sh
...error...
j2@j2-pc:~/spdk-18.10.1$ pip install --upgrade pip
j2@j2-pc:~/spdk-18.10.1$ sudo scripts/pkgdep.sh
Traceback (most recent call last):
  File "/usr/bin/pip", line 9, in <module>
    from pip import main
ImportError: cannot import name main
Error!
```
> [pip问题：Traceback (most recent call last): File "/usr/bin/pip", line 9, in](https://blog.csdn.net/vmxhc1314/article/details/81869676)

将 /usr/bin/pip 文件中：
```python
from pip import  main
if __name__ == '__main__':
    sys.exit(main())
```
改为：
```python
from pip import __main__
if __name__ == '__main__':
    sys.exit(__main__._main())
```
重新pkgdep，有警告
```shell
The directory '/home/j2/.cache/pip/http' or its parent directory is not owned by the current user and the cache has been disabled. Please check the permissions and owner of that directory. If executing pip with sudo, you may want sudo's -H flag.
The directory '/home/j2/.cache/pip' or its parent directory is not owned by the current user and caching wheels has been disabled. check the permissions and owner of that directory. If executing pip with sudo, you may want sudo's -H flag.
...
You are using pip version 8.1.1, however version 18.1 is available.
You should consider upgrading via the 'pip install --upgrade pip' command.
Crypto requires NASM version 2.12.02 or newer. Please install
or upgrade and re-run this script if you are going to use Crypto.
```
编译，测试，
```shell
j2@j2-pc:~/spdk-18.10.1$ ./configure 
j2@j2-pc:~/spdk-18.10.1$ make
j2@j2-pc:~/spdk-18.10.1$ ./test/unit/unittest.sh
j2@j2-pc:~/spdk-18.10.1$ sudo scripts/setup.sh
j2@j2-pc:~/spdk-18.10.1$ sudo examples/nvme/identify/identify
```
setup和identify这里必须用sudo。

# 测试
> [SPDK实战、QoS延时验证：Intel Optane P4800X评测(5)](https://www.sohu.com/a/154479122_314773)
> [SPDK生态工具（二）：性能评估工具](https://www.sdnlab.com/21080.html)
> [spdk（bdev层） + fio测试nvme 设备的性能](https://www.jianshu.com/p/3a38a4d10d94)

## fio
```shell
# --build-static
j2@j2-pc:~/fio-fio-3.12$ ./configure --prefix=/home/j2/fio-fio-3.12/ubuntu16.04.4
j2@j2-pc:~/fio-fio-3.12$ make
j2@j2-pc:~/fio-fio-3.12$ make install
j2@j2-pc:~/spdk-18.10.1$ ./configure --with-fio=/home/j2/fio-fio-3.12
j2@j2-pc:~/spdk-18.10.1$ make
j2@j2-pc:~/spdk-18.10.1$ sudo LD_PRELOAD=examples/nvme/fio_plugin/fio_plugin fio ../spdk.fio
```
job file，

```

```

## perf
```shell
j2@j2-pc:~/spdk-18.10.1$ sudo ./examples/nvme/perf/perf -o 4096 -q 32 -s 4096 -w randwrite -t 1200 -c 0xF -r 'trtype:PCIe traddr:0000:0b:00.0'
Starting SPDK v18.10.1 / DPDK 18.08.0 initialization...
[ DPDK EAL parameters: perf --no-shconf -c 0xF -m 4096 --file-prefix=spdk_pid28398 ]
EAL: Detected 24 lcore(s)
EAL: Detected 2 NUMA nodes
EAL: No free hugepages reported in hugepages-1048576kB
EAL: Probing VFIO support...
EAL: Not enough memory available! Requested: 4096MB, available: 2048MB
EAL: FATAL: Cannot init memory

EAL: Cannot init memory

Failed to initialize DPDK
Unable to initialize SPDK env
./examples/nvme/perf/perf: errors occured
j2@j2-pc:~/spdk-18.10.1$ 
j2@j2-pc:~/spdk-18.10.1$ 
j2@j2-pc:~/spdk-18.10.1$ 
j2@j2-pc:~/spdk-18.10.1$ sudo scripts/setup.sh reset
0000:05:00.0 (8086 0953): uio_pci_generic -> nvme
0000:0b:00.0 (8086 0953): uio_pci_generic -> nvme
0000:81:00.0 (8086 0953): uio_pci_generic -> nvme
0000:84:00.0 (8086 0953): uio_pci_generic -> nvme
0000:88:00.0 (8086 0953): uio_pci_generic -> nvme
0000:00:04.0 (8086 6f20): uio_pci_generic -> ioatdma
0000:80:04.0 (8086 6f20): uio_pci_generic -> ioatdma
0000:00:04.1 (8086 6f21): uio_pci_generic -> ioatdma
0000:80:04.1 (8086 6f21): uio_pci_generic -> ioatdma
0000:00:04.2 (8086 6f22): uio_pci_generic -> ioatdma
0000:80:04.2 (8086 6f22): uio_pci_generic -> ioatdma
0000:00:04.3 (8086 6f23): uio_pci_generic -> ioatdma
0000:80:04.3 (8086 6f23): uio_pci_generic -> ioatdma
0000:00:04.4 (8086 6f24): uio_pci_generic -> ioatdma
0000:80:04.4 (8086 6f24): uio_pci_generic -> ioatdma
0000:00:04.5 (8086 6f25): uio_pci_generic -> ioatdma
0000:80:04.5 (8086 6f25): uio_pci_generic -> ioatdma
0000:00:04.6 (8086 6f26): uio_pci_generic -> ioatdma
0000:80:04.6 (8086 6f26): uio_pci_generic -> ioatdma
0000:00:04.7 (8086 6f27): uio_pci_generic -> ioatdma
0000:80:04.7 (8086 6f27): uio_pci_generic -> ioatdma
j2@j2-pc:~/spdk-18.10.1$ sudo HUGEMEM=4096 scripts/setup.sh
0000:05:00.0 (8086 0953): nvme -> uio_pci_generic
0000:0b:00.0 (8086 0953): nvme -> uio_pci_generic
0000:81:00.0 (8086 0953): nvme -> uio_pci_generic
0000:84:00.0 (8086 0953): nvme -> uio_pci_generic
0000:88:00.0 (8086 0953): nvme -> uio_pci_generic
0000:00:04.0 (8086 6f20): ioatdma -> uio_pci_generic
0000:80:04.0 (8086 6f20): ioatdma -> uio_pci_generic
0000:00:04.1 (8086 6f21): ioatdma -> uio_pci_generic
0000:80:04.1 (8086 6f21): ioatdma -> uio_pci_generic
0000:00:04.2 (8086 6f22): ioatdma -> uio_pci_generic
0000:80:04.2 (8086 6f22): ioatdma -> uio_pci_generic
0000:00:04.3 (8086 6f23): ioatdma -> uio_pci_generic
0000:80:04.3 (8086 6f23): ioatdma -> uio_pci_generic
0000:00:04.4 (8086 6f24): ioatdma -> uio_pci_generic
0000:80:04.4 (8086 6f24): ioatdma -> uio_pci_generic
0000:00:04.5 (8086 6f25): ioatdma -> uio_pci_generic
0000:80:04.5 (8086 6f25): ioatdma -> uio_pci_generic
0000:00:04.6 (8086 6f26): ioatdma -> uio_pci_generic
0000:80:04.6 (8086 6f26): ioatdma -> uio_pci_generic
0000:00:04.7 (8086 6f27): ioatdma -> uio_pci_generic
0000:80:04.7 (8086 6f27): ioatdma -> uio_pci_generic
j2@j2-pc:~/spdk-18.10.1$ ./examples/nvme/perf/perf -h\
> 
./examples/nvme/perf/perf: invalid option -- 'h'
./examples/nvme/perf/perf options [AIO device(s)]...
        [-q io depth]
        [-o io size in bytes]
        [-w io pattern type, must be one of
                (read, write, randread, randwrite, rw, randrw)]
        [-M rwmixread (100 for reads, 0 for writes)]
        [-L enable latency tracking via sw, default: disabled]
                -L for latency summary, -LL for detailed histogram
        [-l enable latency tracking via ssd (if supported), default: disabled]
        [-t time in seconds]
        [-c core mask for I/O submission/completion.]
                (default: 1)]
        [-D disable submission queue in controller memory buffer, default: enabled]
        [-r Transport ID for local PCIe NVMe or NVMeoF]
         Format: 'key:value [key:value] ...'
         Keys:
          trtype      Transport type (e.g. PCIe, RDMA)
          adrfam      Address family (e.g. IPv4, IPv6)
          traddr      Transport address (e.g. 0000:04:00.0 for PCIe or 192.168.100.8 for RDMA)
          trsvcid     Transport service identifier (e.g. 4420)
          subnqn      Subsystem NQN (default: nqn.2014-08.org.nvmexpress.discovery)
         Example: -r 'trtype:PCIe traddr:0000:04:00.0' for PCIe or
                  -r 'trtype:RDMA adrfam:IPv4 traddr:192.168.100.8 trsvcid:4420' for NVMeoF
        [-e metadata configuration]
         Keys:
          PRACT      Protection Information Action bit (PRACT=1 or PRACT=0)
          PRCHK      Control of Protection Information Checking (PRCHK=GUARD|REFTAG|APPTAG)
         Example: -e 'PRACT=0,PRCHK=GUARD|REFTAG|APPTAG'
                  -e 'PRACT=1,PRCHK=GUARD'
        [-s DPDK huge memory size in MB.]
        [-m max completions per poll]
                (default: 0 - unlimited)
        [-i shared memory group ID]
j2@j2-pc:~/spdk-18.10.1$ sudo ./examples/nvme/perf/perf -o 4096 -q 32 -s 4096 -w randwrite -t 16 -c 0xF -r 'trtype:PCIe traddr:0000:05:00.0'
Starting SPDK v18.10.1 / DPDK 18.08.0 initialization...
[ DPDK EAL parameters: perf --no-shconf -c 0xF -m 4096 --file-prefix=spdk_pid29850 ]
EAL: Detected 24 lcore(s)
EAL: Detected 2 NUMA nodes
EAL: No free hugepages reported in hugepages-1048576kB
EAL: Probing VFIO support...
Initializing NVMe Controllers
EAL: PCI device 0000:05:00.0 on NUMA socket 0
EAL:   probe driver: 8086:953 spdk_nvme
Attaching to NVMe Controller at 0000:05:00.0
Attached to NVMe Controller at 0000:05:00.0 [8086:0953]
Associating INTEL SSDPEDMW012T4  (CVCQ6523001M1P2JGN  ) with lcore 3
Associating INTEL SSDPEDMW012T4  (CVCQ6523001M1P2JGN  ) with lcore 2
Associating INTEL SSDPEDMW012T4  (CVCQ6523001M1P2JGN  ) with lcore 1
Associating INTEL SSDPEDMW012T4  (CVCQ6523001M1P2JGN  ) with lcore 0
Initialization complete. Launching workers.
Starting thread on core 3
Starting thread on core 2
Starting thread on core 1
Starting thread on core 0
========================================================
                                                                                            Latency(us)
Device Information                                     :       IOPS       MB/s    Average        min        max
INTEL SSDPEDMW012T4  (CVCQ6523001M1P2JGN  ) from core 3:   68497.19     267.57     467.14       4.87    9674.24
INTEL SSDPEDMW012T4  (CVCQ6523001M1P2JGN  ) from core 2:   68362.69     267.04     468.10       5.03    9715.33
INTEL SSDPEDMW012T4  (CVCQ6523001M1P2JGN  ) from core 1:   68318.69     266.87     468.40       4.87    9824.85
INTEL SSDPEDMW012T4  (CVCQ6523001M1P2JGN  ) from core 0:   68364.38     267.05     468.07       4.96    9825.13
========================================================
Total                                                  :  273542.94    1068.53     467.93       4.87    9825.13

j2@j2-pc:~/spdk-18.10.1$ 
j2@j2-pc:~/spdk-18.10.1$ 
j2@j2-pc:~/spdk-18.10.1$ 
j2@j2-pc:~/spdk-18.10.1$ sudo ./examples/nvme/perf/perf -o 4096 -q 32 -s 4096 -w randwrite -t 16 -c 0x1 -r 'trtype:PCIe traddr:0000:05:00.0'
Starting SPDK v18.10.1 / DPDK 18.08.0 initialization...
[ DPDK EAL parameters: perf --no-shconf -c 0x1 -m 4096 --file-prefix=spdk_pid29858 ]
EAL: Detected 24 lcore(s)
EAL: Detected 2 NUMA nodes
EAL: No free hugepages reported in hugepages-1048576kB
EAL: Probing VFIO support...
Initializing NVMe Controllers
EAL: PCI device 0000:05:00.0 on NUMA socket 0
EAL:   probe driver: 8086:953 spdk_nvme
Attaching to NVMe Controller at 0000:05:00.0
Attached to NVMe Controller at 0000:05:00.0 [8086:0953]
Associating INTEL SSDPEDMW012T4  (CVCQ6523001M1P2JGN  ) with lcore 0
Initialization complete. Launching workers.
Starting thread on core 0
========================================================
                                                                                            Latency(us)
Device Information                                     :       IOPS       MB/s    Average        min        max
INTEL SSDPEDMW012T4  (CVCQ6523001M1P2JGN  ) from core 0:  286650.12    1119.73     111.61       4.82    6628.92
========================================================
Total                                                  :  286650.12    1119.73     111.61       4.82    6628.92

j2@j2-pc:~/spdk-18.10.1$ sudo ./examples/nvme/perf/perf -o 4096 -q 32 -s 4096 -w write -t 4 -c 0x1 -r 'trtype:PCIe traddr:0000:05:00.0'
Starting SPDK v18.10.1 / DPDK 18.08.0 initialization...
[ DPDK EAL parameters: perf --no-shconf -c 0x1 -m 4096 --file-prefix=spdk_pid29863 ]
EAL: Detected 24 lcore(s)
EAL: Detected 2 NUMA nodes
EAL: No free hugepages reported in hugepages-1048576kB
EAL: Probing VFIO support...
Initializing NVMe Controllers
EAL: PCI device 0000:05:00.0 on NUMA socket 0
EAL:   probe driver: 8086:953 spdk_nvme
Attaching to NVMe Controller at 0000:05:00.0
Attached to NVMe Controller at 0000:05:00.0 [8086:0953]
Associating INTEL SSDPEDMW012T4  (CVCQ6523001M1P2JGN  ) with lcore 0
Initialization complete. Launching workers.
Starting thread on core 0
========================================================
                                                                                            Latency(us)
Device Information                                     :       IOPS       MB/s    Average        min        max
INTEL SSDPEDMW012T4  (CVCQ6523001M1P2JGN  ) from core 0:  292685.00    1143.30     109.33       4.91    5617.24
========================================================
Total                                                  :  292685.00    1143.30     109.33       4.91    5617.24

j2@j2-pc:~/spdk-18.10.1$ sudo ./examples/nvme/perf/perf -o 4096 -q 32 -s 4096 -w write -t 4 -c 0x2000 -r 'trtype:PCIe traddr:0000:05:00.0'
Starting SPDK v18.10.1 / DPDK 18.08.0 initialization...
[ DPDK EAL parameters: perf --no-shconf -c 0x2000 -m 4096 --file-prefix=spdk_pid29868 ]
EAL: Detected 24 lcore(s)
EAL: Detected 2 NUMA nodes
EAL: No free hugepages reported in hugepages-1048576kB
EAL: Probing VFIO support...
Initializing NVMe Controllers
EAL: PCI device 0000:05:00.0 on NUMA socket 0
EAL:   probe driver: 8086:953 spdk_nvme
Attaching to NVMe Controller at 0000:05:00.0
Attached to NVMe Controller at 0000:05:00.0 [8086:0953]
Associating INTEL SSDPEDMW012T4  (CVCQ6523001M1P2JGN  ) with lcore 13
Initialization complete. Launching workers.
Starting thread on core 13
========================================================
                                                                                            Latency(us)
Device Information                                     :       IOPS       MB/s    Average        min        max
INTEL SSDPEDMW012T4  (CVCQ6523001M1P2JGN  ) from core 13:  294559.75    1150.62     108.63       4.98    6066.26
========================================================
Total                                                  :  294559.75    1150.62     108.63       4.98    6066.26

j2@j2-pc:~/spdk-18.10.1$ sudo ./examples/nvme/perf/perf -o 4m -q 32 -s 4096 -w write -t 4 -c 0x2000 -r 'trtype:PCIe traddr:0000:05:00.0'
Starting SPDK v18.10.1 / DPDK 18.08.0 initialization...
[ DPDK EAL parameters: perf --no-shconf -c 0x2000 -m 4096 --file-prefix=spdk_pid29874 ]
EAL: Detected 24 lcore(s)
EAL: Detected 2 NUMA nodes
EAL: No free hugepages reported in hugepages-1048576kB
EAL: Probing VFIO support...
Initializing NVMe Controllers
EAL: PCI device 0000:05:00.0 on NUMA socket 0
EAL:   probe driver: 8086:953 spdk_nvme
Attaching to NVMe Controller at 0000:05:00.0
Attached to NVMe Controller at 0000:05:00.0 [8086:0953]
WARNING: controller INTEL SSDPEDMW012T4  (CVCQ6523001M1P2JGN  ) ns 1 has invalid ns size 1200243695616 / block size 512 for I/O size 4
WARNING: Some requested NVMe devices were skipped
No valid NVMe controllers or AIO devices found
j2@j2-pc:~/spdk-18.10.1$ sudo ./examples/nvme/perf/perf -o 4194304 -q 32 -s 4096 -w write -t 4 -c 0x2000 -r 'trtype:PCIe traddr:0000:05:00.0'
Starting SPDK v18.10.1 / DPDK 18.08.0 initialization...
[ DPDK EAL parameters: perf --no-shconf -c 0x2000 -m 4096 --file-prefix=spdk_pid29878 ]
EAL: Detected 24 lcore(s)
EAL: Detected 2 NUMA nodes
EAL: No free hugepages reported in hugepages-1048576kB
EAL: Probing VFIO support...
Initializing NVMe Controllers
EAL: PCI device 0000:05:00.0 on NUMA socket 0
EAL:   probe driver: 8086:953 spdk_nvme
Attaching to NVMe Controller at 0000:05:00.0
Attached to NVMe Controller at 0000:05:00.0 [8086:0953]
Associating INTEL SSDPEDMW012T4  (CVCQ6523001M1P2JGN  ) with lcore 13
Initialization complete. Launching workers.
Starting thread on core 13
========================================================
                                                                                            Latency(us)
Device Information                                     :       IOPS       MB/s    Average        min        max
INTEL SSDPEDMW012T4  (CVCQ6523001M1P2JGN  ) from core 13:     308.25    1233.00  104935.04   22555.34  117980.98
========================================================
Total                                                  :     308.25    1233.00  104935.04   22555.34  117980.98

j2@j2-pc:~/spdk-18.10.1$ sudo ./examples/nvme/perf/perf -o 4194304 -q 32 -s 4096 -w write -t 4 -c 0x1 -r 'trtype:PCIe traddr:0000:05:00.0'
Starting SPDK v18.10.1 / DPDK 18.08.0 initialization...
[ DPDK EAL parameters: perf --no-shconf -c 0x1 -m 4096 --file-prefix=spdk_pid29881 ]
EAL: Detected 24 lcore(s)
EAL: Detected 2 NUMA nodes
EAL: No free hugepages reported in hugepages-1048576kB
EAL: Probing VFIO support...
Initializing NVMe Controllers
EAL: PCI device 0000:05:00.0 on NUMA socket 0
EAL:   probe driver: 8086:953 spdk_nvme
Attaching to NVMe Controller at 0000:05:00.0
Attached to NVMe Controller at 0000:05:00.0 [8086:0953]
Associating INTEL SSDPEDMW012T4  (CVCQ6523001M1P2JGN  ) with lcore 0
Initialization complete. Launching workers.
Starting thread on core 0
========================================================
                                                                                            Latency(us)
Device Information                                     :       IOPS       MB/s    Average        min        max
INTEL SSDPEDMW012T4  (CVCQ6523001M1P2JGN  ) from core 0:     307.75    1231.00  104911.70   22699.09  116968.86
========================================================
Total                                                  :     307.75    1231.00  104911.70   22699.09  116968.86

j2@j2-pc:~/spdk-18.10.1$ sudo ./examples/nvme/perf/perf -o 4194304 -q 32 -s 4096 -w write -t 4 -c 0xf -r 'trtype:PCIe traddr:0000:05:00.0'
Starting SPDK v18.10.1 / DPDK 18.08.0 initialization...
[ DPDK EAL parameters: perf --no-shconf -c 0xf -m 4096 --file-prefix=spdk_pid29886 ]
EAL: Detected 24 lcore(s)
EAL: Detected 2 NUMA nodes
EAL: No free hugepages reported in hugepages-1048576kB
EAL: Probing VFIO support...
Initializing NVMe Controllers
EAL: PCI device 0000:05:00.0 on NUMA socket 0
EAL:   probe driver: 8086:953 spdk_nvme
Attaching to NVMe Controller at 0000:05:00.0
Attached to NVMe Controller at 0000:05:00.0 [8086:0953]
Associating INTEL SSDPEDMW012T4  (CVCQ6523001M1P2JGN  ) with lcore 3
Associating INTEL SSDPEDMW012T4  (CVCQ6523001M1P2JGN  ) with lcore 2
Associating INTEL SSDPEDMW012T4  (CVCQ6523001M1P2JGN  ) with lcore 1
Associating INTEL SSDPEDMW012T4  (CVCQ6523001M1P2JGN  ) with lcore 0
Initialization complete. Launching workers.
Starting thread on core 3
Starting thread on core 2
Starting thread on core 1
Starting thread on core 0
========================================================
                                                                                            Latency(us)
Device Information                                     :       IOPS       MB/s    Average        min        max
INTEL SSDPEDMW012T4  (CVCQ6523001M1P2JGN  ) from core 3:      84.00     336.00  399191.18   28751.61  444618.98
INTEL SSDPEDMW012T4  (CVCQ6523001M1P2JGN  ) from core 2:      83.25     333.00  402924.51   32200.29  443727.62
INTEL SSDPEDMW012T4  (CVCQ6523001M1P2JGN  ) from core 1:      82.50     330.00  406014.98   37059.40  443168.02
INTEL SSDPEDMW012T4  (CVCQ6523001M1P2JGN  ) from core 0:      82.50     330.00  406212.16   35146.04  444680.55
========================================================
Total                                                  :     332.25    1329.00  403564.37   28751.61  444680.55

j2@j2-pc:~/spdk-18.10.1$ sudo ./examples/nvme/perf/perf -o 16777216 -q 32 -s 4096 -w write -t 4 -c 0xf -r 'trtype:PCIe traddr:0000:05:00.0'
Starting SPDK v18.10.1 / DPDK 18.08.0 initialization...
[ DPDK EAL parameters: perf --no-shconf -c 0xf -m 4096 --file-prefix=spdk_pid29892 ]
EAL: Detected 24 lcore(s)
EAL: Detected 2 NUMA nodes
EAL: No free hugepages reported in hugepages-1048576kB
EAL: Probing VFIO support...
Initializing NVMe Controllers
EAL: PCI device 0000:05:00.0 on NUMA socket 0
EAL:   probe driver: 8086:953 spdk_nvme
Attaching to NVMe Controller at 0000:05:00.0
Attached to NVMe Controller at 0000:05:00.0 [8086:0953]
controller IO queue size 4096 less than required
Consider using lower queue depth or small IO size because IO requests may be queued at the NVMe driver.
WARNING: Some requested NVMe devices were skipped
Associating INTEL SSDPEDMW012T4  (CVCQ6523001M1P2JGN  ) with lcore 3
Associating INTEL SSDPEDMW012T4  (CVCQ6523001M1P2JGN  ) with lcore 2
Associating INTEL SSDPEDMW012T4  (CVCQ6523001M1P2JGN  ) with lcore 1
Associating INTEL SSDPEDMW012T4  (CVCQ6523001M1P2JGN  ) with lcore 0
Initialization complete. Launching workers.
Starting thread on core 3
Starting thread on core 2
Starting thread on core 1
Starting thread on core 0
starting I/O failed
starting I/O failed
starting I/O failed
starting I/O failed
========================================================
                                                                                            Latency(us)
Device Information                                     :       IOPS       MB/s    Average        min        max
INTEL SSDPEDMW012T4  (CVCQ6523001M1P2JGN  ) from core 3:      26.50     424.00 1378237.92  189295.82 1681419.63
INTEL SSDPEDMW012T4  (CVCQ6523001M1P2JGN  ) from core 2:      26.50     424.00 1384060.60  199343.94 1682199.80
INTEL SSDPEDMW012T4  (CVCQ6523001M1P2JGN  ) from core 1:      26.25     420.00 1393641.04  201916.80 1682298.07
INTEL SSDPEDMW012T4  (CVCQ6523001M1P2JGN  ) from core 0:      26.25     420.00 1395348.45  206082.71 1682435.40
========================================================
Total                                                  :     105.50    1688.00 1387790.38  189295.82 1682435.40

j2@j2-pc:~/spdk-18.10.1$ sudo ./examples/nvme/perf/perf -o 16777216 -q 32 -s 4096 -w write -t 4 -c 0xf -r 'trtype:PCIe traddr:0000:05:00.0'
Starting SPDK v18.10.1 / DPDK 18.08.0 initialization...
[ DPDK EAL parameters: perf --no-shconf -c 0xf -m 4096 --file-prefix=spdk_pid29903 ]
EAL: Detected 24 lcore(s)
EAL: Detected 2 NUMA nodes
EAL: No free hugepages reported in hugepages-1048576kB
EAL: Probing VFIO support...
Initializing NVMe Controllers
EAL: PCI device 0000:05:00.0 on NUMA socket 0
EAL:   probe driver: 8086:953 spdk_nvme
Attaching to NVMe Controller at 0000:05:00.0
Attached to NVMe Controller at 0000:05:00.0 [8086:0953]
controller IO queue size 4096 less than required
Consider using lower queue depth or small IO size because IO requests may be queued at the NVMe driver.
WARNING: Some requested NVMe devices were skipped
Associating INTEL SSDPEDMW012T4  (CVCQ6523001M1P2JGN  ) with lcore 3
Associating INTEL SSDPEDMW012T4  (CVCQ6523001M1P2JGN  ) with lcore 2
Associating INTEL SSDPEDMW012T4  (CVCQ6523001M1P2JGN  ) with lcore 1
Associating INTEL SSDPEDMW012T4  (CVCQ6523001M1P2JGN  ) with lcore 0
Initialization complete. Launching workers.
Starting thread on core 3
Starting thread on core 2
Starting thread on core 1
Starting thread on core 0
starting I/O failed
starting I/O failed
starting I/O failed
starting I/O failed
========================================================
                                                                                            Latency(us)
Device Information                                     :       IOPS       MB/s    Average        min        max
INTEL SSDPEDMW012T4  (CVCQ6523001M1P2JGN  ) from core 3:      26.50     424.00 1382522.50  190645.81 1683826.64
INTEL SSDPEDMW012T4  (CVCQ6523001M1P2JGN  ) from core 2:      26.25     420.00 1386511.40  198855.35 1685283.25
INTEL SSDPEDMW012T4  (CVCQ6523001M1P2JGN  ) from core 1:      26.25     420.00 1399044.62  208331.21 1685504.39
INTEL SSDPEDMW012T4  (CVCQ6523001M1P2JGN  ) from core 0:      26.25     420.00 1400331.67  206634.96 1685705.08
========================================================
Total                                                  :     105.25    1684.00 1392079.79  190645.81 1685705.08

j2@j2-pc:~/spdk-18.10.1$ sudo ./examples/nvme/perf/perf -o 16777216 -q 32 -s 4096 -w read -t 4 -c 0xf -r 'trtype:PCIe traddr:0000:05:00.0'
Starting SPDK v18.10.1 / DPDK 18.08.0 initialization...
[ DPDK EAL parameters: perf --no-shconf -c 0xf -m 4096 --file-prefix=spdk_pid29909 ]
EAL: Detected 24 lcore(s)
EAL: Detected 2 NUMA nodes
EAL: No free hugepages reported in hugepages-1048576kB
EAL: Probing VFIO support...
Initializing NVMe Controllers
EAL: PCI device 0000:05:00.0 on NUMA socket 0
EAL:   probe driver: 8086:953 spdk_nvme
Attaching to NVMe Controller at 0000:05:00.0
Attached to NVMe Controller at 0000:05:00.0 [8086:0953]
controller IO queue size 4096 less than required
Consider using lower queue depth or small IO size because IO requests may be queued at the NVMe driver.
WARNING: Some requested NVMe devices were skipped
Associating INTEL SSDPEDMW012T4  (CVCQ6523001M1P2JGN  ) with lcore 3
Associating INTEL SSDPEDMW012T4  (CVCQ6523001M1P2JGN  ) with lcore 2
Associating INTEL SSDPEDMW012T4  (CVCQ6523001M1P2JGN  ) with lcore 1
Associating INTEL SSDPEDMW012T4  (CVCQ6523001M1P2JGN  ) with lcore 0
Initialization complete. Launching workers.
Starting thread on core 3
Starting thread on core 2
Starting thread on core 1
Starting thread on core 0
starting I/O failed
starting I/O failed
starting I/O failed
starting I/O failed
========================================================
                                                                                            Latency(us)
Device Information                                     :       IOPS       MB/s    Average        min        max
INTEL SSDPEDMW012T4  (CVCQ6523001M1P2JGN  ) from core 3:      46.50     744.00  717634.73  164784.63  830421.20
INTEL SSDPEDMW012T4  (CVCQ6523001M1P2JGN  ) from core 2:      46.25     740.00  719570.26  168117.04  829957.78
INTEL SSDPEDMW012T4  (CVCQ6523001M1P2JGN  ) from core 1:      46.00     736.00  721743.18  177461.95  830567.32
INTEL SSDPEDMW012T4  (CVCQ6523001M1P2JGN  ) from core 0:      46.00     736.00  722301.30  176895.23  829961.81
========================================================
Total                                                  :     184.75    2956.00  720304.12  164784.63  830567.32
j2@j2-pc:~/spdk-18.10.1$ sudo ./examples/nvme/perf/perf -o 4096 -q 1 -s 2048 -w randwrite -t 4 -c 0x1 -r 'trtype:PCIe traddr:0000:0b:00.0'
Starting SPDK v18.10.1 / DPDK 18.08.0 initialization...
[ DPDK EAL parameters: perf --no-shconf -c 0x1 -m 2048 --file-prefix=spdk_pid487 ]
EAL: Detected 24 lcore(s)
EAL: Detected 2 NUMA nodes
EAL: No free hugepages reported in hugepages-1048576kB
EAL: Probing VFIO support...
Initializing NVMe Controllers
EAL: PCI device 0000:0b:00.0 on NUMA socket 0
EAL:   probe driver: 8086:953 spdk_nvme
Attaching to NVMe Controller at 0000:0b:00.0
Attached to NVMe Controller at 0000:0b:00.0 [8086:0953]
Associating INTEL SSDPEDMW012T4  (CVCQ652400851P2JGN  ) with lcore 0
Initialization complete. Launching workers.
Starting thread on core 0
========================================================
                                                                                            Latency(us)
Device Information                                     :       IOPS       MB/s    Average        min        max
INTEL SSDPEDMW012T4  (CVCQ652400851P2JGN  ) from core 0:  120407.00     470.34       8.26       5.79    4635.11
========================================================
Total                                                  :  120407.00     470.34       8.26       5.79    4635.11

j2@j2-pc:~/spdk-18.10.1$ sudo ./examples/nvme/perf/perf -o 4096 -q 4 -s 2048 -w randwrite -t 4 -c 0x1 -r 'trtype:PCIe traddr:0000:0b:00.0'
Starting SPDK v18.10.1 / DPDK 18.08.0 initialization...
[ DPDK EAL parameters: perf --no-shconf -c 0x1 -m 2048 --file-prefix=spdk_pid492 ]
EAL: Detected 24 lcore(s)
EAL: Detected 2 NUMA nodes
EAL: No free hugepages reported in hugepages-1048576kB
EAL: Probing VFIO support...
Initializing NVMe Controllers
EAL: PCI device 0000:0b:00.0 on NUMA socket 0
EAL:   probe driver: 8086:953 spdk_nvme
Attaching to NVMe Controller at 0000:0b:00.0
Attached to NVMe Controller at 0000:0b:00.0 [8086:0953]
Associating INTEL SSDPEDMW012T4  (CVCQ652400851P2JGN  ) with lcore 0
Initialization complete. Launching workers.
Starting thread on core 0
========================================================
                                                                                            Latency(us)
Device Information                                     :       IOPS       MB/s    Average        min        max
INTEL SSDPEDMW012T4  (CVCQ652400851P2JGN  ) from core 0:  279772.75    1092.86      14.27       4.76    4760.82
========================================================
Total                                                  :  279772.75    1092.86      14.27       4.76    4760.82

j2@j2-pc:~/spdk-18.10.1$ sudo ./examples/nvme/perf/perf -o 409616 -s 2048 -w randwrite -t 4 -c 0x1 -r 'trtype:PCIe traddr:0000:0b:00.0'0'
Starting SPDK v18.10.1 / DPDK 18.08.0 initialization...
[ DPDK EAL parameters: perf --no-shconf -c 0x1 -m 2048 --file-prefix=spdk_pid497 ]
EAL: Detected 24 lcore(s)
EAL: Detected 2 NUMA nodes
EAL: No free hugepages reported in hugepages-1048576kB
EAL: Probing VFIO support...
Initializing NVMe Controllers
EAL: PCI device 0000:0b:00.0 on NUMA socket 0
EAL:   probe driver: 8086:953 spdk_nvme
Attaching to NVMe Controller at 0000:0b:00.0
j2@j2-pc:~/spdk-18.10.1$ sudo ./examples/nvme/perf/perf -o 4096 -q 16 -s 2048 -w randwrite -t 4 -c 0x1 -r 'trtype:PCIe traddr:0000:0b:00.0'
Starting SPDK v18.10.1 / DPDK 18.08.0 initialization...
[ DPDK EAL parameters: perf --no-shconf -c 0x1 -m 2048 --file-prefix=spdk_pid503 ]
EAL: Detected 24 lcore(s)
EAL: Detected 2 NUMA nodes
EAL: No free hugepages reported in hugepages-1048576kB
EAL: Probing VFIO support...
Initializing NVMe Controllers
EAL: PCI device 0000:0b:00.0 on NUMA socket 0
EAL:   probe driver: 8086:953 spdk_nvme
Attaching to NVMe Controller at 0000:0b:00.0
Attached to NVMe Controller at 0000:0b:00.0 [8086:0953]
Associating INTEL SSDPEDMW012T4  (CVCQ652400851P2JGN  ) with lcore 0
Initialization complete. Launching workers.
Starting thread on core 0
========================================================
                                                                                            Latency(us)
Device Information                                     :       IOPS       MB/s    Average        min        max
INTEL SSDPEDMW012T4  (CVCQ652400851P2JGN  ) from core 0:  287862.00    1124.46      55.56       4.90    5568.93
========================================================
Total                                                  :  287862.00    1124.46      55.56       4.90    5568.93

j2@j2-pc:~/spdk-18.10.1$ sudo ./examples/nvme/perf/perf -o 4194304 -q 16 -s 2048 -w randwrite -t 4 -c 0x1 -r 'trtype:PCIe traddr:0000:0b:00.0'
Starting SPDK v18.10.1 / DPDK 18.08.0 initialization...
[ DPDK EAL parameters: perf --no-shconf -c 0x1 -m 2048 --file-prefix=spdk_pid507 ]
EAL: Detected 24 lcore(s)
EAL: Detected 2 NUMA nodes
EAL: No free hugepages reported in hugepages-1048576kB
EAL: Probing VFIO support...
Initializing NVMe Controllers
EAL: PCI device 0000:0b:00.0 on NUMA socket 0
EAL:   probe driver: 8086:953 spdk_nvme
Attaching to NVMe Controller at 0000:0b:00.0
Attached to NVMe Controller at 0000:0b:00.0 [8086:0953]
Associating INTEL SSDPEDMW012T4  (CVCQ652400851P2JGN  ) with lcore 0
Initialization complete. Launching workers.
Starting thread on core 0
========================================================
                                                                                            Latency(us)
Device Information                                     :       IOPS       MB/s    Average        min        max
INTEL SSDPEDMW012T4  (CVCQ652400851P2JGN  ) from core 0:     301.75    1207.00   53260.31   11526.45   62177.25
========================================================
Total                                                  :     301.75    1207.00   53260.31   11526.45   62177.25

j2@j2-pc:~/spdk-18.10.1$ sudo ./examples/nvme/perf/perf -o 4194304 -q 16 -s 2048 -w randread -t 4 -c 0x1 -r 'trtype:PCIe traddr:0000:0b:00.0'
Starting SPDK v18.10.1 / DPDK 18.08.0 initialization...
[ DPDK EAL parameters: perf --no-shconf -c 0x1 -m 2048 --file-prefix=spdk_pid519 ]
EAL: Detected 24 lcore(s)
EAL: Detected 2 NUMA nodes
EAL: No free hugepages reported in hugepages-1048576kB
EAL: Probing VFIO support...
Initializing NVMe Controllers
EAL: PCI device 0000:0b:00.0 on NUMA socket 0
EAL:   probe driver: 8086:953 spdk_nvme
Attaching to NVMe Controller at 0000:0b:00.0
Attached to NVMe Controller at 0000:0b:00.0 [8086:0953]
Associating INTEL SSDPEDMW012T4  (CVCQ652400851P2JGN  ) with lcore 0
Initialization complete. Launching workers.
Starting thread on core 0
========================================================
                                                                                            Latency(us)
Device Information                                     :       IOPS       MB/s    Average        min        max
INTEL SSDPEDMW012T4  (CVCQ652400851P2JGN  ) from core 0:     737.50    2950.00   21703.05    7895.89   25541.66
========================================================
Total                                                  :     737.50    2950.00   21703.05    7895.89   25541.66

j2@j2-pc:~/spdk-18.10.1$ sudo ./examples/nvme/perf/perf -o 16777216 -q 16 -s 2048 -w randwrite -t 4 -c 0x1 -r 'trtype:PCIe traddr:0000:0b:00.0'
Starting SPDK v18.10.1 / DPDK 18.08.0 initialization...
[ DPDK EAL parameters: perf --no-shconf -c 0x1 -m 2048 --file-prefix=spdk_pid524 ]
EAL: Detected 24 lcore(s)
EAL: Detected 2 NUMA nodes
EAL: No free hugepages reported in hugepages-1048576kB
EAL: Probing VFIO support...
Initializing NVMe Controllers
EAL: PCI device 0000:0b:00.0 on NUMA socket 0
EAL:   probe driver: 8086:953 spdk_nvme
Attaching to NVMe Controller at 0000:0b:00.0
Attached to NVMe Controller at 0000:0b:00.0 [8086:0953]
Associating INTEL SSDPEDMW012T4  (CVCQ652400851P2JGN  ) with lcore 0
Initialization complete. Launching workers.
Starting thread on core 0
========================================================
                                                                                            Latency(us)
Device Information                                     :       IOPS       MB/s    Average        min        max
INTEL SSDPEDMW012T4  (CVCQ652400851P2JGN  ) from core 0:      78.25    1252.00  208153.12   47927.36  232573.98
========================================================
Total                                                  :      78.25    1252.00  208153.12   47927.36  232573.98
```

