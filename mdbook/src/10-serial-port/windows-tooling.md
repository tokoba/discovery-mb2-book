# Windows ツール

まず、MB2 を取り外します。

MB2 を再び接続する前に、ターミナルで次のコマンドを実行します。

``` console
$ mode
```

すると、コンピューターに接続されているデバイスの一覧が表示されます。名前が `COM`
で始まるものはシリアルデバイスです。これが今回扱う種類のデバイスです。シリアルモジュールを接続する
*前* の、すべての `COM` ポートの `mode` 出力をメモしておいてください。

次に、MB2 を接続し、再度 `mode` コマンドを実行します。一覧に新しい
`COM` ポートが表示されたら、それが MB2 の
シリアル機能に割り当てられた COM ポートです。

次に `putty` を起動します。GUI が表示されます。

<p align="center">
<img title="PuTTY 設定" src="../assets/putty-settings.png" width="500" />
</p>

起動画面では、"Session" カテゴリーが開いているはずです。そこで "Connection type" として "Serial" を
選択します。"Serial line" フィールドには、前の手順で確認した `COM` デバイスを入力します。
たとえば `COM3` です。

次に、左側のメニューから "Connection/Serial" カテゴリーを選択します。この新しい画面で、
シリアルポートが次のように設定されていることを確認してください。

- "Speed (baud)": 115200
- "Data bits": 8
- "Stop bits": 1
- "Parity": None
- "Flow control": None

最後に、Open ボタンをクリックします。すると、コンソールが表示されます。

<p align="center">
<img title="PuTTY コンソール" src="../assets/putty-console.png" width="500" />
</p>

このコンソールで入力すると、MB2 の上部にある黄色の LED が点滅します。キーを 1 回押すごとに、
LED が 1 回点滅するはずです。なお、コンソールには入力した内容はエコーバックされないため、画面には
何も表示されないままです。