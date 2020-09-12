# メッセージパッシング

Drone OSにおける好ましいスレッド間通信の方法はメッセージパッシングです。Rustの
stdlibが、マルチプロデューサ・シングルコンシューマ・キュー向けに
`std::sync::mpsc` を提供しているように、Droneは、`drone_core::sync::spsc`
の下に3種類のシングルプロデューサ・シングルコンシューマ・キューを提供しています。

## ワンショット

oneshotチャネルは、あるスレッドから別のスレッドに単一の値の所有権を転送するために
使用されます。次のようにしてチャンネルを作成することができます。

```rust
use drone_core::sync::spsc::oneshot;

let (tx, rx) = oneshot::channel();
```

`tx`と`rx`は各々送信パートと受信パートであり、異なるスレッドに渡すことができます。
`tx`パートには`send`メソッドがあり、値として`self`を取ります。これは一回しか
呼び出すことができないことを意味します。

```rust
tx.send(my_message);
```

`rx`パートはfutureです。これは`.await`することができることを意味します。

```rust
let my_message = rx.await;
```

## リング

1つの型の複数の値を渡すためにリングチャネルがあります。これは固定サイズのリング
バッファを割り当てることで動作します。

```rust
use drone_core::sync::spsc::ring;

let (tx, rx) = ring::channel(100);
```

ここでは、`100`が使用するリングバッファのサイズです。`tx`パートを使って、
このチャネルを通じて値を送信します。

```rust
tx.send(value1);
tx.send(value2);
tx.send(value3);
```

`rx`パートはストリームです。

```rust
while let Some(value) = rx.next().await {
    // 受け取った新しい値
}
```

## パルス

何らかのイベントを他のスレッドに繰り返し通知する必要があるが、ペイロードがない場合、
リングチャネルはやりすぎかもしれません。この場合、パルスチャネルがあります。これは
アトミックカウンタにより実現されています。

```rust
use drone_core::sync::spsc::pulse;

let (tx, rx) = pulse::channel();
```

`tx`パートは`send`メソッドを持ち、背後にあるカウンタに加算する数値を取ります。

```rust
tx.send(1);
tx.send(3);
tx.send(100);
```

`rx`パートはストリームです。ストリームのポーリングが成功するたびにカウンタを
クリアし、その値を返し、それが格納されます。

```rust
while let Some(pulses) = rx.next().await {
    // 最後のポーリング以降に`pulses`数のイベントが発生した
}
```

## Futureとストリーム

スレッドトークンは、特定のスレッドに接続するための、これまで説明したチャンネルを作成
するためのメソッドを持ちます。

`add_future`は`fiber`を受け取り、future (oneshotチャネルの`rx`パート) を
返します。futureはファイバが`fib::Complete`を返した時点で解決されます。

```rust
use drone_cortexm::{fib, thr::prelude::*};

let pll_ready = thr.rcc.add_future(fib::new_fn(|| {
    if pll_ready_flag.read_bit() {
        fib::Complete(())
    } else {
        fib::Yielded(())
    }
}));
pll_ready.await;
```

`add_stream_ring`はストリーム（リングチャネルの`rx`パート）を返し、これは
ファイバが`fib::Yielded(Some(...))`または`fib::Complete(Some(...))`を
返すたびに解決されます。

```rust
use drone_cortexm::{fib, thr::prelude::*};

let uart_bytes = thr.uart.add_stream_ring(
    100, // リングバッファサイズ
    || panic!("Ring buffer overflow"),
    fib::new_fn(|| {
        if let Some(byte) = read_uart_byte() {
            fib::Yielded(Some(byte))
        } else {
            fib::Yielded(None)
        }
    }),
);
```

`add_stream_pulse`はストリーム（パルスチェネルの`rx`パート）を返し、これは
ファイバが`fib::Yielded(Some(number))`または`fib::Complete(Some(number))
を返すたびに解決されます。

```rust
use drone_cortexm::{fib, thr::prelude::*};

let sys_tick_stream = thr.sys_tick.add_stream_pulse(
    || panic!("Counter overflow"),
    fib::new_fn(|| fib::Yielded(Some(1))),
);
```
