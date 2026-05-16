# 大きさ

地球の磁場はどれくらい強いのでしょうか？ [`magnetic_field()`] メソッドのドキュメントによると、取得している `x` `y` `z` の値の単位はナノテスラです。つまり、磁場の大きさをナノテスラで求めるために計算しなければならないのは、`x` `y` `z` の値が表す 3 次元ベクトルの大きさだけです。学校で習ったのを覚えているかもしれませんが、これは単に次のように求められます:

``` rust
use libm::sqrtf;
let magnitude = sqrtf(x * x + y * y + z * z);
```

[`magnetic_field()`]: https://docs.rs/lsm303agr/1.1.0/lsm303agr/struct.Lsm303agr.html#method.magnetic_field

Rust の `core` には `sqrtf()` のような浮動小数点数学関数がないため、`no_std`
プログラムではどこかから実装を入手する必要があります。そのために [libm] クレートを使います。

[libm]: https://crates.io/crates/libm

これらをすべてプログラム (`examples/magnitude.rs`) にまとめると、次のようになります:

``` rust
{{#include examples/magnitude.rs}}
```

これを `cargo run --example magnitude` で実行してください。

このプログラムは、磁場の大きさ（強さ）をナノテスラ (`nT`) と
ミリガウス (`mG`、1 `mG` = 100 `nT`) で報告します。地球の磁場の大きさは
`250 mG` から `650 mG` の範囲にあります（大きさは地理的な位置によって変化します）ので、理想的にはその範囲にだいたい収まる値が見えるはずです。センサーがまだキャリブレーションされていないため、値はかなりずれている可能性があります。キャリブレーションについては [appendix 3] を参照してください。キャリブレーションすると、私はおよそ `340 mG` の大きさを確認しています。

[appendix 3]: ../appendix/3-mag-calibration/index.html

いくつか質問があります:

- ボードを動かさない状態では、どのような値が見えますか？ 常に同じ値が見えますか？

- ボードを回転させると、大きさは変化しますか？ 変化するべきでしょうか？