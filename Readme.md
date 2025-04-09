# 借助 GitHub Actions 的 OpenWrt 在线自动编译

复刻仓库，点击 Actions 按钮，选择 HY3000-hanwckf，点击 Run workflow 按钮，等待编译完成即可。

原仓库 https://github.com/lgs2007m/Actions-OpenWrt

---

- hanwckf大佬mt798x闭源仓库- [hanwckf/immortalwrt-mt798x](https://github.com/hanwckf/immortalwrt-mt798x).

- 237大佬mt798x闭源仓库- [padavanonly/immortalwrt-mt798x](https://github.com/padavanonly/immortalwrt-mt798x).

- hanwckf大佬mt798x uboot仓库- [hanwckf/bl-mt798x](https://github.com/hanwckf/bl-mt798x).

- 刷砖也不怕！可以通过串口救砖：[MediaTek Filogic 系列路由器串口救砖教程](https://www.cnblogs.com/p123/p/18046679)

---

## HY3000 workflow 手动运行可选项：
- Set LAN IP Address
- Choose WiFi Driver
- [x] Use nx30pro eeprom and fixed WiFi MAC address
- [x] Use eMMC highspeed
- [ ] Use luci-app-mtk wifi config
- [ ] Not build luci-app-dockerman

- #### 说明

源码中的WAN、LAN地址顺序已修复

| 型号 | eMMC频率 | 备注 |
| ---- | -------- | ---- |
| HY3000M | 52MHz | 支持26MHz和52MHz频率 |

- #### 1. Set LAN IP Address

设置LAN IP地址（路由器登录地址），默认192.168.99.1。  

- #### 2. Choose WiFi Driver

默认使用WiFi驱动版本v7.6.7.2-fw-20240823(recommend)。  

[mtk-openwrt-feeds](https://git01.mediatek.com/plugins/gitiles/openwrt/feeds/mtk-openwrt-feeds/)

mt_wifi的firmware可选，warp默认使用v7.6.7.2配套的warp_20231229-5f71ec，firmware用驱动自带的，不可选。  

**【mt7981的机子上未测试，建议直接使用推荐的选项。】**

根据网络和ChatGPT查询，我理解：  

- mt_wifi 是 MediaTek 的 WiFi 驱动程序，主要用于控制无线网络功能，提供 WiFi 协议栈支持、无线电控制、连接管理等。  
- warp 是 MediaTek 的 WiFi Warp Accelerator 加速框架，通常用于 WiFi 网络数据处理的加速。

mt_wifi 和 warp 中的固件（firmware）是驱动程序与无线芯片之间的中间层。它通常被加载到无线芯片中，以控制其硬件功能，管理无线协议并处理数据传输，会影响无线性能和稳定性。  

| 驱动版本 | 来源 |
| -------- | ---- |
| v7.6.7.2-fw-20240823(recommend) | mtk-openwrt-feeds(20240823) |
| v7.6.6.1-fw-20230306(recommend) | H3C-NX30Pro固件fw-20230306 |
| v7.6.7.2-fw-default | 驱动包自带firmware fw-20231229 |
| v7.6.7.2-fw-20230306 | H3C-NX30Pro固件fw-20230306 |
| v7.6.7.2-fw-20230330 | TP-XDR3030固件fw-20230330 |
| v7.6.7.2-fw-20230411 | H3C-NX30Pro固件fw-20230411 |  
| v7.6.7.2-fw-20230717 | Xiaomi-AX3000T固件fw-20230717 | 
| v7.6.7.2-fw-20231024 | mtk-openwrt-feeds(20231024)的fw-20231024 | 
| v7.6.6.1-fw-default | firmware fw-20220906 |
| v7.6.6.1-fw-20230330 | TP-XDR3030固件fw-20230330 | 
| v7.6.6.1-fw-20230411 | H3C-NX30Pro固件fw-20230411 | 
| v7.6.6.1-fw-20230717 | Xiaomi-AX3000T固件fw-20230717 | 
| v7.6.6.1-fw-20231024 | mtk-openwrt-feeds(20231024)的fw-20231024 | 
| v7.6.6.1-fw-20240823 | mtk-openwrt-feeds(20240823) |

```bash
# SSH查看内核版本
uname -a
# 查看WiFi驱动版本
iwpriv ra0 get_driverinfo
# 查看WiFi驱动mt_wifi mt7981 firmware版本
strings /lib/firmware/7981_WACPU_RAM_CODE_release.bin | grep -E '202[0-9]{6}'
strings /lib/firmware/mt7981_patch_e1_hdr.bin | grep -E '202[0-9]{6}'
strings /lib/firmware/WIFI_RAM_CODE_MT7981.bin | grep -E '202[0-9]{6}'
# 查看WiFi驱动warp mt7981 firmware版本
strings /lib/firmware/7981_WOCPU0_RAM_CODE_release.bin | grep -E '202[0-9]{6}'
```

- #### 3. Use nx30pro eeprom and fixed WiFi MAC address

该选项默认开启，即使用 nx30pro 的高功率 eeprom，不需要请取消打钩。  
不使用独立 FEM 无线功放的 MT7981B 路由器可以通过替换高功率的 eeprom 提高信号强度。  

| 型号 | 2.4G功率 | 5G功率 |
| ---- | -------- | ------ |
| 无FEM | 23dBm | 22dBm |
| NX30 PRO | 25dBm | 24dBm |

开启该选项会使用NX30 PRO的eeprom替换掉固件中的`MT7981_iPAiLNA_EEPROM.bin`文件，并将facotry分区读取的MAC写入到dat以便固定WiFi MAC。  

- #### 4. Use luci-app-mtk wifi config

该选项默认关闭，即按`.mtwifi-cfg.config`配置文件，使用`mtwifi-cfg`配置工具，需要使用旧的luci-app-mtk无线配置工具请打钩。  

- mtwifi-cfg：为mtwifi设计的无线配置工具，兼容openwrt原生luci和netifd，可调整无线驱动的参数较少，配置界面美观友好。  

- luci-app-mtk：源自mtk-sdk提供的配置工具，需要配合wifi-profile脚本使用，可调整无线驱动的几乎所有参数，配置界面较为简陋。  

区别详见大佬的博客[mtwifi无线配置工具说明](https://cmi.hanwckf.top/p/immortalwrt-mt798x/#mtwifi%E6%97%A0%E7%BA%BF%E9%85%8D%E7%BD%AE%E5%B7%A5%E5%85%B7%E8%AF%B4%E6%98%8E) 

`.mtwifi-cfg.config`配置文件中已设置使用mtwifi-cfg配置工具：  

```ini
CONFIG_PACKAGE_luci-app-mtwifi-cfg=y
CONFIG_PACKAGE_luci-i18n-mtwifi-cfg-zh-cn=y
CONFIG_PACKAGE_mtwifi-cfg=y
CONFIG_PACKAGE_lua-cjson=y
```

- #### 5. Not build luci-app-dockerman

该选项默认关闭，即按`.mtwifi-cfg.config`配置文件编译 dockerman，不需要编译 dockerman 请打钩。  
`.mtwifi-cfg.config`配置文件中已设置编译 dockerman：

```ini
CONFIG_PACKAGE_luci-app-dockerman=y  
```

- #### 6. Use eMMC highspeed

该选项默认启用，即使用eMMC的highspeed模式，默认 52MHz.

关闭后则不使用eMMC的highspeed模式，使用26MHz。

该功能未测试，保持默认即可。

---

### 感谢P3TERX的Actions-OpenWrt

- [P3TERX](https://github.com/P3TERX/Actions-OpenWrt)
[Read the details in my blog (in Chinese) | 中文教程](https://p3terx.com/archives/build-openwrt-with-github-actions.html)
