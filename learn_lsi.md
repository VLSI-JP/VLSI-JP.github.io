---
layout: default
title: OpenLANEと半導体設計入門
image: "https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/learn_lsi/learn_lsi_header.png"
---

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/learn_lsi/learn_lsi_header.png)

# OpenLANEと半導体設計入門
質問、修正案、その他連絡は@Cra2yPierr0tマデ

Github : [https://github.com/The-OpenROAD-Project/OpenLane](https://github.com/The-OpenROAD-Project/OpenLane)<br>
ドキュメント : [https://openlane.readthedocs.io/en/latest/](https://openlane.readthedocs.io/en/latest/)

## OpenLANEについて
OpenLANEとはOSSなRTL-to-GDSIIコンパイラであり、20個くらいのOSSを組み合わせて作られている。PDKとRTLとコンフィグを揃えて実行するとGDSIIが生えてくる。本記事ではOpenLANEの使い方及び各フローの説明を行いながら、現代の半導体設計を学んでいく。

OpenLANEの全体的なフローは次のようになっている。[^1]
1. Synthesis
    1. [`yosys`](https://github.com/YosysHQ/yosys) - RTLを論理合成
    2. [`abc`](https://github.com/YosysHQ/yosys) - PDKにマッピング
    3. [`OpenSTA`](https://github.com/The-OpenROAD-Project/OpenSTA) - 静的タイミング解析
2. Floorplan and PDN
    1. [`init_fp`](https://github.com/The-OpenROAD-Project/OpenROAD/tree/master/src/ifp) - コア領域の定義
    2. [`ioplacer`](https://github.com/The-OpenROAD-Project/OpenROAD/tree/master/src/ppl) - 入出力ポートの設置
    3. [`pdn`](https://github.com/The-OpenROAD-Project/OpenROAD/tree/master/src/pdn) - 給電ネットワークの生成
    4. [`tapcell`](https://github.com/The-OpenROAD-Project/OpenROAD/tree/master/src/tap) - weltapとデキャップセルの挿入
3. Placement
    1. [`RePLace`](https://github.com/The-OpenROAD-Project/RePlAce) - グローバル配置
    2. [`Resizer`](https://github.com/The-OpenROAD-Project/OpenROAD/tree/master/src/rsz) - 最適化
    3. [`OpenDP`](https://github.com/The-OpenROAD-Project/OpenROAD/tree/master/src/dpl) - ローカル配置
4. CTS
    1. [`TritonCTS`](https://github.com/The-OpenROAD-Project/OpenROAD/tree/master/src/cts) - クロック供給ネットワーク(クロックツリー)の合成
5. Routing
    1. [`FastRoute`](https://github.com/The-OpenROAD-Project/OpenROAD/tree/master/src/grt) - グローバル配線
    2. [`CU-GR`](https://github.com/cuhk-eda/cu-gr) - グローバル配線(プランB)
    3. [`TritonRoute`](https://github.com/The-OpenROAD-Project/TritonRoute) - ローカル配線
    4. [`SPEF_Extractor`](https://github.com/HanyMoussa/SPEF_EXTRACTOR) - 寄生フォーマット"SPEF"の摘出
6. GDSII Generation
    1. [`Magic`](https://github.com/RTimothyEdwards/magic) - routed defからのGDSIIファイル生成
    2. [`Klayout`](https://github.com/KLayout/klayout) - GDSIIファイル生成(バックアップ)
7. Checks
    1. `Magic` - DRCチェックとアンテナチェック
    2. `Klayout` - DRCチェック
    3. [`Netgen`](https://github.com/RTimothyEdwards/netgen) - LVSチェック
    4. [`CVC`](https://github.com/d-m-bailey/cvc) - 回路妥当性検証
    
![](https://github.com/The-OpenROAD-Project/OpenLane/blob/master/docs/_static/openlane.flow.1.png?raw=true)

出典：[https://openlane.readthedocs.io/en/latest/#openlane-architecture](https://openlane.readthedocs.io/en/latest/#openlane-architecture)

以上を見たところで半導体設計もOpenLANEも完全に理解出来る訳が無いので、これから各項目について説明していく。楽しみましょう。

本記事では以下の適当に作ったALU(`learn_lsi.v`)を題材に解説をしていきます。

```verilog
module learn_lsi(
  input  i_rstn,
  input  i_clk,
  input  [1:0]  i_ctrl,
  input  [31:0] i_data_a,
  input  [31:0] i_data_b,
  output [31:0] o_data
);

  always @(posedge i_clk, negedge i_rstn) begin
    if(!i_rstn) begin
      o_data    <= 32'h0000_0000;
    end else begin
      case(i_ctrl)
        2'b00   : o_data    <= i_data_a + i_data_b;
        2'b01   : o_data    <= i_data_a | i_data_b;
        2'b10   : o_data    <= i_data_a & i_data_b;
        2'b11   : o_data    <= i_data_a ^ i_data_b;
        default : o_data    <= 32'h0000_0000;
      endcase
    end
  end

endmodule
```

## 1. Synthesis

### Logic Synthesis

Logic Synthesis、またの名を**論理合成**とは、HDLで書かれた内容をより単純なANDやORのような論理ゲートやFF等のディジタル回路に変換する処理の事を指します。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/learn_lsi/synthesis.png)

Yosysはこの論理合成を行うツールの一つです。HDLを単純なディジタル回路に変換すると言いましたが、例えばVerilog HDLで書いた回路が実際どんな形式のファイルに変換されるのか気になりませんか？気になりますね、気になるという事にしましょう。
実はYosysの場合、Verilog HDLで書かれた回路は、より単純なVerilog HDLに変換されます。実際にYosysを動かして見てみましょう。

yosysの起動。yosysはexitコマンドで抜けられます。
```bash
$ yosys
-----------
なんやかんや
-----------

yosys> 
```

SystemVerilogモードでVerilogファイルの読み込む。VerilogだけどSVモードなのは気にしなくていい(多分)。
```bash
yosys> read -sv learn_lsi.v
```

トップモジュールを指定して階層構造をチェック
```bash
yosys> hierarchy -top learn_lsi
```

Yosysが処理しやすいように整形してシンプルな最適化を掛ける
```bash
yosys> proc
yosys> opt
```

ちなみにここで`show`コマンドを使うと回路図が見られる。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/learn_lsi/circuit_draw.png)


ディジタル回路を論理回路に変換してシンプルな最適化を掛ける
```bash
yosys> techmap
yosys> opt
```

Verilogファイル`synth.v`に出力
```bash
yosys> write_verilog synth.v
```

さて、出力された`synth.v`が論理合成後のVerilog HDLです。中身を見てみましょう。

```verilog
module learn_lsi(i_rstn, i_clk, i_ctrl, i_data_a, i_data_b, o_data);
  wire _000_;
  wire _001_;
  wire [1:0] _002_;
  wire [1:0] _003_;
  wire [1:0] _004_;
  wire [1:0] _005_;
  wire [1:0] _006_;
  wire [1:0] _007_;
```

なるほどね

```verilog
  assign _008_[1] = _040_[89] |(* src = "learn_lsi.v:0.0-0.0|learn_lsi.v:14.7-20.14|/usr/bin/../share/yosys/techmap.v:593.20-593.31" *)  _040_[121];
  assign _043_[25] = _008_[0] |(* src = "learn_lsi.v:0.0-0.0|learn_lsi.v:14.7-20.14|/usr/bin/../share/yosys/techmap.v:593.20-593.31" *)  _008_[1];
  assign _009_[0] = _040_[24] |(* src = "learn_lsi.v:0.0-0.0|learn_lsi.v:14.7-20.14|/usr/bin/../share/yosys/techmap.v:593.20-593.31" *)  _040_[56];
  assign _009_[1] = _040_[88] |(* src = "learn_lsi.v:0.0-0.0|learn_lsi.v:14.7-20.14|/usr/bin/../share/yosys/techmap.v:593.20-593.31" *)  _040_[120];
  assign _043_[24] = _009_[0] |(* src = "learn_lsi.v:0.0-0.0|learn_lsi.v:14.7-20.14|/usr/bin/../share/yosys/techmap.v:593.20-593.31" *)  _009_[1];
  assign _010_[0] = _040_[23] |(* src = "learn_lsi.v:0.0-0.0|learn_lsi.v:14.7-20.14|/usr/bin/../share/yosys/techmap.v:593.20-593.31" *)  _040_[55];
  assign _010_[1] = _040_[87] |(* src = "learn_lsi.v:0.0-0.0|learn_lsi.v:14.7-20.14|/usr/bin/../share/yosys/techmap.v:593.20-593.31" *)  _040_[119];
  assign _043_[23] = _010_[0] |(* src = "learn_lsi.v:0.0-0.0|learn_lsi.v:14.7-20.14|/usr/bin/../share/yosys/techmap.v:593.20-593.31" *)  _010_[1];
```

なるほど、ちょっと意味が分からないですね。読めるわけがない。

我々が読むには`learn_lsi.v`は複雑すぎました。以下の`simple.v`で試してみましょう。

```verilog
module simple(
  input i_clk,
  input i_data_a,
  input i_data_b,
  input i_ctrl,
  output o_data
);

  always @(posedge i_clk) begin
    if(i_ctrl) begin
      o_data    <= i_data_a & i_data_b;
    end else begin
      o_data    <= i_data_a | i_data_b;
    end
  end

endmodule
```

Yosysで論理合成

```bash
yosys> read -sv simple.v
yosys> hierarchy -top simple
yosys> proc; opt
yosys> techmap; opt
yosys> write_verilog synth.v
```

さて`synth.v`を覗いてみましょう。
```verilog
module simple(i_clk, i_data_a, i_data_b, i_ctrl, o_data);
  (* src = "simple.v:9.3-15.6" *)
  wire _0_;
  (* src = "simple.v:11.20-11.39" *)
  wire _1_;
  (* src = "simple.v:13.20-13.39" *)
  wire _2_;
  (* src = "simple.v:2.9-2.14" *)
  input i_clk;
  wire i_clk;
  (* src = "simple.v:5.9-5.15" *)
  input i_ctrl;
  wire i_ctrl;
  (* src = "simple.v:3.9-3.17" *)
  input i_data_a;
  wire i_data_a;
  (* src = "simple.v:4.9-4.17" *)
  input i_data_b;
  wire i_data_b;
  (* src = "simple.v:6.10-6.16" *)
  output o_data;
  reg o_data;
  (* src = "simple.v:9.3-15.6" *)
  always @(posedge i_clk)
    o_data <= _0_;
  assign _0_ = i_ctrl ? (* src = "simple.v:10.8-10.14|simple.v:10.5-14.8" *) _1_ : _2_;
  assign _1_ = i_data_a &(* src = "simple.v:11.20-11.39" *)  i_data_b;
  assign _2_ = i_data_a |(* src = "simple.v:13.20-13.39" *)  i_data_b;
endmodule
```

あーギリ読めますね、なんか`(* *)`で囲われた何か(正式名称: Attribute)が付いてますが`endmodule`から4行が本体ですね。if文が三項演算子に変換されているのが分かります。

勉強中
## 2. Floorplan and PDN
勉強中
## 3. Placement
勉強中
## 4. CTS
勉強中
## 5. Routing
勉強中
## 6. GDSII Generation
勉強中
## 7. Checks
勉強中

[^1]: https://openlane.readthedocs.io/en/latest/#openlane-design-stages
