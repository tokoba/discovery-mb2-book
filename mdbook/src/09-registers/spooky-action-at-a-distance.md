# 離れた場所での不気味な作用

`OUT` は、ポート `P0` のピンを制御できる唯一のレジスタではありません。`OUTSET` レジスタも
ピンの値を変更できますし、`OUTCLR` も同様です。ただし、`OUTSET` と `OUTCLR` では
ポート `P0` の現在の出力状態を取得することはできません。

`OUTSET` は [Product Specification] に次のように記載されています。

> 6.8.2.2 項。OUTSET - 145 ページ

次のプログラムを見てみましょう。このプログラムの要となるのは `fn print_out` です。この関数は、
`OUT` の現在の値を `RTT` コンソールに出力します（`examples/spooky.rs`）。

``` rust
{{#include examples/spooky.rs}}
```

このプログラムを実行すると、次のように表示されます。

``` console
$ cargo embed
# cargo-embed's console
(..)
15:13:24.055: P0.OUT = 0x000000
15:13:24.055: P0.OUT = 0x200000
15:13:24.055: P0.OUT = 0x280000
15:13:24.055: P0.OUT = 0x080000
15:13:24.055: P0.OUT = 0x000000
```

副作用です！実際には変更していない同じアドレスを何度も読み取っているにもかかわらず、
`OUTSET` または `OUTCLR` に書き込むたびに、その値が変化していることがわかります。

[Product Specification]: https://docs-be.nordicsemi.com/bundle/ps_nrf52833/attach/nRF52833_PS_v1.7.pdf