# 必要なハードウェアと知識

この本を読むうえで主に必要となる知識は、Rust を *ある程度* 知っていることです。ここでいう
*ある程度* をどのくらいか定量化するのは難しいです。ジェネリクスとトレイトの基本に慣れているとかなり役立ちます。クロージャを
どう *使う* かは知っておく必要があります。また、現在の Rust
[edition] のイディオムにも慣れている必要があります。

[edition]: https://rust-lang-nursery.github.io/edition-guide/

また、この教材を進めるには次のものが必要です。

- [Micro:Bit v2]（MB2）ボード。

  [micro:bit v2]: https://tech.microbit.org/hardware/

  このボードは、Amazon や Ali Baba を含む多くの販売元から購入できます。
  販売元の[一覧][0]は、MB2 の製造元である BBC から
  直接入手できます。

  [0]: https://microbit.org/buy/

  <p align="center">
  <img title="micro:bit" src="../assets/microbit-v2.jpg" width="500" />
  </p>

  利用可能な `V2` ボードには複数のバージョンがあります。
  ここでの教材は `V2.00` 向けに書かれていますが、
  どの `V2` ボードでも問題なく動作するはずです。

- micro-B USB ケーブル（特別なものではありません — おそらく何本も持っているでしょう）。これは、
  バッテリーを使わないときに micro:bit ボードへ給電し、ボードと通信するために必要です。  
  ケーブルによっては機器の充電しかできないものもあるため、
  データ転送に対応していることを確認してください。

  <p align="center">
  <img title="micro-B USB ケーブル" src="../assets/usb-cable.jpg" width="500" />
  </p>

  > **注記** 一部の micro:bit キットには、このようなケーブルが同梱されています。 他のモバイル
  > 機器で使っている USB ケーブルでも、micro-B でデータ伝送が可能であれば使えるはずです。

  公式の `micro:bit Go` キットには、USB ケーブルと、USB なしで
  MB2 に給電するための便利なバッテリーパックの両方が含まれています。

> **FAQ**: ちょっと待って、なぜこの特定のハードウェアが必要なのですか？

そのほうが、私にとってもあなたにとってもずっと楽になるからです。

ハードウェアの違いを気にしなくてよければ、この教材ははるかに、はるかに取り組みやすくなります。
これについては私を信じてください。

> **FAQ**: 別の開発ボードでもこの教材を進められますか？

たぶん可能です。主に次の 2 つに依存します。あなたがマイクロコントローラーについてどれだけ
これまでの経験を持っているか、そして／または
あなたの開発ボード向けの高レベル crate がどこかにすでに存在するかどうかです。おそらく最低でも
ここで使っている [`nrf52833-hal`] のような HAL crate は欲しいでしょう。ここで使っている
[`microbit-v2`] のような Board Support crate があるボードのほうが望ましいかもしれません。
別のマイクロコントローラーを使うつもりなら、[Awesome Embedded Rust] を見たり、
単に Web を検索したりして、サポートされている crate を探せます。

[`microbit-v2`]: https://docs.rs/microbit-v2
[`nrf52833-hal`]: https://docs.rs/nrf52833-hal
[Awesome Embedded Rust]: https://github.com/rust-embedded/awesome-embedded-rust

私の考えでは、別の開発ボードを使うと、この文章は初心者向けの親しみやすさと
「追いやすさ」の大半、あるいはほとんどすべてを失います。あらかじめ警告しておきます。

別の Arm ベースの開発ボードを持っていて、自分を完全な
初心者だとは思わないのであれば、[quickstart] プロジェクトテンプレートから始めることを検討してもよいでしょう。

[quickstart]: https://rust-embedded.github.io/cortex-m-quickstart/cortex_m_quickstart/