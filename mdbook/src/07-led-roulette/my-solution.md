# 私の解答

どんな解答になりましたか？

これが私のものです。必要なマトリクスを生成する方法の中では、おそらく最も単純なものの 1 つです（もちろん、いちばん美しいものではありませんが）。

``` rust
{{#include src/main.rs}}
```

もう 1 つあります！ あなたの解答が「release」モードでコンパイルした場合にも動作することを確認してください。

``` console
$ cargo embed --release
```

「release」モードのバイナリをデバッグしたい場合は、別の GDB コマンドを使う必要があります。

``` console
$ gdb ../../../target/thumbv7em-none-eabihf/release/led-roulette
```

Rust コンパイラは、コードをより高速またはより小さくしようとして、release ビルドで生成されるマシン命令を変更します（ときには大幅に）。残念ながら、その後に何が起きているのかを GDB が把握するのは困難です。その結果、GDB を使った release ビルドのデバッグは難しくなることがあります。

バイナリサイズは、常に注意しておくべきものです！ あなたの解答はどれくらいの大きさでしょうか？ release バイナリに対して `size` コマンドを使うと確認できます。

``` console
$ cargo size --release -- -A
    Finished release [optimized + debuginfo] target(s) in 0.02s
led-roulette  :
section              size        addr
.vector_table         256         0x0
.text                6332       0x100
.rodata               648      0x19bc
.data                   0  0x20000000
.bss                 1076  0x20000000
.uninit                 0  0x20000434
.debug_loc           9036         0x0
.debug_abbrev        2754         0x0
.debug_info         96460         0x0
.debug_aranges       1120         0x0
.debug_ranges       11520         0x0
.debug_str          71325         0x0
.debug_pubnames     32316         0x0
.debug_pubtypes     29294         0x0
.Arm.attributes        58         0x0
.debug_frame         2108         0x0
.debug_line         19303         0x0
.comment              109         0x0
Total              283715
```

コードのビルド方法によって、数値はいくらか異なる場合があります。これは問題ありません。

この出力の読み方はわかりますか？ `text` セクションにはプログラムの命令が含まれます。`rodata` セクションには、プログラム命令と一緒に格納される読み取り専用データが含まれます。`data` および `bss` セクションには、RAM に静的に割り当てられた変数（`static` 変数）が含まれます。micro:bit 上のマイクロコントローラの仕様を覚えていれば、そのフラッシュメモリがこの非常に単純なバイナリのサイズの 2 倍にも満たないことに気づくはずです。これは本当に正しいのでしょうか？ サイズ統計からわかるとおり、バイナリの大部分は実際にはデバッグ関連のセクションでできています。しかし、それらがマイクロコントローラに書き込まれることはありません。結局のところ、実行には関係ないからです。