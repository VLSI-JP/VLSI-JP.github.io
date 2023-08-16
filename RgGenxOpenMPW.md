---
layout: default
title: RgGen ✕ OpenMPWでLSIを焼こう！
image: "https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/RISC5000choyen.png"
---
# RgGen ✕ OpenMPWでLSIを焼こう！
質問、修正案、その他連絡は@Cra2yPierr0tマデ

## Abstract
この記事の動機はOpenMPW参加の敷居を下げる事で、手法はRgGenを利用したWishboneインタフェース付きCSRの生成です。出来ました、確かに楽です、これは流行る、RgGenにお布施しよう、以上です。

## Introduction

こんにちは、Cra2yPierr0tです。

OpenMPWの参加者、中々増えませんね。寂しい。
まあ理由は分からなくもないです。難しいんですよね、Verilog以外の諸々が。Verilogを書くのは簡単ですが(書けないお前はザコ)、wishboneとか謎バスプロトコルが出てきますし、OpenLANEのコンフィグは集積回路素人には難しすぎますし、SRAMを取り替える必要がありますし。

というわけでなんとかOpenMPWの敷居を下げていきたい。SRAMに関してはこの前書いたので、次はwishboneをどうにかしよう。そう、**RgGen**で。



百聞は一見に如かず、本記事ではRgGenを使ってお手軽にLSI設計をする流れをご紹介いたします。

## 前提知識
### RgGenについて
**RgGen**はいしたに([@taichi600730](https://twitter.com/taichi600730))さんが製作しているCSR生成ツールで、**設定ファイル** から多彩な機能を持った**CSR** を生成することが出来ます。そして更にCSRに**wishboneインターフェイス**を持たせる事が可能です(その他にAPB, AXI4-liteも可能)。

これが何を意味しているか、つまりRgGenを使えば、wishboneの仕様書を読んでwishboneの制御ロジックを作ったり、あるレジスタはROで、あるレジスタはRWにするといった面倒な作業から解放されます！ 嬉しいですね。

[https://github.com/rggen/rggen](https://github.com/rggen/rggen)

### OpenMPWについて
GoogleとEfablessとSkywaterが結託して始めたシャトルプログラム、製造料・送料全て無料！
アメリカの金でLSIを焼けるぜ！
[https://efabless.com/open_shuttle_program](https://efabless.com/open_shuttle_program)

### Caravelについて
OpenMPWではCaravelというフレームワークを使います。
[https://github.com/efabless/caravel_user_project](https://github.com/efabless/caravel_user_project)

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/caravel_eng.png?raw=true)

CaravelはMGMT CoreとUser Project Areaに分かれています。MGMT CoreはRISC-Vのコアに周辺機器が領域で、ここは固定です。**User Project Areaはユーザーが自由に扱える領域**であり、ここに自分のデザインを挿入します。

MGMT CoreとUser Project AreaはWishboneとLogic Analyzerと呼ばれる二種類のバスで繋がっており、これを介してMGMT CoreのRISC-Vコアから自分のデザインの制御やデータの受送信が可能です。

またUser Project Areaから外部へ38本のGPIOが伸びており、これを使って外部とのデータのやり取りが可能となっています。

#### CaravelのMMIO
Caravelのメモリマップは以下のドキュメントの通りになっており、ユーザーは`0x3000_0000`から`0x8000_0000-1`の範囲を自由に扱うことが可能です。
[https://caravel-harness.readthedocs.io/en/latest/memory-mapped-io-summary.html](https://caravel-harness.readthedocs.io/en/latest/memory-mapped-io-summary.html)

以降では**CSRのベースアドレスは`0x3000_0000`とします**。

#### Caravelのインストール

念のため書いておきます。ここからcaravel_user_projectを落としてくる。
[https://github.com/efabless/caravel_user_project](https://github.com/efabless/caravel_user_project)

templateから作るなりforkするなりcloneするなり好きにしてください、templateからリポジトリを作るとアップデートが面倒かもしれないです。今回はtemplateから生成してcloneします。
```bash
clone git@github.com:<Github ID>/caravel_walkthrough_uart.git
```

リポジトリに入ってdependenciesディレクトリを作る。
```bash
cd caravel_user_project
mkdir dependencies
```

環境変数でOpenLANEのインストール場所とインストールするPDKを選択する。`sky130A`でSkywaterの130nm、`gf180mcuC`でGlobalFoundriesの180nmのPDKがインストールされます。提出したいシャトルに応じて決めてください、今回はSKY130を想定します。
```bash
export OPENLANE_ROOT=$(pwd)/dependencies/openlane_src
export PDK_ROOT=$(pwd)/dependencies/pdks
export PDK=sky130A or export PDK=gf180mcuC
```

インストールを開始
```bash
make setup
```

### OpenLANEとは
OSSのRTL to GDSIIコンパイラ。このGDSIIってやつをファブに提出するとLSIを作ってくれる。OpenLANE自体は約20個のOSSが組み合わさって出来ており、物凄い速度で開発が進められている。
[https://github.com/The-OpenROAD-Project/OpenLane](https://github.com/The-OpenROAD-Project/OpenLane)

### SKY130 PDKとは
SkyWater Technologyの130nmプロセスPDK。PDK(Process Design Kit)はAND回路やOR回路のようなプリミティブな素子の物性情報等をまとめたものです。Googleの名の下にオープンソースになった。
[https://github.com/google/skywater-pdk](https://github.com/google/skywater-pdk)

### GF180 PDKとは
Global Foundriesの180nmプロセスPDK。この前第一回目のシャトル、GFMPW-0があった。
[https://github.com/google/gf180mcu-pdk](https://github.com/google/gf180mcu-pdk)

## 今回作るLSI
今回はCaravelにUARTを追加してみましょう。実はMGMT Coreの方にUARTは既に実装されていますが、それは見なかったことにしてUser Project Areaに自作のUARTハードウェアを追加します。

完成像はUARTで文字を受け取れるRISC-Vマイコンです。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/RISC5000choyen.png?raw=true)

なお、本記事で作成したファイルは全て`caravel_walkthrough_uart`に上げてあります。適宜ご参照下さい。
[https://github.com/Cra2yPierr0t/caravel_walkthrough_uart](https://github.com/Cra2yPierr0t/caravel_walkthrough_uart)

## 仕様を考える

それではLSIにどんな動作をして欲しいか方針を立てましょう。

- `LSI -> PC` : レジスタに1文字入れてビットを立てたら送信、完了したらビットが落ちる
- `LSI <- PC` : 1文字来たら割り込み、文字を読んだら割り込みビットが落ちる

こんなもんか。

## 必要そうなCSRを考える

次に必要そうなCSRは何か考えましょう。受信データと送信データを格納するレジスタは当然必要ですね。その他に送信を開始するためのレジスタと、受信時の割り込みON/OFF、またUARTにはボーレートがあるのでボーレート又は動作周波数を指定するレジスタが必要そうですね。現状何MHzで動くLSIが出来るか不明なため、今回は動作周波数を指定するレジスタにします。ボーレートは115200で固定って事にしましょう。

以上で決めたCSRを表にすると以下の通りになります。

| address       | CSR                  | field                         | access | Description                                 |
| ------------- | -------------------- | ----------------------------- | ------ | ------------------------------------------- |
| `0x3000_0000` | `CLOCK_FREQ`         | `clock_freq[31:0]`            | RW     | LSIの動作周波数                             |
| `0x3000_0004` | `RECEIVED_DATA`      | `reserved[31:8], rx[7:0]`     | RO     | 受信データ                                  |
| `0x3000_0008` | `TRANSMISSION_DATA`  | `reserved[31:8], tx[7:0]`     | RW     | 送信データ                                  |
| `0x3000_000c` | `INTERRUPT_ENABLE`   | `reserved[31:1], irq_en[0]`   | RW     | 受信割り込み有効化                          |
| `0x3000_0010` | `TRANSMISSION_START` | `reserved[31:1], tx_start[0]` | RW     | 1を書き込みで送信開始, 完了時に自動で落ちる |

ここで注意して欲しいのがUARTのような単純なハードウェアでも、自動でビットが落ちる`TRANSMISSION_START`や、値を読んだら割り込みビットが落ちる`RECEIVED_DATA`のように、**特別な機能を持ったCSRも必要となる**、という事です。

## RgGenでCSRを生成

では上記の表を参考にRggenでCSRを作っていきましょう。

### インストール
RgGen本体のインストール
```bash
gem install rggen
```

次にRgGenのVerilogプラグインのインストール
```bash
gem install rggen-verilog
```

### RgGenの使い方
RgGenの詳しい使い方を知りたい場合は本家のリポジトリを見に行ってください。
[https://github.com/rggen/rggen](https://github.com/rggen/rggen)

RgGenには2種類の入力ファイルが必要です。1つ目がコンフィグレーションファイルで、バス幅やアドレス幅、プロトコル等をyamlを使って指定します。2つ目がレジスタマップで、アドレスやビットフィールド、アクセス制御等をyamlを使って指定します。順番に見ていきましょう。

### コンフィグレーションファイル

まずはコンフィグレーションファイルです。以下に今回使う変数とその働きの表を載せます。その他に使える変数に関しては本家のリポジトリを参照してください。
[https://github.com/rggen/rggen](https://github.com/rggen/rggen)

| 変数名          | 機能                         | 例    |
| --------------- | ---------------------------- | --- |
| `bus_width`     | データバスのビット幅         |  32   |
| `address_width` | アドレスのビット幅           |   8  |
| `protocol`      | インターフェイスのプロトコル | apb, axi4lite, wishbone    |

`bus_width`はCaravelでは32bitなので32bitに([user_project_wrapper.v](https://github.com/efabless/caravel_user_project/blob/main/verilog/rtl/user_project_wrapper.v)を参照) 、`address_width`は今回のCSRのサイズは20byteなので5bitにします。
そしてCaravelはMGMT CoreとUser Project Areaはwishboneで繋がっているため、`protocol`は`wishbone`を指定します。

以下を`config.yml`としましょう。

```yaml
bus_width: 32
address_width: 5
protocol: wishbone
```
[https://github.com/Cra2yPierr0t/caravel_walkthrough_uart/blob/main/rggen_configs/config.yml](https://github.com/Cra2yPierr0t/caravel_walkthrough_uart/blob/main/rggen_configs/config.yml)

### レジスタマップ
次にレジスタの構成を指定します。レジスタマップの設定ファイルを`register_map.yml`としましょう。`register_map.yml`は少し長いため、上から順に説明し、最後にファイルの全体をお見せします。

#### レジスタブロックの設定

まずはレジスタブロックの設定です。`register_block`から初め、`name`で名前を、`byte_size`でレジスタブロックのサイズを指定します。



| 変数 | 機能 |
| ---- | ---- |
| `name`     |  レジスタブロックの名前    |
| `byte_size` | レジスタブロックのサイズ |

YAMLでの記述は以下の通り。

```yaml
register_blocks:
  - name: CSR
    byte_size: 20
    registers:
```

`registers`から各レジスタについて記述していきます。

#### CLOCK_FREQ

まずはLSIの動作周波数を決める`CLOCK_FREQ`について記述しましょう。これは32bit幅でアクセスはRWでしたね。

レジスタの名前は`name`で指定し、ビットフィールドは`bit_fields`で指定します。

| 変数 | 機能 |
| --- | ---- |
| `name` | レジスタの名前 |
| `bit_fields` | レジスタのビットフィールド |

`bit_fields`で更に詳しく指定します。

| 変数             | 機能                          |  例   |
| ---------------- | ----------------------------- | --- |
| `name`           | フィールドの名前              |     |
| `bit_assignment` | フィールドのビット幅、LSBなど | `width: 32` |
| `type`           | レジスタの型, 機能            |  `ro, rw`, 後述   |
| `initial_value`  | 初期値                        |     |

最初に検討した通り、`CLOCK_FREQ`の持つフィールドはRWで32bitの`clock_freq`だけですから、`name`は`clock_freq`、`bit_assignment`は`width: 32`、`type`は`rw`にします。

yamlで`CLOCK_FREQ`について記述したのが以下の通りです。

```yaml
- name: CLOCK_FREQ
  bit_fields: 
  - { name: clock_freq, bit_assignment: { width: 32 }, type: rw, initial_value: 1000000}
```

`bit_assignment`には`width: 32`を指定したため、`[31:0]`の幅のフィールドとなります。

`type`に関しては非常に重要で便利なパラメーターですのでよく見ておいてください。`initial_value`は1MHzを示す`1000000`にしておきます。

#### RECEIVED_DATA

次に受信データが格納されるレジスタである`RECEIVED_DATA`について記述していきます。これは8bitの`rx`と、残りの使わない24bitの`reserved`の2つのフィールドを持っていますので、`bit_fields`は2行になります。

yamlで記述したのが以下の通りです。とりあえず見てください。

```yaml
- name: RECEIVED_DATA
  bit_fields: 
  - { name: rx, bit_assignment: { width: 8 }, type: rotrg, initial_value: 0x0}
  - { name: reserved, bit_assignment: { width: 24 }, type: reserved }
```

`rx`のビットフィールドを見てみましょう。`rx`は8bitですので`bit_assignment`は`width: 8`ですね、注目して欲しいのが`type`です。`rotrg`になっていますね、`rotrg`を`type`に指定することで**Read Onlyかつ読み出し時にトリガー信号を発するCSR**になります。嬉しいですね。このトリガー信号を検知して割り込み信号を落とす仕組みを作れそうです。

また`reserved`のビットフィールドを見てみましょう。`width: 24`になっています。これで`rx`フィールドの続きの`[31:8]`の24bitが`reserved`という名前のフィールドになります。

また`type`に`reserved`を指定しています。これで用途を決めていないフィールドを予約という事にしてお茶を濁す事が可能となります。嬉しいですね。追記：`reserved`のフィールドは無くてもいいです。

他に指定可能な`type`に関しては本家のリポジトリを参照してください。
[https://github.com/rggen/rggen](https://github.com/rggen/rggen)

#### TRANSMISSION_DATA

次に送信データを格納するレジスタである`TRANSMISSION_DATA`について記述します。特に目新しい事はしていませんね、8bitの`rw`なフィールドと24bitの`reserved`なフィールドです。

```yaml
- name: TRANSMISSION_DATA
  bit_fields: 
  - { name: tx, bit_assignment: { width: 8 }, type: rw, initial_value: 0x0}
  - { name: reserved, bit_assignment: { width: 24 }, type: reserved }
```

#### INTERRUPT_ENABLE

次に割り込みのON/OFFを決めるレジスタである`INTERRUPT_ENABLE`について記述します。これも同じく新しい事はしていませんね、1bitの`rw`なフィールドと30bitの`reserved`なフィールドです。割り込みはデフォルトでオンにしておきたいので`irq_en`の初期値を`1`にします。

```yaml
- name: INTERRUPT_ENABLE
  bit_fields:
  - { name: irq_en, bit_assignment: { width: 1 }, type: rw, initial_value: 0x1}
  - { name: reserved, bit_assignment: { width: 30 }, type: reserved }
```

#### TRANSMISSION_START

最後に送信開始に使うレジスタである`TRANSMISSION_START`について記述します。フィールドの幅は`INTERRUPT_ENABLE`に似ていますが、`tx_start`の型が`rwc`になっていますね。
`type`に`rwc`を指定するとクリア入力が生えます。送信が完了したらこのクリア入力をアサートして`tx_start`を0にする事が可能になります。嬉しいですね。

```yaml
- name: TRANSMISSION_START
  bit_fields:
  - { name: tx_start, bit_assignment: { width: 1 }, type: rwc, initial_value: 0x0}
  - { name: reserved, bit_assignment: { width: 30 }, type: reserved }
```

以上で全レジスタの設定が完了しました。ここで使った機能はRgGenのごく一部ですので、一度本家リポジトリを確認することをオススメします。

[https://github.com/rggen/rggen](https://github.com/rggen/rggen)

#### 全体像

以下がレジスタマップの全体像です。これを`register_map.yml`としましょう。

```yaml
register_blocks:
  - name: CSR
    byte_size: 20
    registers:
      - name: CLOCK_FREQ
        bit_fields: 
        - { name: clock_freq, bit_assignment: { width: 32 }, type: rw, initial_value: 1000000}
      - name: RECEIVED_DATA
        bit_fields: 
        - { name: rx, bit_assignment: { width: 8 }, type: rotrg, initial_value: 0x0}
        - { name: reserved, bit_assignment: { width: 24 }, type: reserved }
      - name: TRANSMISSION_DATA
        bit_fields: 
        - { name: tx, bit_assignment: { width: 8 }, type: rw, initial_value: 0x0}
        - { name: reserved, bit_assignment: { width: 24 }, type: reserved }
      - name: INTERRUPT_ENABLE
        bit_fields: 
        - { name: irq_en, bit_assignment: { width: 1 }, type: rw, initial_value: 0x1}
        - { name: reserved, bit_assignment: { width: 30 }, type: reserved }
      - name: TRANSMISSION_START
        bit_fields: 
        - { name: tx_start, bit_assignment: { width: 1 }, type: rwc, initial_value: 0x0}
        - { name: reserved, bit_assignment: { width: 30 }, type: reserved }
```
[https://github.com/Cra2yPierr0t/caravel_walkthrough_uart/blob/main/rggen_configs/register_map.yml](https://github.com/Cra2yPierr0t/caravel_walkthrough_uart/blob/main/rggen_configs/register_map.yml)

### 生成
以下のコマンドでCSRを生成します。

```bash
rggen --plugin rggen-verilog -c config.yml register_map.yml
```

実行が完了したら、このコマンドを実行したディレクトリにCSRのファイルが生成されています。

### 生成されたCSR
生成された`CSR.v`の一部を以下に載せます。

```verilog
`include "rggen_rtl_macros.vh"
module CSR #(
  parameter ADDRESS_WIDTH = 5,
  parameter PRE_DECODE = 0,
  parameter [ADDRESS_WIDTH-1:0] BASE_ADDRESS = 0,
  parameter ERROR_STATUS = 0,
  parameter [31:0] DEFAULT_READ_DATA = 0,
  parameter USE_STALL = 1
)(
  input i_clk,
  input i_rst_n,
  input i_wb_cyc,
  input i_wb_stb,
  output o_wb_stall,
  input [ADDRESS_WIDTH-1:0] i_wb_adr,
  input i_wb_we,
  input [31:0] i_wb_dat,
  input [3:0] i_wb_sel,
  output o_wb_ack,
  output o_wb_err,
  output o_wb_rty,
  output [31:0] o_wb_dat,
  output [31:0] o_CLOCK_FREQ_clock_freq,
  input [7:0] i_RECEIVED_DATA_rx,
  output o_RECEIVED_DATA_rx_read_trigger,
  output [7:0] o_TRANSMISSION_DATA_tx,
  output o_INTERRUPT_ENABLE_irq_en,
  input i_TRANSMISSION_START_tx_start_clear,
  output o_TRANSMISSION_START_tx_start
);
  wire w_register_valid;
  wire [1:0] w_register_access;
  wire [4:0] w_register_address;
```
[https://github.com/Cra2yPierr0t/caravel_walkthrough_uart/blob/main/verilog/rtl/CSR.v](https://github.com/Cra2yPierr0t/caravel_walkthrough_uart/blob/main/verilog/rtl/CSR.v)

`i_clk`から`o_wb_dat`まではwishboneのインターフェイスです。その下に`o_CLOCK_FREQ_clock_freq`という32bitの出力信号がありますが、これは`CLOCK_FREQ`ですね。他にも`i_RECEIVED_DATA_rx`という8bitの入力信号があり、これは受信データですし、`i_TRANSMISSION_START_tx_start_clear`は`tx_start`のクリア信号ですね。正しく生成されていますね。

この`CSR.v`を`user_project_wrapper/verilog/rtl/`に突っ込みましょう。

ちなみに同時に生成される`CSR.md`を`$ glow CSR.md`で見ると嬉しい気持ちになります。

### RgGenの共通モジュール
`CSR.v`だけでは使えませんので、rggen-verilog-rtlを`user_project_wrapper/verilog/rtl/`にsubmoduleとして置きましょう。
[https://github.com/rggen/rggen-verilog-rtl](https://github.com/rggen/rggen-verilog-rtl)

```bash
cd user_project_wrapper/verilog/rtl/
git submodule add git@github.com:rggen/rggen-verilog-rtl.git
```

そしてインクルードの関係上、`rggen-verilog-rtl/rggen_rtl_macro.vh`のシンボリックリンクを`verilog/rtl/`以下に設置しましょう。

```bash
cd verilog/rtl
ln -s rggen-verilog-rtl/rggen_rtl_macro.vh rggen_rtl_macro.vh
```


## UARTモジュールを作成

次にUARTモジュールを作成します。ここでは送信を行う`uart_transmission.v`と受信を行う`uart_receive.v`と、これらに加えCSRを統合した`uart.v`を作成します。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/uart.drawio.png?raw=true)


### 送信

ラッッッッッッッッッッ！！！！(UARTくらいは気合で作れた方がよい)

```verilog
module uart_transmission(
  input wire        rst,
  input wire        clk,
  input wire [31:0] clk_div,
  input wire        tx_start,
  input wire [7:0]  tx_data,
  output reg        tx = 1'b1,
  output reg        clear_req = 1'b0
);

  parameter WAIT        = 4'b0000;
  parameter START_BIT   = 4'b0001;
  parameter SEND_DATA   = 4'b0010;
  parameter STOP_BIT    = 4'b0011;
  parameter CLEAR_REQ   = 4'b0100;

  reg [3:0] state;

  reg [31:0] clk_cnt = 32'h0000_0000;

  reg [2:0] tx_index = 3'b000;

  reg [1:0] detect_posedge_start = 2'b00;

  always @(posedge clk) begin
    if(rst) begin
      tx        <= 1'b1;
      state     <= WAIT;
      clear_req <= 1'b0;
      tx_index  <= 3'b000;
      clk_cnt   <= 32'h0000_0000;
      detect_posedge_start <= 2'b00;
    end else begin
      // for safe
      detect_posedge_start <= {detect_posedge_start[0], tx_start}; 
      case(state)
        WAIT        : begin
          tx <= 1'b1;
          clear_req <= 1'b0;
          if(detect_posedge_start == 2'b01) begin
            state <= START_BIT;
          end
        end
        START_BIT   : begin
          tx <= 1'b0;
          if(clk_cnt == (clk_div - 1)) begin
            clk_cnt <= 32'h0000_0000;
            state <= SEND_DATA;
          end else begin
            clk_cnt <= clk_cnt + 32'h0000_0001;
          end
        end
        SEND_DATA   : begin
          tx <= tx_data[tx_index];
          if(clk_cnt == (clk_div - 1)) begin
            clk_cnt <= 32'h0000_0000;
            if(tx_index == 3'b111) begin
              state <= STOP_BIT;
            end
            tx_index <= tx_index + 3'b001;
          end else begin
            clk_cnt <= clk_cnt + 32'h0000_0001;
          end
        end
        STOP_BIT    : begin
          tx <= 1'b1;
          if(clk_cnt == (clk_div - 1)) begin
            clk_cnt <= 32'h0000_0000;
            state <= CLEAR_REQ;
          end else begin
            clk_cnt <= clk_cnt + 32'h0000_0001;
          end
        end
        CLEAR_REQ   : begin
          clear_req <= 1'b1;
          state <= WAIT;
        end
        default     : begin
          tx        <= 1'b1;
          state     <= WAIT;
          clear_req <= 1'b0;
          tx_index  <= 3'b000;
          clk_cnt   <= 32'h0000_0000;
          detect_posedge_start <= 2'b00;
        end
      endcase
    end
  end

endmodule
```
[https://github.com/Cra2yPierr0t/caravel_walkthrough_uart/blob/main/verilog/rtl/UART/uart_transmission.v](https://github.com/Cra2yPierr0t/caravel_walkthrough_uart/blob/main/verilog/rtl/UART/uart_transmission.v)

### 受信

ラッッッッッッッッッッ！！！！

```verilog
module uart_receive (
  input wire        rst,
  input wire        clk,
  input wire [31:0] clk_div,
  input wire        rx,
  input wire        read,
  input wire        irq_en,
  output reg        irq     = 1'b0,
  output reg [7:0]  rx_data = 8'h0
);

  parameter WAIT        = 4'b0000;
  parameter START_BIT   = 4'b0001;
  parameter GET_DATA    = 4'b0010;
  parameter STOP_BIT    = 4'b0011;
  parameter WAIT_READ   = 4'b0100;

  reg [3:0] state;

  reg [31:0] clk_cnt = 32'h0000_0000;

  reg [2:0] rx_index = 3'b000;

  always @(posedge clk) begin
    if(rst) begin
      state     <= WAIT;
      clk_cnt   <= 32'h0000_0000;
      rx_index  <= 3'b000;
      irq       <= 1'b0;
      rx_data   <= 8'h0;
    end else begin
      case(state)
        WAIT      : begin
          irq <= 1'b0;
          if(rx == 1'b0) begin
            state <= START_BIT;
          end
        end
        START_BIT : begin
          // check the middle of wave
          if(clk_cnt == ((clk_div >> 1) - 1)) begin
            clk_cnt <= 32'h0000_0000;
            if(rx == 1'b0) begin
              state <= GET_DATA;
            end
          end else begin
            clk_cnt <= clk_cnt + 32'h0000_0001;
          end
        end
        GET_DATA  : begin
          // get the middle of wave
          if(clk_cnt == (clk_div - 1)) begin
            clk_cnt <= 32'h0000_0000;
            if(rx_index == 3'b111) begin
              state <= STOP_BIT;
            end
            rx_index <= rx_index + 3'b001;
            rx_data[rx_index] <= rx;
          end else begin
            clk_cnt <= clk_cnt + 32'h0000_0001;
          end
        end
        STOP_BIT  : begin
          // check the middle of wave
          if(clk_cnt == (clk_div - 1)) begin
            clk_cnt <= 32'h0000_0000;
            if(rx == 1'b1) begin
              state <= WAIT_READ;
              if(irq_en) begin
                irq <= 1'b1;
              end
            end
          end else begin
            clk_cnt <= clk_cnt + 32'h0000_0001;
          end
        end
        WAIT_READ : begin
          if(read) begin
            irq <= 1'b0;
            state <= WAIT;
          end
        end
        default   : begin
          state     <= WAIT;
          clk_cnt   <= 32'h0000_0000;
          rx_index  <= 3'b000;
          irq       <= 1'b0;
          rx_data   <= 8'h0;
        end
      endcase
    end
  end

endmodule
```
[https://github.com/Cra2yPierr0t/caravel_walkthrough_uart/blob/main/verilog/rtl/UART/uart_receive.v](https://github.com/Cra2yPierr0t/caravel_walkthrough_uart/blob/main/verilog/rtl/UART/uart_receive.v)

### トップモジュール

これは気合ではどうにもなりませんね、注意深くやりましょう。
一番重要なのはwishboneとCSRの接続です。[user_project_wrapper.v](https://github.com/efabless/caravel_user_project/blob/main/verilog/rtl/user_project_wrapper.v#L47-L56)を見てください。`wb_clk_i`から`wbs_dat_o`までがCaravelで使うwishboneの信号線です。RgGenのwishboneとCaravelのwishboneの信号線の名前が若干異なっているので注意が必要ですが、本当に重要な事項は以下の通りです。

- `wb_rst_i`を反転して`i_rst_n`に入力(負論理のため)
- `o_wb_stall`は不使用
- `i_wb_adr`のビット幅
- `wbs_adr_i`の`0x3000_00`部分の判別
- `o_wb_err`は不使用
- `o_wb_rty`は不使用

`wbs_adr_i`において、上位ビットが`0x3000_00`でない場合、CSRのcycとstbに0を入力するロジックを追加しています。

wishboneの接続が完了したら残りはCSRのデータの接続ですが、これは繋げるだけです。

また外部入出力の`io_out`と`io_in`をそれぞれ`tx`と`rx`に接続します。ピン配置はここに載ってます。
[https://caravel-harness.readthedocs.io/en/latest/pinout.html](https://caravel-harness.readthedocs.io/en/latest/pinout.html)

31番は出力で30番は入力なので、`io_oeb`にそれぞれ値を入力します。負論理になっていますので出力に用いる場合は0を入力します。

`vccd1`と`vssd1`は電源ラインですのでとりあえず書いときます。

```verilog
module uart #(
  parameter BAUD_RATE = 115200
)(
`ifdef USE_POWER_PINS
    inout vccd1,	// User area 1 1.8V supply
    inout vssd1,	// User area 1 digital ground
`endif
  // Wishbone Slave ports (WB MI A)
  input wire    wb_clk_i,
  input wire    wb_rst_i,
  input wire    wbs_stb_i,
  input wire    wbs_cyc_i,
  input wire    wbs_we_i,
  input wire    [3:0] wbs_sel_i,
  input wire    [31:0] wbs_dat_i,
  input wire    [31:0] wbs_adr_i,
  output wire   wbs_ack_o,
  output wire   [31:0] wbs_dat_o,

  // IO ports
  input  [`MPRJ_IO_PADS-1:0] io_in,
  output [`MPRJ_IO_PADS-1:0] io_out,
  output [`MPRJ_IO_PADS-1:0] io_oeb,

  // irq
  output [2:0] user_irq
);

  // UART 
  wire  tx;
  wire  rx;

  assign io_oeb[31] =  1'b0;
  assign io_oeb[30] =  1'b1;
  assign io_out[31] = tx;
  assign rx = io_in[30];

  // irq
  wire irq;
  assign user_irq[0] = irq;

  // CSR
  wire [31:0] clk_freq;

  wire [7:0] rx_data;
  wire       irq_en;
  wire       read;

  wire [7:0] tx_data;
  wire tx_clear_req;
  wire tx_start;

  wire i_wb_cyc;
  wire i_wb_stb;
    
  assign i_wb_cyc = (wbs_adr_i[31:8] == 32'h3000_00) ? wbs_cyc_i : 1'b0;
  assign i_wb_stb = (wbs_adr_i[31:8] == 32'h3000_00) ? wbs_stb_i : 1'b0;

  CSR CSR(
    .i_clk      (wb_clk_i   ),
    .i_rst_n    (~wb_rst_i  ),
    .i_wb_cyc   (i_wb_cyc   ),
    .i_wb_stb   (i_wb_stb   ),
    .o_wb_stall (),
    .i_wb_adr   (wbs_adr_i  ),
    .i_wb_we    (wbs_we_i   ),
    .i_wb_dat   (wbs_dat_i ),
    .i_wb_sel   (wbs_sel_i  ),
    .o_wb_ack   (wbs_ack_o ),
    .o_wb_err   (),
    .o_wb_rty   (),
    .o_wb_dat   (wbs_dat_o  ),
    .o_CLOCK_FREQ_clock_freq            (clk_freq   ),
    .i_RECEIVED_DATA_rx                 (rx_data    ),
    .o_RECEIVED_DATA_rx_read_trigger    (read       ),
    .o_TRANSMISSION_DATA_tx             (tx_data    ),
    .o_INTERRUPT_ENABLE_irq_en          (irq_en     ),
    .i_TRANSMISSION_START_tx_start_clear(tx_clear_req),
    .o_TRANSMISSION_START_tx_start      (tx_start   )
  );

  wire [31:0] clk_div;
  assign clk_div = clk_freq / BAUD_RATE;

  uart_receive      receive(
    .rst        (wb_rst_i   ),
    .clk        (wb_clk_i   ),
    .clk_div    (clk_div    ),
    .rx         (rx         ),
    .rx_data    (rx_data    ),
    .read       (read       ),
    .irq_en     (irq_en     ),
    .irq        (irq        )
  );

  uart_transmission transmission(
    .rst        (wb_rst_i   ),
    .clk        (wb_clk_i   ),
    .clk_div    (clk_div    ),
    .tx         (tx         ),
    .tx_data    (tx_data    ),
    .clear_req  (tx_clear_req),
    .tx_start   (tx_start   )
  );

endmodule
```
[https://github.com/Cra2yPierr0t/caravel_walkthrough_uart/blob/main/verilog/rtl/UART/uart.v](https://github.com/Cra2yPierr0t/caravel_walkthrough_uart/blob/main/verilog/rtl/UART/uart.v)

一部しか使わない`io_in`, `io_out`, `io_oeb`をなぜ全てこのモジュールに引き込んでいるのか、疑問に思われるかもしれません。これはuartのIOで`tx`と`rx`だけ出して、`user_project_wrapper`の方で`io_in[30]`や`io_out[31]`と接続しようとすると非自明なLVSエラーに出くわすためです。

`user_project_wrapper`のIOと自分のモジュールを接続する場合は、その信号線の一部ではなく全体を接続するようにしましょう。`user_irq`も然りです。

次はOpenLANEを使ってUARTのRTLからGDSIIを生成しましょう。レイアウトが見られるので嬉しいです。

## OpenLANEの設定ファイルを書く(UART)

ここではUARTのRTLからGDSIIを生成します。まずは`openlane`ディレクトリに`uart`ディレクトリを作成し、`user_proj_example`のコンフィグをコピペしましょう。

OpenMPWにおいてゼロから設定ファイルを書くことは集積回路設計完全理解者でもない限りオススメしません。既存の設定ファイルを自分のデザイン向けに編集するのがオススメです。

```bash
cd caravel_user_project/openlane
mkdir uart
cp user_proj_example/config.json uart/
```

`config.json`は`config.tcl`の場合もあります。

次にコピペしてきた`config.json`を編集します。verilogのファイルパスの追加とダイサイズの変更とクロックの変更が主な作業です。

`DIE_AREA`と`PL_TARGET_DENSITY`はデザインによっては試行錯誤が必要です。

```
{
    "DESIGN_NAME": "uart",
    "DESIGN_IS_CORE": 0,
    "GLB_RT_MAXLAYER": 5,
    "FP_PDN_CHECK_NODES": 0,
    "VERILOG_FILES": [
      "dir::../../verilog/rtl/defines.v", 
      "dir::../../verilog/rtl/rggen-verilog-rtl/rggen_rtl_macros.vh",
      "dir::../../verilog/rtl/rggen-verilog-rtl/*.v",
      "dir::../../verilog/rtl/UART/uart.v",
      "dir::../../verilog/rtl/CSR.v",
      "dir::../../verilog/rtl/UART/uart_receive.v",
      "dir::../../verilog/rtl/UART/uart_transmission.v"
    ],

    "CLOCK_PORT": "wb_clk_i",
    "CLOCK_NET": "wb_clk_i",
    "FP_SIZING": "absolute",
    "DIE_AREA": "0 0 400 400",
    "PL_BASIC_PLACEMENT": 0,
    "PL_TARGET_DENSITY": 0.60,
    "ROUTING_CORES": 16,
    "VDD_NETS": ["vccd1"],
    "GND_NETS": ["vssd1"],
    "DIODE_INSERTION_STRATEGY": 4,
    "RUN_CVC": 1,
    "pdk::sky130*": {
        "FP_CORE_UTIL": 45,
        "RT_MAX_LAYER": "met4",
        "scl::sky130_fd_sc_hd": {
            "CLOCK_PERIOD": 100
        },
        "scl::sky130_fd_sc_hdll": {
            "CLOCK_PERIOD": 100
        },
        "scl::sky130_fd_sc_hs": {
            "CLOCK_PERIOD": 100
        },
        "scl::sky130_fd_sc_ls": {
            "CLOCK_PERIOD": 100,
            "SYNTH_MAX_FANOUT": 5
        },
        "scl::sky130_fd_sc_ms": {
            "CLOCK_PERIOD": 100
        }
    },
    "pdk::gf180mcuC": {
        "STD_CELL_LIBRARY": "gf180mcu_fd_sc_mcu7t5v0",
        "CLOCK_PERIOD": 100,
        "FP_CORE_UTIL": 40,
        "RT_MAX_LAYER": "Metal4",
        "SYNTH_MAX_FANOUT": 4,
        "PL_TARGET_DENSITY": 0.45
    }
}
```
[https://github.com/Cra2yPierr0t/caravel_walkthrough_uart/blob/main/openlane/uart/config.json](https://github.com/Cra2yPierr0t/caravel_walkthrough_uart/blob/main/openlane/uart/config.json)

## UARTのGDSIIを生成

設定ファイルの編集が終わったらリポジトリのルートで`make uart`を実行します。少し待った後、`gds/`に`uart.gds`が生成されています。

```bash
cd caravel_user_project
make uart
```

こちらがklayoutで見た`uart.gds`です。この中にRgGenで生成されたCSRも混ざってます。かわいいですね。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/uartgdsii.png?raw=true)

次はこれをCaravelのUser Project Area、つまり`user_project_wrapper.v`に挿入しましょう。

## `user_project_wrapper`に接続

デフォルトの状態では`user_proj_example`が接続されていますので、これを消し飛ばして`uart`を接続します。

```verilog
/*--------------------------------------*/
/* User project is instantiated  here   */
/*--------------------------------------*/

uart uart (
`ifdef USE_POWER_PINS
	.vccd1(vccd1),	// User area 1 1.8V power
	.vssd1(vssd1),	// User area 1 digital ground
`endif
    .wb_clk_i(wb_clk_i),
    .wb_rst_i(wb_rst_i),

    // MGMT SoC Wishbone Slave

    .wbs_stb_i(wbs_stb_i),
    .wbs_cyc_i(wbs_cyc_i),
    .wbs_we_i(wbs_we_i),
    .wbs_sel_i(wbs_sel_i),
    .wbs_dat_i(wbs_dat_i),
    .wbs_adr_i(wbs_adr_i),
    .wbs_ack_o(wbs_ack_o),
    .wbs_dat_o(wbs_dat_o),

    // IO ports
    .io_in  (io_in      ),
    .io_out (io_out     ),
    .io_oeb (io_oeb     ),

    // irq
    .user_irq (user_irq)
);

endmodule	// user_project_wrapper
```
[https://github.com/Cra2yPierr0t/caravel_walkthrough_uart/blob/main/verilog/rtl/user_project_wrapper.v](https://github.com/Cra2yPierr0t/caravel_walkthrough_uart/blob/main/verilog/rtl/user_project_wrapper.v)

## `user_defines.v`でGPIOの初期状態を設定
上記の通り、GPIOの31番を出力に、30番を入力に用います。本来これはファームウェアで動的に変更できるのですが、`user_defines.v`を編集して設定すると初期から31番が出力で30番が入力になってくれます。

特に使いみちを決めてないピンに関してはUser Project Areaからの出力に設定しておくと楽です。

`user_defines.v`の一部を以下に載せておきます。
```verilog
`define USER_CONFIG_GPIO_27_INIT `GPIO_MODE_USER_STD_OUTPUT
`define USER_CONFIG_GPIO_28_INIT `GPIO_MODE_USER_STD_OUTPUT
`define USER_CONFIG_GPIO_29_INIT `GPIO_MODE_USER_STD_OUTPUT
`define USER_CONFIG_GPIO_30_INIT `GPIO_MODE_USER_STD_INPUT_PULLDOWN
`define USER_CONFIG_GPIO_31_INIT `GPIO_MODE_USER_STD_OUTPUT
`define USER_CONFIG_GPIO_32_INIT `GPIO_MODE_USER_STD_OUTPUT
`define USER_CONFIG_GPIO_33_INIT `GPIO_MODE_USER_STD_OUTPUT
```
[https://github.com/Cra2yPierr0t/caravel_walkthrough_uart/blob/main/verilog/rtl/user_defines.v](https://github.com/Cra2yPierr0t/caravel_walkthrough_uart/blob/main/verilog/rtl/user_defines.v)

## `macro.cfg`で位置を設定
ダイサイズは3000nmx3000nmなので大体中心あたりに置く。
```
uart 1500 1500 N
```
[https://github.com/Cra2yPierr0t/caravel_walkthrough_uart/blob/main/openlane/user_project_wrapper/macro.cfg](https://github.com/Cra2yPierr0t/caravel_walkthrough_uart/blob/main/openlane/user_project_wrapper/macro.cfg)

## OpenLANEの設定ファイルを書く(user_project_wrapper)

`VERILOG_FILES_BLACKBOX`に自分のVerilogのパス、`EXTRA_LEFS`と`EXTRA_GDS`に`lef/`と`gds/`以下に生成されているファイルのパスを設定。`CLOCK_PERIOD`にクロック周期(ns)、`CLOCK_PORT`と`CLOCK_NET`にクロック信号線を設定。あとは`ROUTING_CORES`の値で配線に使うスレッドの数を増やせるので時短になって嬉しいです。
```
{
    "DESIGN_NAME": "user_project_wrapper",
    "ROUTING_CORES": 16,
    "VERILOG_FILES": [
      "dir::../../verilog/rtl/defines.v", 
      "dir::../../verilog/rtl/user_project_wrapper.v"
    ],
    "CLOCK_PERIOD": 100,
    "CLOCK_PORT": "uart.wb_clk_i",
    "CLOCK_NET": "uart.wb_clk_i",
    "FP_PDN_MACRO_HOOKS": "uart vccd1 vssd1 vccd1 vssd1",
    "MACRO_PLACEMENT_CFG": "dir::macro.cfg",
    "VERILOG_FILES_BLACKBOX": [
      "dir::../../verilog/rtl/defines.v", 
      "dir::../../verilog/rtl/UART/uart.v"
    ],
    "EXTRA_LEFS": "dir::../../lef/uart.lef",
    "EXTRA_GDS_FILES": "dir::../../gds/uart.gds",
    "FP_PDN_CHECK_NODES": 0,
    "SYNTH_ELABORATE_ONLY": 1,
```
[https://github.com/Cra2yPierr0t/caravel_walkthrough_uart/blob/main/openlane/user_project_wrapper/config.json](https://github.com/Cra2yPierr0t/caravel_walkthrough_uart/blob/main/openlane/user_project_wrapper/config.json)

## user_project_wrapperのGDSIIを生成
リポジトリのルートで以下のコマンドを実行しましょう。祈りの時間です。これが成功したら提出できたも同然です。
```bash
make user_project_wrapper
```

生成されたGDSIIが以下の通りです。かわいいですね。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/wrappergdsii.png?raw=true)

## テストベンチを作成
もうちょっとだけ続くんじゃ。正しく動作することを確認するためにテストベンチを書きましょう。

テストベンチは`verilog/dv`以下に格納されています。これもゼロから書くのは非常に骨が折れますので、既存の`wb_port`をコピーして編集しましょう。

```bash
cd verilog/dv
cp -r wb_port uart_test
```

とりあえず中身のファイル名は変えときましょう。
```bash
cd uart_test
mv wb_port.c uart_test.c
mv wb_port_tb.v uart_test_tb.v
```
次に`verilog/dv/`以下にあるMakefileに追加したいテストベンチ名を書き加えます。
```bash
PATTERNS = io_ports la_test1 la_test2 wb_port mprj_stimulus uart_test
```
[https://github.com/Cra2yPierr0t/caravel_walkthrough_uart/blob/main/verilog/dv/Makefile](https://github.com/Cra2yPierr0t/caravel_walkthrough_uart/blob/main/verilog/dv/Makefile)

そして`verilog/include/`以下のファイルを全て編集します。
### `includes.gl+sdf.caravel_user_project`

`verilog/gl`以下のファイルのパスを追加

```
// Caravel user project includes		
$USER_PROJECT_VERILOG/gl/user_project_wrapper.v	     
$USER_PROJECT_VERILOG/gl/uart.v
```
[https://github.com/Cra2yPierr0t/caravel_walkthrough_uart/blob/main/verilog/includes/includes.gl+sdf.caravel_user_project](https://github.com/Cra2yPierr0t/caravel_walkthrough_uart/blob/main/verilog/includes/includes.gl+sdf.caravel_user_project)

### `includes.gl.caravel_user_project`

これも`verilog/gl`以下のファイルのパスを追加、形式がちょっと違う

```
# Caravel user project includes	     
-v $(USER_PROJECT_VERILOG)/gl/user_project_wrapper.v	     
-v $(USER_PROJECT_VERILOG)/gl/uart.v    
```
[https://github.com/Cra2yPierr0t/caravel_walkthrough_uart/blob/main/verilog/includes/includes.gl.caravel_user_project](https://github.com/Cra2yPierr0t/caravel_walkthrough_uart/blob/main/verilog/includes/includes.gl.caravel_user_project)

### `includes.rtl.caravel_user_project`

`verilog/rtl`以下の使うファイルのパスを全て追加する。`rggen_rtl_macros.vh`をいろんなファイルがインクルードしているので、`+incdir+`を使う。 このファイルはicarus verilogのcmdfileなのでドキュメントとにらめっこしていい感じに仕上げたのが以下の通りです。

```
# Caravel user project includes
-v $(USER_PROJECT_VERILOG)/rtl/user_project_wrapper.v	     
+incdir+$(USER_PROJECT_VERILOG)/rtl
+incdir+$(USER_PROJECT_VERILOG)/rtl/rggen-verilog-rtl
-y $(USER_PROJECT_VERILOG)/rtl/rggen-verilog-rtl
-v $(USER_PROJECT_VERILOG)/rtl/CSR.v
-v $(USER_PROJECT_VERILOG)/rtl/UART/uart.v
-v $(USER_PROJECT_VERILOG)/rtl/UART/uart_transmission.v
-v $(USER_PROJECT_VERILOG)/rtl/UART/uart_receive.v
```
[https://github.com/Cra2yPierr0t/caravel_walkthrough_uart/blob/main/verilog/includes/includes.rtl.caravel_user_project](https://github.com/Cra2yPierr0t/caravel_walkthrough_uart/blob/main/verilog/includes/includes.rtl.caravel_user_project)

### `uart_test_tb.v`
`wb_port_tb.v`から要らない部分を切除します。理解できないものを理解できるものになるまで削る作業です。これは説明が面倒なのでリポジトリを見てください。

[https://github.com/Cra2yPierr0t/caravel_walkthrough_uart/blob/main/verilog/dv/uart_test/uart_test_tb.v](https://github.com/Cra2yPierr0t/caravel_walkthrough_uart/blob/main/verilog/dv/uart_test/uart_test_tb.v)

### `uart_test.c`

MMIOを使ってA(0x41)を送信するプログラムになっている。序盤の`reg_wb_~`とか`reg_mprj_~`とかはGPIOの設定だと思ってほしい。

```c
#include <defs.h>
#include <stub.c>

#define tx_data (*(volatile uint32_t*)0x30000008)
#define tx_start (*(volatile uint32_t*)0x30000010)


void main()
{
  reg_wb_enable = 1;
  reg_mprj_io_31 = GPIO_MODE_USER_STD_OUTPUT;

  reg_mprj_xfer = 1;
  while (reg_mprj_xfer == 1);

  tx_data = 0x00000041;
  tx_start = 0x000000001;

  while(tx_start != 0x00000000) {}
}
```
[https://github.com/Cra2yPierr0t/caravel_walkthrough_uart/blob/main/verilog/dv/uart_test/uart_test.c](https://github.com/Cra2yPierr0t/caravel_walkthrough_uart/blob/main/verilog/dv/uart_test/uart_test.c)

## テストベンチを実行する
リポジトリのルートで以下のコマンドを実行してテストベンチを開始する。完了すると`verilog/dv/uart_test/`以下に波形ファルが生成されている。

```bash
make verify-uart_test-rtl
```

そして生成された波形ファイルの中身がこちら

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/uart_waves.png?raw=true)

`mprj_io[31]`からUARTで`0x41`が送信されてて、`tx_start`も書き込まれたあと自動で落ちてますね、すげえ！マジで動いてる！

これで送信の動作確認まで完了しました！やったね。受信はテストベンチを書くのがダルいのでやりません。まあ動くやろ。

## プリチェック
プリチェック用のDockerイメージをインストール

```bash
make precheck
```
プリチェックを実行
```bash
make run-precheck
```
プリチェックでデザインに問題が無いことを確認しましょう。

## 提出
あとは`gds/`の中身をリモートリポジトリにアップして、efablessのサイトでリポジトリのURLを登録すれば提出が完了します。お手軽ですね。お疲れ様でした。

## 勧誘
もしOpenMPWに興味を持たれましたら、オープンソース半導体のslackである[open-source-silicon.dev](https://open-source-silicon.dev)に参加するのをおすすめします。ここにはEfablessのエンジニアの方々や各種OSSのコミッタが居ますので、詰まったらすぐに質問することが可能です。またGFMPW-0やMPW-7のシャトルが開始されている時の盛況ぶりは目を見張るものがあります。もし英語での質問に抵抗がありましたら、#japan-regionチャンネルもありますので、お気軽にご参加下さい！

## ポエム
個人でわざわざLSIまで作る意味があるのかというと、無い。手元のHDLの実証はFPGAで十分なので。でもFPGAがあるからこそ、FPGAで実証出来るからこそ、もう一歩踏み込んでLSIにまでしてしまってもいいと思う。だってGoogleがOpenMPWを用意してくれて、お前でもLSIを作れるよって言ってるんだし。しかも無料で。僕もハードルを下げるためにこんな記事を書いてます。出来るんだったら取り敢えずやってみませんか、LSIを個人で作れる機会なんてそうそうありませんよ。

