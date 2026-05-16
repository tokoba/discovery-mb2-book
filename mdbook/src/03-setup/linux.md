# Linux

以下は、いくつかの Linux ディストリビューション向けのインストールコマンドです。

## Ubuntu 20.04 以降 / Debian 10 以降

> **NOTE** `gdb-multiarch` は、Arm Cortex-M プログラムをデバッグする際に使用する GDB コマンドです。
``` console
$ sudo apt install gdb-multiarch minicom libunwind-dev
```

## Fedora 32 以降

> **NOTE** `gdb` は、Arm
> Cortex-M プログラムをデバッグする際に使用する GDB コマンドです。
``` console
$ sudo dnf install gdb minicom libunwind-devel
```

## Arch Linux

> **NOTE** `gdb` は、Arm
> Cortex-M プログラムをデバッグする際に使用する GDB コマンドです。
``` console
$ sudo pacman -S arm-none-eabi-gdb minicom libunwind
```

## その他のディストリビューション

> **NOTE** `arm-none-eabi-gdb` は、Arm Cortex-M プログラムをデバッグする際に使用する GDB コマンドです。

[Arm のビルド済み
ツールチェーン](https://developer.arm.com/downloads/-/arm-gnu-toolchain-downloads) のパッケージがないディストリビューションでは、「Linux
64-bit」ファイルをダウンロードし、その `bin` ディレクトリをパスに追加してください。やり方の一例を示します。

``` console
$ mkdir -p ~/local
$ cd ~/local
$ tar xjf /path/to/downloaded/XXX.tar.bz2
```

次に、使用しているシェルの適切な初期化ファイル
（たとえば `~/.zshrc` や `~/.bashrc`）で、好みのエディターを使って `PATH` に次の内容を追加します。

```
PATH=$PATH:$HOME/local/XXX/bin
```

## udev ルール

これらのルールにより、micro:bit のような USB デバイスを root 権限、つまり `sudo` なしで使用できます。

以下の内容で、このファイルを `/etc/udev/rules.d` に作成してください。

``` console
$ cat /etc/udev/rules.d/69-microbit.rules
```

``` text
# micro:bit 用 CMSIS-DAP
ACTION!="add|change", GOTO="microbit_rules_end"
SUBSYSTEM=="usb", ATTR{idVendor}=="0d28", ATTR{idProduct}=="0204", TAG+="uaccess"
LABEL="microbit_rules_end"
```

次のコマンドで udev ルールを再読み込みします。

``` console
$ sudo udevadm control --reload
```

コンピューターにボードが接続されている場合は、いったん取り外してから再度接続するか、次の
コマンドを実行してください。

``` console
$ sudo udevadm trigger
```

## 権限の確認

USB ケーブルを使って micro:bit をコンピューターに接続します。

micro:bit は `/dev/bus/usb` に USB デバイス（ファイル）として現れるはずです。どのように
列挙されたかを確認してみましょう。

``` console
$ lsusb | grep -i "NXP Arm mbed"
Bus 001 Device 065: ID 0d28:0204 NXP Arm mbed
$ # ^^^        ^^^
```

私の場合、micro:bit はバス #1 に接続され、デバイス #65 として列挙されました。これは
ファイル `/dev/bus/usb/001/065` *が* micro:bit であることを意味します。ファイルの権限を確認しましょう。

``` console
$ ls -l /dev/bus/usb/001/065
crw-rw-r--+ 1 nobody nobody 189, 64 Sep  5 14:27 /dev/bus/usb/001/065
```

権限は `crw-rw-r--+` になっているはずです。末尾の `+` に注意してください。そのうえで、次のコマンドを実行してアクセス権を確認します。

``` console
$ getfacl /dev/bus/usb/001/065
getfacl: Removing leadin '/' from absolute path names
# file: dev/bus/usb/001/065
# owner: nobody
# group: nobody
user::rw-
user:<YOUR-USER-NAME>:rw-
group::rw-
mask::rw-
other::r-
```

上の一覧に、自分のユーザー名が `rw-` 権限付きで表示されるはずです。そうでない場合は、
[udev rules] を確認し、次のコマンドで再読み込みを試してください。

[udev rules]: linux.md#udev-rules

``` console
$ sudo udevadm control --reload
$ sudo udevadm trigger
```

それでは、[next section] に進んでください。

[next section]: verify.md