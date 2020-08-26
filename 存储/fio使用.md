# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [fio doc](https://fio.readthedocs.io/en/latest/fio_doc.html#)
> [从理论到实践，异步I/O模式下NVMe SSD高性能之道](https://blog.csdn.net/Memblaze_2011/article/details/90756131)

# help
help内容，
```shell
j2@j2-pc:~$ fio --help
fio-2.2.10
fio [options] [job options] <job file(s)>
  --debug=options       Enable debug logging. May be one/more of:
                        process,file,io,mem,blktrace,verify,random,parse,
                        diskutil,job,mutex,profile,time,net,rate,compress
  --parse-only          Parse options only, don't start any IO
  --output              Write output to file
  --runtime             Runtime in seconds
  --bandwidth-log       Generate per-job bandwidth logs
  --minimal             Minimal (terse) output
  --output-format=x     Output format (terse,json,normal)
  --terse-version=x     Set terse version output format to 'x'
  --version             Print version info and exit
  --help                Print this page
  --cpuclock-test       Perform test/validation of CPU clock
  --crctest             Test speed of checksum functions
  --cmdhelp=cmd         Print command help, "all" for all of them
  --enghelp=engine      Print ioengine help, or list available ioengines
  --enghelp=engine,cmd  Print help for an ioengine cmd
  --showcmd             Turn a job file into command line options
  --eta=when            When ETA estimate should be printed
                        May be "always", "never" or "auto"
  --eta-newline=time    Force a new line for every 'time' period passed
  --status-interval=t   Force full status dump every 't' period passed
  --readonly            Turn on safety read-only checks, preventing writes
  --section=name        Only run specified section in job file
  --alloc-size=kb       Set smalloc pool to this size in kb (def 1024)
  --warnings-fatal      Fio parser warnings are fatal
  --max-jobs=nr         Maximum number of threads/processes to support
  --server=args         Start a backend fio server
  --daemonize=pidfile   Background fio server, write pid to file
  --client=hostname     Talk to remote backend fio server at hostname
  --remote-config=file  Tell fio server to load this local job file
  --idle-prof=option    Report cpu idleness on a system or percpu basis
                        (option=system,percpu) or run unit work
                        calibration only (option=calibrate)
  --inflate-log=log     Inflate and output compressed log
  --trigger-file=file   Execute trigger cmd when file exists
  --trigger-timeout=t   Execute trigger af this time
  --trigger=cmd         Set this command as local trigger
  --trigger-remote=cmd  Set this command as remote trigger
  --aux-path=path       Use this path for fio state generated files

Fio was written by Jens Axboe <jens.axboe@oracle.com>
                   Jens Axboe <jaxboe@fusionio.com>
                   Jens Axboe <axboe@fb.com>
```
官网注释比较详细，
```shell
--debug=type
Enable verbose tracing type of various fio actions. May be all for all types or individual types separated by a comma (e.g. --debug=file,mem will enable file and memory debugging). Currently, additional logging is available for:

process
Dump info related to processes.
file
Dump info related to file actions.
io
Dump info related to I/O queuing.
mem
Dump info related to memory allocations.
blktrace
Dump info related to blktrace setup.
verify
Dump info related to I/O verification.
all
Enable all debug options.
random
Dump info related to random offset generation.
parse
Dump info related to option matching and parsing.
diskutil
Dump info related to disk utilization updates.
job:x
Dump info only related to job number x.
mutex
Dump info only related to mutex up/down ops.
profile
Dump info related to profile extensions.
time
Dump info related to internal time keeping.
net
Dump info related to networking connections.
rate
Dump info related to I/O rate switching.
compress
Dump info related to log compress/decompress.
? or help
Show available debug options.
--parse-only
Parse options only, don’t start any I/O.

--merge-blktrace-only
Merge blktraces only, don’t start any I/O.

--output=filename
Write output to file filename.

--output-format=format
Set the reporting format to normal, terse, json, or json+. Multiple formats can be selected, separated by a comma. terse is a CSV based format. json+ is like json, except it adds a full dump of the latency buckets.

--bandwidth-log
Generate aggregate bandwidth logs.

--minimal
Print statistics in a terse, semicolon-delimited format.

--append-terse
Print statistics in selected mode AND terse, semicolon-delimited format. Deprecated, use --output-format instead to select multiple formats.

--terse-version=version
Set terse version output format (default 3, or 2 or 4 or 5).

--version
Print version information and exit.

--help
Print a summary of the command line options and exit.

--cpuclock-test
Perform test and validation of internal CPU clock.

--crctest=[test]
Test the speed of the built-in checksumming functions. If no argument is given, all of them are tested. Alternatively, a comma separated list can be passed, in which case the given ones are tested.

--cmdhelp=command
Print help information for command. May be all for all commands.

--enghelp=[ioengine[,command]]
List all commands defined by ioengine, or print help for command defined by ioengine. If no ioengine is given, list all available ioengines.

--showcmd=jobfile
Convert jobfile to a set of command-line options.

--readonly
Turn on safety read-only checks, preventing writes and trims. The --readonly option is an extra safety guard to prevent users from accidentally starting a write or trim workload when that is not desired. Fio will only modify the device under test if rw=write/randwrite/rw/randrw/trim/randtrim/trimwrite is given. This safety net can be used as an extra precaution.

--eta=when
Specifies when real-time ETA estimate should be printed. when may be always, never or auto. auto is the default, it prints ETA when requested if the output is a TTY. always disregards the output type, and prints ETA when requested. never never prints ETA.

--eta-interval=time
By default, fio requests client ETA status roughly every second. With this option, the interval is configurable. Fio imposes a minimum allowed time to avoid flooding the console, less than 250 msec is not supported.

--eta-newline=time
Force a new line for every time period passed. When the unit is omitted, the value is interpreted in seconds.

--status-interval=time
Force a full status dump of cumulative (from job start) values at time intervals. This option does not provide per-period measurements. So values such as bandwidth are running averages. When the time unit is omitted, time is interpreted in seconds. Note that using this option with --output-format=json will yield output that technically isn’t valid json, since the output will be collated sets of valid json. It will need to be split into valid sets of json after the run.

--section=name
Only run specified section name in job file. Multiple sections can be specified. The --section option allows one to combine related jobs into one file. E.g. one job file could define light, moderate, and heavy sections. Tell fio to run only the “heavy” section by giving --section=heavy command line option. One can also specify the “write” operations in one section and “verify” operation in another section. The --section option only applies to job sections. The reserved global section is always parsed and used.

--alloc-size=kb
Set the internal smalloc pool size to kb in KiB. The --alloc-size switch allows one to use a larger pool size for smalloc. If running large jobs with randommap enabled, fio can run out of memory. Smalloc is an internal allocator for shared structures from a fixed size memory pool and can grow to 16 pools. The pool size defaults to 16MiB.

NOTE: While running .fio_smalloc.* backing store files are visible in /tmp.

--warnings-fatal
All fio parser warnings are fatal, causing fio to exit with an error.

--max-jobs=nr
Set the maximum number of threads/processes to support to nr. NOTE: On Linux, it may be necessary to increase the shared-memory limit (/proc/sys/kernel/shmmax) if fio runs into errors while creating jobs.

--server=args
Start a backend server, with args specifying what to listen to. See Client/Server section.

--daemonize=pidfile
Background a fio server, writing the pid to the given pidfile file.

--client=hostname
Instead of running the jobs locally, send and run them on the given hostname or set of hostnames. See Client/Server section.

--remote-config=file
Tell fio server to load this local file.

--idle-prof=option
Report CPU idleness. option is one of the following:

calibrate
Run unit work calibration only and exit.
system
Show aggregate system idleness and unit work.
percpu
As system but also show per CPU idleness.
--inflate-log=log
Inflate and output compressed log.

--trigger-file=file
Execute trigger command when file exists.

--trigger-timeout=time
Execute trigger at this time.

--trigger=command
Set this command as local trigger.

--trigger-remote=command
Set this command as remote trigger.

--aux-path=path
Use the directory specified by path for generated state files instead of the current working directory.

Any parameters following the options will be assumed to be job files, unless they match a job file parameter. Multiple job files can be listed and each job file will be regarded as a separate group. Fio will stonewall execution between each group.
```

# job file
job file的后缀名一般为.fio，
```shell
; -- start job file --
[global]
rw=randread
size=128m

[job1]

[job2]

; -- end job file --
```
global的参数用于所以job，job可覆盖global的参数，等同于命令，--showcmd=jobfile可将jobfile转换成命令
```shell
$ fio --name=global --rw=randread --size=128m --name=job1 --name=job2
```
Let’s look at an example that has a number of processes writing randomly to files:

```shell
; -- start job file --
[random-writers]
ioengine=libaio
iodepth=4
rw=randwrite
bs=32k
direct=0
size=64m
numjobs=4
; -- end job file --
```
等同于命令，
```shell
$ fio --name=random-writers --ioengine=libaio --iodepth=4 --rw=randwrite --bs=32k --direct=0 --size=64m --numjobs=4
```
jobfile可支持include包含，
```shell
; -- start job file including.fio --
[global]
filename=/tmp/test
filesize=1m
include glob-include.fio

[test]
rw=randread
bs=4k
time_based=1
runtime=10
include test-include.fio
; -- end job file including.fio --
; -- start job file glob-include.fio --
thread=1
group_reporting=1
; -- end job file glob-include.fio --
; -- start job file test-include.fio --
ioengine=libaio
iodepth=4
; -- end job file test-include.fio --
```

