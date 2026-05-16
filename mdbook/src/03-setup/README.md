# 開発環境のセットアップ

マイクロコントローラーを扱うには複数のツールが必要です。というのも、コンピューターとは異なるアーキテクチャを扱うことになり、さらに「リモート」デバイス上でプログラムを実行してデバッグする必要があるためです。

## ドキュメント

とはいえ、ツールだけですべてが揃うわけではありません。ドキュメントがなければ、マイクロコントローラーを扱うことはほぼ不可能です。MB2 の公式技術ドキュメントは <https://tech.microbit.org> にあります。本書を通して、ほかの技術ドキュメントも参照していきます。

## ツール

以下に挙げるすべてのツールを使用します。最小バージョンが指定されていないものについては、最近のバージョンであればどれでも動作するはずですが、ここではテストに使用したバージョンを記載しています。

- Rust 1.79.0 以降のツールチェーン。

- `gdb-multiarch`。これはデバッグツールです。テスト済みの最も古いバージョンは 10.2 ですが、ほかのバージョンでもおそらく動作します。お使いのディストリビューションやプラットフォームで `gdb-multiarch` が利用できない場合は、`arm-none-eabi-gdb` でも問題ありません。さらに、通常の `gdb` バイナリの中にも multiarch 機能付きでビルドされているものがあります。これについての詳細は、本書のデバッグの章を参照してください。

- [`cargo-binutils`]。バージョン 0.3.6 以降。

  [`cargo-binutils`]: https://github.com/rust-embedded/cargo-binutils

- [`probe-rs-tools`]。バージョン 0.24.0 以降。

  [`probe-rs-tools`]: https://probe.rs/docs/overview/about-probe-rs/

- Linux および macOS では `minicom`。テスト済みバージョンは 2.7.1 です。ただし、ほかのバージョンでもおそらく動作します。

- Windows では `PuTTY`。

次に、いくつかのツールについて OS に依存しないインストール手順に従ってください。

### `rustc` & Cargo

[https://rustup.rs](https://rustup.rs) の手順に従って rustup をインストールしてください。

すでに rustup をインストール済みであれば、stable チャンネルを使用しており、stable ツールチェーンが最新であることを再確認してください。`rustc -V` は、以下に示すもの以上に新しい日付とバージョンを返すはずです。

``` console
$ rustc -V
rustc 1.79.0 (129f3b996 2024-06-10)
```

### `cargo-binutils`

``` console
$ rustup component add llvm-tools
$ cargo install cargo-binutils --vers '^0.3'
$ cargo size --version
cargo-size 0.3.6
```

### `probe-rs-tools`

**注記** すでに古いバージョンの `probe-run`、`probe-rs`、または `cargo-embed` がシステムにインストールされている場合は、この手順を始める前にそれらを削除してください。将来的に問題を引き起こす可能性があるためです。特に、`probe-run` はもはや公式には存在しません。必要に応じて、次を試してください。

```console
$ cargo uninstall cargo-embed
$ cargo uninstall probe-run
$ cargo uninstall probe-rs
$ cargo uninstall probe-rs-cli
```

`probe-rs-tools` をインストールするには、<https://probe.rs> にアクセスし、そこにある最新のインストール手順に従ってください。

* **注記** `cargo install` を使って `probe-rs-tools` をインストールしたい場合は、以下の手順を試すことができます。この方法では失敗が頻発したという報告がありますが、試してみるのは自由です。

  1. 最新の stable Rust にアップグレードします。

  2. `probe-rs-tools` バイナリの
     [前提条件](https://probe.rs/docs/getting-started/installation/) をインストールします。（リンク先の
     手順は、より一般的な [`probe-rs`](https://probe.rs/) 組み込みデバッグ
     ツールキットのドキュメントの一部です。）

  3. インストールを試します

     ```console
     $ cargo install --locked probe-rs-tools
     ```

`probe-rs-tools` をインストールすると、`probe-rs` や `cargo-embed`（通常は Cargo コマンドとして実行されます）を含む、いくつかの便利なツールがインストールされます。先に進む前に、正しく動作していることを確認してください。

```
$ cargo embed --version
cargo-embed 0.24.0 (git commit: crates.io)
```

### このリポジトリ

この本には、各章で使用する小規模な Rust コードベースもいくつか含まれています。これらを使う最も簡単な方法は、本のソースコードをダウンロードすることです。これは次のいずれかの方法で行えます。

- [リポジトリ](https://github.com/rust-embedded/discovery-mb2/) にアクセスし、緑色の "Code" ボタンをクリックしてから "Download Zip" をクリックします。

- `git` を使ってクローンします（`git` を知っているなら、おそらくすでにインストール済みでしょう）。クローン元は、Zip の方法でリンクしたものと同じリポジトリです。

### OS 固有の手順

次に、使用している OS に対応する手順に従ってください。

- [Linux](linux.md)
- [Windows](windows.md)
- [macOS](macos.md)