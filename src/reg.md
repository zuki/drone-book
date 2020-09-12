# メモリマップドレジスタ

現代のプロセッサアーキテクチャ（ARMなど）は、CPUとペリフェラル間の通信を行うために
メモリマップドI/Oを使用します。メモリマップドレジスタの使用はマイクロコントローラの
プログラミングにおいてかなりの部分を占めています。そのため、Drone OSは複雑なAPIを
提供しており、それらに対するデータ競合のない便利なアクセスを提供してます。

たとえば、STM32F103ではメモリアドレス`0x4001100C`はGPIOC_ODRレジスタに対応して
います。これはGPIOポートCペリフェラルの出力状態を制御するためのレジスタです。

```rust
use core::ptr::write_volatile;

unsafe {
    write_volatile(0x4001_100C as *mut u32, 1 << 13);
}
```

上記のコードは、Droneを使用せず、生のRustでメモリマップドレジスタに書き込む方法の
一例です。このコードは、PC13ピンの出力をロジックハイに設定しています（ポートCの
他のすべてのピンはロジックローにリセットしています）。このコードはあまりにも低レベル
かつエラーを引き起こしやすい上に`unsafe`ブロックを必要とします。

Cortex-M では、SVD (System View Description) フォーマットがあります。通常、
ベンダーは自社のCortex-M MCU用にこのフォーマットのファイルを提供しています。
DroneはサポートしているターゲットごとにこれらのファイルからMCU固有のレジスタAPIを
生成します。そのため、一般的に、リファレンスマニュアルからアドレスやオフセットを
コピーする必要はありません。

プログラムのエントリーポイントである`src/bin.rs`のデフォルトの`reset`関数を
見てみましょう。

```rust
#[no_mangle]
#[naked]
pub unsafe extern "C" fn reset() -> ! {
    mem::bss_init();
    mem::data_init();
    tasks::root(Regs::take(), ThrsInit::take());
    loop {
        processor::wait_for_int();
    }
}
```

この`unsafe`関数は、安全な`root`エントリタスクを呼び出す前に必要なすべての初期化
ルーチンを実行します。これには `Regs::take()`と`ThrsInit::take()`の呼び出しが
含まれています。これらの呼び出しは、ゼロサイズの型である`Regs`と`ThrsInit`の
インスタンスを作成します。これらの呼び出しは`unsafe`です。プログラム全体の
ランタイム中に一度だけ行わなれなければならないからです。

次に、`tasks::root`関数（`handler`から再エクスポートされています）を確認しましょう。

```rust
pub fn handler(reg: Regs, thr_init: ThrsInit) {
    // ISRの終了時にスリープ状態にする
    reg.scb_scr.sleeponexit.set_bit();
}
```

`reg`はオープン構造体（構造体のすべてのフィールドが`pub`）であり、利用可能なすべての
レジスタトークンで構成されています。各レジスタトークンもオープン構造体であり、
レジスタフィールドトークンで構成されています。したがって、次の行

```rust
    reg.scb_scr.sleeponexit.set_bit();
```

は、SCB_SCRレジスタのSLEEPONEXITビットを設定します。

もちろん、現実のアプリケーションでは利用可能なすべてのメモリマップドレジスタを
使用することはありません。`reg`オブジェクトは`root`タスクハンドラ内で開放され、
自動的に削除されることになっています。これをより読みやすくするために、マクロを
使い、`reg`から個々のトークンを論理ブロックに移動させています。

```rust
pub fn handler(reg: Regs, thr_init: ThrsInit) {
    let gpio_c = periph_gpio_c!(reg);
    let sys_tick = periph_sys_tick!(reg);
    beacon(gpio_c, sys_tick)
}
```

これらのマクロはRustの部分移動機能を利用しており、大まかには以下のように展開されます。

```rust
pub fn handler(reg: Regs, thr_init: ThrsInit) {
    let gpio_c = GpioC {
        gpio_crl: reg.gpio_crl,
        gpio_crh: reg.gpio_crh,
        gpio_idr: reg.gpio_idr,
        gpio_odr: reg.gpio_odr,
        // 以下は個々のフィールであることに注意
        // 他のAPB2ペリフェラルは同じレジスタから別のフィールド取り出すことが可能
        rcc_apb2enr_iopcen: reg.rcc_apb2enr.iopcen,
        rcc_apb2enr_iopcrst: reg.rcc_apb2enr.iopcrst,
        // ...
    };
    let sys_tick = SysTick {
        stk_ctrl: reg.stk_ctrl,
        stk_load: reg.stk_load,
        stk_val: reg.stk_val,
        scb_icsr_pendstclr: reg.scb_icsr.pendstclr,
        scb_icsr_pendstset: reg.scb_icsr.pendstset,
    };
    beacon(gpio_c, sys_tick)
}
```

関数ではなく、何故マクロを使うのか疑問に思ったかもしれませんが、次の例は関数が
機能しない理由を示しています。

```rust
fn periph_gpio_c(reg: Regs) -> GpioC {
    GpioC {
        gpio_crl: reg.gpio_crl,
        gpio_crh: reg.gpio_crh,
        gpio_idr: reg.gpio_idr,
        gpio_odr: reg.gpio_odr,
        // Notice that below are individual fields.
        // Other APB2 peripherals may take other fields from this same registers.
        rcc_apb2enr_iopcen: reg.rcc_apb2enr.iopcen,
        rcc_apb2enr_iopcrst: reg.rcc_apb2enr.iopcrst,
        // ...
    }
}

fn periph_sys_tick(reg: Regs) -> GpioC {
    SysTick {
        stk_ctrl: reg.stk_ctrl,
        stk_load: reg.stk_load,
        stk_val: reg.stk_val,
        scb_icsr_pendstclr: reg.scb_icsr.pendstclr,
        scb_icsr_pendstset: reg.scb_icsr.pendstset,
    }
}

pub fn handler(reg: Regs, thr_init: ThrsInit) {
            // --- 移動が発生。なぜなら、`reg`は`Regs`型を持つが、
            //    `Regs`は｀Copy｀トレイトを実装していないため。
    let gpio_c = periph_gpio_c!(reg);
                             // --- ここで値が移動する
    let sys_tick = periph_sys_tick!(reg);
                                 // --- 移動後の値をここで使用している
    beacon(gpio_c, sys_tick)
}
```
