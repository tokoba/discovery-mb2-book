# `0xBAAAAAAD` アドレス

すべてのペリフェラルメモリにアクセスできるわけではありません。このプログラム（`examples/bad.rs`）を見てください。

```rust
{{#include examples/bad.rs}}
```

このアドレスは前に使った `OUT` アドレスに近いですが、このアドレスは *無効* です。つまり、
このアドレスにはレジスタが存在しません。

では、試してみましょう。

``` console
$ cargo run
(..)
Resetting and halting target
Target halted
(gdb) continue
Continuing.

Breakpoint 1, registers::__cortex_m_rt_main_trampoline () at src/07-registers/src/main.rs:9
9	#[entry]
(gdb) continue
Continuing.

Program received signal SIGINT, Interrupt.
registers::__cortex_m_rt_main () at src/07-registers/src/main.rs:10
10	fn main() -> ! {
(gdb) continue
Continuing.

Breakpoint 3, cortex_m_rt::HardFault_ (ef=0x2001ffb8) at src/lib.rs:1046
1046	    loop {}
(gdb) 
```

存在しないメモリを読むという無効な操作を行おうとしたため、プロセッサは *例外*、
つまり *ハードウェア* 例外を発生させました。

多くの場合、例外はプロセッサが無効な操作を実行しようとしたときに発生します。
例外はプログラムの通常の流れを中断し、プロセッサに *例外ハンドラ* を実行させます。
これは単なる関数／サブルーチンです。

例外にはさまざまな種類があります。各種類の例外は異なる条件で発生し、
それぞれ別の例外ハンドラによって処理されます。

`registers` クレートは `cortex-m-rt` クレートに依存しており、このクレートは
`HardFault_` という名前のデフォルトの *ハードフォールト* ハンドラを定義しています。
これは「無効なメモリアドレス」例外を処理します。`embed.gdb` は `HardFault` に
ブレークポイントを設定していたため、デバッガは例外ハンドラの実行中にプログラムを
停止しました。デバッガからこの例外についてさらに詳しい情報を得ることができます。
見てみましょう:

```
(gdb) list
1040  #[allow(unused_variables)]
1041	#[doc(hidden)]
1042	#[cfg_attr(cortex_m, link_section = ".HardFault.default")]
1043	#[no_mangle]
1044	pub unsafe extern "C" fn HardFault_(ef: &ExceptionFrame) -> ! {
1045	    #[allow(clippy::empty_loop)]
1046	    loop {}
1047	}
1048	
1049	#[doc(hidden)]
1050	#[no_mangle]
```

`ef` は、例外が発生する直前のプログラム状態のスナップショットです。これを調べてみましょう:

```
(gdb) print/x *ef
$1 = cortex_m_rt::ExceptionFrame {
  r0: 0x5000a784,
  r1: 0x3,
  r2: 0x2001ff24,
  r3: 0x0,
  r12: 0x1,
  lr: 0x4403,
  pc: 0x43ea,
  xpsr: 0x1000000
}
```

ここにはいくつかのフィールドがありますが、最も重要なのはプログラムカウンタレジスタである `pc` です。この
レジスタ内のアドレスは、例外を発生させた命令を指しています。問題のある命令の周辺を逆アセンブルしてみましょう。

```
(gdb) disassemble /m ef.pc
Dump of assembler code for function core::ptr::read_volatile<u32>:
1654	pub unsafe fn read_volatile<T>(src: *const T) -> T {
   0x000043d2 <+0>:	push	{r7, lr}
   0x000043d4 <+2>:	mov	r7, sp
   0x000043d6 <+4>:	sub	sp, #16
   0x000043d8 <+6>:	str	r0, [sp, #4]
   0x000043da <+8>:	str	r0, [sp, #8]

1655	    // SAFETY: 呼び出し元は `volatile_load` の安全性契約を守らなければならない。
1656	    unsafe {
1657	        assert_unsafe_precondition!(
   0x000043dc <+10>:	b.n	0x43de <core::ptr::read_volatile<u32>+12>
   0x000043de <+12>:	ldr	r0, [sp, #4]
   0x000043e0 <+14>:	movs	r1, #4
   0x000043e2 <+16>:	bl	0x43f4 <core::ptr::read_volatile::precondition_check>
   0x000043e6 <+20>:	b.n	0x43e8 <core::ptr::read_volatile<u32>+22>

1658	            check_language_ub,
1659	            "ptr::read_volatile requires that the pointer argument is aligned and non-null",
1660	            (
1661	                addr: *const () = src as *const (),
1662	                align: usize = align_of::<T>(),
1663	            ) => is_aligned_and_not_null(addr, align)
1664	        );
1665	        intrinsics::volatile_load(src)
   0x000043e8 <+22>:	ldr	r0, [sp, #4]
   0x000043ea <+24>:	ldr	r0, [r0, #0]          ; <-- これです！
   0x000043ec <+26>:	str	r0, [sp, #12]
   0x000043ee <+28>:	ldr	r0, [sp, #12]

1666	    }
1667	}
   0x000043f0 <+30>:	add	sp, #16
   0x000043f2 <+32>:	pop	{r7, pc}

End of assembler dump.

```

この例外は、読み取り命令である `ldr r0, [r0, #0]` 命令によって引き起こされました。この命令
は、`r0` *CPU レジスタ* が示すアドレスのメモリを読み取ろうとしました。ちなみに、CPU
（プロセッサ）レジスタはメモリマップドレジスタではありません。したがって、たとえば
`OUT` のような対応するアドレスはありません。

例外が発生したまさにその瞬間に、`r0` レジスタの値が何だったのか確認できたら便利だと
思いませんか。実は、もう確認しています！先ほど表示した `ef` の値にある `r0` フィールド
が、例外発生時の `r0` レジスタの値です。もう一度示します:

```
(gdb) print/x *ef
$1 = cortex_m_rt::ExceptionFrame {
  r0: 0x5000a784,
  r1: 0x3,
  r2: 0x2001ff24,
  r3: 0x0,
  r12: 0x1,
  lr: 0x4403,
  pc: 0x43ea,
  xpsr: 0x1000000
}
```

`r0` には `0x5000_A784` という値が入っており、これは `read_volatile`
関数を呼び出すときに渡した無効なアドレスです。