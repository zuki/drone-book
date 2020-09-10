# Lチカ

このセクションでは、システムクロック周波数を 72MHz に上げ、PC13 ピンに接続
されているオンボードLEDを点滅させるアプリケーションを書きます。このアプリケー
ションでは、複数のスレッド、future、ストリーム、メモリマップドレジスタ、
ペリフェラルを使用します。

<video autoplay loop muted width="100%">
<source src="./assets/blink.webm" type="video/webm" />
<source src="./assets/blink.mp4" type="video/mp4" />
</video>

この例のコードは[Github](https://github.com/drone-os/bluepill-blink)に
あります。

## プロジェクトの作成

まず、Blue Pillボードのための新規Droneプロジェクトを作成しましょう。

```shell
$ drone new \
        --toolchain nightly-2020-04-30 \ # 必要なrustupコンポーネントをすべて持つ
                                       \ # 最新のnightlyを選択する必要があります
        --device stm32f103 \ # マイクロコントローラ識別子
        --flash-size 128K \ # バイト単位のフラッシュメモリサイズ
        --ram-size 20K \ # バイト単位のRAMサイズ
        bluepill-blink # プロジェクト名
$ cd bluepill-blink
$ just deps
```

新規作成されたアプリケーションを簡単にテストするには、[Hello, world!](./hello-world.md)の
章で紹介したように、Black Magic ProbeをPCに接続し、Blue PillボードをBMPに
接続します。ファームウェアを書き込み、SWOの出力を確認します。

```shell
$ just flash
$ just log
```

"Hello, world!"と表示されたら、次の章に進みましょう。
