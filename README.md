<div align = center>

<img src="./.assets/DogDayAndroid.png" width="200" height="175" alt="banner">

<h1>小米 8 (dipper) 构建 KernelSU</h1>

![License](https://img.shields.io/static/v1?label=License&message=BY-NC-SA&logo=creativecommons&color=green)
![Language](https://img.shields.io/github/languages/top/Xiaomi-sdm845-KSU/Android-Kernel-Builder)
![Issues](https://img.shields.io/github/issues/Xiaomi-sdm845-KSU/Android-Kernel-Builder)
![Pull Requests](https://img.shields.io/github/issues-pr/Xiaomi-sdm845-KSU/Android-Kernel-Builder)
<br>

这是个 Github Action 自动构建小米 8（dipper）KernelSU 内核的项目, 自动同步最新 KernelSU

包含: MI 8 (dipper)
<br>
</div>

---


# 内核选择


MIUI rom 建议选择 dipper-Xiaomi_Kernel_OpenSource-sdm845_构建时间.zip

如: 小米 8 MIUI 12.5 选择 dipper-Xiaomi_Kernel_OpenSource-sdm845_********.zip

类原生rom 建议选择 dipper-NGK_android_kernel_xiaomi_sdm845_构建时间.zip

如: 小米 8 类原生rom 选择 dipper-NGK_android_kernel_xiaomi_sdm845_********.zip

LineageOS rom 建议选择 dipper-android_kernel_xiaomi_sdm845_构建时间.zip

如: 小米 8 LineageOS 22.2 选择 dipper-android_kernel_xiaomi_sdm845_********.zip


---


# 项目结构


| 文件 | 说明 |
| --- | --- |
| `repos.dipper-MIUI.json` | MIUI 内核配置（Xiaomi_Kernel_OpenSource-sdm845, 88-zstd 分支） |
| `repos.dipper-NGK.json` | 类原生内核配置（NGK_android_kernel_xiaomi_sdm845, t-caf-ksu 分支） |
| `repos.dipper-LineageOS.json` | LineageOS 内核配置（LineageOS/android_kernel_xiaomi_sdm845, lineage-22.2 分支） |
| `.github/workflows/build KernelSU_v0.9.5.yml` | 按 `repos*.json` 全量构建 |
| `.github/workflows/build_MIU-Kernel.yml` | 只构建 MIUI 内核（读 `repos*-MIUI.json`） |
| `.github/workflows/build_NGK-Kernel.yml` | 只构建 NGK 内核（读 `repos*-NGK.json`） |
| `.github/workflows/build_LineageOS-Kernel.yml` | 只构建 LineageOS 内核（读 `repos*-LineageOS.json`） |
| `.github/workflows/del.yml` | 清理旧的 workflow 运行记录 |

> **关于 `defconfigFragments`**：LineageOS 内核的基础 defconfig（`vendor/xiaomi/mi845_defconfig`）
> 并不包含触摸屏、指纹、GPS 等机型专属项，这些放在 `arch/arm64/configs/vendor/xiaomi/dipper.config`
> 里，必须额外合并，否则会编出一个「能编译但刷进手机屏幕指纹全废」的内核。
> 所以在 `repos.dipper-LineageOS.json` 的 **`kernelSource` 里**用 `defconfigFragments` 声明该片段
> （workflow 读的是 `matrix.repos.kernelSource.defconfigFragments`），
> workflow 会在 defconfig 之后自动执行 `scripts/kconfig/merge_config.sh` + `olddefconfig`。
> 没有该字段的机型配置行为完全不变。
>
> 该片段里含 `CONFIG_MACH_XIAOMI_SDM845=y` / `CONFIG_MACH_XIAOMI_E1N=y`，它们决定内核
> 走**小米分支**还是**高通参考板分支**（见 `arch/arm64/boot/dts/qcom/Makefile` 开头的
> `ifeq ($(CONFIG_MACH_XIAOMI_SDM845),y)`）。若合并失效，会去编 `sdm845-v2-qvr-evt.dtb`
> 等参考板 DTB，并报 `Reference to non-existent node or label "ts_int_active"` 这类 DTC 错误
> —— **看到这个报错，就是片段没合并成功。**
>
> 这个报错的来历（已逐行核对）：`sdm845-qvr.dtsi` 的触摸屏节点写的是
> `pinctrl-0 = <&ts_int_active &ts_reset_active>`，而 `sdm845-pinctrl.dtsi` 里注册的标签
> 其实只有 `ts_int_active1` / `ts_reset_active1` —— 那个 `ts_int_active` 是**节点名**
> （后面没有冒号，所以不是标签）。dtc 解析 `&X` 时查的是**标签表**，因此失败。
> 这是上游参考板分支里的一处死代码缺陷，**我们永远不该编它**。
>
> 所以：**不要**去 DTS 里补标签、也**不要**改 Makefile 删掉那条 `dtb-y`
> —— Makefile 已经被 `ifeq ($(CONFIG_MACH_XIAOMI_SDM845),y)` 正确门控了，
> 正确做法就是让配置走小米分支。
>
> **两道防线**（4 个 workflow 里都有，专门防上面这种「静默跳过」）：
> 1. **矩阵生成阶段**：用 `jq` 校验 `defconfigFragments` 只能出现在 `kernelSource` 里，
>    写到顶层就直接报错退出。之前正是写错层级导致取到 `null`、合并被静默跳过、
>    CI 报的却是 DTC 错误，完全看不出根因。
> 2. **合并阶段**：校验片段文件真实存在，路径写错时显式失败。
>    `merge_config.sh` 遇到不存在的文件不会报错，只会静默什么都不做。

> **关于 KernelSU 版本**：所有 workflow 里的 `KERNELSU_VERSION` 固定为 `v0.9.5`，**不要改成 `main`**。
> `main` 分支已重构（变成 `kernel/hook/` + `kernel/core/`），不再兼容本项目的 4.9 内核，会直接编译失败：
> - `syscall_fn_t` 只在 `__x86_64__` 下自行定义，arm64 依赖内核提供，而 4.9 内核里没有这个类型
> - `MODULE_IMPORT_NS` 未做版本保护，而 4.9 内核没有这个宏
>
> `v0.9.5` 是**最后一个**带 `kernel_compat.h` 且对上述两项都做了版本保护的版本
> （`v1.0.0` 起保护被移除）。想升级请先确认新版本仍支持 4.9 内核。
> 刷入后，KernelSU 管理器 App 也建议使用与内核模块对应的 v0.9.5。


---


# 致谢


- [Akitlove](https://github.com/Akitlove) : 此项目所用99%+代码来自此作者,包括但不限于内核修复、Github Action
- [KernelSU](https://github.com/tiann/KernelSU) : KernelSU

# 许可

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。
