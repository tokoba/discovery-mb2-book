# UART

このマイクロコントローラ（多くのものと同様）には、UART（「Universal Asynchronous Receiver/Transmitter」）ペリフェラルがあります。MB2 には 2 種類の UART ペリフェラルがあり、古い `UART` と新しい `UARTE`（「Easy DMA を備えた UART」）があります。ハードウェアシリアルポートと通信するために、`UARTE` ペリフェラルを使用します。

この章全体を通して、シリアル通信を使用してマイクロコントローラとコンピュータの間で情報をやり取りします。