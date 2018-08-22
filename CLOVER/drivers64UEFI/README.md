### Drivers64UEFI目录下的驱动作用：

（选项A）BIOS启动过程中要用到drivers32或drivers64目录  

（选项B）UEFI启动过程中则使用driversUEFI目录


必须要提的一点是这些驱动程序只在bootloader运行时有效，
不会影响最终启动的操作系统。

### drivers64UEFI目录下的驱动：

CsmVideoDxe-64.efi ，Clover图形界面的图像驱动，可以有更多的分辨率选择。
（仅限于启动界面）。他基于UEFI BIOS的CSM模块，因此需要CSM可用。这个驱动
比较危险，可能导致开机黑屏、Clover无法启动或启动后系统出现唤醒问题，谨慎使用！

__NTFS.efi__   识别NTFS分区，用于启动Windows EFI系统，比如引导Win8/8.1

__HFSPlus.efi__
HFS+文件系统驱动程序。这个驱动对于通过启动方式B（UEFI）来启动Mac OS X是必须的。

VBoxHFS.efi
HFSPlus.efi的替代品，性能要差一点。

__FSInject.efi__
控制文件系统注入kext到系统的可能性。详细解释请参照WithKexts。

EmuVariableUefi-64.efi 如果不装,登录cloud、imessage等会有激活问题。
需要MacOSX的NVRAM变量的支持

__OsxFatBinaryDrv.efi__
允许加载FAT模块比如boot.efi。（选项B）UEFI需要。

DataHubDxe-64.efi 这是DataHub协议支持强制性的MacOSX的。通常它已经存在，
但有时也可能被忽略了，在这种情况下，你应该在屏幕上看到的警告。
该文件的存在始终是安全的。建议还是使用它，不会产生冲突。

__OsxAptioFixDrv-64.efi__  修复AMI Aptio EFI内存映射。如果没有就不能启动OS X。
OsxLowMemFixDrv-64.efi  是OsxAptioFixDrv的简化版本。两者不要同时使用。

PartitionDxe-64.efi 支持不常见的分区地图，如：混合GPT / MBR或Apple分区图，
但没有为Apple分区优化，也没有为GPT/MBR优化。很可能（选项B）需要。文件的存在始终是安全的。

NvmExpressDxe-64.efi  支持SSD连接到NVM Express总线

VBoxExt2.efi或VBoxExt4.efi  支持Linux的EXT2/3/4文件系统驱动 能够启动Linux efi系统

XhciDxe.efi 支持USB3总线，大多是英特尔控制器。

Usb*.efi, UHCI.efi, EHCI.efi, XHCI.efi
用以解决依赖性关系不满足导致的内建驱动工作不正常的情况（选项B）的一组驱动。

PS2Mouse*.efi, PS2MouseAbsolute*.efi, UsbMouse*.efi
用来使鼠标，触摸板在CloverGUI界面工作的一组驱动，它们对操作系统没有