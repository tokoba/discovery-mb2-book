# ハードウェアを知る

これから扱うハードウェアに慣れていきましょう。

## micro:bit

<p align="center">
<img title="micro:bit" src="../assets/microbit-v2.jpg" />
</p>

以下は、このボード上にある数多くのコンポーネントの一部です。

- 1つの[マイクロコントローラー]。
- 複数の LED、特に背面の LED マトリクス
- 2つのユーザーボタンと、リセットボタン（USB ポートの隣にあるもの）。
- 1つの USB ポート。
- [磁力計]と[加速度計]の両方の機能を持つセンサー

[マイクロコントローラー]: https://en.wikipedia.org/wiki/Microcontroller
[加速度計]: https://en.wikipedia.org/wiki/Accelerometer
[磁力計]: https://en.wikipedia.org/wiki/Magnetometer

これらのコンポーネントの中で最も重要なのはマイクロコントローラーで（「microcontroller unit」の略で「MCU」と
呼ばれることもあります）、これは USB ポートがある側のボード上にある
2つの黒い四角のうち大きい方です。MCU がコードを実行します。
「ボードをプログラムする」という表現を目にすることもあるかもしれませんが、
実際に行っているのは、ボードに搭載されている MCU をプログラムすることです。

ボードのより詳しい説明に興味があれば、
[micro:bit のウェブサイト](https://tech.microbit.org/hardware/) を確認できます。

MCU は非常に重要なので、このボードに載っているものをもう少し詳しく見てみましょう。