# 型安全な操作

`P0` のレジスタの 1 つである `IN` レジスタは、読み取り専用レジスタとして文書化されています。

> 6.8.2.4 IN - 145 ページと 146 ページ

表の 'Access' 列では、このレジスタには 'R' しか示されていないことに注意してください。 このレジスタに書き込むべきではなく、さもないとまずいことが起こるかもしれません。

レジスタには異なる読み取り / 書き込み権限があります。書き込み専用のものもあれば、読み書きできるものもあり、当然、読み取り専用のものもあります。

16 進アドレスを直接扱うのもエラーの元です。すでに見たように、無効なメモリアドレスにアクセスしようとすると例外が発生し、プログラムの実行が妨げられました。

レジスタを「安全」に操作できる API があったらよいと思いませんか？ 理想的には、その API は私が挙げた次の 3 点を表現しているべきです。実際のアドレスを直接いじらないこと、読み取り / 書き込み権限を尊重すること、そしてレジスタの予約済み部分の変更を防ぐことです。

実は、あります！ `registers::init()` は実際に、`P0` および `P1` ポートのレジスタを型安全に操作する API を提供する値を返します。

覚えているかもしれませんが、あるペリフェラルに関連付けられたレジスタのグループはレジスタブロックと呼ばれ、連続したメモリ領域に配置されます。この型安全 API では、各レジスタブロックは `struct` としてモデル化され、その各フィールドが 1 つのレジスタを表します。各レジスタフィールドは、たとえば `u32` に対する別々の newtype であり、読み取り / 書き込み権限に応じて `read`、`write`、`modify` の組み合わせのメソッドを公開します。最後に、これらのメソッドは `u32` のようなプリミティブ値を受け取るのではなく、ビルダーパターンを使って構築でき、レジスタの予約済み部分の変更を防ぐさらに別の newtype を受け取ります。

この API に慣れる最良の方法は、実行中のサンプルをこれに移植することです
(`examples/type-safe.rs`)。

```rust
{{#include examples/type-safe.rs}}
```

最初に気づくのは、マジックアドレスが一切出てこないことです。代わりに、`P0` ポートのレジスタブロック内の `OUT` レジスタを参照するために、`p0.out` という、より人にとってわかりやすい方法を使います。

このレジスタブロックにはクロージャを受け取る [`modify`] メソッドがあります。このクロージャが呼び出される前に、`OUT` レジスタの値が読み取られ、その値が `r` パラメータとしてクロージャに渡されます。`r` の値をもとに、そのメソッドを使って `w` をレジスタの望ましい新しい値へ操作できます。クロージャが返ると、その結果がレジスタに書き込まれます。今回のケースでは、レジスタの現在の値も `w` パラメータに渡されるため、レジスタの残りのビットをそのままにしておきたい場合は、`w` だけを操作すれば済みます。

`modify` メソッドは、書き込みと読み取りの両方を許可するレジスタに対して定義されています。レジスタの値を読み取るだけで更新したくない場合は、[`read`] メソッドを使えます。あるいは、読み取らずに単にレジスタ値を書き込みたい場合は、[`write`] メソッドがあります。

読み取り専用レジスタは `read` だけを公開し、書き込み専用レジスタは `write` だけを公開します。これにより、許可されていない方法でユーザーがレジスタへアクセスすることを防げるため、呼び出しを `unsafe` ブロックで囲む必要はありません。また、正確なレジスタアドレスやビット位置を自分で突き止める必要もありません！

[`write`]: https://docs.rs/svd2rust/latest/svd2rust/#write
[`read`]: https://docs.rs/svd2rust/latest/svd2rust/#read
[`modify`]: https://docs.rs/svd2rust/latest/svd2rust/#modify

このプログラムを実行してみましょう！ プログラムをデバッグしている *間* にできる、興味深いことがいくつかあります。

`p0` は `P0` ポートのレジスタブロックへの参照です。`print p0` は
レジスタブロックのベースアドレスを返し、`print *p0` はその値を表示します。

```sh
$ cargo run
(..)
Target halted
(gdb) set mem inaccessible-by-default off
(gdb) break main.rs:12
Breakpoint 4 at 0x162: main.rs:12. (2 locations)
(gdb) continue
Continuing.

Program received signal SIGINT, Interrupt.
cortex_m_rt::DefaultPreInit () at src/lib.rs:1058
1058	pub unsafe extern "C" fn DefaultPreInit() {}
(gdb) continue
Continuing.

Breakpoint 1, registers::__cortex_m_rt_main_trampoline () at src/07-registers/src/main.rs:7
7	#[entry]
(gdb) continue
Continuing.

Program received signal SIGINT, Interrupt.
registers::__cortex_m_rt_main () at src/07-registers/src/main.rs:8
8	fn main() -> ! {
(gdb) continue
Continuing.

Breakpoint 4.2, registers::__cortex_m_rt_main () at src/07-registers/src/main.rs:12
12	    p0.out.modify(|_, w| w.pin21().set_bit());
(gdb) print *p0                                               ; ⬅️ ここで `*p0` を表示します！
$1 = nrf52833_pac::p0::RegisterBlock {
  _reserved0: [0 <repeats 1284 times>],
  out: nrf52833_pac::generic::Reg<nrf52833_pac::p0::out::OUT_SPEC> {
    register: vcell::VolatileCell<u32> {
      value: core::cell::UnsafeCell<u32> {
        value: 0
      }
    },
    _marker: core::marker::PhantomData<nrf52833_pac::p0::out::OUT_SPEC>
  },
  outset: nrf52833_pac::generic::Reg<nrf52833_pac::p0::outset::OUTSET_SPEC> {
    register: vcell::VolatileCell<u32> {
      value: core::cell::UnsafeCell<u32> {
        value: 0
      }
    },
    _marker: core::marker::PhantomData<nrf52833_pac::p0::outset::OUTSET_SPEC>
  },
  outclr: nrf52833_pac::generic::Reg<nrf52833_pac::p0::outclr::OUTCLR_SPEC> {
    register: vcell::VolatileCell<u32> {
      value: core::cell::UnsafeCell<u32> {
        value: 0
      }
    },
    _marker: core::marker::PhantomData<nrf52833_pac::p0::outclr::OUTCLR_SPEC>
  },
  in_: nrf52833_pac::generic::Reg<nrf52833_pac::p0::in_::IN_SPEC> {
    register: vcell::VolatileCell<u32> {
      value: core::cell::UnsafeCell<u32> {
        value: 0
      }
    },
    _marker: core::marker::PhantomData<nrf52833_pac::p0::in_::IN_SPEC>
  },
  dir: nrf52833_pac::generic::Reg<nrf52833_pac::p0::dir::DIR_SPEC> {
    register: vcell::VolatileCell<u32> {
      value: core::cell::UnsafeCell<u32> {
        value: 3513288704
      }
    },
    _marker: core::marker::PhantomData<nrf52833_pac::p0::dir::DIR_SPEC>
  },
  dirset: nrf52833_pac::generic::Reg<nrf52833_pac::p0::dirset::DIRSET_SPEC> {
    register: vcell::VolatileCell<u32> {
      value: core::cell::UnsafeCell<u32> {
        value: 3513288704
      }
    },
    _marker: core::marker::PhantomData<nrf52833_pac::p0::dirset::DIRSET_SPEC>
  },
  dirclr: nrf52833_pac::generic::Reg<nrf52833_pac::p0::dirclr::DIRCLR_SPEC> {
    register: vcell::VolatileCell<u32> {
      value: core::cell::UnsafeCell<u32> {
        value: 3513288704
      }
    },
    _marker: core::marker::PhantomData<nrf52833_pac::p0::dirclr::DIRCLR_SPEC>
  },
  latch: nrf52833_pac::generic::Reg<nrf52833_pac::p0::latch::LATCH_SPEC> {
    register: vcell::VolatileCell<u32> {
      value: core::cell::UnsafeCell<u32> {
        value: 0
      }
    },
    _marker: core::marker::PhantomData<nrf52833_pac::p0::latch::LATCH_SPEC>
  },
  detectmode: nrf52833_pac::generic::Reg<nrf52833_pac::p0::detectmode::DETECTMODE_SPEC> {
    register: vcell::VolatileCell<u32> {
      value: core::cell::UnsafeCell<u32> {
        value: 0
      }
    },
    _marker: core::marker::PhantomData<nrf52833_pac::p0::detectmode::DETECTMODE_SPEC>
  },
  _reserved9: [0 <repeats 472 times>],
  pin_cnf: [nrf52833_pac::generic::Reg<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC> {
      register: vcell::VolatileCell<u32> {
        value: core::cell::UnsafeCell<u32> {
          value: 2
        }
      },
      _marker: core::marker::PhantomData<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC>
    } <repeats 11 times>, nrf52833_pac::generic::Reg<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC> {
      register: vcell::VolatileCell<u32> {
        value: core::cell::UnsafeCell<u32> {
          value: 3
        }
      },
      _marker: core::marker::PhantomData<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC>
    }, nrf52833_pac::generic::Reg<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC> {
      register: vcell::VolatileCell<u32> {
        value: core::cell::UnsafeCell<u32> {
          value: 2
        }
      },
      _marker: core::marker::PhantomData<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC>
    }, nrf52833_pac::generic::Reg<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC> {
      register: vcell::VolatileCell<u32> {
        value: core::cell::UnsafeCell<u32> {
          value: 2
        }
      },
--Type <RET> for more, q to quit, c to continue without paging--c
      _marker: core::marker::PhantomData<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC>
    }, nrf52833_pac::generic::Reg<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC> {
      register: vcell::VolatileCell<u32> {
        value: core::cell::UnsafeCell<u32> {
          value: 2
        }
      },
      _marker: core::marker::PhantomData<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC>
    }, nrf52833_pac::generic::Reg<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC> {
      register: vcell::VolatileCell<u32> {
        value: core::cell::UnsafeCell<u32> {
          value: 3
        }
      },
      _marker: core::marker::PhantomData<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC>
    }, nrf52833_pac::generic::Reg<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC> {
      register: vcell::VolatileCell<u32> {
        value: core::cell::UnsafeCell<u32> {
          value: 2
        }
      },
      _marker: core::marker::PhantomData<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC>
    }, nrf52833_pac::generic::Reg<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC> {
      register: vcell::VolatileCell<u32> {
        value: core::cell::UnsafeCell<u32> {
          value: 2
        }
      },
      _marker: core::marker::PhantomData<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC>
    }, nrf52833_pac::generic::Reg<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC> {
      register: vcell::VolatileCell<u32> {
        value: core::cell::UnsafeCell<u32> {
          value: 2
        }
      },
      _marker: core::marker::PhantomData<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC>
    }, nrf52833_pac::generic::Reg<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC> {
      register: vcell::VolatileCell<u32> {
        value: core::cell::UnsafeCell<u32> {
          value: 3
        }
      },
      _marker: core::marker::PhantomData<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC>
    }, nrf52833_pac::generic::Reg<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC> {
      register: vcell::VolatileCell<u32> {
        value: core::cell::UnsafeCell<u32> {
          value: 2
        }
      },
      _marker: core::marker::PhantomData<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC>
    }, nrf52833_pac::generic::Reg<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC> {
      register: vcell::VolatileCell<u32> {
        value: core::cell::UnsafeCell<u32> {
          value: 3
        }
      },
      _marker: core::marker::PhantomData<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC>
    }, nrf52833_pac::generic::Reg<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC> {
      register: vcell::VolatileCell<u32> {
        value: core::cell::UnsafeCell<u32> {
          value: 3
        }
      },
      _marker: core::marker::PhantomData<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC>
    }, nrf52833_pac::generic::Reg<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC> {
      register: vcell::VolatileCell<u32> {
        value: core::cell::UnsafeCell<u32> {
          value: 2
        }
      },
      _marker: core::marker::PhantomData<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC>
    }, nrf52833_pac::generic::Reg<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC> {
      register: vcell::VolatileCell<u32> {
        value: core::cell::UnsafeCell<u32> {
          value: 3
        }
      },
      _marker: core::marker::PhantomData<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC>
    }, nrf52833_pac::generic::Reg<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC> {
      register: vcell::VolatileCell<u32> {
        value: core::cell::UnsafeCell<u32> {
          value: 2
        }
      },
      _marker: core::marker::PhantomData<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC>
    }, nrf52833_pac::generic::Reg<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC> {
      register: vcell::VolatileCell<u32> {
        value: core::cell::UnsafeCell<u32> {
          value: 2
        }
      },
      _marker: core::marker::PhantomData<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC>
    }, nrf52833_pac::generic::Reg<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC> {
      register: vcell::VolatileCell<u32> {
        value: core::cell::UnsafeCell<u32> {
          value: 2
        }
      },
      _marker: core::marker::PhantomData<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC>
    }, nrf52833_pac::generic::Reg<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC> {
      register: vcell::VolatileCell<u32> {
        value: core::cell::UnsafeCell<u32> {
          value: 3
        }
      },
      _marker: core::marker::PhantomData<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC>
    }, nrf52833_pac::generic::Reg<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC> {
      register: vcell::VolatileCell<u32> {
        value: core::cell::UnsafeCell<u32> {
          value: 2
        }
      },
      _marker: core::marker::PhantomData<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC>
    }, nrf52833_pac::generic::Reg<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC> {
      register: vcell::VolatileCell<u32> {
        value: core::cell::UnsafeCell<u32> {
          value: 3
        }
      },
      _marker: core::marker::PhantomData<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC>
    }, nrf52833_pac::generic::Reg<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC> {
      register: vcell::VolatileCell<u32> {
        value: core::cell::UnsafeCell<u32> {
          value: 3
        }
      },
      _marker: core::marker::PhantomData<nrf52833_pac::p0::pin_cnf::PIN_CNF_SPEC>
    }]
}
```

こうした newtype やクロージャは、巨大で肥大化したプログラムを生成しそうに聞こえるかもしれません。しかし実際には、[LTO] を有効にしてプログラムをリリースモードでコンパイルすると、`write_volatile` と16進数アドレスを使っていた「unsafe」版とまったく同じ命令が生成されていることがわかります。

[LTO]: https://en.wikipedia.org/wiki/Interprocedural_optimization

`cargo objdump` を使ってアセンブリコードを `release.type-safe.dump` に出力します:

``` console
cargo objdump -q --release --example type-safe -- --disassemble --no-show-raw-insn  > release.type-safe.dump
```

次に、`release.type-safe.dump` の中で `main` を検索します
```
00000158 <main>:
     158:      	push	{r7, lr}
     15a:      	mov	r7, sp
     15c:      	bl	0x160 <registers::__cortex_m_rt_main::h0e9b57c6799332fd> @ imm = #0x0

00000160 <registers::__cortex_m_rt_main::h0e9b57c6799332fd>:
     160:      	push	{r7, lr}
     162:      	mov	r7, sp
     164:      	bl	0x192 <registers::init::hec71dddc40be11b5> @ imm = #0x2a
     168:      	movw	r0, #0x504
     16c:      	movt	r0, #0x5000
     170:      	ldr	r1, [r0]
     172:      	orr	r1, r1, #0x200000
     176:      	str	r1, [r0]
     178:      	ldr	r1, [r0]
     17a:      	orr	r1, r1, #0x80000
     17e:      	str	r1, [r0]
     180:      	ldr	r1, [r0]
     182:      	bic	r1, r1, #0x200000
     186:      	str	r1, [r0]
     188:      	ldr	r1, [r0]
     18a:      	bic	r1, r1, #0x80000
     18e:      	str	r1, [r0]
     190:      	b	0x190 <registers::__cortex_m_rt_main::h0e9b57c6799332fd+0x30> @ imm = #-0x4
```

これによって、`ptr::read_volatile` と `ptr::write_volatile` の呼び出しを使ったものとまったく同じバイナリが生成されることを確認できます。

この話のいちばん素晴らしい点は、GPIO API を実装するためのコードを誰一人として1行も書く必要がなかったことです。すべてのコードは、[svd2rust] ツールを使って System View Description (SVD) ファイルから自動生成されました。この SVD ファイルは実際にはマイクロコントローラのベンダーが提供する XML ファイルで、各マイクロコントローラのレジスタマップが含まれています。このファイルには、レジスタブロックのレイアウト、ベースアドレス、各レジスタの読み取り/書き込み権限、レジスタのレイアウト、レジスタに予約済みビットがあるかどうか、その他多くの有用な情報が含まれています。

[svd2rust]: https://crates.io/crates/svd2rust