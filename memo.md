# 読めるBlue Pill

```
dspace@mini:~$ st-info --probe
Found 1 stlink programmers
 serial:     4a1a16002c135737334d4e00
 hla-serial: "\x4a\x1a\x16\x00\x2c\x13\x57\x37\x33\x4d\x4e\x00"
 flash:      65536 (pagesize: 1024)
 sram:       20480
 chipid:     0x0410
 descr:      F1xx Medium-density
dspace@mini:~$
```

# 読めないBlue Pill

```
dspace@mini:~$ st-info --probe
Found 1 stlink programmers
 serial:     4a1a16002c135737334d4e00
 hla-serial: "\x4a\x1a\x16\x00\x2c\x13\x57\x37\x33\x4d\x4e\x00"
 flash:      0 (pagesize: 0)
 sram:       0
 chipid:     0x0748

dspace@mini:~$ st-flash --debug erase
st-flash 1.6.1
2020-09-16T08:58:51 DEBUG common.c: *** looking up stlink version
2020-09-16T08:58:51 DEBUG common.c: st vid         = 0x0483 (expect 0x0483)
2020-09-16T08:58:51 DEBUG common.c: stlink pid     = 0x3748
2020-09-16T08:58:51 DEBUG common.c: stlink version = 0x2
2020-09-16T08:58:51 DEBUG common.c: jtag version   = 0x22
2020-09-16T08:58:51 DEBUG common.c: swim version   = 0x7
2020-09-16T08:58:51 DEBUG common.c: *** looking up stlink version
2020-09-16T08:58:51 DEBUG common.c: st vid         = 0x0483 (expect 0x0483)
2020-09-16T08:58:51 DEBUG common.c: stlink pid     = 0x3748
2020-09-16T08:58:51 DEBUG common.c: stlink version = 0x2
2020-09-16T08:58:51 DEBUG common.c: jtag version   = 0x22
2020-09-16T08:58:51 DEBUG common.c: swim version   = 0x7
2020-09-16T08:58:51 DEBUG common.c: stlink current mode: mass
2020-09-16T08:58:51 DEBUG usb.c: JTAG/SWD freq set to 0
2020-09-16T08:58:51 DEBUG common.c: *** set_swdclk ***
2020-09-16T08:58:51 DEBUG common.c: stlink current mode: mass
2020-09-16T08:58:51 DEBUG common.c: *** stlink_enter_swd_mode ***
2020-09-16T08:58:51 DEBUG common.c: *** stlink_jtag_reset ***
2020-09-16T08:58:51 DEBUG common.c: *** stlink_reset ***
2020-09-16T08:58:51 DEBUG common.c: *** stlink_write_debug32 5fa0004 to 0xe000ed0c
2020-09-16T08:58:51 DEBUG common.c: Loading device parameters....
2020-09-16T08:58:51 DEBUG common.c: *** stlink_core_id ***
2020-09-16T08:58:51 DEBUG common.c: core_id = 0x00003748
2020-09-16T08:58:51 DEBUG common.c: *** stlink_read_debug32 3748 is 0xe0042000
2020-09-16T08:58:51 WARN common.c: unknown chip id! 0x3748
Failed to connect to target
```

## フラッシュのクリア

Windowsで`STM32 ST-LINK Utility`を立ち上げ、ST-LINKをリセットボタンを押しながら挿入した
ところ認識したので、全エリアクリアした。すると以下の通り、BluePillが見えるようになった。

```
dspace@mini:~$ st-info --probe
Found 1 stlink programmers
 serial:     4a1a16002c135737334d4e00
 hla-serial: "\x4a\x1a\x16\x00\x2c\x13\x57\x37\x33\x4d\x4e\x00"
 flash:      65536 (pagesize: 1024)
 sram:       20480
 chipid:     0x0410
 descr:      F1xx Medium-density

dspace@mini:~$ st-flash --debug erase
st-flash 1.6.1
2020-09-16T10:03:20 DEBUG common.c: *** looking up stlink version
2020-09-16T10:03:20 DEBUG common.c: st vid         = 0x0483 (expect 0x0483)
2020-09-16T10:03:20 DEBUG common.c: stlink pid     = 0x3748
2020-09-16T10:03:20 DEBUG common.c: stlink version = 0x2
2020-09-16T10:03:20 DEBUG common.c: jtag version   = 0x22
2020-09-16T10:03:20 DEBUG common.c: swim version   = 0x7
2020-09-16T10:03:20 DEBUG common.c: *** looking up stlink version
2020-09-16T10:03:20 DEBUG common.c: st vid         = 0x0483 (expect 0x0483)
2020-09-16T10:03:20 DEBUG common.c: stlink pid     = 0x3748
2020-09-16T10:03:20 DEBUG common.c: stlink version = 0x2
2020-09-16T10:03:20 DEBUG common.c: jtag version   = 0x22
2020-09-16T10:03:20 DEBUG common.c: swim version   = 0x7
2020-09-16T10:03:20 DEBUG common.c: stlink current mode: debug (jtag or swd)
2020-09-16T10:03:20 DEBUG usb.c: JTAG/SWD freq set to 0
2020-09-16T10:03:20 DEBUG common.c: *** set_swdclk ***
2020-09-16T10:03:20 DEBUG common.c: stlink current mode: debug (jtag or swd)
2020-09-16T10:03:20 DEBUG common.c: *** stlink_jtag_reset ***
2020-09-16T10:03:20 DEBUG common.c: *** stlink_reset ***
2020-09-16T10:03:20 DEBUG common.c: *** stlink_write_debug32 5fa0004 to 0xe000ed0c
2020-09-16T10:03:20 DEBUG common.c: Loading device parameters....
2020-09-16T10:03:20 DEBUG common.c: *** stlink_core_id ***
2020-09-16T10:03:20 DEBUG common.c: core_id = 0x2ba01477
2020-09-16T10:03:20 DEBUG common.c: *** stlink_read_debug32 20036410 is 0xe0042000
2020-09-16T10:03:20 DEBUG common.c: *** stlink_read_debug32 ffff0040 is 0x1ffff7e0
2020-09-16T10:03:20 INFO common.c: F1xx Medium-density: 20 KiB SRAM, 64 KiB flash in at least 1 KiB pages.
2020-09-16T10:03:20 DEBUG common.c: stlink current mode: debug (jtag or swd)
2020-09-16T10:03:20 DEBUG common.c: stlink current mode: debug (jtag or swd)
2020-09-16T10:03:20 DEBUG common.c: *** stlink_force_debug_mode ***
2020-09-16T10:03:20 DEBUG common.c: *** stlink_status ***
2020-09-16T10:03:20 DEBUG usb.c: core status: 00030003
2020-09-16T10:03:20 DEBUG common.c:   core status: halted
2020-09-16T10:03:20 DEBUG common.c: *** stlink_read_debug32 0 is 0x4002200c
2020-09-16T10:03:20 DEBUG common.c: *** stlink_read_debug32 80 is 0x40022010
2020-09-16T10:03:20 DEBUG common.c: *** stlink_write_debug32 45670123 to 0x40022004
2020-09-16T10:03:20 DEBUG common.c: *** stlink_write_debug32 cdef89ab to 0x40022004
2020-09-16T10:03:20 DEBUG common.c: *** stlink_read_debug32 0 is 0x40022010
2020-09-16T10:03:20 DEBUG common.c: Successfully unlocked flash
2020-09-16T10:03:20 DEBUG common.c: *** stlink_read_debug32 0 is 0x40022010
2020-09-16T10:03:20 DEBUG common.c: *** stlink_write_debug32 4 to 0x40022010
2020-09-16T10:03:20 DEBUG common.c: *** stlink_read_debug32 4 is 0x40022010
2020-09-16T10:03:20 DEBUG common.c: *** stlink_write_debug32 44 to 0x40022010
Mass erasing2020-09-16T10:03:20 DEBUG common.c: *** stlink_read_debug32 1 is 0x4002200c
2020-09-16T10:03:20 DEBUG common.c: *** stlink_read_debug32 1 is 0x4002200c
2020-09-16T10:03:20 DEBUG common.c: *** stlink_read_debug32 1 is 0x4002200c
2020-09-16T10:03:20 DEBUG common.c: *** stlink_read_debug32 20 is 0x4002200c

2020-09-16T10:03:20 DEBUG common.c: *** stlink_read_debug32 20 is 0x4002200c
2020-09-16T10:03:20 DEBUG common.c: *** stlink_read_debug32 4 is 0x40022010
2020-09-16T10:03:20 DEBUG common.c: *** stlink_write_debug32 84 to 0x40022010
2020-09-16T10:03:20 DEBUG common.c: *** stlink_read_debug32 84 is 0x40022010
2020-09-16T10:03:20 DEBUG common.c: *** stlink_write_debug32 80 to 0x40022010
2020-09-16T10:03:20 DEBUG common.c: *** stlink_exit_debug_mode ***
2020-09-16T10:03:20 DEBUG common.c: *** stlink_write_debug32 a05f0000 to 0xe000edf0
2020-09-16T10:03:20 DEBUG common.c: *** stlink_close ***
```

# ST-LINK V2と[stlink](https://github.com/stlink-org/stlink)を使った書き込み方法

ST-LINK V2はopenocdからは使えないみたい（`interface/stlink-v2.cfg`は正規商品用で
`hla_vid_pid`がST-LINK V2のものとは違う）でopenocdがエラーで立ち上がらなかった。そこで
stlinkのst-utilを使って書き込みを行ったところ、うまく行った。

1. 端末AでGDBサーバを立ち上げる
    ```
    $ st-util
    ```

2. 端末Bでgdbを立ち上げる

    ```
    $ cd target/thumbv7m-none-eabi/debug/examples
    $ gdb blinky
    GNU gdb (GDB) 9.2
    ...
    Reading symbols from blinky...
    (gdb) target extended-remote :4242
    Remote debugging using :4242
    0x00000000 in ?? ()
    (gdb) load
    Loading section .vector_table, size 0x130 lma 0x8000000
    Loading section .text, size 0x28d0 lma 0x8000130
    Loading section .rodata, size 0x724 lma 0x8002a00
    Start address 0x08000130, load size 12580
    Transfer rate: 16 KB/sec, 4193 bytes/write.
    ```

# `st-flash`で書き込む

`st-flash`で書き込む場合、ELF形式のファイルをBIN形式に変換する必要がある。

```
$ arm-none-eabi-objcopy -O binary blinky blinky.bin
$ st-flash write blinky.bin 0x8000000
st-flash 1.6.1
2020-09-16T10:41:12 INFO common.c: F1xx Medium-density: 20 KiB SRAM, 64 KiB flash in at least 1 KiB pages.
file blinky.bin md5 checksum: a776c779de16d57c1c9a1648ab9c896d, stlink checksum: 0x00144183
2020-09-16T10:41:12 INFO common.c: Attempting to write 12580 (0x3124) bytes to stm32 address: 134217728 (0x8000000)
2020-09-16T10:41:12 INFO common.c: Flash page at addr: 0x08000000 erased
2020-09-16T10:41:12 INFO common.c: Flash page at addr: 0x08000400 erased
2020-09-16T10:41:12 INFO common.c: Flash page at addr: 0x08000800 erased
2020-09-16T10:41:12 INFO common.c: Flash page at addr: 0x08000c00 erased
2020-09-16T10:41:12 INFO common.c: Flash page at addr: 0x08001000 erased
2020-09-16T10:41:12 INFO common.c: Flash page at addr: 0x08001400 erased
2020-09-16T10:41:12 INFO common.c: Flash page at addr: 0x08001800 erased
2020-09-16T10:41:12 INFO common.c: Flash page at addr: 0x08001c00 erased
2020-09-16T10:41:12 INFO common.c: Flash page at addr: 0x08002000 erased
2020-09-16T10:41:12 INFO common.c: Flash page at addr: 0x08002400 erased
2020-09-16T10:41:12 INFO common.c: Flash page at addr: 0x08002800 erased
2020-09-16T10:41:12 INFO common.c: Flash page at addr: 0x08002c00 erased
2020-09-16T10:41:12 INFO common.c: Flash page at addr: 0x08003000 erased
2020-09-16T10:41:12 INFO common.c: Finished erasing 13 pages of 1024 (0x400) bytes
2020-09-16T10:41:12 INFO common.c: Starting Flash write for VL/F0/F3/F1_XL core id
2020-09-16T10:41:12 INFO flash_loader.c: Successfully loaded flash loader in sram
13/13 pages written
2020-09-16T10:41:12 INFO common.c: Starting verification of write complete
2020-09-16T10:41:12 INFO common.c: Flash written and verified! jolly good!
```

# 壊れたSTM32F4Discovery

```
dspace@mini:~/develop/rust/tock-stm32$ st-info --probe
Found 0 stlink programmers
```

# 壊れていないSTM32F4Discovery

```
dspace@mini:~/develop/rust/tock-stm32$ st-info --probe
Found 1 stlink programmers
 serial:     303636434646333033343331353733343537313532323138
 hla-serial: "\x30\x36\x36\x43\x46\x46\x33\x30\x33\x34\x33\x31\x35\x37\x33\x34\x35\x37\x31\x35\x32\x32\x31\x38"
 flash:      1048576 (pagesize: 16384)
 sram:       196608
 chipid:     0x0413
 descr:      F4xx
