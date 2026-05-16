# Rust Embedded の用語

micro:bit のプログラミングに入る前に、今後のすべての章で重要になるライブラリと
用語を手短に見ておきましょう。

## 抽象化レイヤー

完全にサポートされたマイクロコントローラ、またはマイクロコントローラを搭載したボードでは、
その抽象化レベルを表す用語として、通常は次のものを耳にするでしょう。

### Peripheral Access Crate (PAC)

PAC の役割は、チップのペリフェラルに対する（ある程度）安全な直接インターフェースを提供し、
すべてのビットを望むとおりに設定できるようにすることです（もちろん、誤った設定もできてしまいます）。通常、
PAC を直接扱う必要があるのは、より上位のレイヤーでは要件を満たせない場合か、
それらのためのより高水準なコードを開発している場合だけです。 当然ながら、ここで私たちが
（その多くは暗黙的に）使うことになる PAC は [nRF52] 用のものです。

### Hardware Abstraction Layer (HAL)

HAL の役割は、チップの PAC の上に構築され、このチップ固有の特殊な振る舞いをすべて
知らない人でも実際に使える抽象化を提供することです。 通常、
HAL はペリフェラル全体を単一の struct に抽象化し、たとえばそのペリフェラルを使って
データをやり取りできるようにします。 私たちは [nRF52-hal] を使います。

### Board Support Crate (BSP)

（Rust 以外では、これは通常 Board Support Package と呼ばれるため、この略称になっています。）

BSP の役割は、ボード全体（micro:bit など）をまとめて抽象化することです。 つまり、
マイクロコントローラだけでなく、そのボード上に搭載されている可能性のあるセンサーや LED なども
利用できる抽象化を提供する必要があります。 かなり多くの場合（特にカスタムメイドのボードでは）、
あらかじめ用意された BSP は存在しません。 その代わり、チップ向けの HAL を使い、
センサー用のドライバは自分で実装するか、`crates.io` で探すことになります。 ただ幸いなことに、micro:bit には
[BSP] があるので、HAL に加えてそれも使っていきます。

[nrF52]: https://crates.io/crates/nrf52833-pac
[nrF52-hal]: https://crates.io/crates/nrf52833-hal
[BSP]: https://crates.io/crates/microbit-v2

## レイヤーの統一

次に、Rust Embedded の世界で非常に中心的なソフトウェア
である [`embedded-hal`] を見ていきます。 その名前が示すとおり、これは
先ほど見た第 2 の抽象化レベル、つまり HAL に関係しています。
[`embedded-hal`] の考え方は、
各 HAL における特定のペリフェラルのすべての実装で通常共有される
振る舞いを記述する trait の集合を提供することです。 たとえば、
ピンの電源をオンまたはオフにできる関数は常にあると期待するでしょう。
これは、ボード上の LED を点灯・消灯したり、そのほかの何かに使ったりするためです。

`embedded-hal` を使うと、たとえば温度
センサーのようなハードウェア向けのドライバを、[`embedded-hal`] trait の実装が
存在するあらゆるチップで使える形で書けます。 これは、ドライバを
[`embedded-hal`] trait にのみ依存するように書くことで
実現されます。 そのように書かれたドライバは *プラットフォーム非依存* と呼ばれます。
幸い、`crates.io` から入手するドライバのほとんどはプラットフォーム非依存です。

[`embedded-hal`]: https://crates.io/crates/embedded-hal


## さらに読む

これらの抽象化レベルについてさらに学びたい場合は、Franz Skarman（別名 [TheZoq2]）が
Oxidize 2020 でこのトピックについて講演しています: [An Overview of the Embedded Rust Ecosystem]。

[TheZoq2]: https://github.com/TheZoq2/
[An Overview of the Embedded Rust Ecosystem]: https://www.youtube.com/watch?v=vLYit_HHPaY