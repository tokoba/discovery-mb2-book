# デバッグしてみよう

この小さなプログラムをどうデバッグするのか見ていきましょう。まだ特に面白いバグはありませんが、
デバッグを学ぶにはそういうプログラムが一番です。

## そもそもこれはどう動いているのでしょうか？

プログラムをデバッグする前に、ここで実際に何が起きているのかを少しだけ素早く理解しておきましょう。
前の章では、基板上の 2 つ目のチップの役割と、それが私たちのコンピュータとどのように通信するかを
すでに説明しましたが、では実際にそれをどう使うのでしょうか？

`Embed.toml` の小さなオプション `default.gdb.enabled = true` によって、フラッシュ後に
`cargo embed` がいわゆる「GDB stub」を起動するようになっています。これは、GDB が接続して
「アドレス X にブレークポイントを設定する」といったコマンドを送れるサーバーです。するとサーバーは、
そのコマンドをどう処理するかを自分で判断できます。`cargo embed` の GDB stub の場合は、
そのコマンドを USB 経由で 2 つ目のチップ上の「デバッグプローブ」に転送します。このチップが、
私たちの代わりに MCU とやり取りする役割を担ってくれます。

## デバッグしてみましょう！

現在のシェルでは `cargo-embed` が動作しています。新しいシェルを開いて、プロジェクトディレクトリに
戻ることができます。そこに移動したら、まず次のようにして gdb でバイナリを開く必要があります。

```shell
$ gdb ../../../target/thumbv7em-none-eabihf/debug/examples/init
```

> **注記** インストールした GDB によっては、起動に別のコマンドを使う必要があります。
> どのコマンドだったか忘れた場合は [chapter 3] を確認してください。

[chapter 3]: ../03-setup/index.md#tools

このコマンドの `../../..` が必要なのは、各サンプルプロジェクトが書籍全体を含む「ワークスペース」の
中にあるためです。ワークスペースでは、単一の共有 `target` ディレクトリが使われます。詳しくは
[Workspaces chapter in Rust Book] を参照してください。

> **注記** ここで `cargo-embed` が大量の警告を表示しても心配しないでください。現時点では
> GDB プロトコルを完全には実装していないため、GDB が送るすべてのコマンドを認識できないことが
> あります。GDB がクラッシュしない限り、問題ありません。

[Workspaces chapter in Rust Book]: https://doc.rust-lang.org/book/ch14-03-cargo-workspaces.html#creating-a-workspace

次に、GDB stub に接続する必要があります。デフォルトでは `localhost:1337` で動作しているので、
接続するには次を実行します。

```shell
(gdb) target remote :1337
Remote debugging using :1337
0x00000116 in nrf52833_pac::{{impl}}::fmt (self=0xd472e165, f=0x3c195ff7) at /home/nix/.cargo/registry/src/github.com-1ecc6299db9ec823/nrf52833-pac-0.9.0/src/lib.rs:157
157     #[derive(Copy, Clone, Debug)]
```

> **注記** この章のリポジトリ内のサンプルは、時間の経過とともに変更される可能性があります。
> そのため、行番号やその他のソースの詳細は、ここや以下に示されているものと異なる場合があります。
>
> プログラム開始後に停止せず、次のようにプログラムのより深い場所まで進んでしまう場合は、
> リセットするために `monitor reset halt` を実行してみてください。これは `probe-rs` の
> [a bug](https://github.com/probe-rs/probe-rs/issues/3438) によるもので、詳しくは
> [issue #27](https://github.com/rust-embedded/discovery-mb2/issues/27) を参照してください。
> ```shell
> (gdb) target remote :1337
> Remote debugging using :1337
> init::__cortex_m_rt_main () at mdbook/src/05-meet-your-software/examples/init.rs:19
> 19              asm::nop();
> (gdb) monitor reset halt
> Resetting and halting target
> Target halted
> ```

次にやりたいのは、プログラムの `main` 関数まで進むことです。これを行うには、まずそこに
ブレークポイントを設定し、その後そのブレークポイントに到達するまでプログラムの実行を続けます。

```
(gdb) break main
Breakpoint 1 at 0x104: file src/05-meet-your-software/examples/init.rs, line 9.
Note: automatically using hardware breakpoints for read-only addresses.
(gdb) continue
Continuing.

Breakpoint 1, init::__cortex_m_rt_main_trampoline () at src/05-meet-your-software/examples/init.rs:9
9       #[entry]
```

ブレークポイントは、プログラムの通常の流れを止めるために使えます。`continue` コマンドを使うと、
プログラムはブレークポイントに到達する*まで*自由に実行されます。この場合は、そこに
ブレークポイントがあるため、`main` 関数に到達するまでです。

GDB の出力に「Breakpoint 1」と表示されていることに注意してください。プロセッサが使える
ブレークポイントの数には限りがあるので、こうしたメッセージに注意を払うのは良い考えです。
もしブレークポイントが足りなくなった場合は、`info break` で現在のブレークポイントを一覧表示し、
`delete <breakpoint-num>` で不要なものを削除できます。

より快適にデバッグするために、ここでは GDB の Text User Interface (TUI) を使います。その
モードに入るには、GDB のシェルで次のコマンドを入力します。

```
(gdb) layout src
```

> **注記** Windows ユーザーの皆さんには申し訳ありません。GNU Arm Embedded Toolchain に
> 同梱されている GDB は、この TUI モードをサポートしていません `:-(`。

![GDB セッション](../assets/gdb-layout-src.png "GDB TUI")

GDB の break コマンドは、関数名だけに対して機能するわけではありません。特定の行番号でも
停止できます。13 行目で停止したい場合は、単に次のようにします。

```
(gdb) break 13
Breakpoint 2 at 0x110: file src/05-meet-your-software/examples/init.rs, line 13.
(gdb) continue
Continuing.

Breakpoint 2, init::__cortex_m_rt_main () at src/05-meet-your-software/examples/init.rs:13
(gdb)
```

いつでも、次のコマンドで TUI モードを終了できます。

```
(gdb) tui disable
```

現在、私たちは `_y = x` 文の「上」にいます。この文はまだ実行されていません。つまり、`x` は
初期化されていますが、`_y` には何が入っているか分かりません。`print` コマンドを使って `x` を
調べてみましょう。

```
(gdb) print x
$1 = 42
(gdb) print &x
$2 = (*mut i32) 0x20003fe8
(gdb)
```

予想どおり、`x` には値 `42` が入っています。`print &x` コマンドは、変数 `x` のアドレスを
表示します。ここで興味深いのは、GDB の出力に参照の型が表示されていることです。`*mut i32` は、
可変な `i32` 値へのポインタです。

プログラムの実行を 1 行ずつ進めたい場合は、`next` コマンドを使います。`loop {}` 文まで
進んでみましょう。

```
(gdb) next
16          loop {}
```

これで `_y` も初期化されているはずです。

```
(gdb) print _y
$5 = 42
```

ローカル変数を 1 つずつ表示する代わりに、`info locals` コマンドを使うこともできます。

```
(gdb) info locals
x = 42
_y = 42
(gdb)
```

`loop {}` 文の上で再び `next` を使うと、プログラムはその文を決して通過しないため、
そこで詰まってしまいます。そこで代わりに、`layout asm` コマンドで逆アセンブル表示に切り替え、
`stepi` を使って 1 命令ずつ進めます。あとで再び `layout src` コマンドを実行すれば、
いつでも Rust のソースコード表示に戻れます。

> **注記** うっかり `next` や `continue` コマンドを使って GDB が詰まってしまった場合は、
> `Ctrl+C` を押せば抜け出せます。

```
(gdb) layout asm
```

![GDB セッション](../assets/gdb-layout-asm.png "GDB の逆アセンブル")

TUI モードを使っていない場合は、`disassemble /m` コマンドを使って、現在いる行の周辺を
逆アセンブルできます。
```
(gdb) disassemble /m
Dump of assembler code for function _ZN12init18__cortex_m_rt_main17h3e25e3afbec4e196E:
10      fn main() -> ! {
   0x0000010a <+0>:     sub     sp, #8
   0x0000010c <+2>:     movs    r0, #42 ; 0x2a

11          let _y;
12          let x = 42;
   0x0000010e <+4>:     str     r0, [sp, #0]

13          _y = x;
   0x00000110 <+6>:     str     r0, [sp, #4]

14
15          // 無限ループ。このスタックフレームから抜けないようにするためだけのもの
16          loop {}
=> 0x00000112 <+8>:     b.n     0x114 <_ZN12init18__cortex_m_rt_main17h3e25e3afbec4e196E+10>
   0x00000114 <+10>:    b.n     0x114 <_ZN12init18__cortex_m_rt_main17h3e25e3afbec4e196E+10>

End of assembler dump.
```

左側の太い矢印 `=>` が見えますか？これは、プロセッサが次に実行する命令を示しています。

TUI モードに入っていない場合、`stepi` コマンドを実行するたびに、GDB はプロセッサが次に実行する命令
に対応する文とその行番号を表示します。

```
(gdb) stepi
16          loop {}
(gdb) stepi
16          loop {}
```

もっと興味深いものに進む前に、最後に 1 つ小技を紹介します。次のコマンドを GDB に入力してください。

```
(gdb) monitor reset
(gdb) c
Continuing.

Breakpoint 1, init::__cortex_m_rt_main_trampoline () at src/05-meet-your-software/src/main.rs:9
9       #[entry]
(gdb)
```

これで `main` の先頭に戻りました！

`monitor reset` はマイクロコントローラをリセットし、プログラムのエントリポイントで停止させます。
続く `continue` コマンドは、ブレークポイントが設定されている `main`
関数に到達するまで、プログラムをそのまま自由に実行させます。

この組み合わせは、誤って調べたかったプログラムの一部を通り過ぎてしまったときに便利です。
プログラムの状態を簡単にそのいちばん最初まで巻き戻せます。

> **細かい注意点**: この `reset` コマンドは RAM をクリアしたり変更したりしません。そのメモリには前回の実行時の
> 値が残ります。とはいえ、通常それは問題にならないはずです。ただし、プログラムの挙動が
> *未初期化* 変数の値に依存している場合は別です — ですが、それはまさに未定義
> 動作 (UB) の定義です。

このデバッグセッションはこれで終了です。`quit` コマンドで終えられます。

```
(gdb) quit
A debugging session is active.

        Inferior 1 [Remote target] will be detached.

Quit anyway? (y or n) y
Detaching from program: $PWD/target/thumbv7em-none-eabihf/debug/meet-your-software, Remote target
Ending remote debugging.
[Inferior 1 (Remote target) detached]
```

> **注記** デフォルトの GDB CLI が好みに合わない場合は、[gdb-dashboard] を試してみてください。これは Python
> を使って、デフォルトの GDB CLI をレジスタ、ソースビュー、アセンブリ
> ビューなどを表示するダッシュボードに変えます。

[gdb-dashboard]: https://github.com/cyrus-and/gdb-dashboard#gdb-dashboard

GDB で何ができるのかをもっと知りたい場合は、セクション [GDB の使い方](../appendix/2-how-to-use-gdb/) を参照してください。

次は何でしょう？私が約束した高水準 API です。