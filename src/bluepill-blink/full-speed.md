# 全速動作

データシートによると、STM32F103 MCUは最大72MHzの周波数で動作することができます。
しかし、デフォルトでは8MHzでしか動作しません。チップの最大能力を発揮させるには、
ランタイムでシステム周波数を上げる必要があります。

システムクロックソースには3つのオプションがあります。

- HSI (High Speed Internal) - 8MHzの定速で動作するMCUチップの内部にあるRC
  発振器です。これは起動時に選択されるデフォルトのシステムクロックソースです。

- HSE (High Speed External) - 4～16MHzのオプションの外部共振器です。
  Blue Pillボードには8MHzの水晶がMCUに接続されています（MCUの真横にあるY2と
  マークされた金属ケースに入っているコンポーネント）。

- PLL (Phase-Locked Loop) - HSIやHSEの乗算器として使用できるMCU内部にある
  ペリフェラル。HSIの最大乗数は8で64MHz、HSEの最大乗数は16で理論的には128MHzに
  なりますが、PLLの出力周波数は72MHzを超えることはできません。

以上のことから、72MHzを実現するためには、以下の手順を踏む必要があります。

1. HSE発振器を起動し、安定するのを待つ。
2. HSEを入力、乗数を9とするPLLを起動し、安定するのを待つ。
3. システムクロックのソースとしてPLLを選択する。

まず、プロジェクトレベルの定数用のモジュールを作成します。以下の内容の新しい
ファイル`src/consts.rs`を作成してください。

```rust
//! プロジェクト定数。

/// HSEクリスタル周波数。
pub const HSE_FREQ: u32 = 8_000_000;

/// PLL乗数ファクター。
pub const PLL_MULT: u32 = 9;

/// システムクロック周波数。
pub const SYS_CLK: u32 = HSE_FREQ * PLL_MULT;
```

そして、このモジュールを`src/lib.rs`に登録します。

```rust
pub mod consts;
```

アプリケーションがHSEやPLLのクロックが安定するまで待つ必要がある場合、フラグを
絶えずチェックしてCPUサイクルやエネルギーを無駄にしたくはありません。その代わりに
割り込みを登録して、その割り込みが発生するまでスリープしたいので、RCC割り込みを
使用します。

![Vector Table](../assets/vtable-rcc.png)

リファレンスマニュアルに載っている上の表のうちRCC割り込みの位置だけが必要です。
この割り込みをアプリケーションのベクタテーブルに入れましょう。そのためには、
`src/thr.rs`にある`thr::vtable!`マクロを編集する必要があります。デフォルトでは
以下のようになっています。

```rust
thr::vtable! {
    // ... ヘッダーはスキップ ...

    // --- アロケート済みのスレッド ---

    /// 。全フォールトクラス
    pub HARD_FAULT;
}
```

HardFaultハンドラだけが定義されています。上の表によると、HardFaultは位置番号を
持たないので、名前だけで参照されています。5の位置に新しい割り込みハンドラを追加
する必要があります。

```rust
thr::vtable! {
    // ... ヘッダーはスキップ ...

    // --- アロケート済みのスレッド ---

    /// 全フォールトクラス.
    pub HARD_FAULT;
    /// RCCグローバル割り込み.
    pub 5: RCC;
}
```

新しい割り込みは数値の位置番号を持っているので任意の名前を付けられます。

`src/tasks/root.rs`にあるルートタスクハンドラを開きます。デフォルトでは
次のようになっています。

```rust
//! ルートタスク.

use crate::{thr, thr::ThrsInit, Regs};
use drone_cortexm::{reg::prelude::*, thr::prelude::*};

/// ルートタスクハンドラ.
#[inline(never)]
pub fn handler(reg: Regs, thr_init: ThrsInit) {
    let thr = thr::init(thr_init);

    thr.hard_fault.add_once(|| panic!("Hard Fault"));

    println!("Hello, world!");

    // ISRを抜ける際にスリープ状態に入る.
    reg.scb_scr.sleeponexit.set_bit();
}
```

Drone OSでは、最低の優先度を持つ最初のタスクをrootと呼びます。その関数ハンドラは、
安全でない初期化ルーチンの終了後に`src/bin.rs`にあるプログラムのエントリポイント
から呼び出されます。rootハンドラは`Regs`型と`ThrsInit`型の2つの引数を受け取り
ます。両者とも、一度に一つのインスタンスの存在しか許さない`Token`トレイトを実装
しているゼロサイズの型です。`Token`型のインスタンス化は安全ではないので、
`src/bin.rs` の安全でないエントリポイント内で行われます。

引数`reg`は，利用可能な全てのメモリマップドレジスタを表すトークンセットです。
これは、ルートハンドラ内で個々のレジスタトークンやレジスタフィールドトークンに
分解されることになります。

第2引数`thr_init`の目的はそれを`thr::init`関数に渡すことです。この関数は
スレッドシステムの初期化ルーチンを実行し、`Thrs`型のインスタンスを返します。
`Thrs`も`Regs`同様、ゼロサイズの型ですが、スレッドトークンのための型です。

ルートハンドラはHardFaultスレッドに単発のファイバを追加します。ファイバ本体は
`panic!`マクロを呼び出すだけです。Droneは、パニックメッセージをポート#1のログ
出力に書き込み、セルフリセット要求を発行し、それが実行されるまでブロックすると
いう手順でパニックを処理します。

システムクロック周波数を72MHzに上げるための新しい`async`関数を追加しましょう。
この関数にはRCCとFLASHペリフェラルのレジスタ、およびRCCスレッドトークンが必要です。

```rust
//! ルートタスク.

use crate::{
    consts::{PLL_MULT, SYS_CLK},
    thr,
    thr::ThrsInit,
    Regs,
};
use drone_core::log;
use drone_cortexm::{fib, reg::prelude::*, swo, thr::prelude::*};
use drone_stm32_map::reg;

/// ルートタスクハンドラ.
#[inline(never)]
pub fn handler(reg: Regs, thr_init: ThrsInit) {
    let thr = thr::init(thr_init);

    thr.hard_fault.add_once(|| panic!("Hard Fault"));

    raise_system_frequency(
        reg.flash_acr,
        reg.rcc_cfgr,
        reg.rcc_cir,
        reg.rcc_cr,
        thr.rcc,
    )
    .root_wait();

    println!("Hello, world!");

    // ISRを抜ける際にスリープ状態に入る.
    reg.scb_scr.sleeponexit.set_bit();
}

async fn raise_system_frequency(
    flash_acr: reg::flash::Acr<Srt>,
    rcc_cfgr: reg::rcc::Cfgr<Srt>,
    rcc_cir: reg::rcc::Cir<Srt>,
    rcc_cr: reg::rcc::Cr<Srt>,
    thr_rcc: thr::Rcc,
) {
    // TODO 周波数を72MHzに上げる
}
```

`async`関数は`Future`トレイトを実装しているカスタム型を返す関数のシンタックス
シュガーです。返されたfutureを`.root_wait()`メソッドを使って実行します。`root_wait`メソッドは、ルートタスクコンテキストなど、最も低い優先度の
スレッド内で使用することになっています。そうでない場合、現在プリエンプション
されているスレッドがストールする可能性があります。futureを実行するための
もう一つのオプションは、スレッドトークンの`exec`や`add_exec`メソッドを使う
ことです。

プログラムがまだ動作していることを確認するのが良いでしょう。

```shell
$ just flash
$ just log
```

それでは、`raise_system_frequency`関数を書いていきましょう。まず、NVIC
（Nested Vectored Interrupt Controller）でRCC割り込みを有効にする必要が
あります。そして、HSEとPLLが安定したらRCCペリフェラルが割り込みをトリガできる
ようにします。

```rust
    thr_rcc.enable_int();
    rcc_cir.modify(|r| r.set_hserdyie().set_pllrdyie());
```

次は、HSEクロックを有効にし、それが安定するまで待つようにします。

```rust
    // `hserdyc`と`hserdyf`の所有権をファイバに移す必要がある.
    let reg::rcc::Cir {
        hserdyc, hserdyf, ..
    } = rcc_cir;
    // RCC_CIR_HSERDYFがアサートされたら通知するリスナーをアタッチする.
    let hserdy = thr_rcc.add_future(fib::new_fn(move || {
        if hserdyf.read_bit() {
            hserdyc.set_bit();
            fib::Complete(())
        } else {
            fib::Yielded(())
        }
    }));
    // HSEクロックを有効にする.
    rcc_cr.modify(|r| r.set_hseon());
    // RCC_CIR_HSERDYFがアサートされるまでスリープ.
    hserdy.await;
```

同様にPLLも有効にします。

```rust
    // `pllrdyc`と`pllrdyf`の所有権をファイバに移す必要がある.
    let reg::rcc::Cir {
        pllrdyc, pllrdyf, ..
    } = rcc_cir;
    // RCC_CIR_PLLRDYFがアサートされたら通知するリスナーをアタッチする.
    let pllrdy = thr_rcc.add_future(fib::new_fn(move || {
        if pllrdyf.read_bit() {
            pllrdyc.set_bit();
            fib::Complete(())
        } else {
            fib::Yielded(())
        }
    }));
    rcc_cfgr.modify(|r| {
        r.set_pllsrc() // HSEオシレータクロックをPLLの入力クロックとして選択
            .write_pllmul(PLL_MULT - 2) // 出力周波数 = 入力クロック × PLL_MULT
    });
    // PLLを有効にする.
    rcc_cr.modify(|r| r.set_pllon());
    // RCC_CIR_PLLRDYFがアサートされるまでスリープ.
    pllrdy.await;
```

周波数が上がったのでフラッシュメモリの設定を修正する必要があります。

```rust
    // 48 MHz < SYS_CLK <= 72 Mhzの場合は、2ウェイトステート
    flash_acr.modify(|r| r.write_latency(2));
```

周波数を上げる前に、現在進行中のSWO送信があれば終了するまで待つ必要があります。
また、プロジェクトの`Drone.toml`で定義されている固定ボーレートを維持するように
SWOのプリスケーラを変更します。デバッグプローブが接続されていない場合、これは
no-opとなるので、リリースバイナリではこのままでも安全であることに注意してください。

```rust
    swo::flush();
    swo::update_prescaler(SYS_CLK / log::baud_rate!() - 1);
```

最後に、システムクロックのソースをPLLに切り替えます。

```rust
    rcc_cfgr.modify(|r| r.write_sw(0b10)); // PLLをシステムクロックとして選択
```

以下が`raise_system_frequency`関数の最終リストです。

```rust
async fn raise_system_frequency(
    flash_acr: reg::flash::Acr<Srt>,
    rcc_cfgr: reg::rcc::Cfgr<Srt>,
    rcc_cir: reg::rcc::Cir<Srt>,
    rcc_cr: reg::rcc::Cr<Srt>,
    thr_rcc: thr::Rcc,
) {
    thr_rcc.enable_int();
    rcc_cir.modify(|r| r.set_hserdyie().set_pllrdyie());

    // We need to move ownership of `hserdyc` and `hserdyf` into the fiber.
    let reg::rcc::Cir {
        hserdyc, hserdyf, ..
    } = rcc_cir;
    // Attach a listener that will notify us when RCC_CIR_HSERDYF is asserted.
    let hserdy = thr_rcc.add_future(fib::new_fn(move || {
        if hserdyf.read_bit() {
            hserdyc.set_bit();
            fib::Complete(())
        } else {
            fib::Yielded(())
        }
    }));
    // Enable the HSE clock.
    rcc_cr.modify(|r| r.set_hseon());
    // Sleep until RCC_CIR_HSERDYF is asserted.
    hserdy.await;

    // We need to move ownership of `pllrdyc` and `pllrdyf` into the fiber.
    let reg::rcc::Cir {
        pllrdyc, pllrdyf, ..
    } = rcc_cir;
    // Attach a listener that will notify us when RCC_CIR_PLLRDYF is asserted.
    let pllrdy = thr_rcc.add_future(fib::new_fn(move || {
        if pllrdyf.read_bit() {
            pllrdyc.set_bit();
            fib::Complete(())
        } else {
            fib::Yielded(())
        }
    }));
    rcc_cfgr.modify(|r| {
        r.set_pllsrc() // HSE oscillator clock selected as PLL input clock
            .write_pllmul(PLL_MULT - 2) // output frequency = input clock × PLL_MULT
    });
    // Enable the PLL.
    rcc_cr.modify(|r| r.set_pllon());
    // Sleep until RCC_CIR_PLLRDYF is asserted.
    pllrdy.await;

    // Two wait states, if 48 MHz < SYS_CLK <= 72 Mhz.
    flash_acr.modify(|r| r.write_latency(2));

    swo::flush();
    swo::update_prescaler(SYS_CLK / log::baud_rate!() - 1);

    rcc_cfgr.modify(|r| r.write_sw(0b10)); // PLL selected as system clock
}
```
