# Hello, world!

前の章ではBlue Pillでデバッグプローブを作成し、別のBlue Pillに接続しました。
この章では、このマイクロコントローラ上で初めてのDroneプログラムを実行します。

## Rust

まだRustをインストールされていない場合は、[rustup.rs](https://rustup.rs/)
からインストールしてください。Droneは現在のところ、NightlyチャンネルのRustで
しか利用できません。まず、以下をインストールする必要があります。

```shell
$ rustup toolchain install nightly \
      -c rust-src -c rustfmt -c clippy -c llvm-tools-preview \
      -t thumbv7m-none-eabi
```

すべてのnightlyリリースですべてのコンポーネントが利用できるわけではありません。
上記のコマンドは必要なコンポーネントがすべて利用できる最新のリリースを時間を遡って
探して行きます。

## `just` コマンド

組込み開発では、時々実行する必要のある様々なプロジェクト固有のタスクがある
場合がしばしばあります。そのため、優れたRustクレートである
[`just`](https://github.com/casey/just)の使用をお勧めします。


```shell
$ cargo +stable install just
```

Justは`make`にインスパイアされたコマンドランナーです。プロジェクトのルートに
`Justfile`がある場合はいつでも`just --list`を実行すると、利用可能なすべての
コマンドを知ることができます。さらに、`drone new`コマンドは`Justfile`を
生成します。シェルの設定に`alias j="just"`を加えることをお勧めします。
そうすれば、`just`ではなく`j`とタイプするだけですみます。


## `drone`コマンド

Drone OSプロジェクトは多くのRustクレートで構成されています。しかし、そのための
入り口が一つあります。`drone`コマンドラインユーティリティです。

```shell
$ cargo +nightly install drone
```

これで事前に必要なものはすべて整いましたので、次の段階に進めることができます。
初めてのDroneクレートの作成です。

## 新規プロジェクト

`drone`に新規Droneクレートを作成するよう伝えましょう。ここでは。ターゲットと
なるMCUファミリー（Blue Pillでは`stm32f103`）、フラッシュメモリーサイズ、
RAMサイズとプロジェクト名を指定する必要があります。

```shell
$ drone new --device stm32f103 --flash-size 128K --ram-size 20K hello-world
$ cd hello-world
```

プロジェクトの中で最初に行うことは依存パッケージのインストールです。

```shell
$ just deps
```

Rustを更新したあとにもいつもこのタスクを実行するべきです。

ここでは（先の章で説明したように）次のように2つのBlue Pillを接続したと
仮定します。

![BMP wiring](./assets/bmp-wiring.jpg)

新たに作成したプロジェクトをターゲットとなるBlue Pillに書き込みましょう。
初めてビルトする際には時間がかかる場合があります。


```shell
$ just flash
```

成功した場合は、次のようになります。

![Flash success](./assets/just-flash.png)

最後に、デバイスからのSWO出力をチェックしてください。

```shell
$ just log
```

![SWO output](./assets/just-swo.png)

出力が上のようになっていたら、おめでとうございます。Droneプロジェクトの開発環境が
正しく設定されています。
