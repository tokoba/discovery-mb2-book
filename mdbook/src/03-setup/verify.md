# インストールを確認する

すべてのツールが正しくインストールされたことを確認しましょう。

## cargo-embed の確認

まず、USB ケーブルを使って micro:bit をコンピューターに接続します。

micro:bit の USB ポートのすぐ横にあるオレンジ色の LED が少なくとも点灯するはずです。さらに、これまでに
micro:bit に別のプログラムを書き込んだことがない場合は、micro:bit に最初から入っているデフォルトのプログラムが
背面の赤い LED を点滅させ始めるはずです。これらは無視してもかまいませんし、デモアプリで遊んでもかまいません。

それでは、probe-rs と、ひいては cargo-embed が micro:bit を認識できるか確認しましょう。これには、
次のコマンドを実行します:

``` console
$ probe-rs list
The following debug probes were found:
[0]: BBC micro:bit CMSIS-DAP -- 0d28:0204:990636020005282030f57fa14252d446000000006e052820 (CMSIS-DAP)
```

また、micro:bit のデバッグ機能についてさらに詳しい情報が必要であれば、次を実行できます:

``` console
$ probe-rs info
Probing target via JTAG

Error identifying target using protocol JTAG: The probe does not support the JTAG protocol.

Probing target via SWD

Arm Chip with debug port Default:
Debug Port: DPv1, DP Designer: Arm Ltd
├── 0 MemoryAP
│   └── ROM Table (Class 1), Designer: Nordic VLSI ASA
│       ├── Cortex-M4 SCS   (Generic IP component)
│       │   └── CPUID
│       │       ├── IMPLEMENTER: Arm Ltd
│       │       ├── VARIANT: 0
│       │       ├── PARTNO: Cortex-M4
│       │       └── REVISION: 1
│       ├── Cortex-M3 DWT   (Generic IP component)
│       ├── Cortex-M3 FBP   (Generic IP component)
│       ├── Cortex-M3 ITM   (Generic IP component)
│       ├── Cortex-M4 TPIU  (Coresight Component)
│       └── Cortex-M4 ETM   (Coresight Component)
└── 1 Unknown AP (Designer: Nordic VLSI ASA, Class: Undefined, Type: 0x0, Variant: 0x0, Revision: 0x0)


Debugging RISC-V targets over SWD is not supported. For these targets, JTAG is the only supported protocol. RISC-V specific information cannot be printed.
Debugging Xtensa targets over SWD is not supported. For these targets, JTAG is the only supported protocol. Xtensa specific information cannot be printed.
```

次に、この本のソースコードの `src/03-setup` ディレクトリにいることを確認してください。続いて、次のコマンドを実行します:

```
$ rustup target add thumbv7em-none-eabihf
$ cargo embed --target thumbv7em-none-eabihf
```

すべてが正しく動作していれば、cargo-embed はまずこのディレクトリ内の小さなサンプルプログラムを
コンパイルし、その後それを書き込み、最後に Hello World を表示する見やすいテキストベースのユーザーインターフェースを
開くはずです。

（そうならない場合は、[general troubleshooting] の手順を確認してください。）

[general troubleshooting]: ../appendix/1-general-troubleshooting/index.html

この出力は、今 micro:bit に書き込んだ小さな Rust プログラムから送られてきています。
すべて正常に動作しています。次の章へ進んでください！