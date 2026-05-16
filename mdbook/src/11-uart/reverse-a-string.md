# 文字列を逆順にする

では次に、クライアントが送信したテキストを逆順にして返すようにして、サーバーをもう少し面白くしてみましょう。サーバーは、クライアントが
ENTER キーを押すたびに応答を返します。サーバーからの各応答は新しい行に出力されます。

今回はバッファが必要です。[`heapless::Vec`] を使えます。以下がスターターコードです。

[`heapless::Vec`]: https://docs.rs/heapless/latest/heapless/struct.Vec.html

``` rust
#![no_main]
#![no_std]

use cortex_m_rt::entry;
use core::fmt::Write;
use heapless::Vec;
use rtt_target::rtt_init_print;
use panic_rtt_target as _;

use microbit::{
    hal::prelude::*,
    hal::uarte,
    hal::uarte::{Baudrate, Parity},
};

mod serial_setup;
use serial_setup::UartePort;

#[entry]
fn main() -> ! {
    rtt_init_print!();
    let board = microbit::Board::take().unwrap();

    let mut serial = {
        let serial = uarte::Uarte::new(
            board.UARTE0,
            board.uart.into(),
            Parity::EXCLUDED,
            Baudrate::BAUD115200,
        );
        UartePort::new(serial)
    };

    // 32 バイトの容量を持つバッファ
    let mut buffer: Vec<u8, 32> = Vec::new();

    loop {
        buffer.clear();

        // TODO ユーザーリクエストを受信する。各ユーザーリクエストは ENTER で終わる
        // 注 `buffer.push` は `Result` を返す。エラーは
        // エラーメッセージを返して処理すること。

        // TODO 逆順にした文字列を送り返す
    }
}
```