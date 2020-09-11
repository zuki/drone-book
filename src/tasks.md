# タスク

Drone OSアプリケーションにおいて、タスクは論理的な作業単位です。多くの場合、
それは個別スレッドで実行している`async`関数として表現されます。慣習として、
各タスクは`src/tasks`ディレクトリに個別のモジュールとして配置されます。
このモジュールには少なくとも`handler`という名前のタスクのメイン関数が含まれて
います。そして、この関数は`src/tasks/mod.rs`で次のように再エクスポートされます。

```rust
pub mod my_task;

pub use self::my_task::handler as my_task;
```

タスクスレッドとして未使用の割り込みを使用するのが一般的です。たとえば、
STM32F103では、位置番号53に"UART5グローバル割り込み"があります。
UART5ペリフェラルがアプリケーションで使用されていない場合、その割り込みを
全く別のタスクで再利用することができます。

```rust
thr::vtable! {
    // ... ヘッダはスキップ ...

    // --- 割り当て済みのスレッド ---

    /// 全フォールトクラス.
    pub HARD_FAULT;
    /// `my_task`用のスレッド.
    pub 53: MY_TASK;
}
```

次に、`my_task`が`async`関数であると仮定すると、スレッドはタスクを次のように
して実行できます。f

```rust
use crate::tasks;
use drone_cortexm::thr::prelude::*;

thr.my_task.enable_int();
thr.my_task.set_priority(0xB0);
thr.my_task.exec(tasks::my_task());
```

これにより、`my_task`futreやそのネストしたfutureが`Poll::Pending`を
返すたびに、スレッドはサスペンドします。そして、futureが再びポーリングできる
ようになると再開されます。これは、背後で`core::task::Waker`を渡すことで
実装されており、`wake`されるとスレッドを起動します。
