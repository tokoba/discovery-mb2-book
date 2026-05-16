# RTRM: リファレンスマニュアルを読む

これまでに、nRF52833 の GPIO ピンを見てきました。このチップでは（そして多くのほかのチップでも同様に）、GPIO ピンは *ポート* にグループ化されています。ポートは 2 つあり、それぞれ Port 0 と Port 1 で、略して `P0` と `P1` と呼ばれます。各ポート内のピンには 0 から始まる番号が付けられています。Port 0 には 32 本のピンがあり、`P0.00` から `P0.31` まで、Port 1 には 10 本のピンがあり、`P1.00` から `P1.09` まであります。

まず思い出す必要があるのは、どのピンがどの LED に接続されているかです。以前は回路図をたどってこれを調べました。しかし、それは実はハードモードです。必要な情報は MB2 の [pinmap table] にあります。

[pinmap table]: https://tech.microbit.org/hardware/schematic/#v2-pinmap

その表には次のように書かれています。

- `ROW1`、つまり最上段の LED 行は、ピン `P0.21` に接続されています。`P0.21` は「Port 0 の Pin 21」の短縮表記です。
- `ROW5`、つまり最下段の LED 行は、ピン `P0.19` に接続されています。

ここまでで、最上段と最下段の行をオン/オフするには、`P0.21` と `P0.19` の状態を変更したいのだと分かりました。これらのピンは Port 0 の一部なので、設定には `P0` ペリフェラルを使います。

各ペリフェラルには、それに対応するレジスタ *ブロック* があります。レジスタブロックとは、連続したメモリ上に割り当てられたレジスタの集合です。レジスタブロックの開始アドレスは、そのベースアドレスと呼ばれます。`P0` ペリフェラルのベースアドレスが何かを調べる必要があります。その情報は、マイクロコントローラの [Product Specification] の次の節にあります。

> Section 4.2.4 Instantiation - Page 22

その表によると、`P0` レジスタブロックのベースアドレスは `0x5000_0000` です。

また、各ペリフェラルにはドキュメント内にそれぞれ専用の節があります。これらの節の末尾には、そのペリフェラルのレジスタブロックに含まれるレジスタの表があります。`GPIO` ファミリのペリフェラルについては、その表は次の場所にあります。

> Section 6.8.2 Registers - Page 144

`OUT` は、セット/リセットに使用するレジスタです。`P0` のベースアドレスからのオフセット値は `0x504` です。`OUT` は [Product Specification] で確認できます。

そのレジスタの仕様は、`GPIO` レジスタ表のすぐ下にあります。

> Subsection 6.8.2.1 OUT - Page 145

とにかく、`0x5000_0000` + `0x504` = `0x50000504` です。見覚えがありますね！ ついに！

これが、私たちが書き込みを行っていたレジスタです。ドキュメントにはいくつか興味深いことが書かれています。まず、このレジスタは書き込みも読み出しもできます。次に、このレジスタは 32 ビットのメモリであり、各ビットが対応するピンの状態を表します。つまり、たとえば bit 19 は pin 19 に対応します。そのビットを 1 に設定するとピン出力が有効になり、0 に設定するとリセットされます。さらに、すべてのビットのリセット値が 0 なので、すべてのピン出力はデフォルトで無効になっていることも分かります。

GDB の `examine` コマンド `x` を使います。GDB サーバーの設定によっては、GDB は指定されていないメモリの読み出しを拒否します。次を実行すると、この挙動を無効にできます。

```
set mem inaccessible-by-default off
```

それでは始めましょう。まず `inaccessible-by-default` フラグをオフにしてから、いくつかブレークポイントを設定し、デバイスをリセットして停止します。

```
(gdb) set mem inaccessible-by-default off
(gdb) break 16
Breakpoint 1 at 0x172: file src/07-registers/src/main.rs, line 16.
Note: automatically using hardware breakpoints for read-only addresses.
(gdb) break 19
Breakpoint 2 at 0x17c: file src/07-registers/src/main.rs, line 19.
(gdb) break 22
Breakpoint 3 at 0x184: file src/07-registers/src/main.rs, line 22.
(gdb) break 25
Breakpoint 4 at 0x18c: file src/07-registers/src/main.rs, line 25.
(gdb) monitor reset halt
Resetting and halting target
Target halted
```

よし。では最初のブレークポイント、つまり 16 行目の直前まで実行を進めて、アドレス `0x50000504` にあるレジスタの内容を表示してみましょう。

```
(gdb) c
Continuing.

Breakpoint 1, registers::__cortex_m_rt_main () at src/07-registers/src/main.rs:16
16              *(PORT_P0_OUT as *mut u32) |= 1 << 21;
(gdb) x 0x50000504
0x50000504:     0x00000000
```

この時点で、レジスタの値が `0x00000000`、つまり `0` であることが分かります。これは [Product Specification] の記述と一致しており、そこでは `0` がこのレジスタの「リセット値」だとされています。つまり MCU がリセットされると、このレジスタの値は `0` になります。

先に進みましょう。この行はいくつかの命令（読み出し、ビット単位 OR、書き込み）から成っているので、次のブレークポイントに到達するまで、デバッガに複数回実行を継続させる必要があります。

```
(gdb) c
Continuing.

Program received signal SIGINT, Interrupt.
0x00000174 in registers::__cortex_m_rt_main () at src/07-registers/src/main.rs:16
16              *(PORT_P0_OUT as *mut u32) |= 1 << 21;
(gdb) c
Continuing.

Breakpoint 2, registers::__cortex_m_rt_main () at src/07-registers/src/main.rs:19
19              *(PORT_P0_OUT as *mut u32) |= 1 << 19;
```

19 行目の直前で停止しました。つまり、この時点で 16 行目は完全に実行されています。もう一度 `OUT` レジスタの内容を見てみましょう。

```
(gdb) x 0x50000504
0x50000504:     0x00200000
```

この時点での `OUT` レジスタの値は `0x00200000` で、10 進数では `2097152`、つまり `2^21` です。これは bit 21 が 1 に設定され、それ以外のビットは 0 に設定されていることを意味します。これは 16 行目のコードと対応しており、`1 << 21`、つまり 1 を左に 21 ビットシフトした値を、`OUT` の現在の値（このときは 0）とビット単位 OR したうえで、`OUT` レジスタに書き込んでいます。

`OUT` に `1 << 21`（`OUT[21]= 1`）を書き込むと、`P0.21` は *ハイ* になります。これにより、最上段の LED 行が *オン* になります。実際に最上段が点灯していることを確認してください。

```
(gdb) c
Continuing.
```

ええ、今それを言おうとしていました。では、もう一度 'c' を押して次のブレークポイントまで実行を進め、その値を表示してみましょう。

```
Program received signal SIGINT, Interrupt.
0x0000017e in registers::__cortex_m_rt_main () at src/07-registers/src/main.rs:19
19              *(PORT_P0_OUT as *mut u32) |= 1 << 19;
(gdb) c
Continuing.

Breakpoint 3, registers::__cortex_m_rt_main () at src/07-registers/src/main.rs:22
22              *(PORT_P0_OUT as *mut u32) &= !(1 << 21);
(gdb) x 0x50000504
0x50000504:     0x00280000
```

19 行目では、bit 21 をそのままにして `OUT` の bit 19 を 1 に設定しました。その結果が `0x00280000` で、10 進数では `2621440`、つまり `2^19 + 2^21` です。これは bit 19 と bit 21 の両方が 1 に設定されていることを意味します。

`OUT` に `1 << 19`（`OUT[19]= 1`）を書き込むと、`P0.19` は *ハイ* になります。これにより、最下段の LED 行が *オン* になります。したがって、この時点で最下段が点灯しているはずです。

続く行では、これらの行を再びオフにします。まず最上段、次に最下段です。今回は、ビット単位 AND 演算にビット単位 NOT を組み合わせています。`!(1 << 21)` を計算すると、bit 21 以外のすべてのビットが 1 になります。次にそれを `OUT` の現在の値とビット単位 AND することで、ほかのビットの値を保ったまま、bit 21 だけを 0 にします。

実行を続けて、報告される `OUT` レジスタの値が期待どおりかどうかを確認してください。`main` 関数の末尾にある無限ループにデバイスが入ったら、`CTRL+C` を押して実行を一時停止できます。
```
(gdb) c
Continuing.

Program received signal SIGINT, Interrupt.
0x00000186 in registers::__cortex_m_rt_main () at src/07-registers/src/main.rs:22
22              *(PORT_P0_OUT as *mut u32) &= !(1 << 21);
(gdb) c
Continuing.

Breakpoint 4, registers::__cortex_m_rt_main () at src/07-registers/src/main.rs:25
25              *(PORT_P0_OUT as *mut u32) &= !(1 << 19);
(gdb) x 0x50000504
0x50000504:     0x00080000
(gdb) c
Continuing.

Program received signal SIGINT, Interrupt.
0x0000018e in registers::__cortex_m_rt_main () at src/07-registers/src/main.rs:25
25              *(PORT_P0_OUT as *mut u32) &= !(1 << 19);
(gdb) c
Continuing.
^C
Program received signal SIGINT, Interrupt.
0x00000196 in registers::__cortex_m_rt_main () at src/07-registers/src/main.rs:28
28          loop {}
(gdb) x 0x50000504
0x50000504:     0x00000000
```

そして、この時点ですべてのLEDは再び消灯しているはずです！

[Product Specification]: https://docs-be.nordicsemi.com/bundle/ps_nrf52833/attach/nRF52833_PS_v1.7.pdf