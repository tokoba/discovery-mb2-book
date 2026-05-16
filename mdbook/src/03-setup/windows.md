# Windows

## `arm-none-eabi-gdb`

Arm は Windows 向けの `.exe` インストーラーを提供しています。[こちら][gcc] から入手して、指示に従ってください。
インストール処理が完了する直前に、"Add path to environment variable"
オプションにチェックを入れるか、選択してください。その後、ツールが `%PATH%` に入っていることを確認してください。

``` console
$ arm-none-eabi-gcc -v
(..)
gcc version 5.4.1 20160919 (release) (..)
```

[gcc]: https://developer.arm.com/downloads/-/arm-gnu-toolchain-downloads

## PuTTY

最新の `putty.exe` を [このサイト][this site] からダウンロードし、`%PATH%` 内のどこかに配置してください。

[this site]: http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html

それでは、[次のセクション][next section] に進んでください。

[next section]: verify.md