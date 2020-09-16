# Drone CLI

チップのサポートに関して、Drone CLIは以下を担当します。

1. 新規プロジェクトのための正しいscaffordを生成すること。生成されたプログラムは
   チップにフラッシュする準備ができている。プログラムは、標準出力に文字列`"Hello, world!"`を
   表示する。

2. 正しいリンカスクリプトを生成すること。

3. 1つ以上のデバッグプローブを使ってチップを操作すること。

4. 少なくとも1つのDroneロガー出力をキャプチャする方法。

すべてのプラットフォーム固有のクレートは`drone/src/crates.rs` に登録する必要が
あります。これには、プラットフォームクレート（たとえば、`drone-cortexm`）、
ベンダー固有のマッピング（たとえば、`drone-stm32-map`や`drone-nrf-map`）、
DSO（Drone Serial Output）を実装したクレート（たとえば、`drone-nrf91-dso`）が
含まれます。

特定のマイクロコントローラモデルは`drone/src/devices/registry.rs`に登録する
必要があります。たとえば、以下はNordic Semiconductor社製nRF9160のエントリです。

```rust
    Device {
        name: "nrf9160", // デバイス識別子
        target: "thumbv8m.main-none-eabihf", // Rustターゲットトリプル
        flash_origin: 0x0000_0000, // Flashメモリの開始アドレス
        ram_origin: 0x2000_0000, // RAMの開始アドレス
        // 固有のフラグや機能を持つプラットフォームクレートへのリンク
        platform_crate: PlatformCrate {
            krate: crates::Platform::Cortexm,
            flag: "cortexm33f_r0p2",
            features: &[
                "floating-point-unit",
                "memory-protection-unit",
                "security-extension",
            ],
        },
        // 固有のフラグや機能を持つバインディングクレートへのリンク
        bindings_crate: BindingsCrate {
            krate: crates::Bindings::Nrf,
            flag: "nrf9160",
            features: &["uarte"],
        },
        probe_bmp: None, // BMPは未サポート
        probe_openocd: None, // OpenOCDは未サポート
        probe_jlink: Some(ProbeJlink { device: "NRF9160" }), // J-Link設定
        log_swo: None, // SWOは未サポート
        // DSO実装へのリンク
        log_dso: Some(LogDso { krate: crates::Dso::Nrf91, features: &[] }),
    },
```

`drone` CLI は、様々なデバッグプローブへの統一されたインターフェースを提供します。
現在サポートされているデバッグプローブには、Black Magic Probe、J-Link、OpenOCDの
3種類がありますが、それ自体が異なるデバッガへのインターフェースとなっています。
新しいチップをDroneでサポートするためには、現在知られているプローブの一つを使って
チップを使用するための方法をCLIユーティリティに教える必要があります。あるいは、
そのチップのために全く新しいプローブのサポートを追加することもできます。

CLIユーティリティには内蔵のDroneロガーからデータを捕捉する役割もあります。現在、
SWO（ARMのSerial Wire Output）とDSO（Drone Serial Output）の2つのプロトコル
パーサが実装されています。DSOプロトコルは、チップにハードウェアプロトコルが実装
されていない場合に使用されます。ログ出力は、プローブに内蔵されているリーダ、または
汎用の外部UARTリーダを通じて捕捉することができます。新しいチップでは少なくとも
1つのログメソッドが実装されている必要があります。
