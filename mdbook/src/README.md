# `micro::bit v2 Embedded Discovery Book`

> [Rust] を通してマイクロコントローラの世界を発見しましょう！

[Rust]: https://www.rust-lang.org/

本書は、通常の C/C++ ではなく Rust を学習言語として用いる、マイクロコントローラベースの
組み込みシステムに関する入門コースです。

## 扱う内容

以下のトピックを扱う予定です（いずれ、そうなればと思っています）:

- 「組み込み」(Rust) プログラムの書き方、ビルド、フラッシュ、デバッグの方法。

- マイクロコントローラで一般的に見られる機能（「ペリフェラル」）: デジタル入力と出力、
  パルス幅変調 (PWM)、アナログ-デジタルコンバータ (ADC)、シリアル、I2C、SPI などの
  一般的な通信プロトコル、など。

- マルチタスクの概念: 協調型 vs プリエンプティブ型マルチタスク、割り込み、スケジューラ、など。

- 制御システムの概念: センサー、キャリブレーション、デジタルフィルタ、アクチュエータ、
  開ループ制御、閉ループ制御、など。

## 進め方

- 初心者向けです。マイクロコントローラや組み込みシステムの事前経験は必要ありません。

- ハンズオンです。理論を実践に移すための演習を豊富に用意しています。 *あなた* が行うのは
  ここでの作業の大部分です。

- ツール中心です。開発を容易にするツールを積極的に活用します。GDB を使った "本物の"
  デバッグやロギングも早い段階で導入します。LED をデバッグ手段として使う余地はありません。

## 対象外

本書の対象外となるものは次のとおりです:

- Rust を教えること。その話題については、すでに十分な教材があります。ここではマイクロ
  コントローラと組み込みシステムに焦点を当てます。

- 電気回路理論や電子工学についての包括的な教科書であること。ここでは、一部のデバイスが
  どのように動作するかを理解するために必要な最小限の内容だけを扱います。

- リンカスクリプトやブートプロセスのような詳細を扱うこと。たとえば、既存のツールを使って
  コードをボードに書き込めるようにしますが、それらのツールの仕組みまでは詳しく扱いません。

また、この教材を他の開発ボード向けに移植するつもりもなく、本書では micro:bit 開発ボード
のみを使用します。

## 問題の報告

本書のソースは [this repository] にあります。誤字やコードの問題を見つけた場合は、
[issue tracker] で報告してください。

[this repository]: https://github.com/rust-embedded/discovery-mb2
[issue tracker]: https://github.com/rust-embedded/discovery-mb2/issues

## その他の組み込み Rust リソース

この Discovery book は、[Embedded Working Group] が提供する複数の組み込み Rust リソース
の 1 つにすぎません。全体の一覧は [The Embedded Rust Bookshelf] にあります。ここには
[Frequently Asked Questions] の一覧も含まれています。

[Embedded Working Group]: https://github.com/rust-embedded/wg
[The Embedded Rust Bookshelf]: https://docs.rust-embedded.org
[Frequently Asked Questions]: https://docs.rust-embedded.org/faq.html