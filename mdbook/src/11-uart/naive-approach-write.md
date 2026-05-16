# 素朴なアプローチと `write!`

## 素朴なアプローチ

おそらく、次のようなプログラムを思いついたでしょう（`examples/naive-send-string.rs`）:

```rs
{{#include examples/naive-send-string.rs}}
```

これは完全に有効な実装ですが、いずれは引数のフォーマットなど、`print!` の持つ便利な
機能をすべて使いたくなるかもしれません。どうすればそれができるのか気になるなら、このまま読み進めてください。

## `write!` と `core::fmt::Write`

`core::fmt::Write` トレイトを使うと、それを実装する任意の構造体を、`std` の世界で
`print!` を使うのと基本的に同じ方法で使えるようになります。この場合、`nrf` HAL の `Uart`
構造体は `core::fmt::Write` を実装しているので、先ほどのプログラムを次のようにリファクタリング
できます（`examples/send-string.rs`）:

```rs
{{#include examples/send-string.rs}}
```

このプログラムを micro:bit に書き込むと、あなたが思いついたイテレータベースのプログラムと
機能的に等価であることがわかるでしょう。