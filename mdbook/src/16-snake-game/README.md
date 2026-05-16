# Snake game

ここからは、MB2 の 5×5 LED マトリクスをディスプレイとして、2 つのボタンを操作入力として使って遊べる、基本的な [snake](https://en.wikipedia.org/wiki/Snake_(video_game_genre)) ゲームを実装していきます。これにより、本書の前の章で扱ったいくつかの概念を土台にしつつ、新しい周辺機器や概念についても学んでいきます。

## モジュール性

ここでのソースコードは、おそらく必要以上にモジュール化されています。この細かい粒度のモジュール化によって、ソースコードを少しずつ見ていくことができます。コードはボトムアップで構築します。まず `game`、`controls`、`display` の 3 つのモジュールを作成し、その後これらを組み合わせて最終的なプログラムを構築します。各モジュールにはトップレベルのソースファイルと、1 つ以上のインクルードされるソースファイルがあります。たとえば、`game` モジュールは `src/game.rs`、`src/game/coords.rs`、`src/game/movement.rs` などで構成されます。Rust の `mod` 文は、モジュールのさまざまな構成要素をまとめるために使用されます。*The Rust Programming Language* には、Rust のモジュールシステムについてのよい [description] があります。

[description]: https://doc.rust-lang.org/book/ch07-02-defining-modules-to-control-scope-and-privacy.html