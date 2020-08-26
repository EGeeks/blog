# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 机器
```shell
j2@j2-pc:~$ uname -a
Linux j2-pc 4.13.0-36-generic #40~16.04.1-Ubuntu SMP Fri Feb 16 23:25:58 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
```

# ubuntu16.04 系统自带驱动测试结果
## sync
| rw | ioengine | bs | iodepth | numjobs | bw (MB/s)  | iops |  note |
|:--------|:--------|:--------|:--------|:--------|:-------------|:-------|:-------|
| read | sync | 4k | 1 | 1 |  2160.4 | 553046 | |
| read | sync | 4k | 4 | 1 | 2266.2 | 580125 | |
| read | sync | 4k | 16 | 1  | 2328.7 | 596120 | |
| write | sync | 4k | 1 | 1  |  1434.7 | 367277 | 有时速度为1719.1这个级别 |
| write | sync | 4k | 4 | 1  | 1437.1 | 368115 | 有时速度为1719.1这个级别 |
| write | sync | 4k | 16 | 1  | 1450.2 | 371243 | 有时速度为1719.1这个级别 |
| read | sync | 4m | 1 | 1  |  3215.8 | 803 | |
| read | sync | 4m | 4 | 1  | 3000.8 | 750 | 速度有些漂 |
| read | sync | 4m | 16 | 1  | 3150.8 | 787 | |
| write | sync | 4m | 1 | 1  |  2127.9 | 531 | |
| write | sync | 4m | 4 | 1  | 2091.5 | 522 | |
| write | sync | 4m | 16 | 1  | 2087.7 | 521 | |
可以看到iodepth在sync这种方式下对速度影响不大，甚至没有明确的比例关系，和理论相符。

## libaio
首先格式化磁盘，创建ext4文件系统，参数direct=1
| rw | ioengine | bs | iodepth | numjobs | bw (MB/s)  | iops |  note |
|:--------|:--------|:--------|:--------|:--------|:-------------|:-------|:-------|
| read | libaio | 4k | 4 | 1 |  140.5 | 35119 | |
| read | libaio | 4k | 4 | 4 | 498.7 | 124663 | |
| read | libaio | 4k | 16 | 16  |  1688.5 | 432229 | |
| read | libaio | 4m | 4 | 4  | 2420.3 | 605 | |
| read | libaio | 4m | 16 | 16  | 2308.6 | 577 | |
| write | libaio | 4k | 4 | 1  | 508.5 | 127130 | |
| write | libaio | 4k | 4 | 4  |  1007.5 | 257912 | |
| write | libaio | 4k | 16 | 16  | 827.9 | 206971 | |
| write | libaio | 4m | 4 | 4  | 1191.4 | 297 | |
| write | libaio | 4m | 16 | 16  |  1192.9 | 298 | |

## 软raid
> [Ubuntu 上创建常用磁盘阵列](https://www.cnblogs.com/Ray-liang/p/5996271.html)

md组软raid，4块750，大块连续写，性能可观，达到指标，
```shell
j2@j2-pc:~$ sudo fio -filename=/home/j2/nvme/a.bin -direct=1 -iodepth 1 -thread -rw=read -ioengine=libaio -bs=16m -size=8G -numjobs=1 -group_reporting -name=mytest
mytest: (g=0): rw=read, bs=16M-16M/16M-16M/16M-16M, ioengine=libaio, iodepth=1
fio-2.2.10
Starting 1 thread
Jobs: 1 (f=1)
mytest: (groupid=0, jobs=1): err= 0: pid=3547: Fri Dec 28 16:32:44 2018
  read : io=8192.0MB, bw=7454.5MB/s, iops=465, runt=  1099msec
    slat (usec): min=897, max=10587, avg=1145.32, stdev=459.31
    clat (usec): min=482, max=4821, avg=997.32, stdev=472.46
     lat (usec): min=1863, max=11098, avg=2142.95, stdev=622.42
    clat percentiles (usec):
     |  1.00th=[  548],  5.00th=[  684], 10.00th=[  764], 20.00th=[  820],
     | 30.00th=[  860], 40.00th=[  884], 50.00th=[  900], 60.00th=[  916],
     | 70.00th=[  940], 80.00th=[  988], 90.00th=[ 1128], 95.00th=[ 1992],
     | 99.00th=[ 3120], 99.50th=[ 3504], 99.90th=[ 4832], 99.95th=[ 4832],
     | 99.99th=[ 4832]
    bw (MB  /s): min= 7456, max= 7504, per=100.00%, avg=7480.49, stdev=34.64
    lat (usec) : 500=0.20%, 750=7.81%, 1000=72.85%
    lat (msec) : 2=14.26%, 4=4.49%, 10=0.39%
  cpu          : usr=0.36%, sys=53.19%, ctx=521, majf=0, minf=4097
  IO depths    : 1=100.0%, 2=0.0%, 4=0.0%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     issued    : total=r=512/w=0/d=0, short=r=0/w=0/d=0, drop=r=0/w=0/d=0
     latency   : target=0, window=0, percentile=100.00%, depth=1

Run status group 0 (all jobs):
   READ: io=8192.0MB, aggrb=7454.5MB/s, minb=7454.5MB/s, maxb=7454.5MB/s, mint=1099msec, maxt=1099msec

Disk stats (read/write):
    md0: ios=53632/88, merge=0/0, ticks=0/0, in_queue=0, util=0.00%, aggrios=16384/28, aggrmerge=0/0, aggrticks=10312/0, aggrin_queue=10312, aggrutil=75.86%
  nvme2n1: ios=16384/28, merge=0/0, ticks=10284/0, in_queue=10284, util=75.22%
  nvme1n1: ios=16384/28, merge=0/0, ticks=10316/0, in_queue=10316, util=75.86%
  nvme0n1: ios=16384/28, merge=0/0, ticks=10240/0, in_queue=10240, util=75.22%
  nvme3n1: ios=16384/28, merge=0/0, ticks=10408/0, in_queue=10408, util=73.94%
j2@j2-pc:~$ sudo fio -filename=/home/j2/nvme/a.bin -direct=1 -iodepth 1 -thread -rw=write -ioengine=libaio -bs=16m -size=8G -numjobs=1 -group_reporting -name=mytest
mytest: (g=0): rw=write, bs=16M-16M/16M-16M/16M-16M, ioengine=libaio, iodepth=1
fio-2.2.10
Starting 1 thread
Jobs: 1 (f=1): [W(1)] [-.-% done] [0KB/3788MB/0KB /s] [0/236/0 iops] [eta 00m:00s]
mytest: (groupid=0, jobs=1): err= 0: pid=3608: Fri Dec 28 16:34:52 2018
  write: io=8192.0MB, bw=3855.6MB/s, iops=240, runt=  2125msec
    slat (usec): min=1222, max=3892, avg=1622.14, stdev=226.83
    clat (usec): min=186, max=9122, avg=2521.24, stdev=1031.97
     lat (msec): min=2, max=10, avg= 4.14, stdev= 1.01
    clat percentiles (usec):
     |  1.00th=[  486],  5.00th=[ 1448], 10.00th=[ 1704], 20.00th=[ 1928],
     | 30.00th=[ 2096], 40.00th=[ 2224], 50.00th=[ 2384], 60.00th=[ 2544],
     | 70.00th=[ 2672], 80.00th=[ 2800], 90.00th=[ 3088], 95.00th=[ 4320],
     | 99.00th=[ 6880], 99.50th=[ 7584], 99.90th=[ 9152], 99.95th=[ 9152],
     | 99.99th=[ 9152]
    bw (MB  /s): min= 3785, max= 3968, per=100.00%, avg=3858.64, stdev=86.72
    lat (usec) : 250=0.20%, 500=0.98%, 750=0.20%, 1000=0.20%
    lat (msec) : 2=20.90%, 4=70.90%, 10=6.64%
  cpu          : usr=14.27%, sys=25.09%, ctx=523, majf=0, minf=1
  IO depths    : 1=100.0%, 2=0.0%, 4=0.0%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     issued    : total=r=0/w=512/d=0, short=r=0/w=0/d=0, drop=r=0/w=0/d=0
     latency   : target=0, window=0, percentile=100.00%, depth=1

Run status group 0 (all jobs):
  WRITE: io=8192.0MB, aggrb=3855.6MB/s, minb=3855.6MB/s, maxb=3855.6MB/s, mint=2125msec, maxt=2125msec

Disk stats (read/write):
    md0: ios=0/58048, merge=0/0, ticks=0/0, in_queue=0, util=0.00%, aggrios=0/16437, aggrmerge=0/1, aggrticks=0/6735, aggrin_queue=6735, aggrutil=60.41%
  nvme2n1: ios=0/16439, merge=0/3, ticks=0/6428, in_queue=6432, util=60.41%
  nvme1n1: ios=0/16438, merge=0/0, ticks=0/6912, in_queue=6908, util=56.25%
  nvme0n1: ios=0/16438, merge=0/2, ticks=0/6956, in_queue=6956, util=57.64%
  nvme3n1: ios=0/16436, merge=0/0, ticks=0/6644, in_queue=6644, util=53.83%
```

# 详细
```shell
j2@j2-pc:~$ sudo fio -filename=/dev/nvmen0 -direct=0 -iodepth 1 -thread -rw=read -ioengine=sync -bs=4k -size=8G -numjobs=1 -group_reporting -name=mytest
mytest: (g=0): rw=read, bs=4K-4K/4K-4K/4K-4K, ioengine=sync, iodepth=1
fio-2.2.10
Starting 1 thread
Jobs: 1 (f=1): [R(1)] [80.0% done] [2592MB/0KB/0KB /s] [664K/0/0 iops] [eta 00m:01s]
mytest: (groupid=0, jobs=1): err= 0: pid=4478: Thu Dec 27 14:56:46 2018
  read : io=8192.0MB, bw=2160.4MB/s, iops=553046, runt=  3792msec
    clat (usec): min=1, max=287, avg= 1.27, stdev= 2.32
     lat (usec): min=1, max=287, avg= 1.31, stdev= 2.34
    clat percentiles (usec):
     |  1.00th=[    1],  5.00th=[    1], 10.00th=[    1], 20.00th=[    1],
     | 30.00th=[    1], 40.00th=[    1], 50.00th=[    1], 60.00th=[    1],
     | 70.00th=[    1], 80.00th=[    2], 90.00th=[    2], 95.00th=[    2],
     | 99.00th=[    2], 99.50th=[    2], 99.90th=[    2], 99.95th=[    8],
     | 99.99th=[  199]
    bw (MB  /s): min=    0, max= 2614, per=100.00%, avg=2223.40, stdev=980.52
    lat (usec) : 2=75.60%, 4=24.33%, 10=0.02%, 20=0.03%, 50=0.01%
    lat (usec) : 250=0.01%, 500=0.01%
  cpu          : usr=20.13%, sys=79.79%, ctx=318, majf=0, minf=10
  IO depths    : 1=100.0%, 2=0.0%, 4=0.0%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     issued    : total=r=2097152/w=0/d=0, short=r=0/w=0/d=0, drop=r=0/w=0/d=0
     latency   : target=0, window=0, percentile=100.00%, depth=1

Run status group 0 (all jobs):
   READ: io=8192.0MB, aggrb=2160.4MB/s, minb=2160.4MB/s, maxb=2160.4MB/s, mint=3792msec, maxt=3792msec
j2@j2-pc:~$ sudo fio -filename=/dev/nvmen0 -direct=0 -iodepth 1 -thread -rw=write -ioengine=sync -bs=4k -size=8G -numjobs=1 -group_reporting -name=mytest
mytest: (g=0): rw=write, bs=4K-4K/4K-4K/4K-4K, ioengine=sync, iodepth=1
fio-2.2.10
Starting 1 thread
Jobs: 1 (f=1): [W(1)] [100.0% done] [0KB/1638MB/0KB /s] [0/419K/0 iops] [eta 00m:00s]
mytest: (groupid=0, jobs=1): err= 0: pid=4631: Thu Dec 27 15:13:14 2018
  write: io=8192.0MB, bw=1434.7MB/s, iops=367277, runt=  5710msec
    clat (usec): min=1, max=252, avg= 2.13, stdev= 2.83
     lat (usec): min=1, max=252, avg= 2.18, stdev= 2.87
    clat percentiles (usec):
     |  1.00th=[    2],  5.00th=[    2], 10.00th=[    2], 20.00th=[    2],
     | 30.00th=[    2], 40.00th=[    2], 50.00th=[    2], 60.00th=[    2],
     | 70.00th=[    2], 80.00th=[    2], 90.00th=[    2], 95.00th=[    3],
     | 99.00th=[    3], 99.50th=[    3], 99.90th=[    4], 99.95th=[   12],
     | 99.99th=[  205]
    bw (MB  /s): min=    0, max= 1642, per=100.00%, avg=1486.80, stdev=493.13
    lat (usec) : 2=0.30%, 4=99.60%, 10=0.04%, 20=0.04%, 50=0.01%
    lat (usec) : 250=0.02%, 500=0.01%
  cpu          : usr=15.68%, sys=84.22%, ctx=479, majf=0, minf=20
  IO depths    : 1=100.0%, 2=0.0%, 4=0.0%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     issued    : total=r=0/w=2097152/d=0, short=r=0/w=0/d=0, drop=r=0/w=0/d=0
     latency   : target=0, window=0, percentile=100.00%, depth=1

Run status group 0 (all jobs):
  WRITE: io=8192.0MB, aggrb=1434.7MB/s, minb=1434.7MB/s, maxb=1434.7MB/s, mint=5710msec, maxt=5710msec
```

```shell
j2@j2-pc:~$ sudo fio -filename=/home/j2/nvme/a.bin -direct=1 -iodepth 4 -thread -rw=read -ioengine=libaio -bs=4k -size=8G -numjobs=1 -group_reporting -name=mytest
mytest: (g=0): rw=read, bs=4K-4K/4K-4K/4K-4K, ioengine=libaio, iodepth=4
fio-2.2.10
Starting 1 thread
mytest: Laying out IO file(s) (1 file(s) / 8192MB)
Jobs: 1 (f=1): [R(1)] [100.0% done] [137.4MB/0KB/0KB /s] [35.2K/0/0 iops] [eta 00m:00s]
mytest: (groupid=0, jobs=1): err= 0: pid=5763: Thu Dec 27 15:45:54 2018
  read : io=8192.0MB, bw=140477KB/s, iops=35119, runt= 59715msec
    slat (usec): min=2, max=664, avg= 4.18, stdev= 5.61
    clat (usec): min=7, max=3137, avg=108.74, stdev=65.81
     lat (usec): min=13, max=3142, avg=113.00, stdev=65.98
    clat percentiles (usec):
     |  1.00th=[   46],  5.00th=[   58], 10.00th=[   65], 20.00th=[   72],
     | 30.00th=[   97], 40.00th=[  103], 50.00th=[  106], 60.00th=[  113],
     | 70.00th=[  120], 80.00th=[  123], 90.00th=[  131], 95.00th=[  145],
     | 99.00th=[  322], 99.50th=[  354], 99.90th=[  588], 99.95th=[ 1416],
     | 99.99th=[ 2576]
    bw (KB  /s): min=137320, max=144176, per=100.00%, avg=140474.35, stdev=1349.77
    lat (usec) : 10=0.01%, 20=0.01%, 50=1.44%, 100=32.73%, 250=63.65%
    lat (usec) : 500=2.05%, 750=0.04%, 1000=0.02%
    lat (msec) : 2=0.03%, 4=0.04%
  cpu          : usr=10.22%, sys=20.64%, ctx=1081924, majf=0, minf=131
  IO depths    : 1=0.1%, 2=0.1%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     issued    : total=r=2097152/w=0/d=0, short=r=0/w=0/d=0, drop=r=0/w=0/d=0
     latency   : target=0, window=0, percentile=100.00%, depth=4

Run status group 0 (all jobs):
   READ: io=8192.0MB, aggrb=140477KB/s, minb=140477KB/s, maxb=140477KB/s, mint=59715msec, maxt=59715msec

Disk stats (read/write):
  nvme0n1: ios=2091774/1386, merge=0/31, ticks=207272/0, in_queue=207004, util=99.43%
j2@j2-pc:~$ sudo fio -filename=/home/j2/nvme/a.bin -direct=1 -iodepth 4 -thread -rw=read -ioengine=libaio -bs=4k -size=8G -numjobs=4 -group_reporting -name=mytest
mytest: (g=0): rw=read, bs=4K-4K/4K-4K/4K-4K, ioengine=libaio, iodepth=4
...
fio-2.2.10
Starting 4 threads
Jobs: 3 (f=2): [R(2),_(1),R(1)] [100.0% done] [475.9MB/0KB/0KB /s] [122K/0/0 iops] [eta 00m:00s]
mytest: (groupid=0, jobs=4): err= 0: pid=5811: Thu Dec 27 15:55:42 2018
  read : io=32768MB, bw=498654KB/s, iops=124663, runt= 67290msec
    slat (usec): min=2, max=761, avg= 5.27, stdev=11.07
    clat (usec): min=1, max=4717, avg=121.58, stdev=70.12
     lat (usec): min=12, max=4722, avg=126.94, stdev=70.72
    clat percentiles (usec):
     |  1.00th=[   45],  5.00th=[   60], 10.00th=[   67], 20.00th=[   82],
     | 30.00th=[  100], 40.00th=[  106], 50.00th=[  112], 60.00th=[  119],
     | 70.00th=[  125], 80.00th=[  133], 90.00th=[  163], 95.00th=[  258],
     | 99.00th=[  374], 99.50th=[  474], 99.90th=[  636], 99.95th=[  724],
     | 99.99th=[ 2352]
    bw (KB  /s): min=119184, max=130016, per=25.10%, avg=125149.40, stdev=1852.25
    lat (usec) : 2=0.01%, 4=0.01%, 10=0.01%, 20=0.01%, 50=1.50%
    lat (usec) : 100=28.37%, 250=64.87%, 500=4.91%, 750=0.29%, 1000=0.02%
    lat (msec) : 2=0.01%, 4=0.01%, 10=0.01%
  cpu          : usr=10.06%, sys=21.68%, ctx=4488065, majf=0, minf=751
  IO depths    : 1=0.1%, 2=0.1%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     issued    : total=r=8388608/w=0/d=0, short=r=0/w=0/d=0, drop=r=0/w=0/d=0
     latency   : target=0, window=0, percentile=100.00%, depth=4

Run status group 0 (all jobs):
   READ: io=32768MB, aggrb=498654KB/s, minb=498654KB/s, maxb=498654KB/s, mint=67290msec, maxt=67290msec

Disk stats (read/write):
  nvme0n1: ios=8385723/0, merge=0/0, ticks=931832/0, in_queue=954052, util=100.00%
j2@j2-pc:~$ sudo fio -filename=/home/j2/nvme/a.bin -direct=1 -iodepth 4 -thread -rw=read -ioengine=libaio -bs=4m -size=8G -numjobs=4 -group_reporting -name=mytest
mytest: (g=0): rw=read, bs=4M-4M/4M-4M/4M-4M, ioengine=libaio, iodepth=4
...
fio-2.2.10
Starting 4 threads
Jobs: 2 (f=2): [_(2),R(2)] [87.5% done] [2470MB/0KB/0KB /s] [617/0/0 iops] [eta 00m:02s]
mytest: (groupid=0, jobs=4): err= 0: pid=5848: Thu Dec 27 15:56:10 2018
  read : io=32768MB, bw=2420.3MB/s, iops=605, runt= 13539msec
    slat (usec): min=171, max=3454, avg=377.42, stdev=138.87
    clat (msec): min=4, max=55, avg=24.90, stdev= 9.87
     lat (msec): min=5, max=55, avg=25.28, stdev= 9.87
    clat percentiles (usec):
     |  1.00th=[ 5792],  5.00th=[12224], 10.00th=[15424], 20.00th=[17280],
     | 30.00th=[19072], 40.00th=[20096], 50.00th=[21376], 60.00th=[23424],
     | 70.00th=[29568], 80.00th=[35584], 90.00th=[40704], 95.00th=[42752],
     | 99.00th=[47872], 99.50th=[49408], 99.90th=[51968], 99.95th=[52480],
     | 99.99th=[55552]
    bw (KB  /s): min=358599, max=1275401, per=25.54%, avg=632989.87, stdev=195817.37
    lat (msec) : 10=2.55%, 20=37.26%, 50=59.81%, 100=0.38%
  cpu          : usr=0.10%, sys=6.14%, ctx=8125, majf=0, minf=16388
  IO depths    : 1=0.1%, 2=0.1%, 4=99.9%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     issued    : total=r=8192/w=0/d=0, short=r=0/w=0/d=0, drop=r=0/w=0/d=0
     latency   : target=0, window=0, percentile=100.00%, depth=4

Run status group 0 (all jobs):
   READ: io=32768MB, aggrb=2420.3MB/s, minb=2420.3MB/s, maxb=2420.3MB/s, mint=13539msec, maxt=13539msec

Disk stats (read/write):
  nvme0n1: ios=259384/0, merge=0/0, ticks=5034684/0, in_queue=5048896, util=99.43%
j2@j2-pc:~$ sudo fio -filename=/home/j2/nvme/a.bin -direct=1 -iodepth 16 -thread -rw=read -ioengine=libaio -bs=4k -size=8G -numjobs=16 -group_reporting -name=mytest
mytest: (g=0): rw=read, bs=4K-4K/4K-4K/4K-4K, ioengine=libaio, iodepth=16
...
fio-2.2.10
Starting 16 threads
Jobs: 16 (f=16): [R(16)] [100.0% done] [1706MB/0KB/0KB /s] [437K/0/0 iops] [eta 00m:00s]
mytest: (groupid=0, jobs=16): err= 0: pid=5881: Thu Dec 27 15:58:09 2018
  read : io=131072MB, bw=1688.5MB/s, iops=432229, runt= 77631msec
    slat (usec): min=2, max=6316, avg=11.70, stdev=35.50
    clat (usec): min=1, max=7463, avg=577.75, stdev=301.74
     lat (usec): min=37, max=7468, avg=589.58, stdev=299.03
    clat percentiles (usec):
     |  1.00th=[  141],  5.00th=[  183], 10.00th=[  229], 20.00th=[  326],
     | 30.00th=[  398], 40.00th=[  462], 50.00th=[  532], 60.00th=[  612],
     | 70.00th=[  700], 80.00th=[  812], 90.00th=[  964], 95.00th=[ 1096],
     | 99.00th=[ 1432], 99.50th=[ 1640], 99.90th=[ 2608], 99.95th=[ 2992],
     | 99.99th=[ 3600]
    bw (KB  /s): min=97880, max=156056, per=6.26%, avg=108292.76, stdev=3430.50
    lat (usec) : 2=0.01%, 4=0.01%, 10=0.01%, 20=0.01%, 50=0.01%
    lat (usec) : 100=0.10%, 250=11.85%, 500=34.05%, 750=29.19%, 1000=16.42%
    lat (msec) : 2=8.16%, 4=0.22%, 10=0.01%
  cpu          : usr=9.28%, sys=40.62%, ctx=10560160, majf=0, minf=2779
  IO depths    : 1=0.1%, 2=0.1%, 4=0.1%, 8=0.1%, 16=100.0%, 32=0.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.1%, 32=0.0%, 64=0.0%, >=64=0.0%
     issued    : total=r=33554432/w=0/d=0, short=r=0/w=0/d=0, drop=r=0/w=0/d=0
     latency   : target=0, window=0, percentile=100.00%, depth=16

Run status group 0 (all jobs):
   READ: io=131072MB, aggrb=1688.5MB/s, minb=1688.5MB/s, maxb=1688.5MB/s, mint=77631msec, maxt=77631msec

Disk stats (read/write):
  nvme0n1: ios=33508298/0, merge=0/0, ticks=16417456/0, in_queue=18538620, util=100.00%
j2@j2-pc:~$ sudo fio -filename=/home/j2/nvme/a.bin -direct=1 -iodepth 16 -thread -rw=read -ioengine=libaio -bs=4m -size=8G -numjobs=16 -group_reporting -name=mytest
mytest: (g=0): rw=read, bs=4M-4M/4M-4M/4M-4M, ioengine=libaio, iodepth=16
...
fio-2.2.10
Starting 16 threads
Jobs: 6 (f=4): [_(1),R(1),_(5),R(2),_(1),R(2),_(3),R(1)] [90.5% done] [2537MB/0KB/0KB /s] [634/0/0 iops] [eta 00m:06s]
mytest: (groupid=0, jobs=16): err= 0: pid=5926: Thu Dec 27 15:59:25 2018
  read : io=131072MB, bw=2308.6MB/s, iops=577, runt= 56776msec
    slat (usec): min=158, max=154951, avg=17878.04, stdev=20726.09
    clat (msec): min=18, max=1731, avg=403.50, stdev=214.88
     lat (msec): min=18, max=1787, avg=421.38, stdev=225.79
    clat percentiles (msec):
     |  1.00th=[   30],  5.00th=[   58], 10.00th=[   96], 20.00th=[  194],
     | 30.00th=[  293], 40.00th=[  363], 50.00th=[  416], 60.00th=[  461],
     | 70.00th=[  510], 80.00th=[  570], 90.00th=[  668], 95.00th=[  750],
     | 99.00th=[  963], 99.50th=[ 1057], 99.90th=[ 1303], 99.95th=[ 1434],
     | 99.99th=[ 1696]
    bw (KB  /s): min=15398, max=732882, per=6.52%, avg=154207.86, stdev=75711.79
    lat (msec) : 20=0.03%, 50=3.80%, 100=6.78%, 250=15.08%, 500=42.39%
    lat (msec) : 750=26.95%, 1000=4.23%, 2000=0.73%
  cpu          : usr=0.02%, sys=1.75%, ctx=75471, majf=0, minf=458781
  IO depths    : 1=0.1%, 2=0.1%, 4=0.2%, 8=0.4%, 16=99.3%, 32=0.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.1%, 32=0.0%, 64=0.0%, >=64=0.0%
     issued    : total=r=32768/w=0/d=0, short=r=0/w=0/d=0, drop=r=0/w=0/d=0
     latency   : target=0, window=0, percentile=100.00%, depth=16

Run status group 0 (all jobs):
   READ: io=131072MB, aggrb=2308.6MB/s, minb=2308.6MB/s, maxb=2308.6MB/s, mint=56776msec, maxt=56776msec

Disk stats (read/write):
  nvme0n1: ios=1047097/0, merge=0/0, ticks=315910948/0, in_queue=316514876, util=99.98%
j2@j2-pc:~$ sudo fio -filename=/home/j2/nvme/a.bin -direct=1 -iodepth 16 -thread -rw=write -ioengine=libaio -bs=4m -size=8G -numjobs=16 -group_reporting -name=mytest
mytest: (g=0): rw=write, bs=4M-4M/4M-4M/4M-4M, ioengine=libaio, iodepth=16
...
fio-2.2.10
Starting 16 threads
Jobs: 4 (f=4): [_(4),W(2),_(1),W(1),_(1),W(1),_(6)] [95.7% done] [0KB/1195MB/0KB /s] [0/298/0 iops] [eta 00m:05s]                    
mytest: (groupid=0, jobs=16): err= 0: pid=5994: Thu Dec 27 16:11:13 2018
  write: io=131072MB, bw=1192.9MB/s, iops=298, runt=109884msec
    slat (usec): min=350, max=807847, avg=22297.42, stdev=71377.49
    clat (msec): min=12, max=2760, avg=804.18, stdev=435.80
     lat (msec): min=13, max=2761, avg=826.48, stdev=431.08
    clat percentiles (msec):
     |  1.00th=[   70],  5.00th=[  115], 10.00th=[  200], 20.00th=[  400],
     | 30.00th=[  578], 40.00th=[  709], 50.00th=[  799], 60.00th=[  881],
     | 70.00th=[  996], 80.00th=[ 1139], 90.00th=[ 1385], 95.00th=[ 1582],
     | 99.00th=[ 1926], 99.50th=[ 2024], 99.90th=[ 2376], 99.95th=[ 2507],
     | 99.99th=[ 2671]
    bw (KB  /s): min= 4447, max=412023, per=6.75%, avg=82464.87, stdev=47440.58
    lat (msec) : 20=0.01%, 50=0.10%, 100=3.56%, 250=8.42%, 500=13.77%
    lat (msec) : 750=18.59%, 1000=26.14%, 2000=28.85%, >=2000=0.57%
  cpu          : usr=0.78%, sys=1.48%, ctx=41446, majf=0, minf=2825479
  IO depths    : 1=0.1%, 2=0.1%, 4=0.2%, 8=0.4%, 16=99.3%, 32=0.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.1%, 32=0.0%, 64=0.0%, >=64=0.0%
     issued    : total=r=0/w=32768/d=0, short=r=0/w=0/d=0, drop=r=0/w=0/d=0
     latency   : target=0, window=0, percentile=100.00%, depth=16

Run status group 0 (all jobs):
  WRITE: io=131072MB, aggrb=1192.9MB/s, minb=1192.9MB/s, maxb=1192.9MB/s, mint=109884msec, maxt=109884msec

Disk stats (read/write):
  nvme0n1: ios=0/1046796, merge=0/21, ticks=0/650741096, in_queue=651135108, util=99.97%
j2@j2-pc:~$ sudo fio -filename=/home/j2/nvme/a.bin -direct=1 -iodepth 4 -thread -rw=write -ioengine=libaio -bs=4m -size=8G -numjobs=4 -group_reporting -name=mytest
mytest: (g=0): rw=write, bs=4M-4M/4M-4M/4M-4M, ioengine=libaio, iodepth=4
...
fio-2.2.10
Starting 4 threads
Jobs: 3 (f=3): [W(3),_(1)] [96.6% done] [0KB/1187MB/0KB /s] [0/296/0 iops] [eta 00m:01s]
mytest: (groupid=0, jobs=4): err= 0: pid=6072: Thu Dec 27 16:23:21 2018
  write: io=32768MB, bw=1191.4MB/s, iops=297, runt= 27505msec
    slat (usec): min=264, max=40073, avg=771.32, stdev=1294.90
    clat (msec): min=1, max=100, avg=52.34, stdev= 9.54
     lat (msec): min=1, max=101, avg=53.11, stdev= 9.48
    clat percentiles (msec):
     |  1.00th=[   26],  5.00th=[   41], 10.00th=[   43], 20.00th=[   45],
     | 30.00th=[   48], 40.00th=[   50], 50.00th=[   52], 60.00th=[   55],
     | 70.00th=[   56], 80.00th=[   59], 90.00th=[   64], 95.00th=[   70],
     | 99.00th=[   81], 99.50th=[   86], 99.90th=[   95], 99.95th=[   96],
     | 99.99th=[  101]
    bw (KB  /s): min=194661, max=636430, per=25.19%, avg=307340.73, stdev=40690.86
    lat (msec) : 2=0.01%, 10=0.05%, 20=0.77%, 50=38.95%, 100=60.21%
    lat (msec) : 250=0.01%
  cpu          : usr=2.14%, sys=3.38%, ctx=8157, majf=0, minf=27695
  IO depths    : 1=0.1%, 2=0.1%, 4=99.9%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     issued    : total=r=0/w=8192/d=0, short=r=0/w=0/d=0, drop=r=0/w=0/d=0
     latency   : target=0, window=0, percentile=100.00%, depth=4

Run status group 0 (all jobs):
  WRITE: io=32768MB, aggrb=1191.4MB/s, minb=1191.4MB/s, maxb=1191.4MB/s, mint=27505msec, maxt=27505msec

Disk stats (read/write):
  nvme0n1: ios=0/261215, merge=0/5, ticks=0/9335096, in_queue=9339868, util=99.72%
j2@j2-pc:~$ sudo fio -filename=/home/j2/nvme/a.bin -direct=1 -iodepth 4 -thread -rw=write -ioengine=libaio -bs=4k -size=8G -numjobs=4 -group_reporting -name=mytest
mytest: (g=0): rw=write, bs=4K-4K/4K-4K/4K-4K, ioengine=libaio, iodepth=4
...
fio-2.2.10
Starting 4 threads
Jobs: 4 (f=4): [W(4)] [100.0% done] [0KB/1019MB/0KB /s] [0/261K/0 iops] [eta 00m:00s]
mytest: (groupid=0, jobs=4): err= 0: pid=6105: Thu Dec 27 16:24:31 2018
  write: io=32768MB, bw=1007.5MB/s, iops=257912, runt= 32525msec
    slat (usec): min=2, max=4231, avg=13.64, stdev=15.27
    clat (usec): min=0, max=7398, avg=47.84, stdev=80.27
     lat (usec): min=10, max=7410, avg=61.54, stdev=81.85
    clat percentiles (usec):
     |  1.00th=[   26],  5.00th=[   32], 10.00th=[   35], 20.00th=[   38],
     | 30.00th=[   41], 40.00th=[   42], 50.00th=[   44], 60.00th=[   45],
     | 70.00th=[   47], 80.00th=[   48], 90.00th=[   52], 95.00th=[   55],
     | 99.00th=[  227], 99.50th=[  258], 99.90th=[  532], 99.95th=[  684],
     | 99.99th=[ 4768]
    bw (KB  /s): min=238648, max=269344, per=25.01%, avg=258005.59, stdev=5333.04
    lat (usec) : 2=0.01%, 4=0.01%, 10=0.01%, 20=0.30%, 50=83.95%
    lat (usec) : 100=14.11%, 250=1.06%, 500=0.46%, 750=0.07%, 1000=0.01%
    lat (msec) : 2=0.01%, 4=0.01%, 10=0.02%
  cpu          : usr=7.68%, sys=87.18%, ctx=108062, majf=0, minf=280
  IO depths    : 1=0.1%, 2=0.1%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     issued    : total=r=0/w=8388608/d=0, short=r=0/w=0/d=0, drop=r=0/w=0/d=0
     latency   : target=0, window=0, percentile=100.00%, depth=4

Run status group 0 (all jobs):
  WRITE: io=32768MB, aggrb=1007.5MB/s, minb=1007.5MB/s, maxb=1007.5MB/s, mint=32525msec, maxt=32525msec

Disk stats (read/write):
  nvme0n1: ios=0/8363629, merge=0/5, ticks=0/125168, in_queue=126152, util=100.00%
j2@j2-pc:~$ sudo fio -filename=/home/j2/nvme/a.bin -direct=1 -iodepth 4 -thread -rw=write -ioengine=libaio -bs=4k -size=8G -numjobs=1 -group_reporting -name=mytest
mytest: (g=0): rw=write, bs=4K-4K/4K-4K/4K-4K, ioengine=libaio, iodepth=4
fio-2.2.10
Starting 1 thread
Jobs: 1 (f=1): [W(1)] [100.0% done] [0KB/502.9MB/0KB /s] [0/129K/0 iops] [eta 00m:00s]
mytest: (groupid=0, jobs=1): err= 0: pid=6139: Thu Dec 27 16:25:43 2018
  write: io=8192.0MB, bw=508524KB/s, iops=127130, runt= 16496msec
    slat (usec): min=2, max=542, avg= 5.24, stdev= 6.68
    clat (usec): min=8, max=5849, avg=25.35, stdev=35.36
     lat (usec): min=11, max=5854, avg=30.75, stdev=36.24
    clat percentiles (usec):
     |  1.00th=[   15],  5.00th=[   16], 10.00th=[   17], 20.00th=[   17],
     | 30.00th=[   18], 40.00th=[   20], 50.00th=[   25], 60.00th=[   27],
     | 70.00th=[   28], 80.00th=[   29], 90.00th=[   30], 95.00th=[   32],
     | 99.00th=[   41], 99.50th=[  239], 99.90th=[  306], 99.95th=[  318],
     | 99.99th=[  532]
    bw (KB  /s): min=474072, max=534064, per=99.97%, avg=508352.00, stdev=15619.31
    lat (usec) : 10=0.01%, 20=37.79%, 50=61.51%, 100=0.14%, 250=0.09%
    lat (usec) : 500=0.45%, 750=0.01%, 1000=0.01%
    lat (msec) : 2=0.01%, 4=0.01%, 10=0.01%
  cpu          : usr=24.59%, sys=68.86%, ctx=78124, majf=0, minf=85
  IO depths    : 1=0.1%, 2=0.1%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     issued    : total=r=0/w=2097152/d=0, short=r=0/w=0/d=0, drop=r=0/w=0/d=0
     latency   : target=0, window=0, percentile=100.00%, depth=4

Run status group 0 (all jobs):
  WRITE: io=8192.0MB, aggrb=508523KB/s, minb=508523KB/s, maxb=508523KB/s, mint=16496msec, maxt=16496msec

Disk stats (read/write):
  nvme0n1: ios=0/2085781, merge=0/3, ticks=0/21660, in_queue=21396, util=87.60%
j2@j2-pc:~$ sudo fio -filename=/home/j2/nvme/a.bin -direct=1 -iodepth 16 -thread -rw=write -ioengine=libaio -bs=4k -size=8G -numjobs=16 -group_reporting -name=mytest
mytest: (g=0): rw=write, bs=4K-4K/4K-4K/4K-4K, ioengine=libaio, iodepth=16
...
fio-2.2.10
Starting 16 threads
Jobs: 4 (f=4): [_(7),W(2),_(3),W(1),_(1),W(1),_(1)] [100.0% done] [0KB/863.4MB/0KB /s] [0/221K/0 iops] [eta 00m:00s]
mytest: (groupid=0, jobs=16): err= 0: pid=6169: Thu Dec 27 16:29:13 2018
  write: io=131072MB, bw=827886KB/s, iops=206971, runt=162121msec
    slat (usec): min=2, max=24581, avg=74.40, stdev=508.52
    clat (usec): min=1, max=38071, avg=1159.61, stdev=1982.52
     lat (usec): min=12, max=38289, avg=1234.12, stdev=2045.48
    clat percentiles (usec):
     |  1.00th=[  243],  5.00th=[  318], 10.00th=[  362], 20.00th=[  426],
     | 30.00th=[  478], 40.00th=[  532], 50.00th=[  588], 60.00th=[  652],
     | 70.00th=[  740], 80.00th=[  876], 90.00th=[ 1656], 95.00th=[ 5664],
     | 99.00th=[10304], 99.50th=[12480], 99.90th=[15296], 99.95th=[17280],
     | 99.99th=[21888]
    bw (KB  /s): min=32216, max=118512, per=6.26%, avg=51789.93, stdev=4685.88
    lat (usec) : 2=0.01%, 4=0.01%, 10=0.01%, 20=0.01%, 50=0.02%
    lat (usec) : 100=0.05%, 250=1.12%, 500=33.40%, 750=36.29%, 1000=13.40%
    lat (msec) : 2=6.59%, 4=2.00%, 10=5.99%, 20=1.12%, 50=0.02%
  cpu          : usr=1.65%, sys=53.62%, ctx=15358339, majf=0, minf=7521
  IO depths    : 1=0.1%, 2=0.1%, 4=0.1%, 8=0.1%, 16=100.0%, 32=0.0%, >=64=0.0%
     submit    : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.0%, 32=0.0%, 64=0.0%, >=64=0.0%
     complete  : 0=0.0%, 4=100.0%, 8=0.0%, 16=0.1%, 32=0.0%, 64=0.0%, >=64=0.0%
     issued    : total=r=0/w=33554432/d=0, short=r=0/w=0/d=0, drop=r=0/w=0/d=0
     latency   : target=0, window=0, percentile=100.00%, depth=16

Run status group 0 (all jobs):
  WRITE: io=131072MB, aggrb=827886KB/s, minb=827886KB/s, maxb=827886KB/s, mint=162121msec, maxt=162121msec

Disk stats (read/write):
  nvme0n1: ios=0/33547369, merge=0/31, ticks=0/1289400, in_queue=1316952, util=100.00%
j2@j2-pc:~$   
```

