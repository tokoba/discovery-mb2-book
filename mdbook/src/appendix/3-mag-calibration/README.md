# 磁力計のキャリブレーション

センサーを使ってアプリケーションを開発しようとする前に行うべき非常に重要なことの 1 つは、
その出力が本当に正しいかを確認することです。そうでない場合は、センサーを
キャリブレーションする必要があります。あるいは、センサーが壊れている可能性もあります。可能であれば、
使用前および使用中にセンサーの健全性をチェックするのは非常に良い考えです。

私の場合、2 台の異なる MB2 で、LSM303AGR の磁力計はキャリブレーションなしだとかなり
ずれています。（z 軸が壊れているように見える個体も 1 台あります。メーカーにはこれを検出するのに役立つ追加の
ハードウェアと手順がありますが、ここではその複雑さは扱いません。）

磁力計をキャリブレーションするための、メーカーが規定した手順があります。このキャリブレーションには
かなり多くの数学（行列）が関わるため、ここでは詳しく扱いません。詳細に興味がある場合は、この [Design Note]
でその手順が説明されています。

[Design Note]: https://www.st.com/resource/en/design_tip/dt0103-compensating-for-magnetometer-installation-error-and-hardiron-effects-using-accelerometerassisted-2d-calibration-stmicroelectronics.pdf

幸いなことに、micro:bit 向けの元の C++ ソフトウェアを作成した CODAL グループは、メーカーの
キャリブレーション機構（またはそれに類するもの）を、すでに C++ で [here] に実装しています。

[here]: https://github.com/lancaster-university/codal-microbit-v2/blob/006abf5566774fbcf674c0c7df27e8a9d20013de/source/MicroBitCompassCalibrator.cpp

この C++ のキャリブレーションを Rust に移植したものは `src/lib.rs` にあります。これは
Matlab から C++、そして Rust へと翻訳されたものであり、いくつか興味深い選択がされていることに注意してください。特に、
キャリブレーション済みの値を読むときは *軸が反転されます*。これは、USB
コネクタが前になるように上から見たとき、キャリブレーション済みの値の X、Y、Z 軸が「標準的な」（右、前、上）
向きになるようにするためです。

このキャリブレーターの使い方は、ここにある `src/main.rs` で示されています。

ユーザーがどのようにキャリブレーションを行うかは、C++ 版のこの動画で示されています。（冒頭の
表示は無視してください — キャリブレーションはだいたい半分くらいから始まります。）

<p align="center">
<video src="https://video.microbit.org/support/compass+calibration.mp4" loop="true" autoplay="true" />
</p>

LED マトリクス上のすべての LED が点灯するまで、micro:bit を傾ける必要があります。点滅するカーソルが
現在の対象 LED を示します。

キャリブレーション行列はデモプログラムによって表示されることに注意してください。この行列は、
たとえば [chapter 12] のコンパスプログラムのようなプログラムにハードコードすることもできますし、
（あるいは何らかの方法で flash のどこかに保存してもよく）、ユーザーがプログラムを実行するたびに
再キャリブレーションする必要をなくせます。

[chapter 13]: ../../13-led-compass/index.html