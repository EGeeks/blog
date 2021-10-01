# 作者
QQ群：**852283276**
微信：**arm80x86**
微信公众号：**青儿创客基地**
B站：[主页 `https://space.bilibili.com/208826118`](https://space.bilibili.com/208826118)

# 参考
> [phy基础知识总结 common register总结](https://blog.csdn.net/wuheshi/article/details/79085546)
> [RTL8211F WOL(wake on lan)调试](https://blog.csdn.net/GCQ19961204/article/details/108549856)

# mdio
网卡驱动或者独立的mdio驱动，首先会注册到系统总线，然后开始扫描总线上的PHY芯片，
```bash
mdiobus_register
  mdiobus_scan
    get_phy_device
      get_phy_id
      phy_device_create
    phy_device_register
      phy_scan_fixups
```
在`phy_device_create`中，注意phy_device成员变量的初始值，
```c
struct phy_device *phy_device_create(struct mii_bus *bus, int addr, int phy_id,
				     bool is_c45,
				     struct phy_c45_device_ids *c45_ids)
{
	struct phy_device *dev;
...
	dev->speed = 0;
	dev->duplex = -1;
	dev->pause = 0;
	dev->asym_pause = 0;
	dev->link = 1;
	dev->interface = PHY_INTERFACE_MODE_GMII;

	dev->autoneg = AUTONEG_ENABLE;
...
}
```

注意`phy_scan_fixups`可以对phy进行一些预设，可以在MAC驱动里注册fixup函数，在这儿就会进行调用，linux提供了两种注册方法，`phy_register_fixup_for_uid`，例如，
```c
if (IS_BUILTIN(CONFIG_PHYLIB))
	phy_register_fixup_for_uid(PHY_ID_AR8031, 0xffffffff,
				   ar8031_phy_fixup);
```
第二种，`phy_register_fixup_for_id`，例如，
```c
if (of_machine_is_compatible("atmel,sama5d4ek") &&
   IS_ENABLED(CONFIG_PHYLIB)) {
	phy_register_fixup_for_id("fc028000.etherne:00",
					ksz8081_phy_fixup);
}
```

# phy
注意`phy_scan_fixups`在`config_init`之前被调用，网卡驱动会调用`of_phy_connect`函数，
```bash
of_phy_connect
  phy_connect_direct
    phy_attach_direct
      phydev->state = PHY_READY;
      phy_init_hw
        phydev->drv->soft_reset
        phy_scan_fixups
        phydev->drv->config_init
      phy_resume
        phydrv->resume
    phy_prepare_link
    phy_start_machine
    phy_start_interrupts
```
网卡驱动会调用`phy_start`函数，执行`phydev->state = PHY_UP`，
```cpp
void phy_start(struct phy_device *phydev)
{
	mutex_lock(&phydev->lock);
	
	switch (phydev->state) {
	case PHY_STARTING:
		phydev->state = PHY_PENDING;
		break;
	case PHY_READY:
		phydev->state = PHY_UP;
		break;
	case PHY_HALTED:
		phydev->state = PHY_RESUMING;
	default:
		break;
	}
	mutex_unlock(&phydev->lock);
}
```
phy状态机函数，进入`PHY_UP`后，设置`needs_aneg`，开启自协商，
```cpp
void phy_state_machine(struct work_struct *work)
{
...
	switch (phydev->state) {
	case PHY_DOWN:
	case PHY_STARTING:
	case PHY_READY:
	case PHY_PENDING:
		break;
	case PHY_UP:
		needs_aneg = true;

		phydev->link_timeout = PHY_AN_TIMEOUT;

		break;
...
	if (needs_aneg)
		err = phy_start_aneg(phydev);
}
```
`phy_start_aneg`，默认值`AUTONEG_ENABLE`，
```cpp
phy_start_aneg
  phydev->drv->config_aneg
  phydev->state = PHY_AN;
```
进入`PHY_AN`后，
```cpp
phy_read_status(phydev);
  phydev->drv->read_status
if (!phydev->link) {
	phydev->state = PHY_NOLINK;
	netif_carrier_off(phydev->attached_dev);
	phydev->adjust_link(phydev->attached_dev);
	break;
}
err = phy_aneg_done(phydev);
if (err < 0)
	break;

if (err > 0) {
	phydev->state = PHY_RUNNING;
	netif_carrier_on(phydev->attached_dev);
	phydev->adjust_link(phydev->attached_dev);
} else if (0 == phydev->link_timeout--)
	needs_aneg = true;
break;
```
网卡驱动通过`of_phy_connect`来连接phy，也可以通过`phy_find_first`自动查找到总线上的第一个phy设备，然后调用`phy_connect`。
```cpp
ftgmac100_mii_probe
  //drivers\net\phy\phy_device.c
  phy_find_first
  phy_connect
  phy_attached_info
```
phy读写寄存器函数，c22/c45函数，
```cpp
//include\linux\phy.h
/**
 * phy_read - Convenience function for reading a given PHY register
 * @phydev: the phy_device struct
 * @regnum: register number to read
 *
 * NOTE: MUST NOT be called from interrupt context,
 * because the bus read/write functions may wait for an interrupt
 * to conclude the operation.
 */
static inline int phy_read(struct phy_device *phydev, u32 regnum)
{
	return mdiobus_read(phydev->mdio.bus, phydev->mdio.addr, regnum);
}

/**
 * phy_write - Convenience function for writing a given PHY register
 * @phydev: the phy_device struct
 * @regnum: register number to write
 * @val: value to write to @regnum
 *
 * NOTE: MUST NOT be called from interrupt context,
 * because the bus read/write functions may wait for an interrupt
 * to conclude the operation.
 */
static inline int phy_write(struct phy_device *phydev, u32 regnum, u16 val)
{
	return mdiobus_write(phydev->mdio.bus, phydev->mdio.addr, regnum, val);
}
```

# ethtool设置网卡参数
```bash
$ ethtool -s eth0 speed 100 duplex full autoneg off
```
驱动通过`ethtool_ops`来执行命令，
```cpp
static struct ethtool_ops xemacps_ethtool_ops = {
	.get_settings   = xemacps_get_settings,
	.set_settings   = xemacps_set_settings,
	.get_drvinfo    = xemacps_get_drvinfo,
	.get_link       = ethtool_op_get_link, /* ethtool default */
	.get_ringparam  = xemacps_get_ringparam,
	.get_wol        = xemacps_get_wol,
	.set_wol        = xemacps_set_wol,
	.get_pauseparam = xemacps_get_pauseparam,
	.set_pauseparam = xemacps_set_pauseparam,
#ifdef CONFIG_XILINX_PS_EMAC_HWTSTAMP
	.get_ts_info    = xemacps_get_ts_info,
#endif
};
```
函数`xemacps_set_settings`，
```cpp
xemacps_set_settings
  phy_ethtool_sset
    phy_start_aneg
```
函数`phy_ethtool_sset`，
```cpp
int phy_ethtool_sset(struct phy_device *phydev, struct ethtool_cmd *cmd)
{
	u32 speed = ethtool_cmd_speed(cmd);

	if (cmd->phy_address != phydev->addr)
		return -EINVAL;

	/* We make sure that we don't pass unsupported values in to the PHY */
	cmd->advertising &= phydev->supported;

	/* Verify the settings we care about. */
	if (cmd->autoneg != AUTONEG_ENABLE && cmd->autoneg != AUTONEG_DISABLE)
		return -EINVAL;

	if (cmd->autoneg == AUTONEG_ENABLE && cmd->advertising == 0)
		return -EINVAL;

	if (cmd->autoneg == AUTONEG_DISABLE &&
	    ((speed != SPEED_1000 &&
	      speed != SPEED_100 &&
	      speed != SPEED_10) ||
	     (cmd->duplex != DUPLEX_HALF &&
	      cmd->duplex != DUPLEX_FULL)))
		return -EINVAL;

	phydev->autoneg = cmd->autoneg;

	phydev->speed = speed;

	phydev->advertising = cmd->advertising;

	if (AUTONEG_ENABLE == cmd->autoneg)
		phydev->advertising |= ADVERTISED_Autoneg;
	else
		phydev->advertising &= ~ADVERTISED_Autoneg;

	phydev->duplex = cmd->duplex;

	/* Restart the PHY */
	phy_start_aneg(phydev);

	return 0;
}
```

# genphy驱动
在函数`genphy_config_init`中通过`MII_ESTATUS`确定是否支持`SUPPORTED_1000baseT_Full`，在函数`phy_probe`中`phydev->supported = phydrv->features;`，同时`phydev->advertising = phydev->supported;`，
```c
//drivers\net\phy\phy_device.c
int genphy_config_init(struct phy_device *phydev)
{
	int val;
	u32 features;

	features = (SUPPORTED_TP | SUPPORTED_MII
			| SUPPORTED_AUI | SUPPORTED_FIBRE |
			SUPPORTED_BNC);

	/* Do we support autonegotiation? */
	val = phy_read(phydev, MII_BMSR);
	if (val < 0)
		return val;

	if (val & BMSR_ANEGCAPABLE)
		features |= SUPPORTED_Autoneg;

	if (val & BMSR_100FULL)
		features |= SUPPORTED_100baseT_Full;
	if (val & BMSR_100HALF)
		features |= SUPPORTED_100baseT_Half;
	if (val & BMSR_10FULL)
		features |= SUPPORTED_10baseT_Full;
	if (val & BMSR_10HALF)
		features |= SUPPORTED_10baseT_Half;

	if (val & BMSR_ESTATEN) {
		val = phy_read(phydev, MII_ESTATUS);
		if (val < 0)
			return val;

		if (val & ESTATUS_1000_TFULL)
			features |= SUPPORTED_1000baseT_Full;
		if (val & ESTATUS_1000_THALF)
			features |= SUPPORTED_1000baseT_Half;
	}

	phydev->supported &= features;
	phydev->advertising &= features;

	return 0;
}
//drivers\net\phy\phy_device.c
static int phy_probe(struct device *dev)
{
	struct phy_device *phydev = to_phy_device(dev);
	struct device_driver *drv = phydev->dev.driver;
	struct phy_driver *phydrv = to_phy_driver(drv);
	int err = 0;

	phydev->drv = phydrv;

	/* Disable the interrupt if the PHY doesn't support it
	 * but the interrupt is still a valid one
	 */
	if (!(phydrv->flags & PHY_HAS_INTERRUPT) &&
	    phy_interrupt_is_valid(phydev))
		phydev->irq = PHY_POLL;

	if (phydrv->flags & PHY_IS_INTERNAL)
		phydev->is_internal = true;

	mutex_lock(&phydev->lock);

	/* Start out supporting everything. Eventually,
	 * a controller will attach, and may modify one
	 * or both of these values
	 */
	phydev->supported = phydrv->features;
	of_set_phy_supported(phydev);
	phydev->advertising = phydev->supported;

	/* Set the state to READY by default */
	phydev->state = PHY_READY;

	if (phydev->drv->probe)
		err = phydev->drv->probe(phydev);

	mutex_unlock(&phydev->lock);

	return err;
}
//drivers\net\phy\phy_device.c
static struct phy_driver genphy_driver[] = {
{
	.phy_id		= 0xffffffff,
	.phy_id_mask	= 0xffffffff,
	.name		= "Generic PHY",
	.soft_reset	= genphy_soft_reset,
	.config_init	= genphy_config_init,
	.features	= PHY_GBIT_FEATURES | SUPPORTED_MII |
			  SUPPORTED_AUI | SUPPORTED_FIBRE |
			  SUPPORTED_BNC,
	.config_aneg	= genphy_config_aneg,
	.aneg_done	= genphy_aneg_done,
	.read_status	= genphy_read_status,
	.suspend	= genphy_suspend,
	.resume		= genphy_resume,
	.driver		= { .owner = THIS_MODULE, },
}, {
	.phy_id         = 0xffffffff,
	.phy_id_mask    = 0xffffffff,
	.name           = "Generic 10G PHY",
	.soft_reset	= gen10g_soft_reset,
	.config_init    = gen10g_config_init,
	.features       = 0,
	.config_aneg    = gen10g_config_aneg,
	.read_status    = gen10g_read_status,
	.suspend        = gen10g_suspend,
	.resume         = gen10g_resume,
	.driver         = {.owner = THIS_MODULE, },
} };
//include\linux\phy.h
#define PHY_DEFAULT_FEATURES	(SUPPORTED_Autoneg | \
				 SUPPORTED_TP | \
				 SUPPORTED_MII)

#define PHY_10BT_FEATURES	(SUPPORTED_10baseT_Half | \
				 SUPPORTED_10baseT_Full)

#define PHY_100BT_FEATURES	(SUPPORTED_100baseT_Half | \
				 SUPPORTED_100baseT_Full)

#define PHY_1000BT_FEATURES	(SUPPORTED_1000baseT_Half | \
				 SUPPORTED_1000baseT_Full)

#define PHY_BASIC_FEATURES	(PHY_10BT_FEATURES | \
				 PHY_100BT_FEATURES | \
				 PHY_DEFAULT_FEATURES)

#define PHY_GBIT_FEATURES	(PHY_BASIC_FEATURES | \
				 PHY_1000BT_FEATURES)
```
通过`genphy_read_status`确定链路状态，
```c
int genphy_read_status(struct phy_device *phydev)
{
	int adv;
	int err;
	int lpa;
	int lpagb = 0;
	int common_adv;
	int common_adv_gb = 0;

	/* Update the link, but return if there was an error */
	err = genphy_update_link(phydev);
	if (err)
		return err;

	phydev->lp_advertising = 0;

	if (AUTONEG_ENABLE == phydev->autoneg) {
		if (phydev->supported & (SUPPORTED_1000baseT_Half
					| SUPPORTED_1000baseT_Full)) {
			lpagb = phy_read(phydev, MII_STAT1000);
			if (lpagb < 0)
				return lpagb;

			adv = phy_read(phydev, MII_CTRL1000);
			if (adv < 0)
				return adv;

			phydev->lp_advertising =
				mii_stat1000_to_ethtool_lpa_t(lpagb);
			common_adv_gb = lpagb & adv << 2;
		}

		lpa = phy_read(phydev, MII_LPA);
		if (lpa < 0)
			return lpa;

		phydev->lp_advertising |= mii_lpa_to_ethtool_lpa_t(lpa);

		adv = phy_read(phydev, MII_ADVERTISE);
		if (adv < 0)
			return adv;

		common_adv = lpa & adv;

		phydev->speed = SPEED_10;
		phydev->duplex = DUPLEX_HALF;
		phydev->pause = 0;
		phydev->asym_pause = 0;

		if (common_adv_gb & (LPA_1000FULL | LPA_1000HALF)) {
			phydev->speed = SPEED_1000;

			if (common_adv_gb & LPA_1000FULL)
				phydev->duplex = DUPLEX_FULL;
		} else if (common_adv & (LPA_100FULL | LPA_100HALF)) {
			phydev->speed = SPEED_100;

			if (common_adv & LPA_100FULL)
				phydev->duplex = DUPLEX_FULL;
		} else
			if (common_adv & LPA_10FULL)
				phydev->duplex = DUPLEX_FULL;

		if (phydev->duplex == DUPLEX_FULL) {
			phydev->pause = lpa & LPA_PAUSE_CAP ? 1 : 0;
			phydev->asym_pause = lpa & LPA_PAUSE_ASYM ? 1 : 0;
		}
	} else {
		int bmcr = phy_read(phydev, MII_BMCR);

		if (bmcr < 0)
			return bmcr;

		if (bmcr & BMCR_FULLDPLX)
			phydev->duplex = DUPLEX_FULL;
		else
			phydev->duplex = DUPLEX_HALF;

		if (bmcr & BMCR_SPEED1000)
			phydev->speed = SPEED_1000;
		else if (bmcr & BMCR_SPEED100)
			phydev->speed = SPEED_100;
		else
			phydev->speed = SPEED_10;

		phydev->pause = 0;
		phydev->asym_pause = 0;
	}

	return 0;
}
EXPORT_SYMBOL(genphy_read_status);
```

