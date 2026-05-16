# LEDルーレット

それでは、「本物の」アプリケーションを作ってみましょう。目標は、この回転するライト表示を実現することです。

<p align="center">
<video src="../assets/roulette_fast.mp4" width="500" loop="true" autoplay="true"/>
</p>

LEDピンを個別に扱うのはかなり面倒なので（特に今回のように、基本的にそのほとんどすべてを
使わなければならない場合は）、前に説明した `microbit-v2` BSPクレートを使って、
MB2 の LED「ディスプレイ」を扱うことができます。これは次のように動作します（`examples/light-it-all.rs`）:

```rust
{{#include examples/light-it-all.rs}}
```

例で示した Rust 配列 `light_it_all` には、LED が点灯している場所に 1、消灯している場所に 0 が
入っています。`show()` の呼び出しには、BSP のディスプレイコードが遅延に使用するタイマー、配列の *コピー*、
そしてこの表示を表示してから復帰するまでの時間（ミリ秒）を渡します。