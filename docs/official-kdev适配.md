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





















---



