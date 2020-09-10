#  Blue PillをBlack Magic Probeにする

この章ではBlue PillボードからBlack Magic Probeを作成する過程を説明します。
この過程は、Ubuntu 18.04.3 LTSとArch Linux 5.3.7でテスト指定しています。

## 準備

この過程には次のパッケージがインストールされている必要があります。

```shell
$ sudo apt install build-essential \
                   curl \
                   dfu-util \
                   gcc-arm-none-eabi \
                   gdb-multiarch \
                   git \
                   python \
                   python-pip
```

Arch Linuxでは次の通り。

```
$ sudo pacman -S curl \
                 dfu-util \
                 git \
                 python \
                 python-pip
$ yaourt -S arm-none-eabi-gcc \
            gdb-multiarch
```

`dialout` グループのメンバーになっていると便利です。そうすると、BMPや
USB-to-UARTアダプタを使用する際にスーバーユーザ権限が不要になるからです。

```shell
$ sudo adduser $(id -un) dialout
```

Arch Linuxの場合は`uucp`グループに追加します。

```shell
$ sudo gpasswd -a $(id -un) uucp
```

グループの変更を有効にするために再ログインが必要になるかもしれません。

stm32loaderスクリプトを入手し、その実行に必要なpythonライブラリを
インストールします。

```shell
$ git clone https://github.com/jsnyder/stm32loader
$ pip install pyserial
```

BMPファームウェアを入手します。

```shell
$ git clone https://github.com/blacksphere/blackmagic
$ cd blackmagic
$ git submodule update --init --recursive
```

BMPリポジトリはこのプローブ用のudev規則を提供しています。規則は、udevに
GDBエンドポイントには`/dev/ttyBmpGdb`を、UARTには`/dev/ttyBmpTarg`を
シンボリックリンクするよう命令しています。また、スーパーユーザ権限なしに
BMPフィームウェアのアップデートができるように許可しています。

```shell
$ sudo cp driver/99-blackmagic.rules /etc/udev/rules.d/
$ sudo udevadm control --reload-rules
```

## ビルド

正しいプローブホストを選択する必要があります。ここでは、`swlink`です。

```shell
$ make PROBE_HOST=swlink
```

![Building](./assets/blackmagic-make.png)

これにより2つのバイナリ`src/blackmagic_dfu.bin`と`src/blackmagic.bin`が
生成されます。前者はブートローダであり、USB-to-UARTアダプタで書き込むものです。
後者は実際のファームウェアであり、ブートローダの助けを借りてUSB経由でロードする
ものです。

## ブートローダの書き込み

1. 次の表に従い、USB-to-UARTアダプタをBlue Pillに接続します。

   | USB-to-UART | Blue Pill |
   |-------------|-----------|
   | GND         | GND       |
   | RXD         | A9        |
   | TXD         | A10       |

   **警告:** まだ電源を接続しないでください。手順5で、USBを通じてボードに電源を
   供給します。5Vまたは3.3VピンとUSBを併用すると、ボードにダメージを与える
   可能性があります。

2. USB-to-UARTアダプタのジャンパをVCCと3V3がショートする位置に設定します。
   これにより、アダプタの出力電圧が3.3Vに設定されます。A9ピンとA10ピンは5ボルト
   耐圧なので厳密にはこれは不要です。

3. Blue PillのBOOT0ジャンパを1に設定して、工場出荷時にプログラムされたブート
   ローダからブートするようにします。このブートローダはUART経由でボードを
   プログラミングする役割を果たします。

![CH340G connected to Blue Pill](./assets/bluepill-ch340g.jpg)

4. USB-to-UARTアダプタをPCに接続する前に、システムジャーナルファイルを開きます。

   ```shell
   $ journalctl -f
   ```

   USB-to-UARTアダプタを接続すると、それに付与された名前がわかります。

![CH340G in journal](./assets/ch340g-journal.png)

1. USBケーブルをBlue Pillに接続し、書き込みを開始します。`/dev/ttyUSB0`は
   手順4で調べた名前に置き換えてください。処理が始まらない場合は、Blue Pillの
   リセットボタンを押下してください。

   ```shell
   $ ../stm32loader/stm32loader.py -p /dev/ttyUSB0 -e -w -v src/blackmagic_dfu.bin
   ```

![Successful load](./assets/stm32loader.png)

1. Blue PillのBOOT0ジャンパを0に戻します。

![Reset Blue Pill jumpers](./assets/bluepill-jumpers.jpg)

## ファームウェアの書き込み

この段階で、USB-to-UARTアダプタをBlue PillとPCから外すことができます。
ファームウェアはUSBポート経由で書き込みます。

```shell
$ dfu-util -d 1d50:6018,:6017 -s 0x08002000:leave -D src/blackmagic.bin
```

![Successful load](./assets/dfu-util.png)

稼働するかチェックします。Blue Pillを再接続し、GDBセッションを開きます。

```shell
$ gdb-multiarch
```

GDBのプロンプトで次のコマンドを入力します。

```text
target extended-remote /dev/ttyBmpGdb
monitor version
```

![GDB check](./assets/gdb-monitor-version.png)

出力が上のようになったら、おめでとう。これであなたのBlue PillはBlack Magic
Probeになりました。次回、ファームウェアのアップグレードが必要な時には、上の
`dfu-util`コマンドを再実行するだけです。

## ワイヤリング

以下は一般的なピンアウトの説明とBlue Pillとの接続例です。

| Black Magic Probe | Function            | Blue Pill Target |
|-------------------|---------------------|------------------|
| GND               | GND                 | GND              |
| SWCLK             | JTCK/SWCLK          | SWCLK            |
| SWIO              | JTMS/SWDIO          | SWIO             |
| A15               | JTDI                |                  |
| B3                | JTDO                |                  |
| B4                | JNTRST              | R                |
| B6                | UART1 TX            |                  |
| B7                | UART1 RX            | B3               |
| A3                | UART2 RX (TRACESWO) |                  |

![BMP wiring](./assets/bmp-wiring.jpg)

## 公式BMPとの比較

![Blue Pill and Official BMP](./assets/official-bmp-comparison.jpg)

公式BMPには利点がいくつかあります。There are a few advantages of the official BMP:

- Cortexデバッグコネクタの搭載
- ターゲットへの給電が可能
- ターベット電圧の感知が可能
- より多くのLED
- より堅牢な回路

これらの利点は重大なものではありません。しかし、公式のハードウェアを購入する
ことによりBMPプロジェクトをサポートすることができます。
