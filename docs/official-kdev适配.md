# official-kdev适配

适配目标：

** 使用官方android系统的kernel、内核模块、配合开源发行版的rootfs构建完整系统镜像 **


## 三分区适配 

![](./images/28734888951000.png)

* uboot采用rockchip-linux仓库即可

```shell
https://github.com/rockchip-linux/u-boot.git

branch: next-dev
```

* 修改配置，默认引导boot分区的extlinux.conf

```shell
# git diff
diff --git a/include/configs/evb_rk3588.h b/include/configs/evb_rk3588.h
index 6685bb5a44..efb44f9a28 100644
--- a/include/configs/evb_rk3588.h
+++ b/include/configs/evb_rk3588.h
@@ -20,7 +20,8 @@
 #define CONFIG_SYS_MMC_ENV_DEV         0
 
 #undef CONFIG_BOOTCOMMAND
-#define CONFIG_BOOTCOMMAND RKIMG_BOOTCOMMAND
+#define CONFIG_BOOTCOMMAND "sysboot mmc 0:2 any 0x00500000 extlinux.conf"
+
 
 #endif
 #endif
```


## 内核模块提取

```shell
root@armbian:/lib/modules/5.10.66# ls -alh *.ko
-rw-r--r-- 1 1001 1001 3.0M Jul 10 04:35 bcmdhd.ko
-rw-r--r-- 1 1001 1001 630K Jul 10 04:35 r8168.ko
root@armbian:/lib/modules/5.10.66#
 ```

从官方镜像中提取ko，其中bcmdhd用于wlan0模块，r8168是另一个千兆网卡

还有一个pgdrv.ko

r8168.ko — 正常网卡驱动

用途：让 RTL8168/8111 系列千兆网卡作为网络接口（eth0等）正常工作，负责收发数据包、链路协商、WoL 等。
使用场景：日常启动后自动加载（modprobe r8168或内核自动绑定），使系统能上网。
来源：Realtek 官方出的 Linux 网卡驱动，替代内核自带 r8169（部分旧硬件更稳定）。

pgdrv.ko — Realtek PG Tool（Programming）驱动

用途：不是用来上网的，是给 Realtek PG Tool（rtnicpg）​ 提供底层访问，用于读写网卡 EEPROM / eFUSE、烧写 MAC 地址、修改 VID/PID、写 Option ROM​ 等量产配置操作。
使用场景：工厂产线或维护时临时加载——必须先 rmmod r8168（或 r8169），再 insmod pgdrv.ko，运行 rtnicpg工具操作完后卸载。
注意：加载 pgdrv后网卡不会有网络接口，不能上网。


## rpc_pipefs挂载报错

启动日志pipefs报错

```shell
         Starting rpcbind.service - RPC bind portmap service...
         Starting systemd-journal-catalog-u…ervice - Rebuild Journal Catalog...
         Starting systemd-resolved.service - Network Name Resolution...
         Starting systemd-update-utmp.servi…ord System Boot/Shutdown in UTMP...
[FAILED] Failed to mount run-rpc_pipefs.mount - RPC Pipe File System.
See 'systemctl status run-rpc_pipefs.mount' for details.
[DEPEND] Dependency failed for rpc_pipefs.target.
[DEPEND] Dependency failed for rpc-gssd.ser… service for NFS client and server.
[  OK  ] Reached target nfs-client.target - NFS client services.
[  OK  ] Started rpcbind.service - RPC bind portmap service.
[  OK  ] Reached target remote-fs-pre.targe…reparation for Remote File Systems.
[  OK  ] Reached target remote-fs.target - Remote File Systems.
```


```shell
root@armbian:~# journalctl -u run-rpc_pipefs.mount --no-pager
Jul 10 05:40:36 armbian systemd[1]: Mounting run-rpc_pipefs.mount - RPC Pipe File System...
Jul 10 05:40:36 armbian mount[673]: mount: /run/rpc_pipefs: unknown filesystem type 'rpc_pipefs'.
Jul 10 05:40:36 armbian mount[673]:        dmesg(1) may have more information after failed mount system call.
Jul 10 05:40:36 armbian systemd[1]: run-rpc_pipefs.mount: Mount process exited, code=exited, status=32/n/a
Jul 10 05:40:36 armbian systemd[1]: run-rpc_pipefs.mount: Failed with result 'exit-code'.
Jul 10 05:40:36 armbian systemd[1]: Failed to mount run-rpc_pipefs.mount - RPC Pipe File System.
root@armbian:~# 

```


默认官方内核不支持rpc_pipefs


```shell
root@armbian:~# 
root@armbian:~# cat /proc/filesystems | grep rpc_pipefs
root@armbian:~# zcat /proc/config.gz |grep rpc_pipefs
root@armbian:~# 
root@armbian:~# 
```













---



