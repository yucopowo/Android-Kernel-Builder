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


---


# 项目结构


| 文件 | 说明 |
| --- | --- |
| `repos.dipper-MIUI.json` | MIUI 内核配置（Xiaomi_Kernel_OpenSource-sdm845, 88-zstd 分支） |
| `repos.dipper-NGK.json` | 类原生内核配置（NGK_android_kernel_xiaomi_sdm845, t-caf-ksu 分支） |
| `.github/workflows/build KernelSU_v0.9.5.yml` | 按 `repos*.json` 全量构建 |
| `.github/workflows/build_MIU-Kernel.yml` | 只构建 MIUI 内核（读 `repos*-MIUI.json`） |
| `.github/workflows/build_NGK-Kernel.yml` | 只构建 NGK 内核（读 `repos*-NGK.json`） |
| `.github/workflows/del.yml` | 清理旧的 workflow 运行记录 |


---


# 致谢


- [Akitlove](https://github.com/Akitlove) : 此项目所用99%+代码来自此作者,包括但不限于内核修复、Github Action
- [KernelSU](https://github.com/tiann/KernelSU) : KernelSU

# 许可

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。
