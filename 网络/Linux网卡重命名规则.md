# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [linux网卡命名规则](https://blog.csdn.net/hzj_001/article/details/81587824)
> [Linux重命名网卡名称](https://www.cnblogs.com/feiquan/p/9228066.html)
> [Linux网卡命名enp3s0说明](https://blog.csdn.net/weixin_34123613/article/details/89390993)
> [Linux Ubuntu 修改网卡名字](https://blog.csdn.net/u011521019/article/details/70218642)
> [Predictable Network Interface Names](https://www.freedesktop.org/wiki/Software/systemd/PredictableNetworkInterfaceNames/)
> [systemd/src/udev/udev-builtin-net_id.c](https://github.com/systemd/systemd/blob/master/src/udev/udev-builtin-net_id.c#L20)
> [redhat修改网卡名称](https://www.cnblogs.com/mountain2011/p/9098741.html)
> [linux修改网卡名称（一般修改为eth0）（redHat7）](https://blog.csdn.net/baobingji/article/details/84557185)
> [centos7/redhat7更改网卡名称为eth0](https://blog.csdn.net/feinifi/article/details/77883461)
> [Linux系统修改网卡名称（eth1修改为eth0）](https://www.linuxidc.com/Linux/2018-08/153407.htm)

# ubuntu
```bash
$ sudo vim /etc/default/grub
GRUB_CMDLINE_LINUX="net.ifnames=0 biosdevname=0"
$ sudo apt-get install grub2-common
$ sudo update-grub
$ sudo vim /etc/network/interfaces
```
另外一种需要编辑`/etc/udev/rules.d/70-persistent-net.rules`，例如，`ATTR{address}=="bc:30:5b:9c:ae:79"` 表示MAC地址，`KERNEL=="eth*"` 是原网卡名，`NAME="eth0"` 更改网卡名。
```bash
SUBSYSTEM=="net", ACTION=="add", DRIVERS=="?*", ATTR{address}=="bc:30:5b:b1:cd:be", ATTR{dev_id}=="0x0", ATTR{type}=="1", KERNEL=="eth*", NAME="eth0"
```
systemd，
```c
/* retrieve on-board index number and label from firmware */
static int dev_pci_onboard(sd_device *dev, struct netnames *names) {
        unsigned long idx, dev_port = 0;
        const char *attr, *port_name = NULL;
        size_t l;
        char *s;
        int r;

        /* ACPI _DSM — device specific method for naming a PCI or PCI Express device */
        if (sd_device_get_sysattr_value(names->pcidev, "acpi_index", &attr) < 0) {
                /* SMBIOS type 41 — Onboard Devices Extended Information */
                r = sd_device_get_sysattr_value(names->pcidev, "index", &attr);
                if (r < 0)
                        return r;
        }

        r = safe_atolu(attr, &idx);
        if (r < 0)
                return r;
        if (idx == 0 && !naming_scheme_has(NAMING_ZERO_ACPI_INDEX))
                return -EINVAL;

        /* Some BIOSes report rubbish indexes that are excessively high (2^24-1 is an index VMware likes to
         * report for example). Let's define a cut-off where we don't consider the index reliable anymore. We
         * pick some arbitrary cut-off, which is somewhere beyond the realistic number of physical network
         * interface a system might have. Ideally the kernel would already filter his crap for us, but it
         * doesn't currently. */
        if (idx > ONBOARD_INDEX_MAX)
                return -ENOENT;

        /* kernel provided port index for multiple ports on a single PCI function */
        if (sd_device_get_sysattr_value(dev, "dev_port", &attr) >= 0)
                dev_port = strtoul(attr, NULL, 10);

        /* kernel provided front panel port name for multiple port PCI device */
        (void) sd_device_get_sysattr_value(dev, "phys_port_name", &port_name);

        s = names->pci_onboard;
        l = sizeof(names->pci_onboard);
        l = strpcpyf(&s, l, "o%lu", idx);
        if (port_name)
                l = strpcpyf(&s, l, "n%s", port_name);
        else if (dev_port > 0)
                l = strpcpyf(&s, l, "d%lu", dev_port);
        if (l == 0)
                names->pci_onboard[0] = '\0';

        if (sd_device_get_sysattr_value(names->pcidev, "label", &names->pci_onboard_label) < 0)
                names->pci_onboard_label = NULL;

        return 0;
}
```

