# patch-razer-firmware

**DISCLAIMER: The following steps will probably BRICK YOUR DEVICE and definitely VOIDS YOUR WARRANTY, I won't take any responsibility for any damage to your device or warranty.**

**Make sure you HAVE READ the DISCLAIMER and understand what you're doing.**

**Asking razer for the firmware is better than just patching firmware.**

Just follow the steps from [Luke Granger-Brown
:Secure Boot Shenanigans
](https://lukegb.com/posts/2016-11-11-secure-boot-shenanigans/).

NOTE: Use [UEFITool](https://github.com/LongSoft/UEFITool) `0.26.0` to patch the firmware if the newest version doesn't support patching. When using `0.26.0`, just ignore the parsing warnings and check the volume integrity after patching, make sure the firmware is patched correctly.