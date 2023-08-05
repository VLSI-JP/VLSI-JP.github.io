---
layout: default
title: 無から始める自作CPU
image: "https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/letsmakecpu.png"
---

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/letsmakecpu.png)

# 無から始める自作CPU(作成中)

**CPUは作れる！！！！！！！！ご存知でしたか！！！？？？？？？**

魔法の石、究極のブラックボックス、人類の英知たるCPUは不世出の天才にしか作れないとお思いですか？ **違います**！適切な知識さえあれば割と凡庸な人間でも作ることが出来ます！！！作りましょう！！

この記事はコンピュータサイエンスやプログラミングの知識が無くても、CPUを自作出来る所までお前を持っていく事を目的としています。やっていきましょう。

## 基礎知識

ここではCPUを作るのに必要な知識を説明します。いくつかは知っているかもしれません、もしくはどれも初めて見る単語かもしれません。ここで全て覚えろとは言いません。とりあえずざっと読んで必要になったら読み返してください。

### CPU

CPU、我々が作る対象です。CPUはとは一体なんでしょうか？概要すら知らないのに作ろうとするのは流石に無謀と言えます。ちょっとだけ先に知っておきましょう。

プログラミングという単語は皆さん人生のどこかで聞いたことがあるでしょう。最近の若人は中学高校の授業で実際にプログラミングをした経験があるかもしれませんし、実務で散々やっている方もいるかもしれませんし、報道番組で「時代はプログラミングだ！」とか与太話を聞いた事があるかもしれません。プログラミング、プログラムを書いて画面に何かが出たり音が出たりモーターが動いたりするアレですね。あなたがこの記事を読んでいるSafariやChromeもプログラムですし、YoutubeもTwitterもInstagramもTiktokもプログラムです。ああ素晴らしきかなプログラム。プログラムが無ければお前は生きてはいけません。

```c
#include <stdio.h>
int main(){
    printf("Hello World!\n");
    return 0;
}
```
↑ 素晴らしいプログラム

ですが考えてみてください、そのプログラム、一体どこで動いているのでしょうか？説明できますか？ その答えがCPUです。

CPUとはCentral Processing Unit、中央演算装置の略で、石みたいなガラスみたいな物質で出来ています。見た目は大体こんな感じです。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/cpu_image.jpeg)

この世に存在するプログラムは全てCPUの上で動いています。このCPUが無ければプログラムは動かせませんし、いくらプログラムを書いても意味がありません。我々の生活基盤はこの石に支えられているという訳ですね、ありがとうCPU。消えないでCPU。本記事ではそんなCPUを作ります。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/world_of_cpu.png)

### ディジタル回路

CPUは**ディジタル回路**というもので構成されています。ディジタル回路、難しそうな単語ですね。安心してください、ディジタル回路は単純なルールの集まりに過ぎません。少しずつ学んでいきましょう。

#### ディジタル回路が扱う値

ディジタル**回路**と呼ぶからには、ディジタル回路は電気的な何かを扱う回路なんだろうと思われてるかもしれません。正解です。ディジタル回路は電圧の高低を扱う回路です。具体的には0Vと5Vや、0Vと1.2Vなどを入力に取りますが、実際に何Vなのかは回路によってまちまちなので入力をHigh, Lowとするか、それすら見づらいし面倒なので電圧の高い入力を1、電圧の低い入力を0とする表記が一般的です。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/digital.png)

では実際にどのようなディジタル回路が存在するのか見ていきましょう。

#### NOT

一番シンプルなディジタル回路、**NOTゲート**です。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/not.png)

これは入力を反転します。例えば0が入力されたら1を出力し、1が入力されたら0を出力します。
以下にNOTの真理値表(そういうのがある)を置いておきます。

| `A` | `NOT A` |
| --  | --- |
| `0` | `1` |
| `1` | `0` |

記号としては`~x`とか`!x`とかが使われがちです。

#### OR

次は**ORゲート**です。これは入力の論理和、入力に一つでも1があるなら1を出力し、入力が全て0なら0を出力します。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/or.png)

真理値表は以下の通りです。

| `A` | `B` | `A AND B` |
| - | - | ------- |
| `0` | `0` | `0` |
| `0` | `1` | `1` |
| `1` | `0` | `1` |
| `1` | `1` | `1` |

記号としては`a | b`が使われがちです。

#### AND

次は**ANDゲート**です。これは入力の論理積、入力の全てが1なら1を出力し、それ以外なら0を出力します。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/and.png)

真理値表は以下の通りです。

| `A` | `B` | `A AND B` |
| - | - | ------- |
| `0` | `0` | `0` |
| `0` | `1` | `0` |
| `1` | `0` | `0` |
| `1` | `1` | `1` |

記号としては`a & b`が使われがちです。そのまんまですね。

#### NAND

次は**NANDゲート**、これはANDゲートの出力にNOTしたものです、入力の全てが1なら0を出力し、それ以外なら1を出力します。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/nand.png)

真理値表は以下の通りです。

| `A` | `B` | `A NAND B` |
| - | - | ------- |
| `0` | `0` | `1` |
| `0` | `1` | `1` |
| `1` | `0` | `1` |
| `1` | `1` | `0` |

記号としては`~(a & b)`で表せます。少し難易度が上がりましたかね？

NANDゲートの面白い特徴として、NANDゲートから他の全ての論理回路を構成できるという性質があります(Functional completeness)。実際にNANDゲートからORゲートを作るとこんな感じになります。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/or_nand.png)

本当にORゲートになってるかわからない？真理値表を書くとこの通り、ORゲートになっている事が分かります。

| `A` | `B` | `A NAND A` | `B NAND B` | `(A NAND A) NAND (B NAND B)` |
| - | - | -------- | -------- | -------------------------- |
| `0` | `0` |       `1`  |        `1` | `0`  |
| `0` | `1` |       `1`  |        `0` |                         `1`  |
| `1` | `0` |       `0`  |        `1` |                         `1`  |
| `1` | `1` |       `0`  |        `0` |                         `1`  |

このNANDゲートから他のディジタル回路を作り、それらを組み合わせCPUを作り、OSをその上で動かして、そのOSの上でテトリスを動かそうぜ！という熱い本が存在します。買うといいです。

[コンピュータシステムの理論と実装](https://amzn.asia/d/bmBqWAs)

#### XOR

論理回路ラスト、**XORゲート**です。これは**排他的論理和**と呼び、入力が異なる場合に1を出力し、同じ場合に0を出力します。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/xor.png)

真理値表は以下の通りです。

| `A` | `B` | `A XOR B` |
| - | - | ------- |
| `0` | `0` | `0` |
| `0` | `1` | `1` |
| `1` | `0` | `1` |
| `1` | `1` | `0` |

記号としては`a ^ b`が使われがちです。

#### MUX

少し発展したディジタル回路、**MUX(マルチプレクサ)**です。これは入力を選択する回路です。選択用信号Sが0の場合はAからの入力を出力し、Sが1の場合はBからの入力を出力します。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/mux.png)

真理値表は以下の通りです。

| `A` | `B` | `S` | `Y` |
| - | - | - | - |
| `0` | `0` | `0` | `0` |
| `0` | `0` | `1` | `0` |
| `0` | `1` | `0` | `0` |
| `0` | `1` | `1` | `1` |
| `1` | `0` | `0` | `1` |
| `1` | `0` | `1` | `0` |
| `1` | `1` | `0` | `1` |
| `1` | `1` | `1` | `1` |

このマルチプレクサ、次のようにNOTとANDとORで作ることが出来ます。`Y = (A & S) | (B & ~S)`。

#### HalfAdder

次はディジタル回路で少し面白い回路を紹介します。HalfAdder、**半加算器**です。以下のANDとXORで構成された回路を見てください。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/halfadder.png)

この回路の真理値表を書いてみましょう。以下の通りです。

| `A` | `B` | `C` | `S` |
| --- | --- | --- | --- |
| `0` | `0` | `0` | `0` |
| `0` | `1` | `0` | `1` |
| `1` | `0` | `0` | `1` |
| `1` | `1` | `1` | `0` |

CがAND回路の出力でSがXOR回路の出力になっています。当然っちゃ当然ですね。ですがよく見てください、このCとS、AとBの二進数の加算になっていますね。
ちなみにSは和を意味するSumの略で、Cは繰り上がりを意味するCarryの略です。

上記の表と下の数式を見比べればよく分かります。

$$
\begin{align}
0_{(2)} + 0_{(2)} &= 00_{(2)} \\
0_{(2)} + 1_{(2)} &= 01_{(2)} \\
1_{(2)} + 0_{(2)} &= 01_{(2)} \\
1_{(2)} + 1_{(2)} &= 10_{(2)}
\end{align}
$$

この様に加算と同じ動作をする回路のことを**加算器**と呼びます。覚えておきましょう。

#### FullAdder

さて、半加算器は二進数の1桁同士の加算を行う回路でした。更に2桁、10桁の加算を行いたい時、この半加算器を並べるだけでは複数桁の加算を行う事は出来ません。
加算をやった事のある皆さんはご存知でしょうが、加算は**繰り上がり**が発生する演算です。よって、この半加算器を繰り上がり入力を受け取るように拡張する必要があります。
その回路を**全加算器**、FullAdderと呼びます。実際に見てみましょう。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/fulladder.png)

半加算器２つとORで構成されていますね。新しく追加された全加算器の入力のXは繰り上がり入力を意味します。真理値表を見てみましょう。

| `A` | `B` | `X` | `C` | `S` |
| --- | --- | --- | --- | --- |
| `0` | `0` | `0` | `0` | `0` |
| `0` | `0` | `1` | `0` | `1` |
| `0` | `1` | `0` | `0` | `1` |
| `0` | `1` | `1` | `1` | `0` |
| `1` | `0` | `0` | `0` | `1` |
| `1` | `0` | `1` | `1` | `0` |
| `1` | `1` | `0` | `1` | `0` |
| `1` | `1` | `1` | `1` | `1` |

入力のA, B, Xと出力のC, Sを下の二進数の加算の式を見比べてみましょう。一致していますね。

$$
\begin{align}
0_{(2)} + 0_{(2)} + 0_{(2)} &= 00_{(2)} \\
0_{(2)} + 0_{(2)} + 1_{(2)} &= 01_{(2)} \\
0_{(2)} + 1_{(2)} + 0_{(2)} &= 01_{(2)} \\
0_{(2)} + 1_{(2)} + 1_{(2)} &= 10_{(2)} \\
1_{(2)} + 0_{(2)} + 0_{(2)} &= 01_{(2)} \\
1_{(2)} + 0_{(2)} + 1_{(2)} &= 10_{(2)} \\
1_{(2)} + 1_{(2)} + 0_{(2)} &= 10_{(2)} \\
1_{(2)} + 1_{(2)} + 1_{(2)} &= 11_{(2)} 
\end{align}
$$

この全加算器を用いる事で、以下のように複数桁の加算を作ることが可能となります。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/adder4.png)

二進数の4桁の加算の例

$$
\begin{align}
0011_{(2)} + 1001_{(2)} &= 1100_{(2)} \\
0010_{(2)} + 0011_{(2)} &= 0101_{(2)}
\end{align}
$$

この例は4桁の加算器ですが、8桁でも32桁でも作ることが出来ます。この世の足し算はこの回路で実行されているんですね。

#### D-FF

さてD-FF、謎の物体が出てきました。高校数学でANDやORをなんか見たことあるなあ、という人もD-FFは高校数学で学びません。これは**D型フリップフロップ**というものです。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/dff.png)

突然ですが、回路が情報を記憶する為には何が必須だと思いますか？それは**時間**です。時間の概念が無ければ記憶は意味を持ちえません。という訳で回路に時間の概念を導入します。
ディジタル回路における時間の概念、それは**クロック**です。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/clock.png)

クロックとは一定の周波数で0と1を行ったり来たりする信号を指します。クロックに関係する用語として、0から1になる**立ち上がりエッジ(posedge)**と、1から0になる**立ち下がりエッジ(negedge)**がありますので覚えておきましょう。

クロックを理解した所でこのD-FF、これは信号を記憶する論理素子です。具体的な動作としては、クロックの立ち上がりエッジで入力を出力し、その他のタイミングでは出力を維持します。図にすると分かりやすいです。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/dff_wave.png)

このフリップフロップ、別名レジスタ(register)とも呼ばれる事があります、覚えておいて損はないです。

#### MUXによるD-FFの改良

最後にこのD-FFをMUXで少し改良して、データを記憶するタイミングを制御できるようにしましょう。

以下では、MUXの`w_en`でD-FFへの入力を、D-FFからの出力か外部からの入力`w_data`かを選択できるようにしています。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/dff_improve.png)

波形は以下の通りです。`w_en`でD-FFに対する書き込みを制御出来ていますね。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/dff_improve_wave.png)

以上で我々はディジタル回路の基礎を完全に理解することが出来ました。やったね。
次はこのディジタル回路を作り出せる、FPGAの紹介です。

### FPGA

ディジタル回路を現実世界で実装したい時、選択肢は主に３つ存在しています。

1. ロジックIC
2. LSI
3. FPGA

ちなみに下記の３つ以外でCPUを作っている人を見かけたらそいつは異常な変態ですので近づきましょう。

ロジックICは秋葉で以下の黒いムカデみたいな奴を沢山買ってきて無限に半田付けするやつですね。

![](https://2.bp.blogspot.com/-khfR1UvFhbw/XDXbZILRDOI/AAAAAAABQ_k/FVT0GCYVkV4PgTySWyA63UIkk29pP0DkwCLcBGAs/s800/computer_ic_syusekikairo_long.png)

楽しいですが骨が折れるので今回はパス。

次のLSIですが工場に土下座して大金を払って石を焼いてもらう奴ですね。最近は土下座だけで焼かせてくれるようになりましたが土下座はしたくないのでパス。

![](https://user-images.githubusercontent.com/48832611/205153031-033603ce-ccac-4f02-b91a-920f15859ff1.png)

↑ 工場に土下座した例

そして最後の選択肢、**FPGA**です。**FPGA**とはField Programmable Gate Arrayの略で、一言で言ってしまうとディジタル回路を内部で自由に生成できるチップです。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/fpga.png)

詳しい仕組みの説明はここでは省きますが、どんな論理ゲートにもなれる特殊な論理ゲートが大量に詰まっていると思って頂ければ十分です。このFPGAでディジタル回路を作る場合、HDLというディジタル回路を設計する為の言語を使い、まるでプログラミングの様に回路を作る事が出来ます。

以後ではHDLの一種である、Verilog HDLを学んでいきましょう。

## Verilog HDL入門

この章ではVerilog HDLについて解説していきます。Verilog HDLとはハードウェア記述言語(Hardware Description Language)の一種で、ディジタル回路の設計・検証に用いられます。早い話**ハードウェア用のプログラミング言語**みたいなものです。Verilog HDLで記述されたディジタル回路は、**論理合成**という処理を通して回路に変換されます。その他のHDLとしてSystemVerilog, VHDL等が存在しています。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/learn_lsi/synthesis.png)

本書では、開発環境の快適さと対応ツールの多さからVerilog HDLを採用しています。

### 開発の流れ

ここではVerilog HDLを用いた開発の流れを簡潔に説明します。


![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/flow.png)

開発はシミュレーションと実機検証に分かれています。基本的に書いたコードの動作確認は上段のシミュレーションで行い、現実世界での動作の確認は下段の実機検証の流れで行います。

シミュレーションにおいて、回路を記述したVerilog HDLファイルとテストベンチを書き、それをシミュレータ上で動かし動作を検証します。実機検証では、利用するソフトウェアはEDA(Electronic Design Automation)のみです。シミュレーションで検証したVerilog HDLファイルをEDAに渡し、EDAでビルドを行いFPGA等の実機で動作を確認します。

本書の大半の作業はシミュレーションで行いますので基本はPCを触ってればいいです。実機検証に関しては環境構築も作業も非常に煩雑であるため、本記事の後半で解説します。

### 開発環境を作ろう

初めに開発環境を構築していきましょう。大半の初心者は環境構築で躓く事が経験的に知られていますので、出来る限り丁寧に解説します。

#### テキストエディタのインストール

visual studio codeをインストールせえ

#### Verilog HDLシミュレータのインストール

icarus verilogとgtkwaveをインストールをせえ

### 開発環境に慣れよう

開発環境の構築が完了したら、次は実際に開発環境を触って開発の流れを体感しましょう。

#### Verilogファイルとテストベンチ

とりあえず以下のコードを`day01.v`として保存しましょう。
何を書いているのか分からなくて不安でしょうが、今は写経していただくだけで結構です。

```verilog
module adder8(
     input wire       clk,
     input wire [7:0] in0,
     input wire [7:0] in1,
     output reg [7:0] out
);
    always @(posedge clk) begin
        out <= in0 + in1;
    end
endmodule
```

次に以下のコードを`day01_tb.v`として保存して下さい。
```verilog
module addr8_tb;
    reg             clk = 1'b0;
    reg    [7:0]    in0 = 8'h00;
    reg    [7:0]    in1 = 8'h00;
    wire   [7:0]    out;
    
    initial begin
        $dumpfile("wave.vcd");
        $dumpvars(0, DUT);
    end
    
    always #1 begin
        clk <= ~clk;
    end
    
    adder8 DUT(
        .clk    (clk    ),
        .in0    (in0    ),
        .in1    (in1    ),
        .out    (out    )
    );
    
    initial begin
        #2
        in0 = 8'h05;
        in1 = 8'h04;
        #2
        in0 = 8'hff;
        in1 = 8'h01;
        #2
        $finish;
    end
endmodule
```

さて、今しがた`day01.v`と`day01_tb.v`というファイルを作りました。`day01.v`はVerilog HDLで書いたディジタル回路です。この他に`day01_tb.v`というファイルも書きましたね、Verilogをシミュレータ上で動かす為にはディジタル回路を記述したVerilogファイルの他に、**テストベンチ**と呼ばれるVerilogファイルが必要です。

テストベンチとは文字通り回路のテストを行うためのVerilogファイルです。例えば、あなたが何か回路を作成したとして、その回路が意図した通りに動作するか確かめたい場合、回路に入力を与え、その出力を調べる必要がありますね？テストベンチはそういった回路へ入力を与えたり、出力を確認したり、内部信号を検査したり、回路内の波形ファイルを出力したりするのに使います。

具体的なテストベンチの書き方は後々解説しますので、今はとりあえず手元でシミュレーションを動かしてみましょう。

#### シミュレーションを実行

以下のコマンドを実行してください。

```bash
$ iverilog day01_tb.v day01.v
$ vvp a.out
```

実行が完了したらディレクトリ内に`wave.vcd`というファイルが生成されている筈です。これは波形ファイルです。波形ファイルは以下のコマンドで開きます。

```bash
$ gtkwave wave.vcd
```

GTKwaveの使い方は気合で慣れてください。信号を選択してAppendボタンを押すと多分見れますので適当に触ってみてください。

以上の通り、Verilog HDLでディジタル回路とテストベンチを記述し、シミュレーションを実行。波形を見てデバッグ、そして再びシミュレーションを実行という流れになります。この流れは今後何度も繰り返します。今全てを覚える必要はありませんので大丈夫です、そのうち手癖で開発を回せるようになります。

### Verilogの文法を学ぼう(基礎１)

では本格的にVerilog HDLを学んでいきましょう。最初は必須の構文です。

#### module

Verilog HDLではシステム全体をモジュールという単位で分割して構成します。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/module.png)

モジュールのイメージ

モジュール名は`module`の後ろに記述し、回路の内容は`module` ~ `endmodule`の間に記述します。

```verilog
module test_module();
 // ここに回路を記述
endmodule
```

#### 入出力ポート

回路には入出力が必要ですね、回路の入出力ポートは`input`, `output`で作成します。入力信号名は`input`の後ろに宣言し、出力信号名は`output`の後ろに宣言します。IOの個数や順番に決まりはありませんが、最後の宣言にはカンマが必要ないことに注意しましょう。

```verilog
module test_module (
    input    test_input1, // 入力信号線の定義
    input    test_input2,
    output   test_output1 // 出力信号線の定義
);
 // 回路
endmodule
```
その他に入出力両方に使える`inout`が存在しますが必須ではありませんので必要になった時に各自で調べてください。

#### wire型

次は`wire`です。人生のどこかで見た回路の上には配線がありましたね、あの配線は信号を伝達する信号線です。`wire`ではその信号線を生成します。

`wire`の使い方は以下の通りです。

```verilog
wire [31:0] 信号線名;
```

`wire`の後ろにある`[31:0]`は信号線の幅です。この場合は32本の幅を持った信号線になります。`[15:0]`にすれば16本の幅を持った信号線になりますし、`[3:0]`にすると4本の幅を持った信号線になります。

また、以下のように信号線の幅の記述を省略すると1本の幅を持った信号線になります。

```verilog
wire 信号線名;
```

#### 演算子

Verilog HDLには様々な演算子が存在します。これらの演算子は複数ビットを纏めて演算が可能です。以下によく使う演算子を載せておくきますので適宜参照してください。

#### assign文

`wire`で作られた信号線にはキーワード`assign`を用いて入力できます。`assign`を用いて書かれた回路は、入力が変化すると即座に出力が変化する**組み合わせ回路**となります。
以下のコードでは`test_output0`に`test_input`の値をそのまま入力しています。

```verilog
module test_module (
    input [9:0] test_input,
    output wire [9:0] test_output0,
    output wire [4:0] test_output1
);
    assign test_output0 = test_input;
    // 入力の上位5ビットを出力に直結している
    assign test_output1 = test_input[9:5]; 
endmodule
```
`test_output1`に入力している`test_input`の直後に書いてある`[9:5]`は**スライス**という操作であり、`test_input`の5ビットから9ビットの計5ビットを抜き出して、`test_output1`に出力しています。スライスはよく使う操作ですので覚えておきましょう。

#### 数値リテラル

Verilog HDLで数値を記述する際は、`ビット幅'進数 数値`という形式で記述します。`進数`は`b`で2進数、`d`で10進数、`h`で16進数です。`ビット幅`を指定しない場合は処理系によりけりです(Quartusでは32ビット)。
```verilog
    assign test_wire1 = 5'b10101; //2進5ビット
    assign test_wire2 = 8'd43;    //10進8ビット 
    assign test_wire3 = 16'hdead; //16進16ビット
```

### Verilogの文法を学ぼう(練習１)

覚えてばかりもつまらないでしょうし、ここで少し手を動かしてみましょう。

#### 練習問題１

以下の回路と同等のモジュール`problem1`を作成してください。入出力は画像の通りです。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/problem1.png)

#### 模範解答１

```verilog
module problem1(
  input       [7:0] i_p0,
  input       [7:0] i_p1,
  input       [7:0] i_p2,
  output wire [7:0] o_p
);

    wire [7:0] w_p;

    assign w_p = i_p0 & i_p1;
    assign o_p = w_p | i_p2;

endmodule
```

### Verilogの文法を学ぼう(基礎２)

#### 定数
Verilogには定数として`define`と`parameter`が存在するが、`define`は定数の利用時に定数名の前に｀が必要でキモいので`parameter`を使うことを推奨する。
```verilog
module test_module(
    // 省略
);
    parameter TEISU1 = 123;
    parameter TEISU2 = 32'h5555_5555;
    parameter TEISU3 = 32;
    
    wire [TEISU3-1:0] w1;
    assign w1 = TEISU2;
endmodule
```

#### reg型
#### always文

過去及び現在の値から出力を決定する回路である**順序回路**を書く際には`always`文を用いる。型が`reg`の変数にはこの`always`文内でのみ入力が可能である。`@(posedge clk)`は信号`clk`が0から1に立ち上がった時のみ`always`文内の記述を実行する、という意味である。`negedge`の場合は1から0に立ち下がった場合という意味になる。
後述するが、`reg`変数に入力する際は`<=`と`=`で意味が変わってくるので注意。
接続するときは<=というイメージ
(多分out(t)=out(t-1)のように、代入先と元に時間差がある時には<=するのかな？)
```verilog
module test_module (
    input  wire clk,    // 入出力の信号線も種類を定義できる
    input  wire [4:0] test_input,
    output reg  [4:0] test_output
);
    
    always @(posedge clk) begin // 順序回路の定義
        test_output <= test_output + test_input;
    end
endmodule
```

#### if文

if文は順序回路及び後述するfunction文で記述可能であり、`begin`~`end`で処理を囲う。
```verilog
module test_module(
    // 省略
);
    // 省略

    always @(posedge clk) begin
        if (test_wire) begin
            test_output <= 1;
        end else begin
            test_output <= 0;
        end
    end
endmodule
```


#### function文

assignで書けない複雑な組み合わせ回路の場合、function文を利用する。`function`~`endfunction`で回路の内容を囲み、`function 返り値のビット幅 回路名(input 入力信号)`と記述する。functionの返り値はfunction名に入力する形で記述する。
```verilog
    assign test_wire = test_function(test_input);

    function [4:0] test_function(input [2:0] func_in);
    begin
        case(func_in)
            3'b000 : test_function = 5'b00001;
            3'b001 : test_function = 5'b10101;
            default : test_function = 5'b11111;
        endcase
    end
    endfunction
```

#### case文

case文は、always文とfunction文内で記述可能であり、`case`~`endcase`で囲んで利用する。`case`の後に条件に利用する信号名を記述し、コロン`:`の後ろにマッチした場合の処理を記述する。どの条件にも当てはまらない場合の処理は`default`の後ろに記述する。`default`を記述していない場合、処理系に依るが警告かエラーが出る。
```verilog
always @(posedge clk) begin
    case(test_input)
        5'b0000 : test_out <= 5'b10101;
        5'b0001 : test_out <= 5'b00110;
        5'b0010 : test_out <= 5'b11001;
        default : test_out <= 5'b11111;
    endcase
end
```

#### 三項演算子

三項演算子はいつでも使える(?)。
代入先 = (条件) ? (条件が真の時に代入したい値) :(条件が偽の時に代入したい値)
ネストも可能。

#### モジュール呼び出し

作成したモジュールは`モジュール名 インスタンス名`で生成する。またIOは`.IO名(信号線|変数)`で接続する。
※インスタンスは他言語でいうオブジェクトだと理解すればよい(これ説明いらない感じ？)(気になる人もいるだろうし残しとこう)
```verilog

module test_module(
    // 省略
);
    wire [5:0] w1;
    reg  [5:0] r1;
    wire [5:0] w2;
    
    small_module sm(
        .in1    (w1    ),
        .in2    (r1    ),
        .out1   (w2    )
    );
    
    small_module sm_danger(w1, r1, w2); //こうも書ける(危険)
endmodule
```

#### generate文
#### 体操：UARTで送信回路を作る

### FPGAでVerilogを動かす

### シミュレーション方法
シミュレーション用の関数と構文を説明した後、サンプルコードを載せる。

#### システムタスク
シミュレーション用の関数(システムタスク)の解説
* `$dumpfile` : 波形ファイルの名前設定
* `$dumpvars` : 内部信号を波形に出力するモジュールを設定
* `$display`  : 標準出力に信号を出力する
* `$finish` : シミュレーションを終了する(必須)

#### シミュレーション用構文
シミュレーションに使うverilogの構文の解説
* クロック生成
```verilog
always #1 begin
    clk <= !clk;
end
```
1秒毎にclkの値を反転させる。これをクロック信号として扱う
* 時間経過で入力を変化
```verilog
initial begin
    #2
    data_i <= 8'h05;
    #2
    data_i <= 8'h11;
end
```
`#2`で2秒待機した後`data_i`への入力を変化させている。この場合`clk`の反転は1秒毎なので`#2`で1クロック待機となる。

#### テストベンチの例
適当なモジュール例(インクリメントするだけのモジュール)
```verilog
module test (
    input   clk,
    input   [7:0] data_i,
    output  [7:0] data_o
);

    reg [7:0] data;

    always @(posedge clk) begin
        data <= data_i + 8'h1;
    end
    assign data_o = data;
endmodule
```
テストベンチの例
```verilog
module test_sim();
    reg             clk = 0;
    reg     [7:0]   data_i = 8'h00;
    wire    [7:0]   data_o;

    initial begin
        $dumpfile("wave.vcd");  // 波形ファイル
        $dumpvars(0, test);     // testインスタンスの全信号を出力
    end

    test test(
        .clk        (clk    ), 
        .data_i     (data_i     ),
        .data_o     (data_o     )
    );

    always #1 begin    // クロックの生成 10秒毎に反転
        clk <= !clk;
    end

    initial begin
        data_i  <= 8'h11;
        #2     // 20秒 (1 クロック)待機
        data_i  <= 8'h20;
        #2
        data_i  <= 8'h30;
        $display("data_o : %d\n", data_o); // 標準出力にも出力可能
        #10
        $finish;    // 終了
    end
endmodule
```

#### ツール解説
* icarus verilog : オープンソースのVerilogシミュレータ(Linux, Windows, Mac?)
    * iverilog : シミュレータ用のVerilogコンパイラ
    * vvp : シミュレーション用バーチャルマシン(実行環境)
* gtkwave : オープンソースの波形ビューワー

使い方
```bash
iverilog -o 出力ファイル名 -s トップモジュール名 Verilogファイル名
vvp 出力ファイル名
gtkwave 波形ファイル名
```
実行例
```bash
iverilog -s test_sim test_sim.v test.v 
vvp a.out
gtkwave wave.vcd &
```

## Quartus入門
### 論理合成
### ピンアサイン
### 書き込み

## 自作CPU入門

さあ始めましょう

### プログラムのおしごと

### プログラムが動く流れ

CPUはプログラムをどのように実行しているのでしょうか？CPUはプログラムをそのまま実行している訳ではありません。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/prog_flow.png)

現代では、プログラムは**コンパイラ**というソフトウェアによって**アセンブリ**という中間言語に変換され、そのアセンブリは**アセンブラ**というソフトウェアによって`011010101...`のようなバイナリに変換されます。
CPUとってプログラムはこのバイナリになって初めて実行可能な形式になるというわけです。

### 命令セットアーキテクチャ 

先程、プログラムはバイナリに変換されて初めてCPUが実行可能な形になると言いましたね、プログラミング言語がこの世に沢山あるのと同じように、実は変換後のバイナリもこの世にはいくつか種類が存在しています。このバイナリの種類の事を命令セットアーキテクチャ(Instruction Set Architecture)、**ISA**と呼びます。基本的に、CPUはISAが異なるバイナリを実行できません。例えばMacのソフトウェアはMacのCPU以外では動きませんし、PCゲームのファイルをスマホに突っ込んでも動きませんね、動かそうとするな。

具体的なISAとしては、x86_64, AArch64, RISC-Vなどが存在しています。

本記事では筆者が定義した本記事独自のISA、**Z16**のCPUを作ります。Z16のCPUを作る前に一度、本項でZ16のアセンブリに親しみを持っておきましょう。

#### Z16の概要

Z16は本記事の筆者が定義したISAであり、16-bitの固定長命令セットです。16個の16-bitレジスタと16個の命令を持ちます。実装の容易さだけを考えて作りました。

RISC-Vみたいなイケてるロゴを持ってるISAに憧れがあるのでZ16のロゴを以下に置いておきます。ライセンスフリーです。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/z16_logo.png)

#### Z16のレジスタ

レジスタの内訳は以下の通り。全部で16個であり、アドレス0のレジスタ`ZR`は常に値が0な特殊なレジスタ、レジスタ`B1`~`B3`は後述しますが分岐命令の対象にすることが可能なレジスタです。残りのレジスタ`G0`~`G11`は自由に使えるレジスタとなっています。

| アドレス | 名前 | 説明 |
| -------- | ---- | ---- |
|    `0x0` | `ZR` | 常に値が0のレジスタ |
|    `0x1` | `B1` | 分岐命令用レジスタ |
|    `0x2` | `B2` | 分岐命令用レジスタ |
|    `0x3` | `B3` | 分岐命令用レジスタ |
|    `0x4` | `G0` | 汎用レジスタ |
|    `0x5` | `G1` | 汎用レジスタ |
|    `0x6` | `G2` | 汎用レジスタ |
|    `0x7` | `G3` | 汎用レジスタ |
|    `0x8` | `G4` | 汎用レジスタ |
|    `0x9` | `G5` | 汎用レジスタ |
|    `0xA` | `G6` | 汎用レジスタ |
|    `0xB` | `G7` | 汎用レジスタ |
|    `0xC` | `G8` | 汎用レジスタ |
|    `0xD` | `G9` | 汎用レジスタ |
|    `0xE` | `G10` | 汎用レジスタ |
|    `0xF` | `G11` | 汎用レジスタ |

#### Z16の命令

次はZ16の命令を見ていきましょう。

##### 演算命令

演算命令のビットフィールドの形式は以下の通り。4bit区切りになっており、下位4bitから順にオペコード、ディスティネーションレジスタ、ソースレジスタ１、ソースレジスタ２となっている。

`MSB | rs2[3:0] | rs1[3:0] | rd[3:0] | opcode[3:0] | LSB`

###### ADD

加算命令、`ADD RS2 RS1 RD`と記述し、RS2とRS1の値を加算してRDに格納する。オペコードは`4'h0`であるため、仮に`ADD G3 G2 G1`は`16'h7650`となる。また、RS2にゼロレジスタZRを指定することでRS1からRDに値を移動させるだけの命令として使うことが可能である。

$$
\text{RS2} + \text{RS1} \rightarrow \text{RD}
$$

###### SUB

減算命令、`SUB RS2 RS1 RD`と記述し、RS2からRS1の値を減算してRDに格納する。オペコードは`4'h1`であるため、仮に`SUB G3 G2 G1`は`16'h7651`となる。

$$
\text{RS2} - \text{RS1} \rightarrow \text{RD}
$$

###### MUL

乗算命令、`MUL RS2 RS1 RD`と記述し、RS2とRS1の値を乗算してRDに格納する。オペコードは`4'h2`であるため、仮に`MUL G3 G2 G1`は`16'h7652`となる。

$$
\text{RS2}\times \text{RS1} \rightarrow \text{RD}
$$

###### DIV

除算命令、`DIV RS2 RS1 RD`と記述し、RS2をRS1で割った商をRDに格納する。オペコードは`4'h3`であるため、仮に`DIV G3 G2 G1`は`16'h7653`となる。

$$
\text{RS2} \div \text{RS1} \rightarrow \text{RD}
$$

###### OR

OR命令、`OR RS2 RS1 RD`と記述し、RS2とRS1の値のORをRDに格納する。オペコードは`4'h4`であるため、仮に`OR G3 G2 G1`は`16'h7654`となる。

$$
\text{RS2} \parallel \text{RS1} \rightarrow \text{RD}
$$

###### AND

AND命令、`AND RS2 RS1 RD`と記述し、RS2とRS1の値のANDをRDに格納する。オペコードは`4'h5`であるため、仮に`AND G3 G2 G1`は`16'h7655`となる。

$$
\text{RS2} \& \text{RS1} \rightarrow \text{RD}
$$

###### XOR

XOR命令、`XOR RS2 RS1 RD`と記述し、RS2とRS1の値のXORをRDに格納する。オペコードは`4'h6`であるため、仮に`XOR G3 G2 G1`は`16'h7656`となる。

$$
\text{RS2} \oplus \text{RS1} \rightarrow \text{RD}
$$

###### SLL

論理左シフト命令、`SLL RS2 RS1 RD`と記述し、RS1の値をRS2の値だけ論理左シフトしてRDに格納する。オペコードは`4'h7`であるため、仮に`SLL G3 G2 G1`は`16'h7657`となる。

$$
\text{RS1} << \text{RS2} \rightarrow \text{RD}
$$

###### SRL

論理右シフト命令、`SRL RS2 RS1 RD`と記述し、RS1の値をRS2の値だけ論理右シフトしてRDに格納する。オペコードは`4'h8`であるため、仮に`SRL G3 G2 G1`は`16'h7658`となる。

$$
\text{RS1} >> \text{RS2} \rightarrow \text{RD}
$$

###### 演算命令まとめ

以下の計算を行いたい場合、アセンブリとそのバイナリは以下の通りになります。

$$
f(x, y) = x\times y + x - y
$$

xがG0、yがG1、結果をG7に格納する事にします。

アセンブリ

```
MUL G0 G1 G2
SUB G0 G1 G3
ADD G2 G3 G7
```

バイナリ

```
16'h4562
16'h4571
16'h67B0
```

演算命令の一覧表

`MSB | rs2[3:0] | rs1[3:0] | rd[3:0] | opcode[3:0] | LSB`

| opcode[3:0] | name | operation | assembly | binary |
| ----------- | ---- | --------- | ------- | ------ |
| `4'h0` | ADD | `rs2 + rs1 ➜ rd` | `ADD G3 G2 G1` | `16'h7650` |
| `4'h1` | SUB | `rs2 - rs1 ➜ rd` | `SUB G3 G2 G1` | `16'h7651` |
| `4'h2` | MUL | `rs2 * rs1 ➜ rd` | `MUL G3 G2 G1` | `16'h7652` |
| `4'h3` | DIV | `rs2 / rs1 ➜ rd` | `DIV G3 G2 G1` | `16'h7653` |
| `4'h4` | OR | `rs2 | rs1 ➜ rd` | `OR G3 G2 G1` | `16'h7654` |
| `4'h5` | AND | `rs2 & rs1 ➜ rd` | `AND G3 G2 G1` | `16'h7655` |
| `4'h6` | XOR | `rs2 ^ rs1 ➜ rd` | `XOR G3 G2 G1` | `16'h7656` |
| `4'h7` | SLL | `rs1 << rs2 ➜ rd` | `SLL G3 G2 G1` | `16'h7657` |
| `4'h8` | SRL | `rs1 >> rs2 ➜ rd` | `SRL G3 G2 G1` | `16'h7658` |

##### 即値命令

先で挙げた演算命令だけでは、G0に100を足す、G7から1を引くなどの操作を行うことが出来ないため、命令に値を埋め込みたい場合が発生する。そのような命令に直接埋め込まれている値を**即値(immediate)**と呼び、即値を含む命令を**即値命令**と呼ぶ。

即値命令のビットフィールドの形式は以下の通り、LSBから下位4bitがオペコード、そこから4bitがディスティネーションレジスタ、上位8bitが即値となっている。

`MSB | imm[7:0] | rd[3:0] | opcode[3:0] | LSB`

###### ADDI

即値加算命令。`ADDI IMM RD`と記述し、RDにIMMを加算してRDに格納する。IMMが取れる値の範囲は符号付き8bitなので-128~127。オペコードは`4'h9`であるため、仮に`ADDI 100 G0`は`16'h6449`、`ADDI -1 G2`は`16'hFF69`となる。

$$
\text{IMM} + \text{RD} \rightarrow \text{RD}
$$

###### 即値命令まとめ

以下の計算を行いたい場合、アセンブリとそのバイナリは以下の通りになります。

$$
f(x, y) = 3\times x + 100 + y
$$

xがG0、yがG1、結果をG7に格納する事にします。

アセンブリ

```
ADDI 3 G4
MUL G4 G0 G2
ADDI 100 G1
ADD G2 G1 G7
```

バイナリ

```
16'h0389
16'h8462
16'h6459
16'h65B0
```

`MSB | imm[7:0] | rd[3:0] | opcode[3:0] | LSB`

| opcode[3:0] | name | operation | assembly | binary |
| ----------- | ---- | --------- | ------- | ------ |
| `4'h9` | ADDI | `imm + rd ➜ rd` | `ADDI 100 G0` | `16'h6449` |

##### メモリ命令

Z16ではレジスタは16個あると先に説明しましたが、このままではCPUはデータを16個しか持つ事が出来ません。これではあまり実用的なCPUとは言えません。そこで**メモリ**という記憶装置を作り、扱えるデータの量を増やします。CPUがメモリを扱えるようにするため、メモリからデータを読み書きする専用の命令を定義します。

###### LOAD

Loadはメモリからデータを取り出す操作であり、Z16ではLOADというLoadを行う命令を一つ持っている。

LOAD命令のビットフィールドの形式は以下の通り、LSBから下位4bitはオペコード、そこから4bitずつにディスティネーションレジスタ、ソースレジスタ１、即値となっている。

`MSB | imm[3:0] | rs1[3:0] | rd[3:0] | opcode[3:0] | LSB`

LOAD命令。`LOAD IMM RS1 RD`と記述し、IMMとRS1の値の和をメモリアドレスとし、メモリのそのアドレスにあるデータをRDに格納する。IMMが取れる値の範囲は符号付き4bitであるため-8~7。オペコードは`4'hA`であるため、仮に`LOAD 0 G0 G1`は`16'h045A`となる。

$$
\text{Memory}[\text{IMM} + \text{RS1}] \rightarrow \text{RD}
$$

###### STORE

Storeはメモリにデータを書き込む操作であり、Z16ではSTOREというStoreを行う命令を一つ持っている。

STORE命令のビットフィールドは以下の通り、LSBから下位4bitがオペコード、そこから4bitずつに即値、ソースレジスタ１、ソースレジスタ２となっている。

`MSB | rs2[3:0] | rs1[3:0] | imm[3:0] | opcode[3:0] | LSB`

STORE命令。`STORE RS2 RS1 IMM`と記述し、IMMとRS1の値の和をメモリアドレスとし、RS2の値をメモリのそのアドレスに格納する。IMMが取れる値の範囲は符号付き4bitであるため-8~7。オペコードは`4'hB`であるため、仮に`STORE G1 G0 0`は`16'h540B`となる。

$$
\text{RS2} \rightarrow \text{Memory}[\text{RS1} + \text{IMM}]
$$

###### メモリ命令まとめ

`MSB | imm[3:0] | rs1[3:0] | rd[3:0] | opcode[3:0] | LSB`

`MSB | rs2[3:0] | rs1[3:0] | imm[3:0] | opcode[3:0] | LSB`

| opcode[3:0] | name | operation | assembly | binary |
| ---- | --------- | ------------- | ------- |
| `4'hA` | LOAD | `[rs1 + imm] ➜ rd` | `LOAD 4 G5 G6` | `16'h49AA` |
| `4'hB` | STORE | `rs2 ➜ [rs1 + imm]` | `STORE G5 G6 2` | `16'h9A2B` |

##### ジャンプ命令

ジャンプ命令とは文字通りジャンプする命令であり、プログラムの流れを強制的に変更します。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/jump.png)

ジャンプ命令のビットフィールドは以下の通り、LSBから下位4bitがオペコード、そこから4bitずつにディスティネーションレジスタ、ソースレジスタ１、即値となっている。

`MSB | imm[3:0] | rs1[3:0] | rd[3:0] | opcode[3:0] | LSB`

###### JAL

**JAL**はJump Absolute and Link の略であり、`JAL IMM RS1 RD`と記述する。動作としてはIMMとRS1の値の和にジャンプすると同時に、JALの次の命令のアドレスをRDに格納する。

即値のフィールドは符号付き4bitであり、-8~7の値を取る。

$$
\begin{align}
\text{PC} + 2 &\rightarrow \text{RD} \\
\text{IMM} + \text{RS1} &\rightarrow \text{PC}
\end{align}
$$

###### JRL

**JRL**はJump Relative and Linkの略であり、`JRL IMM RS1 RD`と記述する。動作としてはIMMとRS1の値とJRLが存在するアドレスの和にジャンプすると同時に、JRLの次の命令のアドレスをRDに格納する。

即値のフィールドは符号付き4bitであり、-8~7の値を取る。

$$
\begin{align}
\text{PC} + 2 &\rightarrow \text{RD} \\
\text{IMM} + \text{RS1} + \text{PC} &\rightarrow \text{PC}
\end{align}
$$

特殊な使い方として、`JRL 0 ZR G0`とすることでプログラムを停止させる事が可能である。

##### ジャンプ命令まとめ

JALのような、指定したアドレスに飛ぶジャンプの事を**絶対ジャンプ**と呼び、JRLのような自身のアドレスを基準に飛ぶジャンプのことを**相対ジャンプ**と呼ぶ。

以下の命令列では、アドレス4の位置にJRLが存在しています。この命令列が実行されると、JRLでアドレスAの命令にジャンプしAND命令が実行されます。またレジスタG6には6が格納されます。

```
0: ADD G0 G1 G2
2: SUB G0 G1 G2
4: JRL 6 ZR G6
6: MUL G0 G1 G2
8: DIV G0 G1 G2
A: AND G0 G1 G2
C: XOR G0 G1 G2
E: ADD G0 G1 G2
```

ジャンプ命令一覧表

`MSB | imm[3:0] | rs1[3:0] | rd[3:0] | opcode[3:0] | LSB`

| opcode[3:0] | name | operation | assembly | binary |
| ----------- | ---- | --------- | -------- | ------ |
| `4'hC` | JAL | `pc + 2 ➜ rd` <br>`imm + rs1 ➜ pc` | `JAL 4 G7 G8` | `16'h4BCC` |
| `4'hD` | JRL | `pc + 2 ➜ rd` <br>`imm + rs1 + pc ➜ pc` | `JRL 4 G7 G8` | `16'h4BCD` |

##### 分岐命令

分岐命令とは文字通り分岐を行う命令であり、強制的にプログラムの流れを変えるジャンプ命令とは異なり、条件に合致した場合にプログラムの流れを変える。

分岐命令のビットフィールドは以下の通りであり、LSBから下位4bitがオペコード、そこから2bitがソースレジスタ１、続いて2bitがソースレジスタ２、MSBから8bitが即値となっている。

`MSB | imm[7:0] | rs2[1:0] | rs1[1:0] | opcode[3:0] | LSB`

レジスタを指定するフィールドの幅がそれぞれ2bitであるため、0~3番目のレジスタ`ZR, B0, B1, B2`のみをソースレジスタに指定できる。

###### BEQ

**BEQ**とは、Branch Equalの略であり、`BEQ RS2 RS1 IMM`と記述する。動作としてはRS1とRS2の値が等しい場合に自身のアドレスに即値を加えたアドレスにジャンプする。オペコードは`4'hE`であるため、仮に`BEQ B2 B1 100`は`16'h649E`となる。

即値のフィールドは符号付き8bitであり、-128~127の値を取る。

$$
\begin{align}
\text{if}\ \text{RS2}\ &== \text{RS1} \\
\text{then}\ \text{imm} &+ \text{PC} \rightarrow \text{PC}
\end{align}
$$

###### BLT

**BLT**とは、Branch Less Thanの略であり、`BLT RS2 RS1 IMM`と記述する。動作としてはRS2の値がRS1の値より大きい場合に自身のアドレスに即値を加えたアドレスにジャンプする。オペコードは`4'hF`であるため、仮に`BLT B2 ZR 100`は`16'h644F`となる。

即値のフィールドは符号付き8bitであり、-128~127の値を取る。

$$
\begin{align}
\text{if}\ \text{RS2}\ &> \text{RS1} \\
\text{then}\ \text{imm} &+ \text{PC} \rightarrow \text{PC}
\end{align}
$$

###### 分岐命令まとめ

分岐命令を用いる事で、かなり柔軟にプログラムを書くことが可能となる。以下の例は1~10の総和を求め、アドレス8で停止するプログラムである。

アセンブリ

```
0: ADDI 10 B1
2: ADD B1 B2 B2
4: ADDI -1 B1
6: BLT B1 ZR -4
8: JRL 0 ZR G11
```

バイナリ

```
16'h0A19
16'h1220
16'hFF19
16'hFC4F
16'h00FD
```

分岐命令一覧表

`MSB | imm[7:0] | rs2[1:0] | rs1[1:0] | opcode[3:0] | LSB`

| opcode[3:0] | name | operation | assembly | binary |
| ----------- | ---- | --------- | -------- | ------ |
| `4'hE` | Branch equal | `if rs2 == rs1 `<br>`then imm + pc ➜ pc` | `BEQ B1 B2 -4` | `16'hFC5E` |
| `4'hF` | Branch Less than | `if rs2 > rs1 `<br>`then imm + pc ➜ pc` | `BLT B1 B2 -12` | `16'hF45F` |

#### Z16のエミュレータ

作成中

### ディジタルビルディングブロック

ここではCPUの各モジュールについて、その働きを説明していきます。

#### Instr Mem

まずは一番左にあるモジュール、**Instr Mem**です。これは**命令メモリ(Instruction Memory)**と呼び、中に命令が入っています。命令メモリ内の各命令にはアドレスが振られており、命令メモリにアドレスを入力すると命令が出力されます。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/instr_mem.png)

#### PC

次はCPUの中を見ていきましょう。このCPUの中にある**PC**は**プログラムカウンタ(Program Counter)**と呼び、命令メモリにアドレスを供給します。通常、プログラムは上から下に実行されるのでプログラムカウンタが出す値は毎クロック増えていきます。

プログラムカウンタ・命令メモリ・CPUの関係をまとめると、プログラムカウンタがアドレスを命令メモリに入力し、命令メモリが命令をCPUへを出力するという流れになります。このデータの流れを**フェッチ(Fetch)**と呼びます。覚えておきましょう。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/fetch.png)

#### Decoder

PCの次は**Decoder**です。これは**デコーダ**と呼び、命令から各種制御信号を生成します。具体的な動作の説明は他の部品の説明をしてから行いますので、とりあえず今は命令に応じて各部品を制御するモジュールだと認識しておいてください。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/decoder.png)

#### Register File

**Register File**、これは**レジスタファイル**と呼び、CPUが少量のデータを保持するのに使います。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/regfile.png)

動作としてはアドレスを入力するとデータを出力する**読み出し**と、

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/regfile_read.png)

アドレスとデータを入力するとレジスタファイルに書き込まれる**書き込み**を行います。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/regfile_write.png)

#### ALU

次はALUです。これは**算術論理演算ユニット(Arithmetic Logic Unit)**の略称で、文字通り論理演算(and, or, xor, シフト, etc..)と算術演算(加算, 減算, etc...)を行うモジュールです。CPUにおいて"計算"はこのALUが行っていると考えていただいてかまいません。ALUは制御信号と２つのデータを受け取り、１つのデータを出力します。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/alu.png)

#### Data Mem

最後に一番右にあるモジュール**Data Mem**、これは**データメモリ**と呼び。CPUが大量のデータを保存するのに使います。データメモリ対して、CPUはデータの書き込みと読み出しを行います。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/data_mem.png)

メモリに対してのデータ書き込みと読み出しにはそれぞれ名前があり、メモリへのデータ書き込みを**ストア(Store)**、

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/data_mem_store.png)

メモリからのデータ読み出しを**ロード(Load)**と呼びます。Store命令、Load命令とよく使われる単語ですので覚えておきましょう。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/data_mem_load.png)

### マイクロアーキテクチャ

#### Load命令

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/path_load.png)

#### Store命令

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/path_store.png)

#### 算術命令

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/path_exe.png)

#### ジャンプ命令
#### 分岐命令


## 自作CPUでプログラムを動かそう

## CPUを実用的にしよう
### MMIO
### 割り込み

## CPUを高速にしよう
### パイプライン化
### キャッシュ
### より高度なアーキテクチャ

## この先へ
