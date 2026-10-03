# 伪 DC 调光

[English](README.md) | [下载已适配版本](https://github.com/jawnd/PseudoDCDimming-marble-fixed/releases/latest)

通过软件增益为部分 OLED 屏幕在低亮度下启用类 DC 调光方式。

这是一个 Xposed（LSPosed）模块，仅支持 Android 12 及以上版本。

## 红米 Note 12 Turbo 适配说明

本仓库包含红米 Note 12 Turbo 的适配修复。该机的设备代号是 `marble`，常见型号标识为 `23049RAD8C`。

在已测试的 Android 13 / SDK 33 ROM 中，系统显示服务实际调用的不是上游模块所 hook 的四参数背光方法，而是带 HDR 参数的六参数重载方法。由于六参数方法没有被 hook，模块收不到实时亮度回调，界面会显示 `NaN`，最低硬件亮度不会锁定，软件增益也不会生效。

修复后的版本按以下顺序选择方法：优先 hook 六参数 HDR 重载，其次兼容五参数重载，最后回退到原来的四参数方法。HDR 标志和 HDR 增益因子保持原样，只改写伪 DC 调光需要的 SDR/背光亮度参数，因此不会主动改变 HDR 参数。

该修复已在已解锁、root 的红米 Note 12 Turbo 上验证：LSPosed 对 `android` 进程启用模块后，重启即可收到实时背光回调。将最低硬件亮度设为 50% 后，把系统亮度拉到最低，硬件背光请求保持在 50% 阈值，软件增益继续降低实际显示亮度；使用另一台手机的相机观察到了预期的类 DC 调光效果。

如果你的系统框架版本更换后仍然无法识别实时亮度，可能是 `BacklightAdapter.setBacklight` 的参数签名发生了变化。请先查看 [PATCH_NOTES.md](PATCH_NOTES.md) 中的根因和方法签名，再针对对应重载调整 hook。

## 下载与安装

[下载最新适配版本](https://github.com/jawnd/PseudoDCDimming-marble-fixed/releases/latest)

1. 下载并安装 Release 页面中的 APK。
2. 在 LSPosed 中启用模块，并勾选作用域 `android`。
3. 重启手机。
4. 打开模块，选择一个可以接受的最低硬件亮度。已验证的红米 Note 12 Turbo 使用了 50%。
5. 将系统亮度滑块继续拉低，低于硬件阈值的部分由 Android 软件增益完成。

## 原理

通过限制亮度控制的最小硬件值，并通过 degamma-gain-regamma 矩阵变换缩小输出信号，使实际显示亮度匹配预期亮度，同时保持较高的 PWM 频率和占空比。

[详细原理](details.zh.md)

## 限制

* 严重依赖厂商对屏幕亮度控制以及响应曲线的校准。如果厂商在校准时使用了不同的响应曲线，或者亮度控制存在非线性行为，都可能影响开启模块后的显示质量；
* 可能与其他颜色变换功能冲突；
* 可能与 HDR 显示冲突；
* 红米 Note 12 Turbo 的修复是在上述 Android 13 / SDK 33 显示栈上验证的，其他 ROM 版本可能暴露不同的框架方法签名。

## 配置与验证

可以使用相机的高速快门模式放大频闪效应，或使用专业仪器测量 PWM 频率和占空比，然后选择一个可以接受的数值作为最低硬件亮度。已验证的红米 Note 12 Turbo 将最低硬件亮度设为 50%，再把系统亮度拉到最低来观察伪 DC 效果。

详细的故障原因、方法签名和代码修复见 [PATCH_NOTES.md](PATCH_NOTES.md)。

## 致谢

灵感来自 [ztc1997/FakeDCBacklight](https://github.com/ztc1997/FakeDCBacklight)。本项目额外实现了立即应用以及稳定启用前后亮度的功能。
