---
layout: default
title: OpenMPWでSRAMを使う
---
# OpenMPWでSRAMを使う

読む時間が無い場合は例となるリポジトリを用意したので適当に参考にして頑張ってほしい。
[https://github.com/Cra2yPierr0t/sky130_sram_test](https://github.com/Cra2yPierr0t/sky130_sram_test)

---

OpenMPWで自作のメモリを使ったら**本当に酷い目にあった**。どうやらOpenRAMで生成したメモリを使うと幸せになれるらしいので色々試行錯誤した結果をここに記す。

OpenRAMは任意の構成のSRAMを生成してくれるOSSらしい。
[https://openram.org](https://openram.org)

素敵なOSSであるが、今回はこれを直接使う訳ではない。というか使い方がわからない。ドキュメントどこにあるねん。実はCaravelを用いたSKY130 PDKのインストールと同時に、OpenRAMでビルドされた1kB, 2kBのSRAMも一緒にインスールされているので今回はそれを使おう。

SRAMのVerilogとGDSIIは以下のディレクトリに用意されている。
```
$PDK_ROOT/sky130B/libs.ref/sky130_sram_macros/
```

このディレクトリには32x256, 8x1024の1kBのメモリと、32x512の2kBメモリが用意されている。

このメモリを使う順序は以下の通り。

1. `user_project_wrapper.v`にメモリを設置
2. `config.tcl`を編集
3. `macro.cfg`で位置を指定
4. ビルド

以降は32x256の1kBのメモリを例に使い方を説明する。基本的に他のメモリも同じ方法で使うことが出来るが、8x1024の1kBのメモリは留意事項があるため後で補足する。

## 1. `user_project_wrapper.v`にメモリを設置

まずは以下のように`user_project_wrapper.v`にメモリを設置する。他のモジュールと同じようにインスタンス名は自由に決めて良いし、複数個インスタンス化出来る。また`ifdef~endif`で囲まれている部分は書いておいた方がよい。

```verilog
    // write    : when web0 = 0, csb0 = 0
    // read     : when web0 = 1, csb0 = 0, maybe 3 clock delay...?
    // read     : when csb1 = 0, maybe 3 clock delay...?
    sky130_sram_1kbyte_1rw1r_32x256_8 rx_mem(
    `ifdef USE_POWER_PINS
        .vccd1  (vccd1          ),
        .vssd1  (vssd1          ),
    `endif
        // RW
        .clk0   (clk0),   // clock
        .csb0   (csb0),   // active low chip select
        .web0   (web0),   // active low write control
        .wmask0 (wmask0), // write mask (4 bit)
        .addr0  (addr0),  // addr (8 bit)
        .din0   (din0),   // data in (32 bit)
        .dout0  (dout0),  // data out (32 bit)
        // R
        .clk1   (clk1),   // clock
        .csb1   (csb1),   // active low chip select
        .addr1  (addr1),  // addr (8 bit)
        .dout1  (dout1)   // data out (32 bit)
    );
```

SRAMの各IOの役割については以下の通り。後ろの数字は省略している。

| 名前    | I/O | 幅 | 役割                                             |
| ------- | --- | --- | ------------------------------------------------ |
| `clk`   | I   | 1bit | クロック入力、立ち下がりで書き込み及び読み出し。       |
| `csb`   | I   | 1bit | チップセレクト信号バー。0で選択、1で非選択       |
| `web`   | I   | 1bit | 書き込みイネーブルバー。0で書き込み、1で読み出し |
| `wmask` | I   | 4bit(SRAMによる) | バイトイネーブル。8ビット単位     |
| `addr`  | I   | 8bit(SRAMによる) | アドレス |
| `din`   | I   | 32bit(SRAMによる) | データ入力 |
| `dout`  | O   | 32bit(SRAMによる) | データ出力 |

データ書き込み、読み出しの方法は以下の波形の通りであり、読み出しには遅延が存在する。また`clk`の周期は20ns以上にする必要がある。[^1]

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/MPWRAM/wave.png?raw=true)

~~データ読み出しの遅延に関しては、SRAMのコード内に†FIXME: This delay is arbitrary†とか書いてあるので少し待ったほうが良い。~~

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/MPWRAM/arbi.png?raw=true)

筆者は半導体設計に詳しくないので推測だが、今回使うSRAMのようなメガセルは`user_project_wrapper.v`でしかインスタンス化出来ない。

## 2. `config.tcl`を編集

自分のモジュールとSRAMを接続できたら、次は`user_project_wrapper`の`config.tcl`を編集する。
編集された`config.tcl`の例はここに置いてある。
[https://github.com/Cra2yPierr0t/Vthernet-SoC/blob/main/openlane/user_project_wrapper/config.tcl](https://github.com/Cra2yPierr0t/Vthernet-SoC/blob/main/openlane/user_project_wrapper/config.tcl)

編集する変数は`FP_PDN_MACRO_HOOKS`、`VERILOG_FILES_BLACKBOX`、`EXTRA_LEFS`、`EXTRA_GDS_FILES`の4つ。

### `FP_PDN_MACRO_HOOKS`

`FP_PDN_MACRO_HOOKS`には`インスタンス名 <vdd_net> <gnd_net> <vdd_pin> <gnd_pin>`を設定する。基本的には`インスタンス名 vccd1 vssd1 vccd1 vssd1`と書けば良い。
```verilog
set ::env(FP_PDN_MACRO_HOOKS) "\
	rx_mem vccd1 vssd1 vccd1 vssd1"
```

複数インスタンス化している場合以下のようにコンマ, スペースで区切る。コンマの後ろにスペースを置かないとエラーが発生する。
```verilog
set ::env(FP_PDN_MACRO_HOOKS) "\
	rx_mem1 vccd1 vssd1 vccd1 vssd1, \
	rx_mem2 vccd1 vssd1 vccd1 vssd1"
```

### `VERILOG_FILES_BLACKBOX`

`VERILOG_FILES_BLACKBOX`にはSRAMのVerilogファイルのパスを設定する。これは合成には使われず、シミュレーションを行う場合にそのファイルが参照される。

以下に32x256の1kBのメモリを使う場合の例を載せる。
```bash
$::env(PDK_ROOT)/sky130B/libs.ref/sky130_sram_macros/verilog/sky130_sram_1kbyte_1rw1r_32x256_8.v \
```

### `EXTRA_LEFS`

`EXTRA_LEFS`には用いるSRAMのLEFファイルのパスを設定する。

以下に32x256の1kBのメモリを使う場合の例を載せる。
```bash
$::env(PDK_ROOT)/sky130B/libs.ref/sky130_sram_macros/lef/sky130_sram_1kbyte_1rw1r_8x1024_8.lef \
```

### `EXTRA_GDS_FILES`

`EXTRA_GDS_FILES`には用いるSRAMのGDSIIファイルのパスを設定する。

以下に32x256の1kBのメモリを使う場合の例を載せる。
```bash
$::env(PDK_ROOT)/sky130B/libs.ref/sky130_sram_macros/gds/sky130_sram_1kbyte_1rw1r_8x1024_8.gds \
```

### DRCエラーの回避

指摘により追加。
`config.tcl`に以下の三行を追加してmagicによるDRCエラーを回避する。

```bash
set ::env(MAGIC_DRC_USE_GDS) 0
set ::env(RUN_MAGIC_DRC) 0
set ::env(QUIT_ON_MAGIC_DRC) 0
```

OpenRAMのSRAMを使う場合magicがどうしてもDRCエラーを発生させるらしい、magicによるDRCは行わなくなるがプリチェックには(殆ど場合)通るので安心してほしい。


これで最低限の`config.tcl`の編集が完了した。

## 3. `macro.cfg`で位置を指定

次に`macro.ctg`でメモリを設置するチップ上の位置を指定する。フォーマットは`インスタンス名 X座標 Y座標 Orientation`となっている。
座標の単位はμmであり、メガセルの左下の位置を指定する。Orientationがどの部分のOrientationを示しているのかは知らないが基本的にNにしておけば問題ない。

`macro.cfg`のパスを`MACRO_PLACEMENT_CFG`に設定する。これはおそらく既に存在している。

以下に複数個インスタンスがある場合の例を示す。
```bash
rx_mem1 100 100 N
rx_mem2 700 100 N
```

以下が生成されたレイアウトであり、指定した通り(100, 100)にメモリの左下が来ている。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/MPWRAM/layout.png?raw=true)

マニュアルには`MACRO_PLACEMENT_CFG`は設定しなくても良さげな事が書いてあるが、筆者が試した限りでは上手く行かなかった。

## 4. ビルド

通常のCaravelにおけるuser_project_wrapperのビルドを実行すれば良い。
```bash
make user_project_wrapper
```

問題なければレイアウト上にメモリが生成されている筈である。

## `sky130_sram_1kbyte_1rw1r_8x1024_8`を使う場合の注意点

執筆段階(mpw-7a)で`NUM_MASKS`の値を2にしないとエラーが発生する。OpenRAMの中の人が直したと言っていたのでもう必要ないかもしれないが、`sky130_sram_1kbyte_1rw1r_8x1024_8`を使ってエラーが発生したら試してみてほしい。ただ安全を取るなら32x256を使うべきだと思われる。

一応以下に例を載せる。`NUM_MASKS`と`wmask0`に変更を加えている。
```verilog
sky130_sram_1kbyte_1rw1r_8x1024_8 #(.NUM_WMASKS(2)) rx_mem(
    ...
    .web0   (rx_data_vb     ),  // active low write control
    .wmask0 ({wmask0, wmask0}), // write mask (1 bit)
    .addr0  (rx_addr        ),  // addr (10 bit)
    ...
```

## 大きなメモリが欲しい場合
僕も欲しい。大人しくOpenRAMを使って生成するか、1kBのSRAMをたくさん置いてアドレスをいい感じにするモジュールを作ればいいと思います。

OpenMPWでメモリをどう作るかは死活問題な気がするのでOpenRAMの使い方を知りたいですね。

--- 

[^1]: ありがとうMatthew Guthausサン
