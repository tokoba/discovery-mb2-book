# 操作

主人公は micro:bit の前面にある 2 つのボタンで操作します。ボタン A でヘビは左に曲がり、ボタン B で右に曲がります。

ボタン入力を並行的に処理するために、`microbit::pac::interrupt` マクロを使用します。割り込みは、MB2 の General Purpose Input/Output Tasks and Events（GPIOTE）ペリフェラルによって生成されます。

## `controls` モジュール

グローバルな可変状態として、2 つの別々の情報を追跡する必要があります。`GPIOTE` ペリフェラルへの参照と、次に曲がる方向の記録です。

共有データは、内部可変性とロックを可能にするために `RefCell` でラップされます。`RefCell` について詳しくは、[RefCell documentation] と Rust Book] の [interior mutability chapter] を読んでください。さらに、この `RefCell` は安全なアクセスを可能にするために `cortex_m::interrupt::Mutex` でラップされます。`cortex_m` クレートが提供する Mutex は、[critical section] の概念を使います。Mutex 内のデータには、`cortex_m::interrupt::free` に渡された関数またはクロージャの内部からのみアクセスできます（ここでは分かりやすさのため `interrupt_free` にリネームしています）。これにより、その関数またはクロージャ内のコード自体が割り込まれないことが保証されます。

[RefCell documentation]: https://doc.rust-lang.org/std/cell/struct.RefCell.html
[interior mutability chapter]: https://doc.rust-lang.org/book/ch15-05-interior-mutability.html
[critical section]: https://en.wikipedia.org/wiki/Critical_section

### 初期化

まず、ボタンを初期化します（`src/controls/init.rs`）。

```rust
{{#include src/controls/init.rs}}
```

nRF52 上の `GPIOTE` ペリフェラルには 8 つの「チャネル」があり、それぞれを `GPIO` ピンに接続して、立ち上がりエッジ（低から高への信号遷移）や立ち下がりエッジ（高から低への信号遷移）を含む特定のイベントに反応するよう設定できます。ボタンは `GPIO` ピンであり、押されていないときは信号が高く、それ以外では低くなります。したがって、ボタン押下は立ち下がりエッジです。

初期化時に `init_channel()` 関数をやや不格好な形で使っているのは、ボタン初期化コードのコピー＆ペーストを避けるためです。MB2 向けの各種組み込みクレートがこれまで隠していた型は、ときどき少し手ごわく見えます。HAL クレートと PAC クレートの型構造は少し独特で、慣れが必要なので、いずれ調べてみることをお勧めします。特に、microbit 上の各ピンは *それぞれ固有の型を持っている* ことに注目してください。初期化で使っている `degrade()` 関数の目的は、それらを共通の型に変換し、その型を `init_channel()`、ひいては `input_pin()` の引数として無理なく使えるようにすることです。

`channel0` を `button_a` に、`channel1` を `button_b` に接続します。どちらの場合も、立ち下がりエッジ（`hi_to_lo`）でイベントを生成するようボタンを設定します。`GPIOTE` ペリフェラルへの参照は `GPIO` Mutex に保存します。その後、`GPIOTE` 割り込みを `unmask` して、ハードウェアがそれらを伝播できるようにし、さらに `unpend` を呼んで pending 状態の割り込みをすべてクリアします（これは、割り込みが unmask される前に生成されていた可能性があります）。

### 割り込みハンドラ

次に、割り込みを処理するコードを書きます。`nrf52833_hal` クレートから再エクスポートされている `interrupt` マクロを使います。処理したい割り込みと同じ名前の関数を定義し（一覧は
[here](https://docs.rs/nrf52833-hal/latest/nrf52833_hal/pac/enum.Interrupt.html) で確認できます）、それに `#[interrupt]` を付けます（`src/controls/interrupt.rs`）。

```rust
{{#include src/controls/interrupt.rs}}
```

`GPIOTE` 割り込みが生成されたら、各ボタンが押されたかどうかを確認します。ボタン A だけが押されていた場合は、ヘビが左に曲がるべきであることを記録します。ボタン B だけが押されていた場合は、ヘビが右に曲がるべきであることを記録します。それ以外の場合は、ヘビは曲がらないことを記録します。（両方のボタンが「同時に」押される可能性は極めて低いです。ボタン押下はほぼ瞬時に検出され、この割り込みハンドラは非常に高速に実行されるため、この状況を起こすには両方のボタンを間に合うように押し下げるのは難しいでしょう。同様に、このコードが見逃して「どちらのボタンも押されていない」と報告してしまうほど短時間だけボタンを押すのも難しいはずです。それでも、Rust はこうした予期しないケースも考慮することを強制します。すべての可能性を確認しない限り、コードはコンパイルされません。）該当する曲がる方向は `TURN` Mutex に保存されます。これらはすべて `interrupt_free` ブロック内で行われ、これにより、この割り込みを処理している間に他のイベントで割り込まれないことが保証されます。

最後に、次の曲がる方向を取得する単純な関数を公開します（`src/controls.rs`）。

```rust
{{#include src/controls.rs}}
```

この関数は単に `TURN` Mutex の現在の値を返します。引数として 1 つの真偽値 `reset` を取ります。`reset` が `true` の場合、`TURN` の値はリセットされ、つまり `Turn::None` に設定されます。

次は、高忠実度のゲーム表示をサポートする機能を作成します。