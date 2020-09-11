# プロセス

Drone OSのプロセスは特別なブロッキングコールで中断することができる特殊な種類の
ファイバです。プロセスは動的に割り当てられた専用のスタックを使用します。
Cortex-Mプラットフォームにおいて、Droneは`SVC`アセンブリ命令とSVCall例外を
使ってプロセスを実装しています。そのため、プロセスを使用する前に、Droneスーパー
バイザをプロジェクトに追加する必要があります。

## スーパバイザ

次の内容の新規ファイルを`src/sv.rs`に作成します。

```rust
//! スーパバイザ.

use drone_cortexm::{
    sv,
    sv::{SwitchBackService, SwitchContextService},
};

sv! {
    /// スーパバイザ型.
    pub struct Sv;

    /// サービスの配列.
    static SERVICES;

    SwitchContextService;
    SwitchBackService;
}
```

そして、新規作成したモジュールを`src/lib.rs`に登録します。

```rust
pub mod sv;
```

`src/thr.rs`にある`thr::vtable!`マクロを次のように変更します。

```rust
use crate::sv::Sv;

thr::vtable! {
    use Thr;
    use Sv; // <-- スーパバイザ型を登録

    // ... 型定義はスキップ ...

    // --- 割り当て済みのスレッド ---

    /// 全フォールトクラス.
    pub HARD_FAULT;
    /// システムサービスコール.
    fn SV_CALL; // <-- 新規外部割り込みハンドラを追加
}
```

また、SVCall用の外部ハンドラをアタッチするために`src/bin.rs`を変更する必要があります。

```rust
use drone_cortexm::sv::sv_handler;
use CRATE_NAME::sv::Sv;

/// ベクタテーブル.
#[no_mangle]
pub static VTABLE: Vtable = Vtable::new(Handlers {
    reset,
    sv_call: sv_handler::<Sv>,
});
```

## プロセスを使用する

まず、前章のジェネレータファイバの例を思い出しましょう。

```rust
use core::pin::Pin;
use drone_cortexm::{
    fib,
    fib::{Fiber, FiberState},
};

let mut fiber = fib::new(|| {
    let mut state = 0;
    while state < 3 {
        state += 1;
        yield state;
    }
    state
});
assert_eq!(Pin::new(&mut fiber).resume(()), FiberState::Yielded(1));
assert_eq!(Pin::new(&mut fiber).resume(()), FiberState::Yielded(2));
assert_eq!(Pin::new(&mut fiber).resume(()), FiberState::Yielded(3));
assert_eq!(Pin::new(&mut fiber).resume(()), FiberState::Complete(3));
```

このファイバはDroneプロセスを使って次のように書き換えることができます。

```rust
use crate::sv::Sv;
use core::pin::Pin;
use drone_cortexm::{
    fib,
    fib::{Fiber, FiberState},
};

let mut fiber = fib::new_proc::<Sv, _, _, _, _>(128, |_, yielder| {
    let mut state = 0;
    while state < 3 {
        state += 1;
        yielder.proc_yield(state);
    }
    state
});
assert_eq!(Pin::new(&mut fiber).resume(()), FiberState::Yielded(1));
assert_eq!(Pin::new(&mut fiber).resume(()), FiberState::Yielded(2));
assert_eq!(Pin::new(&mut fiber).resume(()), FiberState::Yielded(3));
assert_eq!(Pin::new(&mut fiber).resume(()), FiberState::Complete(3));
```

違いは、クロージャ引数の内部コードが完全に同期化されていることです。`proc_yield`
コールは`SVC`アセンブリ命令に変換されます。この命令は実行コンテキストを直ちに再び
呼び出し元に切り替えます。プロセスの`resume`メソッドが呼び出されると、ジェネレータ
と同じように最後のyield点から継続します。

`fib::new_proc`関数は、スタックサイズを第一引数に取ります。スタックはすぐに
ヒープから確保されます。この関数を安全にするために、オーバーフローの可能性から
スタックを保護するためにプロセッサのMPUが使用されます。STM32F103のようなMPUを
持たないプロセッサでは、この関数はパニックになります。しかし、スタックオーバー
フローに関する保証はありませんが、そのようなシステムでもプロセスを使用することは
可能です。`unsafe`とマークされている`new_proc_unchecked`関数を使うことが
できます。

ジェネレータとは異なり、プロセスは入力データを受け取ることができます。また、
`yield`キーワードとは異なり、`proc_yield`関数は必ずしも`()`を返すわけでは
ありません。以下にそのようなプロセスの例を示します。

```rust
let mut fiber = fib::new_proc::<Sv, _, _, _, _>(128, |input, yielder| {
    let mut state = input;
    while state < 4 {
        state += yielder.proc_yield(state);
    }
    state
});
assert_eq!(Pin::new(&mut fiber).resume(1), FiberState::Yielded(1));
assert_eq!(Pin::new(&mut fiber).resume(2), FiberState::Yielded(3));
assert_eq!(Pin::new(&mut fiber).resume(3), FiberState::Complete(6));
```
