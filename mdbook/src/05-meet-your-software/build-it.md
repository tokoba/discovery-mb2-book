# ビルドする

最初のステップは、"binary" クレートをビルドすることです。マイクロコントローラーはあなたのコンピューターとは異なるアーキテクチャを持っているため、クロスコンパイルする必要があります。Rust の世界では、クロスコンパイルは `rustc` または Cargo に追加の `--target` フラグを渡すだけで済むほど簡単です。複雑なのは、そのフラグの引数、つまりターゲットの *名前* を見つけることです。

すでに知っているとおり、micro:bit v2 のマイクロコントローラーには Cortex-M4F プロセッサーが搭載されています。`rustc` は Cortex-M アーキテクチャ向けのクロスコンパイル方法を知っており、そのアーキテクチャ内の異なるプロセッサーファミリーをカバーする複数のターゲットを提供しています。

- `thumbv6m-none-eabi`、Cortex-M0 および Cortex-M1 プロセッサー向け
- `thumbv7m-none-eabi`、Cortex-M3 プロセッサー向け
- `thumbv7em-none-eabi`、Cortex-M4 および Cortex-M7 プロセッサー向け
- `thumbv7em-none-eabihf`、Cortex-M4**F** および Cortex-M7**F** プロセッサー向け
- `thumbv8m.main-none-eabi`、Cortex-M33 および Cortex-M35P プロセッサー向け
- `thumbv8m.main-none-eabihf`、Cortex-M33**F** および Cortex-M35P**F** プロセッサー向け

ここでの "Thumb" は、コードサイズ削減のためにより小さい命令を持つ Arm 命令セットの一種を指します（しゃれです）。`hf`/`F` の部分は、ハードウェア浮動小数点アクセラレーションを意味します。これにより、小数を含む数値計算（「浮動小数点」計算）がずっと高速になります。

micro:bit v2 では、`thumbv7em-none-eabihf` ターゲットを使います。

クロスコンパイルの前に、ターゲット向けの標準ライブラリのプリコンパイル済みバージョン（実際にはその縮小版）をダウンロードする必要があります。これは `rustup` を使って行います。

``` console
$ rustup target add thumbv7em-none-eabihf
```

上の手順は一度だけ行えば十分です。その後、ツールチェーンを更新するたびに `rustup` がこのターゲットを更新します（使用する `core` ライブラリを含む標準ライブラリ `rust-std` コンポーネントを再インストールします）。したがって、[verifying your setup] のときに必要なターゲットをすでに追加しているなら、この手順は省略できます。

[verifying your setup]: ../03-setup/verify.html#verifying-cargo-embed


`rust-std` コンポーネントが配置されたので、これで Cargo を使ってプログラムをクロスコンパイルできます。Git リポジトリの `mdbook/src/05-meet-your-software` ディレクトリにいることを確認してから、ビルドしてください。この初期コードはサンプルなので、そのようにコンパイルします。

``` console
$ cargo build --example init
   Compiling semver-parser v0.7.0
   Compiling proc-macro2 v1.0.86
   ...

    Finished dev [unoptimized + debuginfo] target(s) in 33.67s
```

> **NOTE** このクレートは必ず最適化 *なし* でコンパイルしてください。提供されている `Cargo.toml` ファイルと上記のビルドコマンドにより、`cargo` に `--release` フラグを渡さない限り、最適化は無効になります。

これで、実行可能ファイルが生成されました。この実行可能ファイルはまだ LED を点滅させません。これは、この章の後半で土台として使う簡略化版にすぎません。動作確認として、生成された実行可能ファイルが実際に Arm バイナリであることを確認してみましょう。（以下のコマンドは、

    readelf -h ../../../target/thumbv7em-none-eabihf/debug/examples/init

`readelf` があるシステムでは、これと等価です。）

``` console
$ cargo readobj --example init -- --file-headers
    Finished dev [unoptimized + debuginfo] target(s) in 0.01s
ELF Header:
  Magic:   7f 45 4c 46 01 01 01 00 00 00 00 00 00 00 00 00
  Class:                             ELF32
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              EXEC (Executable file)
  Machine:                           Arm
  Version:                           0x1
  Entry point address:               0x117
  Start of program headers:          52 (bytes into file)
  Start of section headers:          793112 (bytes into file)
  Flags:                             0x5000400
  Size of this header:               52 (bytes)
  Size of program headers:           32 (bytes)
  Number of program headers:         4
  Size of section headers:           40 (bytes)
  Number of section headers:         21
  Section header string table index: 19
```

これらの数値が完全に一致しなくても心配はいりません。これらの多くは、現在のビルド環境にかなり依存します。

次は、プログラムをマイクロコントローラーに書き込みます。