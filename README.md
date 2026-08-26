# pstore-screen

`pstore-screen` 是一款仿 Linux DRM panic 屏幕的安卓内核崩溃屏显驱动，用于解决高通 GKI 2.0 机型无法导出日志的问题，将 pstore/printk 日志绘制到当前屏幕缓冲区，并提供文本日志与二维码两种模式，且支持分页显示，便于加载完整日志。

## 主要功能特性

- 黑底白字显示日志，顶部固定使用黄色标题：

```
===================
  Pstore Log Dump
===================
```

- 采用内置精简 8x16 ASCII 字库；不可显示的 UTF-8 字符显示为 `�`，原始字节仍保留在日志容器和二维码中；
- 根据扫描输出尺寸和物理参数选择字号与安全边距；真实换行只产生硬换行，字面量 `\n` 仍按普通文本显示；
- 电源键单击切换文本/二维码模式，音量键切换上一页或下一页；三击电源键请求关机，长按电源键请求重启；
- 未识别到有效 ramoops 保留区时，日志捕获器仍可保留 printk 最后 256 KiB；
- 显示后端通过已知前缀描述符注册，不修改 DRM 公共结构体或既有导出 KMI 符号。

## 安装配置方法

分支名即目标安卓 GKI 内核版本：`5.10`、`5.15`、`6.1`、`6.6` 和 `6.12`。每个分支只包含该版本的驱动源码和补丁等：

```
compat/                     build/bazel编译所需的abi符号补丁
patches/                    需要应用的内核补丁/配置选项
src/                        驱动核心源码
```

驱动安装方法如下：
1. 拉取本仓库，并切换本仓库至安卓 GKI 内核源码对应版本分支；
2. 将 `src/` 下全部源码复制到内核源码仓库 `common/` 下对应位置；
3. 把 `patches` 中的文件全部复制到内核源码根目录下，并通过以下指令修补内核源码：

```
patch -p1 -F 3 < ./pstore-screen-kernel.patch
cat ./pstore_screen_config >> ./arch/arm64/configs/gki_defconfig
```

注：在极少数情况下，若内核 DRM 驱动为模块形式（`CONFIG_DRM=m`）且希望将该驱动构建入 DRM 驱动模块，则应额外开启 DRM sidecar 模式（`CONFIG_PSTORE_SCREEN_DRM_SIDECAR=y`，）但由于模块加载时机问题，可能无法显示早期内核崩溃日志；

4. 若要为 bazel/build.sh 构建提供兼容并持久集成，建议额外导入 `compat/` 下的补丁；
5. 按 GKI 内核构建流程编译内核。

## 致谢

本项目二维码编码核心部分代码基于 Project Nayuki 的 [QR Code Generator library](https://github.com/nayuki/QR-Code-generator)，并保留其许可声明。感谢 Project Nayuki 对相关代码与文档的贡献。
