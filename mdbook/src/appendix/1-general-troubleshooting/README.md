# 一般的なトラブルシューティング

## `cargo-embed` の問題

`cargo-embed` に関する問題のほとんどは、Linux で `udev` ルールが正しくインストールされていないことに関連しているため、その点が正しいことを確認してください。

行き詰まった場合は、[`discovery` issue tracker] で issue を作成するか、[Rust
Embedded matrix channel] または [probe-rs matrix channel] を訪れて、そこで助けを求めることができます。

[`discovery` issue tracker]: https://github.com/rust-embedded/discovery-mb2/issues
[Rust Embedded matrix channel]: https://matrix.to/#/#rust-embedded:matrix.org
[probe-rs matrix channel]: https://matrix.to/#/#probe-rs:matrix.org

## Cargo の問題

### 「`core` の crate が見つからない」

*症状:*

```
   Compiling volatile-register v0.1.2
   Compiling rlibc v1.0.0
   Compiling r0 v0.1.0
error[E0463]: can't find crate for `core`

error: aborting due to previous error

error[E0463]: can't find crate for `core`

error: aborting due to previous error

error[E0463]: can't find crate for `core`

error: aborting due to previous error

Build failed, waiting for other jobs to finish...
Build failed, waiting for other jobs to finish...
error: Could not compile `r0`.

To learn more, run the command again with --verbose.
```

*原因:*

マイクロコントローラー用の適切なターゲット `thumbv7em-none-eabihf` のインストールを忘れています。

*修正:*

適切なターゲットをインストールしてください。

``` console
$ rustup target add thumbv7em-none-eabihf
```

### デバイスに書き込めない: `No loadable segments were found in the ELF file`

*症状:*
```console
> cargo embed
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.04s
      Config default
      Target /home/user/embedded/target/thumbv7em-none-eabihf/debug/examples/init
 WARN probe_rs::flashing::loader: No loadable segments were found in the ELF file.
       Error No loadable segments were found in the ELF file.
```

*原因:*

Cargo は、ターゲットデバイスの要件に合わせてプログラムをどのようにビルドし、リンクするかを把握する必要があります。
そのため、`.cargo/config.toml` ファイルに正しいパラメーターを設定する必要があります。

*修正:*

正しいパラメーターを含む `.cargo/config.toml` ファイルを追加してください:
```toml
{{#include ../../05-meet-your-software/.cargo/config.toml}}
```

詳細については、[Embedded Setup](../../05-meet-your-software/embedded-setup.md) を参照してください。