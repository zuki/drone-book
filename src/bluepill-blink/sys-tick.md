# タイマー利用

この章では、Blue Pill基板の緑色LEDに接続されているPC13ピンを定期的にアサート・
ディアサートするためにタイマーペリフェラルを使用します。STM32F103 MCUは、4種類、
7個のタイマーを搭載しています。ここでは、すべてのCortex-M MCUに存在する
SysTickタイマーを使用します。

Droneはすでに`drone_cortexm::drv::timer::Timer`トレイトの形でタイマー
ペリフェラルのための汎用インターフェースを持っており、
`drone_cortexm::drv::sys_tick::SysTick`というSysTickドライバの実装も
持っています。しかし、この例では、割り込みとメモリマップドレジスタを直接使用します。

まず、タイマペリフェラルで使用する割り込みを割り当てる必要があります。
リファレンスマニュアルを参照しましょう。

![Vector Table](../assets/vtable-sys-tick.png)

前章のRCC割り込みとは異なり、SysTickは位置値を持ちません。つまり、正確な名前を
使って、位置番号を持つすべての割り込みの前に宣言する必要があります。

```rust
thr::vtable! {
    // ... ヘッダーはスキップ ...

    // --- アロケート済みのスレッド ---

    /// 全フォールトクラス.
    pub HARD_FAULT;
    /// システムティックタイマー.
    pub SYS_TICK;
    /// RCCグローバル割り込み.
    pub 5: RCC;
}
```

リファレンスマニュアルによれば、SysTickクロックの周波数はシステムクロックを
8で割った値です。これを定数モジュール`src/consts.rs`に追加しましょう。

```rust
/// SysTickクロック周波数.
pub const SYS_TICK_FREQ: u32 = SYS_CLK / 8;
```

ルートハンドラを変更しましょう。

```rust
//! ルートタスク.

use crate::{
    consts::{PLL_MULT, SYS_CLK, SYS_TICK_FREQ},
    thr,
    thr::ThrsInit,
    Regs,
};
use drone_core::log;
use drone_cortexm::{fib, reg::prelude::*, swo, thr::prelude::*};
use drone_stm32_map::{
    periph::{
        gpio::{periph_gpio_c, GpioC, GpioPortPeriph},
        sys_tick::{periph_sys_tick, SysTickPeriph},
    },
    reg,
};
use futures::prelude::*;

/// レシーバがあまりにも多くのティックを逃した場合はエラーが返される.
#[derive(Debug)]
pub struct TickOverflow;

/// ルートタスクハンドラ.
#[inline(never)]
pub fn handler(reg: Regs, thr_init: ThrsInit) {
    let thr = thr::init(thr_init);
    let gpio_c = periph_gpio_c!(reg);
    let sys_tick = periph_sys_tick!(reg);

    thr.hard_fault.add_once(|| panic!("Hard Fault"));

    raise_system_frequency(
        reg.flash_acr,
        reg.rcc_cfgr,
        reg.rcc_cir,
        reg.rcc_cr,
        thr.rcc,
    )
    .root_wait();

    beacon(gpio_c, sys_tick, thr.sys_tick)
        .root_wait()
        .expect("beacon fail");

    // ISRを抜ける際にスリープ状態に入る.
    reg.scb_scr.sleeponexit.set_bit();
}

// この関数はそのまま.
async fn raise_system_frequency(...) {...}

async fn beacon(
    gpio_c: GpioPortPeriph<GpioC>,
    sys_tick: SysTickPeriph,
    thr_sys_tick: thr::SysTick,
) -> Result<(), TickOverflow> {
    Ok(())
}
```

エラー型`TickOverflow`を追加しました。これについては後で検討します。

```rust
#[derive(Debug)]
pub struct TickOverflow;
```

ルートハンドラのはじめに2つの`periph_*!`マクロコールを追加しました。これらの
マクロは`reg`構造体の一部を取り、それを `gpio_c`構造体と`sys_tick`構造体に
移動させます。`reg`、`gpio_c`、`sys_tick`はいずれもZST（ゼロサイズ型）なので
このマクロは実行時には何もしませんが、所有権が移動されたことを型システムに通知
します。

```rust
    let gpio_c = periph_gpio_c!(reg);
    let sys_tick = periph_sys_tick!(reg);
```

これらの構造体は、対応するペリフェラルに関連するすべてのレジスタを保持しています。
これらのペリフェラル構造体を`beacon`という名前の新しい`async`関数に渡します。
今回、この関数は`Result`型を返し、これをパニックで処理します。

```rust
    beacon(gpio_c, sys_tick, thr.sys_tick)
        .root_wait()
        .expect("beacon fail");
```

`beacon`関数を書いていきましょう。SysTickタイマーが毎秒SysTick割り込みを
発生するよう構成します。

```rust
    // 割り込みが発生するたびに通知するリスナをアタッチする。
    let mut tick_stream = thr_sys_tick.add_stream_pulse(
        // このクロージャは、最後のストリームポーリング以降のティック数を
        // レシーバが保持できなくなった場合に呼び出される。その場合、
        // 最終的な値として `TickOverflow` エラーがストリーム上に送信される。
        || Err(TickOverflow),
        // 割り込みが発生するごとに呼び出されるファイバ。1 tickを
        // ストリームに送信する。
        fib::new_fn(|| fib::Yielded(Some(1))),
    );
    // タイマーの現在値をクリアする。
    sys_tick.stk_val.store(|r| r.write_current(0));
    // カウンタが0になった際に`stk_val`にロードする値を設定する。
    // 1秒当たりのSysTickクロックの回数を設定することで、
    // 1秒毎にリロードがトリガーされる。
    sys_tick.stk_load.store(|r| r.write_reload(SYS_TICK_FREQ));
    sys_tick.stk_ctrl.store(|r| {
        r.set_tickint() // カウントダウンし、0になったらSysTick割り込みを発生
            .set_enable() // カウンターを開始
    });
```

ここまでで`tick_stream`変数が`Stream`型のインスタンスを保持するようになりました。
ストリームの各項目を`await`し、終了を待ちます。`tick`変数は、最後のストリーム
ポーリングから経過したパルス数（ここでは秒数）です。スレッドが大幅に中断されて
いなければ、通常は単に`1`であると想定されます。

```rust
    while let Some(tick) = tick_stream.next().await {
        for _ in 0..tick?.get() {
            println!("sec");
        }
    }
```

このプログラムを書き込んで、SW0出力を見てみましょう。

```shell
$ just flash
$ just log
```

次のような出力になるはずです。 "sec"行が毎秒ごとに無限にプリントされます。

```text
================================== LOG OUTPUT ==================================
sec
sec
sec
sec
sec
```

いよいよGPIOペリフェラルを使って、Blue Pillの緑色LEDを駆動する時がきました。

![Blue Pill Schematics](../assets/bluepill-schematics-leds.png)

上のBlue Pillの回路図によると、PC13がLow（GNDにショート）の時はD2に電流が流れ、
High（VCCにショート）の時は流れません。PC13ピンを設定しましょう。以下を
`beacon`関数の先頭に置きます。

```rust
    gpio_c.rcc_busenr_gpioen.set_bit(); // GPIOポートCのクロックを有効化
    gpio_c.gpio_crh.modify(|r| {
        r.write_mode13(0b10) // 出力モード、最高速度 2 MHz
            .write_cnf13(0b00) // 汎用出力プッシュプル
    });
```

125ミリ秒ごとに起きるようにタイマーの速度を上げましょう。`stk_load`の
初期化コードを次のように変更します。

```rust
    // カウンタが0になったときに `stk_val`レジスタにロードする値を設定する。
    // ここでは、1秒あたりのSysTickクロック数を8で割った値を設定しているので、
    // 125msごとにリロードがトリガーされることになる。
    sys_tick
        .stk_load
        .store(|r| r.write_reload(SYS_TICK_FREQ / 8));
```

ストリームループを変更します。

```rust
    // 0から7まで周期する値。1周が1秒を表す。
    let mut counter = 0;
    while let Some(tick) = tick_stream.next().await {
        for _ in 0..tick?.get() {
            // 各秒ごとにメッセージをプリント.
            if counter == 0 {
                println!("sec");
            }
            match counter {
                // 0ミリ秒と250ミリ秒の時にピンをlowにする
                0 | 2 => {
                    gpio_c.gpio_bsrr.br13.set_bit();
                }
                // 125, 375, 500, 625, 750, 875の各ミリ秒のときは
                // ピンをhighにする
                _ => {
                    gpio_c.gpio_bsrr.bs13.set_bit();
                }
            }
            counter = (counter + 1) % 8;
        }
    }
```

次のコマンドでアプリケーションをBlue Pillに書き込みます。

```shell
$ just flash
```

すると次のような結果になるはずです。

<video autoplay loop muted width="100%">
<source src="../assets/blink.webm" type="video/webm" />
<source src="../assets/blink.mp4" type="video/mp4" />
</video>

このアプリケーションの全コードは
[Github](https://github.com/drone-os/bluepill-blink)にあります。
