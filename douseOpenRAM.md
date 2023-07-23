---
layout: default
title: ダーリン！OpenRAMを使うっちゃ！
---
# ダーリン！OpenRAMを使うっちゃ！

この記事はHDL Advent Calendar 2022 一日目の記事です。

[https://qiita.com/advent-calendar/2022/hdl](https://qiita.com/advent-calendar/2022/hdl)


集積回路設計素人の皆さんこんにちは、Cra2yPierr0tです。

OpenMPWでCPUを焼く際に、「おっしゃ！RAM作ったろ！」と雑にVerilogで下記のようなRAMを書いたら地獄(8時間のビルド, 無限に消費されるメモリ, 永遠に消えないエラー)を見ました。

```verilog
module data_mem(addr, w_data, w_en, r_data, clock);
    input [7:0] addr;
    input [7:0] w_data;
    input w_en;
    input clock;
    output [7:0] r_data;

    reg [7:0] mem[0:255];
    
    assign r_data = mem[addr];

    always @(posedge clock) begin
        if(w_en) begin
            mem[addr] <= w_data;
        end else begin
            mem[addr] <= mem[addr];
        end
    end
endmodule
```
[https://github.com/cpu-dev/jacaranda-8/blob/master/src/charlatan/data_mem.v](https://github.com/cpu-dev/jacaranda-8/blob/master/src/charlatan/data_mem.v)

そんな地獄を我々以外が味わってはいけない、**決して**。

---

というわけでOpenMPWのプロジェクトでRAMが欲しい場合は**OpenRAM**を使いましょう。これを使えば地獄を見ずに済みます。

[https://openram.org](https://openram.org)

OpenRAMはカリフォルニア大学サンタクルーズ校のVLSI設計・自動化研究室が作成したSRAMコンパイラで、SRAMとその他情報を生成してくれるツールです。素敵ですね。

ただ現状インストールに若干問題があるのとドキュメントがGoogleスライドしかない(どうして...)ので、OpenMPWのSKY130プロジェクトにおける使い方をここで説明します。

## インストール

[注意] devブランチで大改造が行われており、ここで書かれているインストール方法は将来使えなくなる可能性が高いです。

リポジトリをクローン

```bash
git clone git@github.com:VLSIDA/OpenRAM.git
```

環境変数を設定
```bash
cd OpenRAM
export OPENRAM_HOME=$(pwd)/compiler
export OPENRAM_TECH=$(pwd)/technology
export PYTHONPATH=$OPENRAM_HOME
```

Pythonパッケージをインストール
```bash
pip install -r requirements.txt
```

SKY130 PDKのインストール
```bash
mkdir pdks
mkdir PDK_ROOT=$(pwd)/pdks
make install
```

## 設定
どんなSRAMを生成するかの設定ファイルを作成する。設定ファイルはPythonになっている。
例となる設定ファイルが`macros/configs/`に既に用意されており、これをコピペして編集するのが最も手っ取り早い。

編集すべき変数は以下の通り。

### SRAMのサイズ指定

| 変数名     | 意味                  | 例   |
| ---------- | --------------------- | ---- |
| word_size  | 1ワードのサイズ(bits) | 8    |
| num_words  | ワードの数            | 256 |

### マスク指定

| 変数名     | 意味                                    | 例  |
| ---------- | --------------------------------------- | --- |
| write_size | マスクの1ビットに対応するビット幅(bits) | 4   |

### ポート指定

| 変数名       | 意味         | 例  |
| ------------ | ------------ | --- |
| num_rw_ports | 読み出し兼書き込みポートの数 | 0 |
| num_r_ports  | 読み出しポートの数 | 1 |
| num_w_ports  | 書き込みポートの数 | 1 |

### 例

256byteのメモリの設定を作る例を以下に記す。

適当なSKY130の設定ファイルを別名で複製。
```bash
cd macros/configs
cp sky130_sram_1kbyte_1r1w_8x1024_8.py sky130_sram_256byte.py
```

編集した結果が以下。
```python
word_size = 8 # Bits
num_words = 256
human_byte_size = "{:.0f}kbytes".format((word_size * num_words)/1024/8)

# Allow byte writes
write_size = 4 # Bits

# Dual port
num_rw_ports = 0
num_r_ports = 1
num_w_ports = 1
ports_human = '1w1r'

import os
exec(open(os.path.join(os.path.dirname(__file__), 'sky130_sram_common.py')).read())
```

## ビルド
ビルドは`macros`以下で`make <設定ファイル名>`を実行して行う。かなり時間が掛かる。
`<設定ファイル名>`だが、これは設定ファイル名から拡張子を取り除いた文字列であることに注意。

### 例
```bash
cd macros
make sky130_sram_256byte
```

## ビルド結果

生成物は`macros`以下に格納されている。

生成されたSRAMのGDSIIファイルがこちら。かわいいね。
![](https://github.com/Cra2yPierr0t/Cra2yPierr0t.github.io/blob/master/images/sky130_256.png?raw=true)

生成されたhtmlファイルにはSRAMのダイサイズや動作周波数、消費電力、その他ファイルの説明等が書かれている。

![](https://github.com/Cra2yPierr0t/Cra2yPierr0t.github.io/blob/master/images/sram_info.png?raw=true)

## OpenLANEで使ってみる

これと同じ
https://vlsi.jp/OpenMPWSRAM.html

## おわりに
OpenRAMを使うと人生を無駄にせずに済みます。皆さんOpenRAMを使いましょう。