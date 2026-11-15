# Project Overview: Realtek RTL8723DE Linux Kernel Wireless Driver (`rtl8723de`)

## Purpose and Hardware Target

`rtl8723de` is the Linux kernel wireless LAN and Bluetooth coexistence driver module (`5.1.1.8_21285.20171026_COEX20170111-1414`) for the Realtek RTL8723DE 802.11b/g/n PCIe network adapter (PCI device ID `10ec:d723`). It builds as an out-of-tree kernel module (`rtl8723de.ko`) via Kbuild or Dynamic Kernel Module Support (`dkms.conf`).

## Driver Architecture and Subsystem Boundaries

### 1. Core 802.11 Stack (`core/`)
- **MLME and Station Management**: [core/rtw_mlme.c](file:///botworkspace/rtl8723de/core/rtw_mlme.c), [core/rtw_mlme_ext.c](file:///botworkspace/rtl8723de/core/rtw_mlme_ext.c), [core/rtw_sta_mgt.c](file:///botworkspace/rtl8723de/core/rtw_sta_mgt.c), [core/rtw_ap.c](file:///botworkspace/rtl8723de/core/rtw_ap.c), and [core/rtw_ieee80211.c](file:///botworkspace/rtl8723de/core/rtw_ieee80211.c) implement 802.11 beacon parsing, authentication, association, station state tables, and soft-AP mode.
- **Transmit and Receive Data Paths**: [core/rtw_xmit.c](file:///botworkspace/rtl8723de/core/rtw_xmit.c) and [core/rtw_recv.c](file:///botworkspace/rtl8723de/core/rtw_recv.c) manage frame aggregation (A-MPDU / A-MSDU), sequence numbering, defragmentation, and DMA ring buffer descriptor preparation.
- **Cryptographic Security**: [core/rtw_security.c](file:///botworkspace/rtl8723de/core/rtw_security.c), [core/rtw_wapi.c](file:///botworkspace/rtl8723de/core/rtw_wapi.c), and [core/rtw_wapi_sms4.c](file:///botworkspace/rtl8723de/core/rtw_wapi_sms4.c) implement WEP, TKIP, AES-CCMP (WPA/WPA2), and WAPI SMS4 frame encryption/decryption.
- **Power Saving, RF, and Bluetooth Coexistence**: [core/rtw_pwrctrl.c](file:///botworkspace/rtl8723de/core/rtw_pwrctrl.c), [core/rtw_rf.c](file:///botworkspace/rtl8723de/core/rtw_rf.c), [core/rtw_btcoex.c](file:///botworkspace/rtl8723de/core/rtw_btcoex.c), [core/rtw_p2p.c](file:///botworkspace/rtl8723de/core/rtw_p2p.c), [core/rtw_tdls.c](file:///botworkspace/rtl8723de/core/rtw_tdls.c), and [core/efuse/rtw_efuse.c](file:///botworkspace/rtl8723de/core/efuse/rtw_efuse.c) handle IPS/LPS power states, eFuse MAC/calibration reading, Wi-Fi Direct, and shared-antenna Bluetooth traffic arbitration.

### 2. Hardware Abstraction Layer (`hal/`)
- **HAL Common and HALMAC**: [hal/hal_intf.c](file:///botworkspace/rtl8723de/hal/hal_intf.c), [hal/hal_com.c](file:///botworkspace/rtl8723de/hal/hal_com.c), [hal/hal_com_phycfg.c](file:///botworkspace/rtl8723de/hal/hal_com_phycfg.c), [hal/hal_phy.c](file:///botworkspace/rtl8723de/hal/hal_phy.c), [hal/hal_dm.c](file:///botworkspace/rtl8723de/hal/hal_dm.c), and [hal/hal_halmac.c](file:///botworkspace/rtl8723de/hal/hal_halmac.c) provide hardware register programming, MAC/BB/RF initialization sequences ([hal/HalPwrSeqCmd.c](file:///botworkspace/rtl8723de/hal/HalPwrSeqCmd.c)), and firmware C2H/H2C mailbox commands.
- **RTL8723DE Chip-Specific Driver (`hal/rtl8723d/`)**: Implements RTL8723DE PCIe DMA descriptor handling, 8051 firmware loading, baseband/RF calibration, and single/dual-antenna selection (`ant_sel`).
- **PHY Dynamic Mechanism (`hal/phydm/`)**: [hal/phydm/phydm.c](file:///botworkspace/rtl8723de/hal/phydm/phydm.c), [phydm_dig.c](file:///botworkspace/rtl8723de/hal/phydm/phydm_dig.c), [phydm_antdiv.c](file:///botworkspace/rtl8723de/hal/phydm/phydm_antdiv.c), [phydm_rainfo.c](file:///botworkspace/rtl8723de/hal/phydm/phydm_rainfo.c), [phydm_cfotracking.c](file:///botworkspace/rtl8723de/hal/phydm/phydm_cfotracking.c), and [phydm_noisemonitor.c](file:///botworkspace/rtl8723de/hal/phydm/phydm_noisemonitor.c) implement adaptive Initial Gain (DIG), rate adaptation, carrier frequency offset tracking, and antenna diversity.
- **Bluetooth Coexistence Algorithms (`hal/btc/`)**: Chip-specific 1-antenna and 2-antenna TDD/FDD coexistence state machines.

### 3. Linux OS Dependency Layer (`os_dep/`, `platform/`, `include/`)
- [os_dep/osdep_service.c](file:///botworkspace/rtl8723de/os_dep/osdep_service.c) and [os_dep/linux/os_intfs.c](file:///botworkspace/rtl8723de/os_dep/linux/os_intfs.c): Kernel module initialization (`module_init`/`module_exit`), `net_device` registration, kernel threads, spinlocks, and timers.
- [os_dep/linux/pci_intf.c](file:///botworkspace/rtl8723de/os_dep/linux/pci_intf.c) and [pci_ops_linux.c](file:///botworkspace/rtl8723de/os_dep/linux/pci_ops_linux.c): Linux `pci_driver` probe/remove, IRQ registration, and PCIe MMIO/DMA mapping.
- [os_dep/linux/ioctl_cfg80211.c](file:///botworkspace/rtl8723de/os_dep/linux/ioctl_cfg80211.c) and [ioctl_linux.c](file:///botworkspace/rtl8723de/os_dep/linux/ioctl_linux.c): Linux `cfg80211`/`nl80211` and Wireless Extensions (`wext`) userspace interfaces for NetworkManager and `wpa_supplicant`.
- [os_dep/linux/rtw_proc.c](file:///botworkspace/rtl8723de/os_dep/linux/rtw_proc.c): Diagnostics exposed under `/proc/net/rtl8723de/`.

### 4. Build and Packaging Files
- [Makefile](file:///botworkspace/rtl8723de/Makefile), [Kconfig](file:///botworkspace/rtl8723de/Kconfig), and [dkms.conf](file:///botworkspace/rtl8723de/dkms.conf): Kbuild and DKMS build definitions.
- [NEW_readme.txt](file:///botworkspace/rtl8723de/NEW_readme.txt), [clean](file:///botworkspace/rtl8723de/clean), [runwpa](file:///botworkspace/rtl8723de/runwpa), [wlan0dhcp](file:///botworkspace/rtl8723de/wlan0dhcp), and [ifcfg-wlan0](file:///botworkspace/rtl8723de/ifcfg-wlan0): Helper scripts and DKMS removal/troubleshooting notes.
