```
dspace@mini:~/source/blackmagic/src$ st-flash --reset write blackmagic_dfu.bin 0x8000000
st-flash 1.6.1
2020-09-13T15:19:35 INFO common.c: F1xx Medium-density: 20 KiB SRAM, 64 KiB flash in at least 1 KiB pages.
file blackmagic_dfu.bin md5 checksum: 2d7aca3796d74cfac627132fa823a7, stlink checksum: 0x000a7453
2020-09-13T15:19:35 INFO common.c: Attempting to write 7308 (0x1c8c) bytes to stm32 address: 134217728 (0x8000000)
2020-09-13T15:19:35 INFO common.c: Flash page at addr: 0x08000000 erased
2020-09-13T15:19:35 INFO common.c: Flash page at addr: 0x08000400 erased
2020-09-13T15:19:35 INFO common.c: Flash page at addr: 0x08000800 erased
2020-09-13T15:19:35 INFO common.c: Flash page at addr: 0x08000c00 erased
2020-09-13T15:19:35 INFO common.c: Flash page at addr: 0x08001000 erased
2020-09-13T15:19:35 INFO common.c: Flash page at addr: 0x08001400 erased
2020-09-13T15:19:35 INFO common.c: Flash page at addr: 0x08001800 erased
2020-09-13T15:19:35 INFO common.c: Flash page at addr: 0x08001c00 erased
2020-09-13T15:19:35 INFO common.c: Finished erasing 8 pages of 1024 (0x400) bytes
2020-09-13T15:19:35 INFO common.c: Starting Flash write for VL/F0/F3/F1_XL core id
2020-09-13T15:19:36 INFO flash_loader.c: Successfully loaded flash loader in sram
  8/8 pages written
2020-09-13T15:19:36 INFO common.c: Starting verification of write complete
2020-09-13T15:19:36 INFO common.c: Flash written and verified! jolly good!

dspace@mini:~/source/blackmagic/src$ st-flash --flash=128k write blackmagic.bin 0x8002000
st-flash 1.6.1
2020-09-13T15:20:16 INFO common.c: F1xx Medium-density: 20 KiB SRAM, 64 KiB flash in at least 1 KiB pages.
Forcing flash size: --flash=0x00020000
file blackmagic.bin md5 checksum: 341e1f524a8cc9be4864cffce3e9730, stlink checksum: 0x007ed2a0
2020-09-13T15:20:16 INFO common.c: Attempting to write 89036 (0x15bcc) bytes to stm32 address: 134225920 (0x8002000)
2020-09-13T15:20:16 INFO common.c: Flash page at addr: 0x08002000 erased
2020-09-13T15:20:16 INFO common.c: Flash page at addr: 0x08002400 erased
2020-09-13T15:20:16 INFO common.c: Flash page at addr: 0x08002800 erased
2020-09-13T15:20:16 INFO common.c: Flash page at addr: 0x08002c00 erased
2020-09-13T15:20:16 INFO common.c: Flash page at addr: 0x08003000 erased
2020-09-13T15:20:16 INFO common.c: Flash page at addr: 0x08003400 erased
2020-09-13T15:20:16 INFO common.c: Flash page at addr: 0x08003800 erased
2020-09-13T15:20:16 INFO common.c: Flash page at addr: 0x08003c00 erased
2020-09-13T15:20:16 INFO common.c: Flash page at addr: 0x08004000 erased
2020-09-13T15:20:16 INFO common.c: Flash page at addr: 0x08004400 erased
2020-09-13T15:20:16 INFO common.c: Flash page at addr: 0x08004800 erased
2020-09-13T15:20:16 INFO common.c: Flash page at addr: 0x08004c00 erased
2020-09-13T15:20:16 INFO common.c: Flash page at addr: 0x08005000 erased
2020-09-13T15:20:16 INFO common.c: Flash page at addr: 0x08005400 erased
2020-09-13T15:20:16 INFO common.c: Flash page at addr: 0x08005800 erased
2020-09-13T15:20:16 INFO common.c: Flash page at addr: 0x08005c00 erased
2020-09-13T15:20:16 INFO common.c: Flash page at addr: 0x08006000 erased
2020-09-13T15:20:16 INFO common.c: Flash page at addr: 0x08006400 erased
2020-09-13T15:20:16 INFO common.c: Flash page at addr: 0x08006800 erased
2020-09-13T15:20:16 INFO common.c: Flash page at addr: 0x08006c00 erased
2020-09-13T15:20:16 INFO common.c: Flash page at addr: 0x08007000 erased
2020-09-13T15:20:16 INFO common.c: Flash page at addr: 0x08007400 erased
2020-09-13T15:20:16 INFO common.c: Flash page at addr: 0x08007800 erased
2020-09-13T15:20:16 INFO common.c: Flash page at addr: 0x08007c00 erased
2020-09-13T15:20:16 INFO common.c: Flash page at addr: 0x08008000 erased
2020-09-13T15:20:16 INFO common.c: Flash page at addr: 0x08008400 erased
2020-09-13T15:20:16 INFO common.c: Flash page at addr: 0x08008800 erased
2020-09-13T15:20:17 INFO common.c: Flash page at addr: 0x08008c00 erased
2020-09-13T15:20:17 INFO common.c: Flash page at addr: 0x08009000 erased
2020-09-13T15:20:17 INFO common.c: Flash page at addr: 0x08009400 erased
2020-09-13T15:20:17 INFO common.c: Flash page at addr: 0x08009800 erased
2020-09-13T15:20:17 INFO common.c: Flash page at addr: 0x08009c00 erased
2020-09-13T15:20:17 INFO common.c: Flash page at addr: 0x0800a000 erased
2020-09-13T15:20:17 INFO common.c: Flash page at addr: 0x0800a400 erased
2020-09-13T15:20:17 INFO common.c: Flash page at addr: 0x0800a800 erased
2020-09-13T15:20:17 INFO common.c: Flash page at addr: 0x0800ac00 erased
2020-09-13T15:20:17 INFO common.c: Flash page at addr: 0x0800b000 erased
2020-09-13T15:20:17 INFO common.c: Flash page at addr: 0x0800b400 erased
2020-09-13T15:20:17 INFO common.c: Flash page at addr: 0x0800b800 erased
2020-09-13T15:20:17 INFO common.c: Flash page at addr: 0x0800bc00 erased
2020-09-13T15:20:17 INFO common.c: Flash page at addr: 0x0800c000 erased
2020-09-13T15:20:17 INFO common.c: Flash page at addr: 0x0800c400 erased
2020-09-13T15:20:17 INFO common.c: Flash page at addr: 0x0800c800 erased
2020-09-13T15:20:17 INFO common.c: Flash page at addr: 0x0800cc00 erased
2020-09-13T15:20:17 INFO common.c: Flash page at addr: 0x0800d000 erased
2020-09-13T15:20:17 INFO common.c: Flash page at addr: 0x0800d400 erased
2020-09-13T15:20:17 INFO common.c: Flash page at addr: 0x0800d800 erased
2020-09-13T15:20:17 INFO common.c: Flash page at addr: 0x0800dc00 erased
2020-09-13T15:20:17 INFO common.c: Flash page at addr: 0x0800e000 erased
2020-09-13T15:20:17 INFO common.c: Flash page at addr: 0x0800e400 erased
2020-09-13T15:20:17 INFO common.c: Flash page at addr: 0x0800e800 erased
2020-09-13T15:20:17 INFO common.c: Flash page at addr: 0x0800ec00 erased
2020-09-13T15:20:17 INFO common.c: Flash page at addr: 0x0800f000 erased
2020-09-13T15:20:17 INFO common.c: Flash page at addr: 0x0800f400 erased
2020-09-13T15:20:17 INFO common.c: Flash page at addr: 0x0800f800 erased
2020-09-13T15:20:17 INFO common.c: Flash page at addr: 0x0800fc00 erased
2020-09-13T15:20:17 INFO common.c: Flash page at addr: 0x08010000 erased
2020-09-13T15:20:17 INFO common.c: Flash page at addr: 0x08010400 erased
2020-09-13T15:20:17 INFO common.c: Flash page at addr: 0x08010800 erased
2020-09-13T15:20:17 INFO common.c: Flash page at addr: 0x08010c00 erased
2020-09-13T15:20:17 INFO common.c: Flash page at addr: 0x08011000 erased
2020-09-13T15:20:17 INFO common.c: Flash page at addr: 0x08011400 erased
2020-09-13T15:20:17 INFO common.c: Flash page at addr: 0x08011800 erased
2020-09-13T15:20:17 INFO common.c: Flash page at addr: 0x08011c00 erased
2020-09-13T15:20:17 INFO common.c: Flash page at addr: 0x08012000 erased
2020-09-13T15:20:17 INFO common.c: Flash page at addr: 0x08012400 erased
2020-09-13T15:20:17 INFO common.c: Flash page at addr: 0x08012800 erased
2020-09-13T15:20:17 INFO common.c: Flash page at addr: 0x08012c00 erased
2020-09-13T15:20:17 INFO common.c: Flash page at addr: 0x08013000 erased
2020-09-13T15:20:17 INFO common.c: Flash page at addr: 0x08013400 erased
2020-09-13T15:20:17 INFO common.c: Flash page at addr: 0x08013800 erased
2020-09-13T15:20:17 INFO common.c: Flash page at addr: 0x08013c00 erased
2020-09-13T15:20:17 INFO common.c: Flash page at addr: 0x08014000 erased
2020-09-13T15:20:17 INFO common.c: Flash page at addr: 0x08014400 erased
2020-09-13T15:20:17 INFO common.c: Flash page at addr: 0x08014800 erased
2020-09-13T15:20:17 INFO common.c: Flash page at addr: 0x08014c00 erased
2020-09-13T15:20:17 INFO common.c: Flash page at addr: 0x08015000 erased
2020-09-13T15:20:17 INFO common.c: Flash page at addr: 0x08015400 erased
2020-09-13T15:20:17 INFO common.c: Flash page at addr: 0x08015800 erased
2020-09-13T15:20:17 INFO common.c: Flash page at addr: 0x08015c00 erased
2020-09-13T15:20:17 INFO common.c: Flash page at addr: 0x08016000 erased
2020-09-13T15:20:17 INFO common.c: Flash page at addr: 0x08016400 erased
2020-09-13T15:20:17 INFO common.c: Flash page at addr: 0x08016800 erased
2020-09-13T15:20:17 INFO common.c: Flash page at addr: 0x08016c00 erased
2020-09-13T15:20:17 INFO common.c: Flash page at addr: 0x08017000 erased
2020-09-13T15:20:17 INFO common.c: Flash page at addr: 0x08017400 erased
2020-09-13T15:20:17 INFO common.c: Flash page at addr: 0x08017800 erased
2020-09-13T15:20:17 INFO common.c: Finished erasing 87 pages of 1024 (0x400) bytes
2020-09-13T15:20:17 INFO common.c: Starting Flash write for VL/F0/F3/F1_XL core id
2020-09-13T15:20:17 INFO flash_loader.c: Successfully loaded flash loader in sram
  6/87 pages written2020-09-13T15:20:18 ERROR flash_loader.c: flash loader run error
2020-09-13T15:20:18 ERROR common.c: stlink_flash_loader_run(0x8003800) failed! == -1
stlink_fwrite_flash() == -1

dspace@mini:~/source/blackmagic/src$ st-flash --flash=128k write blackmagic.bin 0x8002000
st-flash 1.6.1
2020-09-13T15:31:41 INFO common.c: F1xx Medium-density: 20 KiB SRAM, 64 KiB flash in at least 1 KiB pages.
Forcing flash size: --flash=0x00020000
file blackmagic.bin md5 checksum: 341e1f524a8cc9be4864cffce3e9730, stlink checksum: 0x007ed2a0
2020-09-13T15:31:41 INFO common.c: Attempting to write 89036 (0x15bcc) bytes to stm32 address: 134225920 (0x8002000)
2020-09-13T15:31:41 INFO common.c: Flash page at addr: 0x08002000 erased
2020-09-13T15:31:41 INFO common.c: Flash page at addr: 0x08002400 erased
2020-09-13T15:31:41 INFO common.c: Flash page at addr: 0x08002800 erased
2020-09-13T15:31:41 INFO common.c: Flash page at addr: 0x08002c00 erased
2020-09-13T15:31:41 INFO common.c: Flash page at addr: 0x08003000 erased
2020-09-13T15:31:41 INFO common.c: Flash page at addr: 0x08003400 erased
2020-09-13T15:31:41 INFO common.c: Flash page at addr: 0x08003800 erased
2020-09-13T15:31:41 INFO common.c: Flash page at addr: 0x08003c00 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x08004000 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x08004400 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x08004800 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x08004c00 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x08005000 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x08005400 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x08005800 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x08005c00 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x08006000 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x08006400 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x08006800 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x08006c00 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x08007000 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x08007400 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x08007800 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x08007c00 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x08008000 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x08008400 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x08008800 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x08008c00 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x08009000 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x08009400 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x08009800 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x08009c00 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x0800a000 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x0800a400 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x0800a800 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x0800ac00 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x0800b000 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x0800b400 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x0800b800 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x0800bc00 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x0800c000 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x0800c400 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x0800c800 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x0800cc00 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x0800d000 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x0800d400 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x0800d800 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x0800dc00 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x0800e000 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x0800e400 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x0800e800 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x0800ec00 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x0800f000 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x0800f400 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x0800f800 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x0800fc00 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x08010000 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x08010400 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x08010800 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x08010c00 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x08011000 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x08011400 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x08011800 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x08011c00 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x08012000 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x08012400 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x08012800 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x08012c00 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x08013000 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x08013400 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x08013800 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x08013c00 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x08014000 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x08014400 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x08014800 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x08014c00 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x08015000 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x08015400 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x08015800 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x08015c00 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x08016000 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x08016400 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x08016800 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x08016c00 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x08017000 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x08017400 erased
2020-09-13T15:31:42 INFO common.c: Flash page at addr: 0x08017800 erased
2020-09-13T15:31:42 INFO common.c: Finished erasing 87 pages of 1024 (0x400) bytes
2020-09-13T15:31:42 INFO common.c: Starting Flash write for VL/F0/F3/F1_XL core id
2020-09-13T15:31:42 INFO flash_loader.c: Successfully loaded flash loader in sram
 87/87 pages written
2020-09-13T15:31:45 INFO common.c: Starting verification of write complete
2020-09-13T15:31:46 ERROR common.c: Verification of flash failed at offset: 57344
stlink_fwrite_flash() == -1
