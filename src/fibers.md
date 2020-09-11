# ファイバ

Drone OSのファイバは本質的に有限状態マシンです。ファイバは、型レベルでいえば
`Fiber`トレイトを実装している無名型のインスタンスです。このトレイとは
`drone_core::fib`で次のように定義されています。

```rust
pub trait Fiber {
    type Input;
    type Yield;
    type Return;

    fn resume(
        self: Pin<&mut Self>,
        input: Self::Input,
    ) -> FiberState<Self::Yield, Self::Return>;
}

pub enum FiberState<Y, R> {
    Yielded(Y),
    Complete(R),
}
```

`Fiber`と`FiberState`は`core::ops`の`Generator`と`GeneratorState`に
似ていますが、入力パラメータが追加されています。また、ジェネレータ同様、完了後に
ファイバを再開することは不正です。

ファイバーは、`drone_cortexm::fib::new_*`ファミリのコンストラクタを使って
複数の方法で作成することができます。例えば、再開後すぐに完了するファイバは
`FnOnce`クロージャから作成することができます。

```rust
use core::pin::Pin;
use drone_cortexm::{
    fib,
    fib::{Fiber, FiberState},
};

let mut fiber = fib::new_once(|| 4);
assert_eq!(Pin::new(&mut fiber).resume(()), FiberState::Complete(4));
```

完了までに複数のyield点を含むファイバは、`FnMut`クロージャから作成することが
できます。

```rust
let mut state = 0;
let mut fiber = fib::new_fn(move || {
    if state < 3 {
        state += 1;
        fib::Yielded(state)
    } else {
        fib::Complete(state)
    }
});
assert_eq!(Pin::new(&mut fiber).resume(()), FiberState::Yielded(1));
assert_eq!(Pin::new(&mut fiber).resume(()), FiberState::Yielded(2));
assert_eq!(Pin::new(&mut fiber).resume(()), FiberState::Yielded(3));
assert_eq!(Pin::new(&mut fiber).resume(()), FiberState::Complete(3));
```

また、同等なファイバをRustのジェネレータこうbンを使って作成することもできます。

```rust
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

この章で説明したファイバは、Drone OSタスクの主要な構成要素です。しかし、ファイバ
にはもう一種類あり、次の章で説明します。
