# ZMK Shield Cornix

这是一个基于42键Corne键盘的ZMK固件分支。

## Bootloader恢复说明

原版RMK固件移除了SoftDevice，因此在刷新`zmk.uf2`之前，需要先恢复SoftDevice。具体步骤请参考[bootloader/README.md](./bootloader/README.md)。

### 软设备恢复步骤

1. 下载适用于nRF52840的bootloader。
2. 按照`bootloader/README.md`中的说明提取并转换SoftDevice。
3. 使用`uf2`工具刷新恢复的SoftDevice。

更多细节请查看[bootloader/README.md](./bootloader/README.md)。