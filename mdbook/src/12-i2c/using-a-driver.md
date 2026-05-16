# ドライバを使う

第5章ですでに説明したとおり、`embedded-hal` は、ハードウェアとやり取りできるプラットフォームに依存しないコードを書くために使える抽象化を提供します。実際、[LED ルーレット](../07-led-roulette/index.html) の章とこの章のここまででハードウェアとやり取りするために使ってきたすべてのメソッドは、`embedded-hal` で定義されたトレイトのものでした。ここで初めて、`embedded-hal` が提供するトレイトを実際に使います。

embedded Rust がサポートするすべてのプラットフォーム向けに、私たちの LSM303AGR 用ドライバを実装するのは無意味です（そして、今後登場するかもしれない新しいものについても同様です）。これを避けるために、`embedded-hal` のトレイトを実装したジェネリック型を受け取るドライバを書くことで、プラットフォームに依存しないバージョンのドライバを提供できます。幸いなことに、これはすでに [`lsm303agr`] クレートで行われています。そのため、実際の加速度計と磁力計の値の読み取りは、基本的にはプラグアンドプレイになります（加えて、少しドキュメントを読むだけです）。実際、`crates.io` のページには、Raspberry Pi を使って加速度計データを読み取るために必要なことがすべてすでに書かれています。あとはそれを私たちのチップ向けに合わせるだけです。

[`lsm303agr`]: https://crates.io/crates/lsm303agr

リンク先のページにある Raspberry Pi Linux のサンプルコードを見てください。

すでに [前のページ](read-a-single-register.md) で [`embedded_hal::blocking::i2c`] トレイトを実装するオブジェクトのインスタンスを作成する方法はわかっているので、サンプルコードを適応させるのは簡単です（`examples/show-accel.rs`）:

[`embedded_hal::blocking::i2c`]: https://docs.rs/embedded-hal/0.2.6/embedded_hal/blocking/i2c/index.html

```rust
{{#include examples/show-accel.rs}}
```

前のスニペットと同じように、次のようにして試せるはずです。
```console
$ cargo embed --example show-accel
```

さらに、micro:bit を少し（物理的に）動かすと、表示されている加速度の値が変化するのがわかるはずです。