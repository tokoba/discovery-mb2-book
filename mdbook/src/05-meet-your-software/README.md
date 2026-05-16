# ソフトウェアに触れてみましょう

この章では、*非常に*単純なプログラムをいくつかビルド、実行、デバッグする方法を学びます。ここでの目標は、
MB2 Rust プログラミングの詳細に立ち入ること（まだ）ではなく、そのプロセスの仕組みに慣れることです。

まず、この本の以降の部分で使われる慣例について簡単に説明します。私たちは、次のようにして
本全体のコピーを取得することを想定しています。

```
git clone http://github.com/rust-embedded/discovery-mb2
```

この本の「ソースコード」は `discovery-mb2/mdbook/src` にあります。あなたのコピーでもそこへ移動して、
少し見て回ってください。各章のディレクトリには、Markdown の原文テキストと、その章にあるすべてのプログラムの
完全なソースの両方が含まれています。`src/main.rs` のようなパスを参照する場合、それは作業中の章を起点とした
場所を意味します。たとえば、あなたの `discovery-mb2` には
`mdbook/src/05-meet-your-software/examples/init.rs` というファイルがあります。この章では、そのファイルを単に
`examples/init.rs` として参照します。

Rust コードには基本的に 2 種類あります。「binary」の実行可能プログラムと、「library」コードです。この本では、
library コードはそれほど大きな役割を果たしません。binary プログラムのソースコードは、いくつかの場所に置けます。

* `src/main.rs` にあるプログラムは、`cargo embed` または `cargo
  run` によって自動的にコンパイルおよび実行されます。特別なフラグは必要ありません。

* `examples/foo.rs` にあるプログラムは、`cargo embed --example foo` または
  `cargo run --example foo` でコンパイルおよび実行できます。
  
* `src/bin/bar.rs` にあるプログラムは、`cargo embed --bin bar` または
  `cargo run --bin bar` でコンパイルおよび実行できます。

これは紛らわしいですが、Cargo の標準的な慣例です。

それでは先に進み、これらすべてを実際に扱っていきましょう。