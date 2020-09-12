# メモリマップドレジスタAPIの概要

このセクションでは、レジスタトークンとフィールドトークンの最も一般的なメソッドの例を
提供します。完全なAPIについては`drone_core::reg`と`drone_cortexm::reg`、
両モジュールのドキュメントを参照してください。

## 全レジスタ

RCC_CRレジスタの値を読み取る:

```rust
let val = rcc_cr.load();
```

HSIRDYはシングルビットフィールドなので、このメソッドは`boo`値を返し、対応する
ビットが設定されているか、クリアされているかを示します。

```rust
val.hsirdy() // `val & (1 << 1) != 0`に相当
```

HSITRIMはRCC＿CRレジスタの中央にある5ビットのフィールドです。このメソッドは
このフィールドのビットだけを先頭にシフトした整数を返します。

```rust
val.hsitrim() // `(val >> 3) & ((1 << 5) - 1)`に相当
```

RCC_CRレジストをそのデフォルト値にリセットします。デフォルト値はリファレンス
マニュアルに指定されています。

```rust
rcc_cr.reset();
```

次の行は、RCC_CRレジスタに新しい値を書き込みます。その値はレジスタのデフォルト値です
が、HSIONには1が、HSITRIMには14が設定されます。

```rust
rcc_cr.store(|r| r.set_hsion().write_hsitrim(14));
// シングルビットフィールドには、"set_"の他に "clear_"と "toggle_"と
// いう接頭辞がある。
```

そして最後に、次の行は上記の全てを組み合わせたもので、read-modify-write操作を
行います。

```rust
rcc_cr.modify(|r| r.set_hsion().write_hsitrim(14));
```

指定されていないフィールドをデフォルトにリセットする`store`とは異なり、
`modify`メソッドは他のフィールドの値をそのまま保持します。

## レジスタフィールド

レジスタフィールドトークンしか持っていない場合は、そのフィールドだけに影響を与える
操作しか実行できません。他の兄弟フィールドには影響を与えません。

```rust
rcc_cr_hsirdy.read_bit(); // `rcc_cr.load().hsirdy()`に相当
rcc_cr_hsitrim.read_bits(); // `rcc_cr.load().hsitrim()`に相当
rcc_cr_hsirdy.set_bit(); // `rcc_cr.modify(|r| r.set_hsirdy())`に相当
rcc_cr_hsirdy.clear_bit(); // `rcc_cr.modify(|r| r.clear_hsirdy())`に相当
rcc_cr_hsirdy.toggle_bit(); // `rcc_cr.modify(|r| r.toggle_hsirdy())`に相当
rcc_cr_hsitrim.write_bits(14); // `rcc_cr.modify(|r| r.write_hsitrim(14))`に相当
```

また、同じレジスタの複数のフィールドのトークンを持っている場合は、
1つのread-modify-write操作を次のように行うことができます。

```rust
rcc_cr_hsion.modify(|r| {
    rcc_cr_hsion.set(r);
    rcc_cr_hsitrim.write(r, 14);
});
// これは次に相当する
rcc_cr.modify(|r| r.set_hsion().write_hsitrim(14));
```
