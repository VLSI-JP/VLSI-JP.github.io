---
layout: default
title: ディジタル回路とVerilog入門
image: "https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/DigitalCircuit_And_Verilog.png"
---

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/DigitalCircuit_And_Verilog.png)

# ディジタル回路とVerilog入門

ディジタル回路とVerilog入門では、CPUを作る前に必要な基礎知識、そして作るために必要な道具の使い方を学んでいきます。

## 基礎知識

ここではCPUを作るのに必要な知識を説明します。覚える必要はありません。

### CPU

CPU、我々が作る対象です。CPUはとは一体なんでしょうか？概要すら知らないのに作ろうとするのは流石に無謀と言えます。ちょっとだけ先に知っておきましょう。

プログラミングという単語は皆さん人生のどこかで聞いたことがあるでしょう。最近の中高生はプログラミングの授業があるんですかね、気の毒ですね。プログラミング、プログラムを書いてゲームを作ったりモーターを動かしたりするアレですね。あなたがこの記事を読んでいるSafariやChromeもプログラムですし、YoutubeもTwitterもInstagramもプログラムです。ああ素晴らしきかなプログラム。プログラムが無ければお前は生きてはいけません。

```c
#include <stdio.h>
int main(){
    printf("Hello World!\n");
    return 0;
}
```
↑ 素晴らしいプログラム

ですが考えてみてください、そのプログラム、一体どこで動いているのでしょうか？ その答えが**CPU**です。

CPUとはCentral Processing Unit、中央演算装置の略で、石みたいなガラスみたいな物質で出来ています。見た目は大体こんな感じです。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/cpu_image.jpeg)

この世に存在するプログラムは全てCPUの上で動いています。このCPUが無ければプログラムは動かせませんし、いくらプログラムを書いても意味がありません。我々の生活基盤はこの石に支えられているという訳ですね、ありがとうCPU。愛してるCPU。本記事ではそんなCPUを作ります。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/world_of_cpu.png)

### 我々は如何にしてCPUを作るか

CPUを現実世界で作成する場合、選択肢として主にロジックIC・VLSI・FPGAの３つが存在しています。

1. ロジックIC
2. VLSI
3. FPGA

#### ロジックIC

ロジックICによるCPUの自作は現在、おそらく最も知名度を獲得しているCPUの作成方法です。ロジックICとは黒いムカデの形をした単純な動作をする電子部品であり、これらを組み合わせてCPUを構成します。

![](https://2.bp.blogspot.com/-khfR1UvFhbw/XDXbZILRDOI/AAAAAAABQ_k/FVT0GCYVkV4PgTySWyA63UIkk29pP0DkwCLcBGAs/s800/computer_ic_syusekikairo_long.png)
<p style="text-align:center"> <b>黒いムカデ</b></p>

このロジックICを用いた自作CPUに関しては、**CPUの創り方**という2003年出版の伝説的な名著が存在しています。このメイドが描かれた本をご存知の方も居るのではないでしょうか。

<p style="text-align:center"><iframe src="https://book.mynavi.jp/ec_products_detail_html/id=22065" frameborder="0" width="220" height="250" scrolling="yes"></iframe></p>

この本の存在と知名度が、一般人がCPUを自作するという一見不可能なチャレンジを現実味のあるものへと変えてくれたと私は考えています。実際、この本の血脈は受け継がれ、最近では一種類のICだけで16bitCPUを作る猛者が現れたり、自作CPUをトピックとした会が開催されるなど、多くの影響を現代まで及ぼしています。

[NAND(74HC00)だけで16bitCPUを作る[NLP-16]](https://cherry-takuan.org/article/?id=3)

[第2回 自作CPUを語る会(2023/12/03開催予定)](https://connpass.com/event/287012/)

そんなCPUを自作する伝統的な方法であるロジックICですが、多くの電子部品と少なからずの電子工作の知識が要求され、片手間にやるには骨が折れますので本記事ではパスします。

2023年12月03日に、自作CPUに興味のある人間で集まる自作CPUを語る会の第2回が開催予定ですので行くといいです。

#### VLSI

さて次のVLSIですが、こちらはCPUの作成方法として現代では異論の余地なく最もメジャーな方法です。

VLSIとは超大規模集積回路、Very Large Scale Integrationの略であり、先程のロジックICの中身を超大量に１つのチップ内に並べた物体です。Core i7やRyzen 7といった世にはばかるCPUは大体VLSIです。

VLSIは他の工業製品と同じく工場(ファブと呼ばれる)で大量生産されており、ファブへの発注にそれはそれは莫大な金が掛かります。大学が半導体製造装置を所有している場合、所属と目的によってはおそらく低額(または無料)で利用が可能ですが、ちゃらんぽらんな本記事の為に利用する事は不可能です。

よって本記事ではVLSIによる自作CPUはパスします。


![](https://user-images.githubusercontent.com/48832611/205153031-033603ce-ccac-4f02-b91a-920f15859ff1.png)
<p style="text-align:center"><b>Googleの金を使った例<a href="https://vlsi.jp/OpenMPW.html">(詳細)</a></b></p>

と言いつつ最近個人でも無料(Googleの金)でLSIを作らせてくれるプログラム、OpenMPWがスタートしました。VLSIが自作CPUの現実的な選択肢となる時代は既に始まっています。この記事で自作CPUを理解した後にチャレンジするのも良いでしょう。

#### FPGA

そして最後の選択肢、**FPGA**です。

**FPGA**とはField Programmable Gate Arrayの略で、一言で言ってしまうと回路を内部で自由に生成できるチップです。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/fpga.png)
<p style="text-align:center"><b>FPGA</b></p>

詳しい仕組みの説明はここでは省きますが、どんな回路にもなれる特殊な回路が大量に詰まっていると思って頂ければ十分です。このFPGAでCPUを作る場合、HDLという回路を設計する為の言語を使い、まるでプログラミングの様に回路を作る事が出来ます。
一昔前はこのFPGA、１万円は余裕で超える程度には値が張り、値段の時点で既に入門のハードルが高いものとなっていました。しかし現在、安価なFPGAも出現し始め、勇気の要らない値段でFPGAが手に入るようになってきました。

本記事では入手性、値段、開発の容易さから、CPUの作成手段としてFPGAを採用しています。

### 二進数と16進数

続いて知って頂きたいのが**二進数**です。二進数は数値の表記方法の１つであり、CPUを作るにはどうしても必要な知識ですのでここで説明します。

大前提として我々は普段、数値を`0, 1, 2, 3, 4, 5, 6, 7, 8, 9`の10個の数字で表記していますね。そして9よりも大きい数値を表記したい場合、9より大きい数字を我々は持っていませんので、**繰り上がり**という操作を行い、`10, 11, 12, ...`と表記します。これを十進数と呼びます。

#### 二進数

二進数とは、我々が`0, 1`しか数字を持っていない場合の数値の表記方法です。二進数において数値を0から順に数えていくと、まずは`0, 1,` と進み、我々は1より大きい数字を持っていませんので繰り上がりを行い、`10, 11, 100, 101, ...`と表記します。数字が0と1しか無い場合の数値の表記方法が二進数です。また二進数の桁数を**ビット**と呼びます。例えば`10101110`は8ビットです。

#### 16進数

追加で知って頂きたいのが**16進数**です。これは逆に我々が`0, 1, 2, 3, 4, 5, 6, 7, 8, 9, A, B, C, D, E, F`の16個の数字を持っている場合の数値の表記方法です。これは二進数との相性が良いためよく使われます。16進数において数値を0から順に数えていくと、`0, 1, 2, 3, 4, 5, 6, 7, 8, 9`と進み、我々は9より大きい数字としてAを持っていますので、`A, B, C, D, E, F`と進み、Fより大きい数字は持っていませんので繰り上がりを行い、`10, 11, 12, ...`と表記していきます。16進数は二進数と相性が良いと言いましたが、これは16進数と二進数で4ビットごとの繰り上がりのタイミングが一致しているためです。また16進数の数値を表記する場合は、十進数の値と区別するため`0x`を数値の先頭に付ける場合があります。

まあ難しかったらググってください。

以下に16進数と十進数と二進数の対応表を載せておきます。

| 十進数 | 16進数 | 二進数 |
| ------ | ------ | ------ |
|    `0` |  `0x0` |    `0` |
|    `1` |  `0x1` |    `1` |
|    `2` |  `0x2` |   `10` |
|    `3` |  `0x3` |   `11` |
|    `4` |  `0x4` |  `100` |
|    `5` |  `0x5` |  `101` |
|    `6` |  `0x6` |  `110` |
|    `7` |  `0x7` |  `111` |
|    `8` |  `0x8` | `1000` |
|    `9` |  `0x9` | `1001` |
|   `10` |  `0xA` | `1010` |
|   `11` |  `0xB` | `1011` |
|   `12` |  `0xC` | `1100` |
|   `13` |  `0xD` | `1101` |
|   `14` |  `0xE` | `1110` |
|   `15` |  `0xF` | `1111` |
|   `16` | `0x10` |`10000` |
|   `17` | `0x11` |`10001` |
|   `18` | `0x12` |`10010` |
|   `19` | `0x13` |`10011` |
|   `20` | `0x14` |`10100` |

#### 二進数の負の数

ここでは二進数の負の数について学びます。10進数で負の数を表現したい場合は`-10`のように数の前にマイナスの記号を付けるだけで簡単です。二進数で負の数を表現するためには２の補数という表現方法を使うのが一般的です。

詳しい定義は省きますが、二進数の数値を負の数にしたい場合は、`ビット反転して+1`を行うだけです。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/binary_minus.png)

またこの負の数の表現方法の場合、先頭のビットを見て0なら正の数であり、1なら負の数であると確認することが可能です。二進数の値は上記の符号付きと、符号なしに分類できます。例えば8bitで表現できる値の範囲は、符号なし8bitでは0~255ですが、符号付き8bitだと-128~127になります。二進数の値で計算する時は符号付きの値を扱ってるのか符号なしの値を扱っているのか注意しましょう。

この２の補数によって、二進数における引き算が可能となります。

### ディジタル回路とは

現代のCPUは**ディジタル回路**というもので構成されています。

ディジタル**回路**と呼ぶからには、電気的な何かを扱う物体だと思われるかもしれません。正解です。ディジタル回路は電圧の高低を扱う回路です。具体的には0Vと5Vや、0Vと1.2Vなどを入力に取りますが、実際に何Vなのかは回路によってまちまちなので入力をHigh, Lowとするか、それすら見づらいし面倒なので電圧の高い入力を1、電圧の低い入力を0とする表記が一般的です。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/digital.png)

ディジタル回路の具体的な例は後に回して、とりあえず今はCPUがこの0と1を扱うディジタル回路で構成されていると思っていてください。

### Verilog HDLとは

Verilog HDLとはハードウェア記述言語(Hardware Description Language)の一種で、ディジタル回路の設計・検証に用いられます。早い話**ハードウェア用のプログラミング言語**みたいなものです。

Verilog HDLで記述されたディジタル回路は、**論理合成**という処理を通して回路に変換されます。その他のHDLとしてSystemVerilog, VHDL等が存在しています。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/learn_lsi/synthesis.png)

本記事では、開発環境の快適さと対応ツールの多さからVerilog HDLを採用しています。

基礎知識は以上です。続いては、実際に手を動かしながらCPUを作るための技術を身に着けていきます。

## Verilog HDL入門

本章では、Verilog HDLを実際に書きながらディジタル回路の構成方法を学び、CPUを作る基礎的な技能を身に着けていきます。

### 開発の流れ

ここではVerilog HDLを用いた開発の流れを簡潔に説明します。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/flow.png)

開発は**シミュレーション**と**実機検証**に分かれています。基本的に書いたコードの動作確認は上段のシミュレーションで行い、現実世界での動作の確認は下段の実機検証の流れで行います。

シミュレーションにおいて、回路を記述したVerilog HDLファイルとテストベンチと呼ばれるファイルを書き、それをシミュレータ上で動かし動作を検証します。実機検証では、利用するソフトウェアはEDA(Electronic Design Automation)ツールのみです。シミュレーションで検証したVerilog HDLファイルをEDAに渡し、EDAでビルドを行いFPGA等の実機で動作を確認します。

本記事の大半の作業はシミュレーションで行いますので基本はPCを触ってればいいです。実機検証に関しては環境構築も作業も非常に煩雑であるため、本記事の後半で解説します。

### 開発環境構築

初めに開発環境を構築していきましょう。大半の初心者は環境構築で躓く事が経験的に知られています。心を強く持ちましょう。

以下ではWindows環境向けに解説を進めていきますが、LinuxやMacをお使いの方でも同名のソフトウェアをインストールして頂ければ本記事の内容を進めることが出来ます。

#### テキストエディタのインストール

テキストエディタとは文字を書く為のソフトウェアです。メモ帳でも自作CPUを出来なくはないですが、メモ帳は使い辛いのでVisual Studio Codeをインストールしましょう。もしVSCode以外にお気に入りのテキストエディタがありましたら(vimとか)、そのままお使いいただいて構いません。

このページ([https://code.visualstudio.com](https://code.visualstudio.com))からダウンロードを行ってください。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/vscode_download.png)

[https://code.visualstudio.com](https://code.visualstudio.com)

#### Verilog HDLシミュレータのインストール

本記事ではシミュレータとしてIcarus Verilogを利用します。

このページ([http://bleyer.org/icarus/](http://bleyer.org/icarus/))の**Downloadの一番上のリンク**からダウンロードしてください。v12がv13とかv14とかになってるかもしれませんが、とりあえず**Downloadの一番上のリンク**です。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/verilog_download.png)

[http://bleyer.org/icarus/](http://bleyer.org/icarus/)

またインストール時の注意点として**１つだけ絶対に押すボタンが存在します。** 以下の画像のSelect Additional TasksのAdd executable folder(s) to the user PATHです。**絶対にチェックを入れてください。** それ以外は適当にNextなりFinishなりポチポチ押してけばいいです。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/verilog_install.png)

インストールが終わりましたら、正しくインストール出来ているか確かめるためにソフトウェアの検索欄に`cmd`と入力してコマンドプロンプトを起動してください。起動方法をご存知ならPowerShellでもターミナルでも構いません。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/run_cmd.png)

そして`iverilog`コマンドと`vvp`コマンドを実行してください。以下のような表示が出たら成功です。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/check_iverilog.png)

また`gtkwave`コマンドを実行してください。何かウィンドウが出たら成功です。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/check_gtkwave.png)

### 開発環境に慣れる

環境構築が完了しましたら、次は実際に開発環境を触って開発の流れを体感しましょう。

まずは`verilog_test`という名前で新たにフォルダを作成してください。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/make_folder.png)

#### Verilogファイルとテストベンチ

そしてVSCodeを開き、`New File..`から先程作成したフォルダに`adder8.v`という名前のファイルを作成します。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/vscode_newfile.png)
![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/vscode_newfile_adder8.png)

そうしましたら以下のコードを`adder8.v`に書き写してください。何を書いているのか分からなくて不安でしょうが、今は写経していただくだけで結構です。

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

書き写しましたら上書き保存してください。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/vscode_write_adder8.png)

次に`adder8_tb.v`という名前で新たにファイルを作成し、以下のコードを書き写してください。

```verilog
module adder8_tb;
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

書き写しましたら上書き保存してください。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/vscode_write_adder8_tb.png)

さて、今しがた`adder8.v`と`adder8_tb.v`というファイルを作りました。`adder8.v`はVerilog HDLで書いた8bit同士の値の加算を行うディジタル回路です。また`adder8_tb.v`というファイルも書きましたね、これは**テストベンチ**というものです。Verilogをシミュレータ上で動かす為には、ディジタル回路を記述したVerilogファイルの他に、**テストベンチ**と呼ばれるVerilogファイルが必要です。

テストベンチとは文字通り回路のテストを行うためのVerilogファイルです。例えば、あなたが何か回路を作成したとして、その回路が意図した通りに動作するか確かめたい場合、回路に入力を与え、その出力を調べる必要がありますね？テストベンチはそういった回路へ入力を与えたり、出力を確認したり、内部信号を検査したり、回路内の波形ファイルを出力したりするのに使います。

具体的なテストベンチの書き方は後々解説しますので、今はとりあえず手元でシミュレーションを動かしてみましょう。

#### シミュレーションを実行

回路とテストベンチが書き終わりましたら、作成したファイルがあるフォルダを開き、白い部分をShiftを押しながら右クリックをして「ターミナルで開く」をクリックしてください。これで`verilog_test`フォルダ内でターミナルを開くことができます。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/open_terminal.png)

以下のコマンドを実行してください。

```bash
iverilog adder8_tb.v adder8.v
vvp a.out
```

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/run_iverilog.png)

実行が完了したらフォルダ内に`wave.vcd`というファイルが生成されている筈です。これは波形ファイルです。波形ファイルは以下のコマンドで開きます。

```bash
gtkwave wave.vcd
```

このコマンドでGTKwaveが起動します。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/run_gtkwave.png)

GTKWaveでは信号名をダブルクリックか、信号名を選択してAppendボタンを押すと波形を見ることができます。実際に`in0`と`in1`の加算結果が`out`に入力されているのが確認できると思います。適当に触ってみて気合で慣れてください。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/use_gtkwave.png)

以上のVerilog HDLでディジタル回路とテストベンチを記述し、シミュレーションを実行。波形を見てデバッグが開発の流れになります。この一連の流れは今後何度も繰り返します。今全てを覚える必要はありませんので大丈夫です、そのうち手癖で開発を回せるようになります。

1. Verilog HDLで回路を書く
2. Verilog HDLでテストベンチを書く
3. シミュレーションを実行
4. 波形を見る

### Verilogの基礎

開発環境の構築が完了しましたら、本格的にVerilogを用いてディジタル回路の基礎を学んでいきます。

#### wire型

まず最初はwire型です。人生で一度は何かしらの回路を見た経験があると思いますが、その回路の上には配線がありましたね、あの配線は信号を伝達する信号線です。Verilogにおいて、`wire`というキーワードを用いることで信号線を定義する事が出来ます。

`wire`の使い方は以下のように、`wire 信号幅 信号線名;`の順で記述します。行の終わりにはセミコロン`;`を必ず付けてください。

```verilog
wire [31:0] 信号線名;
```

`wire`の後ろにある`[31:0]`は信号線の幅です。この場合は32本の幅を持った信号線になります。`[15:0]`にすれば16本の幅を持った信号線になりますし、`[3:0]`にすると4本の幅を持った信号線になります。

また、以下のように信号線の幅の記述を省略すると1本の幅を持った信号線になります。

```verilog
wire 信号線名;
```

#### assign文

`wire`で作られた信号線にはキーワード`assign`を用いて値の入力や、信号線の接続ができます。`assign`を用いて書かれた回路は、入力が変化すると即座に出力が変化する**組み合わせ回路**となります。

この`assign`を用いた信号の入力時に、演算子を用いて演算を行うことが可能です。以下の例では信号線`w_a`に信号線`w_b`を接続しています。行の終わりにセミコロン`;`を必ず付けてください。

```verilog
  assign w_a = w_b;
```

実際に、assign文を用いてwireで定義された信号線に対して別の信号線を接続できる事を確認してみます。以下のHDLを`Basic.v`というファイル名で保存してください。

```verilog
module Basic;

  // --- おまじないここから ---
  initial begin
    $dumpfile("wave.vcd");
    $dumpvars(0, Basic);
  end

  reg r_a;
  // --- おまじないここまで ---

  wire w_x;

  assign w_x = r_a;

  // --- おまじないここから ---
  initial begin
    r_a = 1'b0;
    #2
    r_a = 1'b1;
    #2
    $finish;
  end
  // --- おまじないここまで ---

endmodule
```
<p style="text-align:center"> <b>Basic.v</b></p>

`Basic.v`では、信号線`w_x`を定義した後、assign文を用いて`r_a`の値を信号線`w_x`へ入力しています。`// --- おまじないここから ---`と`// --- おまじないここまで ---`で囲まれた部分は後ほど学びますので、今はおまじないだと思ってください。

そして、この`Basic.v`をシミュレータで実際に動かしてみましょう。

```bash
iverilog Basic.v
vvp a.out
gtkwave wave.vcd
```

`r_a`と`w_x`の波形を覗いてみると、`r_a`の値が`w_x`に入力されているのが分かると思います。

#### 数値リテラル

Verilog HDLで数値を記述する際は、`ビット幅'進数 数値`という形式で記述します。`進数`は`b`で2進数、`d`で10進数、`h`で16進数です。`ビット幅`を指定しない場合のビット幅は処理系によって変わります。(Quartusでは32ビット)。

```verilog
5'b10101; //2進5ビット
8'd43;    //10進8ビット 
16'hdead; //16進16ビット
```

| 進数 | 文字 | 例 |
| ---- | ---- | -- |
| 2    | `b`  | `8'b0011 1101` |
| 10   | `d`  | `8'd61` |
| 16   | `h`  | `8'h3d` |

<p style="text-align:center"> <b>Verilogの数値リテラル</b></p>

### ディジタル回路の基礎

wire型とassign文で信号線を繋げることが出来ました。ただ、線を繋げるだけでは何も面白くありません。この次は代表的なディジタル回路について知り、Verilogで実際に使ってみます。

#### NOT

最初は一番シンプルなディジタル回路、**NOTゲート**です。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/not.png)

これは入力を反転します。例えば0が入力されたら1を出力し、1が入力されたら0を出力します。
以下にNOTの真理値表(そういうのがある)を置いておきます。

| `A` | `NOT A` |
| --  | --- |
| `0` | `1` |
| `1` | `0` |

<p style="text-align:center"> <b>NOTゲートの真理値表</b></p>

Verilogでは`~信号名`で信号をNOTゲートに入力することが出来ます。実際に使ってみましょう。`Basic.v`のassign文を以下のように編集してください。

```verilog
  assign w_x = ~r_a;
```

これは`r_a`の値をNOTゲートに入力し、その出力を`w_x`に接続しています。シミュレータで動作を確認しましょう。

```bash
iverilog Basic.v
vvp a.out
gtkwave wave.vcd
```

`r_a`と`w_x`の波形を覗いてみると、`r_a`の値が反転されて`w_x`に入力されているのが分かると思います。

#### OR

次は**ORゲート**です。これは入力の論理和、入力の少なくともどちらか一方が1なら1を出力します。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/or.png)

真理値表は以下の通りです。

| `A` | `B` | `A OR B` |
| - | - | ------- |
| `0` | `0` | `0` |
| `0` | `1` | `1` |
| `1` | `0` | `1` |
| `1` | `1` | `1` |

<p style="text-align:center"> <b>ORゲートの真理値表</b></p>

Verilogでは`信号名1 | 信号名2`で信号をORゲートに入力することが出来ます。実際に使ってみる前に、`Basic.v`に少々手を加えます。以下の内容を`Basic.v`に書き込んでください。

```verilog
module Basic;

  // --- おまじないここから ---
  initial begin
    $dumpfile("wave.vcd");
    $dumpvars(0, Basic);
  end

  reg r_a;
  reg r_b;
  // --- おまじないここまで ---

  wire w_x;

  assign w_x = r_a | r_b;

  // --- おまじないここから ---
  initial begin
    r_a = 1'b0;
    r_b = 1'b0;
    #2
    r_a = 1'b0;
    r_b = 1'b1;
    #2
    r_a = 1'b1;
    r_b = 1'b0;
    #2
    r_a = 1'b1;
    r_b = 1'b1;
    #2
    $finish;
  end
  // --- おまじないここまで ---

endmodule
```

先程編集した`Basic.v`のassign文の部分で`r_a`と`r_b`の値をORゲートに入力し、信号線`w_x`に入力しています。

```verilog
  assign w_x = r_a | r_b;
```

以下のコマンドを打ち、シミュレータで動作を確認しましょう。

```bash
iverilog Basic.v
vvp a.out
gtkwave wave.vcd
```

`r_a`, `r_b`, `w_x`の波形と真理値表を見比べてみると、`r_a`と`r_b`の値がORされて`w_x`に入力されているのが分かると思います。

#### AND

次は**ANDゲート**です。これは入力の論理積、入力の両方が1なら1を出力します。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/and.png)

真理値表は以下の通りです。

| `A` | `B` | `A AND B` |
| - | - | ------- |
| `0` | `0` | `0` |
| `0` | `1` | `0` |
| `1` | `0` | `0` |
| `1` | `1` | `1` |

<p style="text-align:center"> <b>ANDゲートの真理値表</b></p>

Verilogでは`信号名1 & 信号名2`で信号をANDゲートに入力することが出来ます。そのまんまですね。実際に使ってみましょう。

`Basic.v`のassign文の部分を編集し、ORゲートをANDゲートに変更します。

```verilog
  assign w_x = r_a & r_b;
```

以下のコマンドを打ち、シミュレータで実際にANDゲートになっている事を確認しましょう。

```bash
iverilog Basic.v
vvp a.out
gtkwave wave.vcd
```

`r_a`, `r_b`, `w_x`の波形を覗いてみると、`r_a`と`r_b`の値がANDされて`w_x`に入力されているのが分かると思います。

#### NAND

次は**NANDゲート**、これはANDゲートの出力にNOTしたものです、入力の少なくともどちらか一方が0なら1を出力します。それ以外なら1を出力します。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/nand.png)

真理値表は以下の通りです。

| `A` | `B` | `A NAND B` |
| - | - | ------- |
| `0` | `0` | `1` |
| `0` | `1` | `1` |
| `1` | `0` | `1` |
| `1` | `1` | `0` |

<p style="text-align:center"> <b>NANDゲートの真理値表</b></p>

Verilogでは、NANDはNOTゲートとANDゲートを組み合わせて`~(信号名1 & 信号名2)`で表せます。カッコ()は数式と同じく優先順位を表しており、カッコの中身が優先され先に実行されます。少し難易度が上がりましたかね？実際に使ってみましょう。

`Basic.v`のassign文の部分を編集し、ANDゲートをNANDゲートに変更します。

```verilog
  assign w_x = ~(r_a & r_b);
```

以下のコマンドを打ち、シミュレータで実際にNANDゲートになっている事を確認しましょう。

```bash
iverilog Basic.v
vvp a.out
gtkwave wave.vcd
```

`r_a`, `r_b`, `w_x`の波形を覗いてみると、`r_a`と`r_b`の値のNANDが`w_x`に入力されているのが分かると思います。

NANDゲートの面白い特徴として、NANDゲートから他の全ての論理回路を構成できるという性質があります(Functional completeness)。実際にNANDゲートからORゲートを作るとこんな感じになります。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/or_nand.png)

本当にORゲートになっているか確かめるために真理値表を書くとこの通り、ORゲートになっている事が分かります。

| `A` | `B` | `A NAND A` | `B NAND B` | `(A NAND A) NAND (B NAND B)` |
| - | - | -------- | -------- | -------------------------- |
| `0` | `0` |       `1`  |        `1` | `0`  |
| `0` | `1` |       `1`  |        `0` |                         `1`  |
| `1` | `0` |       `0`  |        `1` |                         `1`  |
| `1` | `1` |       `0`  |        `0` |                         `1`  |

<p style="text-align:center"> <b>NANDゲートで作成したORゲートの真理値表</b></p>

このNANDゲートから他のディジタル回路を作り、それらを組み合わせCPUを作り、OSをその上で動かして、そのOSの上でテトリスを動かそうぜ！という熱い本が存在します。買うといいです。

[コンピュータシステムの理論と実装](https://amzn.asia/d/bmBqWAs)

ちなみにVerilogでは以下のようにNANDゲートでORゲートを作ることが出来ますが、めっちゃ分かりづらいので読まなくていいです。

```verilog
  assign w_x = ~(~(r_a & r_a)) | (~(r_b & r_b));
```

#### XOR

論理回路ラスト、**XORゲート**です。これは**排他的論理和**と呼び、入力が異なる場合に1を出力し、同じ場合は0を出力します。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/xor.png)

真理値表は以下の通りです。

| `A` | `B` | `A XOR B` |
| - | - | ------- |
| `0` | `0` | `0` |
| `0` | `1` | `1` |
| `1` | `0` | `1` |
| `1` | `1` | `0` |

<p style="text-align:center"> <b>XORゲートの真理値表</b></p>

Verilogでは`信号名1 ^ 信号名2`で信号をXORゲートに入力することが出来ます。実際に使ってみましょう。

`Basic.v`のassign文の部分を編集し、XORゲートを生成します。ちなみに記号^は日本語キーボードの「へ」を押すと出てきます。

```verilog
  assign w_x = r_a ^ r_b;
```

以下のコマンドを打ち、シミュレータで実際にXORゲートになっている事を確認しましょう。

```bash
iverilog Basic.v
vvp a.out
gtkwave wave.vcd
```

`r_a`, `r_b`, `w_x`の波形を覗いてみると、`r_a`と`r_b`の値がXORされて`w_x`に入力されているのが分かると思います。

#### MUXと三項演算子

少し発展したディジタル回路、**MUX(マルチプレクサ)**です。これは入力を選択する回路です。選択用信号Sが0の場合はAからの入力を出力し、Sが1の場合はBからの入力を出力します。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/mux.png)

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

<p style="text-align:center"> <b>マルチプレクサの真理値表</b></p>

このマルチプレクサ、次のようにNOTとANDとORで作ることが出来ます。`Y = (A & ~S) | (B & S)`。

実際にVerilogでマルチプレクサを使ってみる前に、`Basic.v`に少し手を加えます。かなり長いのでコピペで結構です。

```verilog
module Basic;

  // --- おまじないここから ---
  initial begin
    $dumpfile("wave.vcd");
    $dumpvars(0, Basic);
  end

  reg r_a;
  reg r_b;
  reg r_s;
  // --- おまじないここまで ---

  wire w_x;

  assign w_x = (r_a & ~r_s) | (r_b & r_s);

  // --- おまじないここから ---
  initial begin
    r_a = 1'b0;
    r_b = 1'b0;
    r_s = 1'b0;
    #2
    r_a = 1'b1;
    r_b = 1'b0;
    r_s = 1'b0;
    #2
    r_a = 1'b0;
    r_b = 1'b1;
    r_s = 1'b0;
    #2
    r_a = 1'b1;
    r_b = 1'b1;
    r_s = 1'b0;
    #2
    r_a = 1'b0;
    r_b = 1'b0;
    r_s = 1'b1;
    #2
    r_a = 1'b1;
    r_b = 1'b0;
    r_s = 1'b1;
    #2
    r_a = 1'b0;
    r_b = 1'b1;
    r_s = 1'b1;
    #2
    r_a = 1'b1;
    r_b = 1'b1;
    r_s = 1'b1;
    #2
    $finish;
  end
  // --- おまじないここまで ---

endmodule
```

`Basic.v`のassign文の部分でマルチプレクサを作成しています。

```verilog
  assign w_x = (r_a & ~r_s) | (r_b & r_s);
```

以下のコマンドを打ち、シミュレータで実際にマルチプレクサになっている事を確認しましょう。

```bash
iverilog Basic.v
vvp a.out
gtkwave wave.vcd
```

`r_a`, `r_b`, `r_s`, `w_x`の波形を覗いてみると、`r_s`が0の時は`w_x`に`r_a`の値が入力され、`r_s`が1の時は`w_x`に`w_b`の値が入力されており、マルチプレクサが出来ている事が確認できると思います。

論理ゲートを用いてマルチプレクサを作ることが出来ましたが、信号を選択したい時に毎回ANDとNOTを使ってマルチプレクサを作るのは非常に面倒です。そこでVerilogでは**三項演算子**というものを用いて簡単にマルチプレクサを表す事が出来ます。

三項演算子は`?`と`:`の２つの記号を組み合わせて使います。

```
S ? B : A
```

これは信号Sが0の場合は信号Aが選択され、信号Sが1の場合は信号Bが選択されます。`Basic.v`をこの三項演算子を使って書き直してみましょう。

```verilog
  assign w_x = r_s ? r_b : r_a;
```

以下のコマンドを打ち、シミュレータで実際にマルチプレクサになっている事を確認しましょう。

```bash
iverilog Basic.v
vvp a.out
gtkwave wave.vcd
```

`r_a`, `r_b`, `r_s`, `w_x`の波形を覗いてみると、三項演算子でマルチプレクサを生成出来ている事が確認できると思います。

#### HalfAdder

次はディジタル回路で少し面白い回路を紹介します。HalfAdder、**半加算器**です。以下のANDとXORで構成された回路を見てください。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/halfadder.png)

この回路の真理値表を書いてみましょう。以下の通りです。

| `A` | `B` | `C` | `S` |
| --- | --- | --- | --- |
| `0` | `0` | `0` | `0` |
| `0` | `1` | `0` | `1` |
| `1` | `0` | `0` | `1` |
| `1` | `1` | `1` | `0` |

CがAND回路の出力でSがXOR回路の出力になっています。当然っちゃ当然ですね。ですがよく見てください、このCとS、AとBの二進数の加算結果の二桁目と一桁目になっていますね。
ちなみにCは繰り上がりを意味するCarryの略で、Sは和を意味するSumの略です、

上記の表と下の数式を見比べればよく分かります。

<p>
$$
\begin{align}
0_{(2)} + 0_{(2)} &= 00_{(2)} \\
0_{(2)} + 1_{(2)} &= 01_{(2)} \\
1_{(2)} + 0_{(2)} &= 01_{(2)} \\
1_{(2)} + 1_{(2)} &= 10_{(2)}
\end{align}
$$
</p>

この様に加算と同じ動作をする回路のことを**加算器**と呼びます。覚えておきましょう。

それではVerilogで半加算器を作成してみます。まずは`Basic.v`にORゲートの検証の時に使ったコードをコピペしてください。

そして信号線`w_x`の宣言を消し、新たにCarryとSumを入力する信号線`w_c`と`w_s`を定義します。

```verilog
  wire w_c;
  wire w_s;
```

続いてassign文で図と同じようにANDとXORを用いて半加算器を作成します。

```verilog
  assign w_c = r_a & r_b;
  assign w_s = r_a ^ r_b;
```

そうしましたら以下のコマンドを打ち、シミュレータで半加算器が作成できている事を確認しましょう。

```bash
iverilog Basic.v
vvp a.out
gtkwave wave.vcd
```

`r_a`, `r_b`, `w_c`, `w_s`の波形を覗いてみると、半加算器を作成出来ている事が確認できると思います。


#### FullAdder

さて、半加算器は二進数の1桁同士の加算を行う回路でした。更に2桁、10桁の加算を行いたい時、この半加算器を並べるだけでは複数桁の加算を行う事は出来ません。
加算をやった事のある皆さんはご存知でしょうが、加算は**繰り上がり**が発生する演算です。よって、この半加算器を繰り上がり入力を受け取るように拡張する必要があります。
その回路を**全加算器**、FullAdderと呼びます。実際に見てみましょう。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/fulladder.png)

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

<p>
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
</p>

それでは実際に全加算器をVerilogで作成していきます。回路を作る前にまずは`Basic.v`に手を加えます。以下のコードを`Basic.v`にコピペしてください。

```verilog
module Basic;

  // --- おまじないここから ---
  initial begin
    $dumpfile("wave.vcd");
    $dumpvars(0, Basic);
  end

  reg r_a;
  reg r_b;
  reg r_x;
  // --- おまじないここまで ---

  wire w_c;
  wire w_s;

  // ここに全加算器を記述

  // --- おまじないここから ---
  initial begin
    r_a = 1'b0;
    r_b = 1'b0;
    r_x = 1'b0;
    #2
    r_a = 1'b1;
    r_b = 1'b0;
    r_x = 1'b0;
    #2
    r_a = 1'b0;
    r_b = 1'b1;
    r_x = 1'b0;
    #2
    r_a = 1'b1;
    r_b = 1'b1;
    r_x = 1'b0;
    #2
    r_a = 1'b0;
    r_b = 1'b0;
    r_x = 1'b1;
    #2
    r_a = 1'b1;
    r_b = 1'b0;
    r_x = 1'b1;
    #2
    r_a = 1'b0;
    r_b = 1'b1;
    r_x = 1'b1;
    #2
    r_a = 1'b1;
    r_b = 1'b1;
    r_x = 1'b1;
    #2
    $finish;
  end
  // --- おまじないここまで ---

endmodule
```

コピペ出来ましたら`// ここに全加算器を記述`の部分に全加算器を記述していきます。

全加算器は半加算器を２つ、ORゲートを１つ使った今までで最も複雑な回路であるのに加え、２つの半加算器を繋ぐ中間の信号線も存在しています。出来る所から作っていきましょう。

まずは信号`r_a`と`r_b`の値を半加算器に入力した際の、CarryとSumの出力信号を意味する中間的な信号線を`wire`を用いて新たに定義します。名前は適当に`w_s_ab`と`w_c_ab`とします。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/make_fa_1.png)

```verilog
wire w_c_ab;
wire w_s_ab;
```

そうしましたら半加算器を作成して`w_c_ab`と`w_s_ab`に信号を入力していきます。

```verilog
assign w_c_ab = r_a & r_b;
assign w_s_ab = r_a ^ r_b;
```

続いて`r_a`と`r_b`のSumと`r_x`の値を半加算器に入力した際の、Carryの出力信号を意味する中間的な信号線を新たに定義します。名前は適当に`w_c_abx`とします。Sumの方は`w_s`に入力すればいいので新たに定義する必要はありません。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/make_fa_2.png)

```verilog
wire w_c_abx;
```

それでは２つ目の半加算器を作成して`w_c_abx`と`w_s`に信号を入力してきます。

```verilog
assign w_c_abx = w_s_ab & r_x;
assign w_s     = w_s_ab ^ r_x; 
```

最後に`w_c`に`r_a`と`r_b`のCarryと`w_c_abx`のORを入力します。

```verilog
assign w_c = w_c_ab | w_c_abx;
```

これで全加算器が完成しました。完成した全加算器の全体像を以下に載せておきます。

```verilog
  wire w_c;
  wire w_s;
  wire w_c_ab;
  wire w_s_ab;
  wire w_c_abx;

  assign w_c_ab = r_a & r_b;
  assign w_s_ab = r_a ^ r_b;

  assign w_c_abx = w_s_ab & r_x;
  assign w_s     = w_s_ab ^ r_x;

  assign w_c = w_c_ab | w_c_abx;
```

それでは以下のコマンドを打ち、シミュレータで全加算器が作成できている事を確認しましょう。

```bash
iverilog Basic.v
vvp a.out
gtkwave wave.vcd
```

`r_a`, `r_b`, `r_x`, `w_c`, `w_s`の波形を覗いてみると、全加算器を作成出来ている事が確認できると思います。

#### 複数桁の加算器

この全加算器を用いる事で、以下のように複数桁の加算を作ることが可能となります。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/adder4.png)

二進数の4桁の加算の例

<p>
$$
\begin{align}
0011_{(2)} + 1001_{(2)} &= 1100_{(2)} \\
0010_{(2)} + 0011_{(2)} &= 0101_{(2)}
\end{align}
$$
</p>

この例は4桁の加算器ですが、8桁でも32桁でも作ることが出来ます。この世の足し算はこの回路で実行されているんですね。

#### 加算器と加算演算子

これで我々はディジタル回路で足し算を行う事が出来るようになりましたが、足し算を行いたい度に先程の様に加算器を作成していては大変非効率です。

Verilogでは**加算演算子**を用いる事で、簡単に加算器を生成することが可能です。

Verilogにおいて、加算演算子は`+`の記号になっています。実際に使ってみましょう。

```verilog
A + B
```

まずは`w_c`と`w_s`を１つの信号線にまとめ、新たに`w_add`という2bitの幅を持つ信号線を定義します。`w_c`は加算の繰り上がりビットですので`w_add`の2bit目、`w_s`は加算結果ですので`w_add`の1bit目になります。

```verilog
wire [1:0] w_add;
```

そして`r_a`と`r_b`と`r_x`を加算演算子で加算した結果を`w_add`に入力します。

```verilog
assign w_add = r_a + r_b + r_x;
```

以下に加算演算子を使った際の全体像を載せておきます。手作業で加算器を作成した時に比べて圧倒的に簡単ですね。

```verilog
wire [1:0] w_add;

assign w_add = r_a + r_b + r_x;
```

それでは以下のコマンドを打ち、シミュレータで全加算器が作成できている事を確認しましょう。

```bash
iverilog Basic.v
vvp a.out
gtkwave wave.vcd
```

`r_a`, `r_b`, `r_x`, `w_add`の波形を覗いてみると、加算演算子で全加算器を作成出来ている事が確認できると思います。

#### 演算子

Verilogには様々な演算子が存在し、これらの演算子は複数ビットを纏めて演算が可能です。以下によく使う演算子を載せておきますので適宜参照してください。

| 演算名 | 演算子 | 例 |
| --- | --- | --- |
| NOT | `~` | `~A` |
| OR | `|` | `A | B` |
| AND | `&` | `A & B` |
| XOR | `^` | `A ^ B` |
| 加算 | `+` | `A + B` |
| 減算 | `-` | `A - B` |
| 乗算 | `*` | `A * B` |
| 除算 | `/` | `A / B` |
| 左シフト | `<<` | `A << Bj` |
| 右シフト | `>>` | `A >> B` |
| 算術右シフト | `>>>` | `A >>> B` |

<p style="text-align:center"><b>Verilogの演算子</b></p>

### ディジタル回路の構造化

先程までの内容で、我々はVerilogで基礎的なディジタル回路を構成する方法を学びました。しかし、[開発の流れ](#開発の流れ)の部分でVerilog HDLにおける開発は、回路を記述したVerilogと、テストベンチの２つのファイルに分かれていると説明しましたが、先程まで`Basic.v`という１つのファイルしかありませんでした。

これはVerilogにスムーズに入門して頂くために、`Baisc.v`にディジタル回路とテストベンチの両方を書き込んでいたためです。これはあまり標準的とは言えませんので、以降ではディジタル回路とテストベンチの２つのファイルに分けていきます。ここではその方法を学んでいきましょう。

#### module

Verilog HDLではシステム全体をモジュールという単位で分割して構成します。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/module.png)

<p style="text-align:center"><b>モジュールのイメージ</b></p>

Verilogでモジュールを作成するには`module`というキーワードを用います。例えば`test_module`という名前のモジュールを作成したい場合は、`module test_module();`のように、`module`の後ろにモジュール名を記述します。そして回路の内容を`module モジュール名();` ~ `endmodule`の間に記述します。

```verilog
module test_module();
 // ここに回路を記述
endmodule
```

#### 入出力ポート

回路には入出力が必要です、回路の入出力ポートは`input`, `output`というキーワードで定義します。入力信号名は`input`の後ろに宣言し、出力信号名は`output`の後ろに宣言します。IOの個数や順番に決まりはありませんが、最後の宣言にはカンマが必要ないことに注意しましょう。

```verilog
module test_module (
    input    i_data1, // 入力信号線の定義
    input    i_data2,
    output   o_data1  // 出力信号線の定義
);
 // 回路
endmodule
```
その他に入出力両方に使える`inout`が存在しますが必須ではありませんので必要になった時に各自で調べてください。

#### 加算モジュールを作る

それでは、このmoduleを使って全加算器のモジュールを作成してみましょう。今回は勉強の為に加算演算子の存在を忘れます。`FullAdder.v`というファイルを新たに作り、そこに書き込んでいきます。

まずはモジュールを定義します。今回はモジュール名を`FullAdder`とします。

```verilog
module FullAdder();
endmodule
```

続いて入出力ポートを定義します。全加算器の回路図を思い出して頂くと、入力にはA, B, Xが必要であり、出力にはS, Cが必要です。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/fulladder.png)

`input`, `output`を使って定義しましょう。

```verilog
module FullAdder(
  input i_a,
  input i_b,
  input i_x,
  output o_c,
  output o_s
);
endmodule
```
<p style="text-align:center"><b>FullAdder.v</b></p>

入出力が定義出来ましたら、[FullAdder](#fulladder)で作った全加算器の回路をコピペし、一部の信号を編集します。`w_c`, `w_s`の宣言を消し、`r_a`, `r_b`, `r_x`, `w_c`, `w_s`をそれぞれ`i_a`, `i_b`, `i_x`, `o_c`, `o_s`に編集してください。

```verilog
module FullAdder(
  input i_a,
  input i_b,
  input i_x,
  output o_c,
  output o_s
);

  wire w_c_ab;
  wire w_s_ab;
  wire w_c_abx;

  assign w_c_ab = i_a & i_b;
  assign w_s_ab = i_a ^ i_b;

  assign w_c_abx = w_s_ab & i_x;
  assign o_s     = w_s_ab ^ i_x;

  assign o_c = w_c_ab | w_c_abx;

endmodule
```
<p style="text-align:center"><b>FullAdder.v</b></p>

これで全加算器のモジュールが完成しました。しかし今作成した`FullAdder`では、構成要素である半加算器を直接ディジタル回路で記述しており、かっこよくありません。そこで半加算器のモジュールを新たに作り、全加算器モジュール内で呼び出してみましょう。

#### モジュール呼び出し

Verilogを用いて作成したモジュールは`モジュール名 インスタンス名`で呼び出す事が可能です。また入出力ポートは`.ポート名(信号線名)`で接続します。

また１つのモジュールから複数の回路を生成できます。

```verilog
module test_module(
    // 省略
);

  wire [7:0] w_data1;
  wire [7:0] w_data2;
  wire [7:0] w_data3;
  wire [7:0] w_data4;

  wire [7:0] w_out1;
  wire [7:0] w_out2;
    
  small_module sm1(
    .in1    (w_data1    ),
    .in2    (w_data2    ),
    .out1   (w_out1     )
  );
    
  small_module sm2(
    .in1    (w_data3    ),
    .in2    (w_data4    ),
    .out1   (w_out2     )
  );
    
endmodule
```

#### 全加算器で半加算器を利用する

それでは全加算器モジュールから半加算器モジュールを呼び出してみましょう。まずは半加算器のモジュールを記述したファイル`HalfAdder.v`を新たに作成します。

```verilog
module HalfAdder(
  input i_a,
  input i_b,
  output o_c,
  output o_s
);

  assign o_c = i_a & i_b;
  assign o_s = i_a ^ i_b;

endmodule
```
<p style="text-align:center"><b>HalfAdder.v</b></p>

そして、先程学んだモジュールを呼び出しを用いて`FullAdder`内で`HalfAdder`を呼び出してみます。全加算器は２つの半加算器を持っていましたので、１つ目の半加算器の名前を`HA1`、２つ目の半加算器の名前を`HA2`とします。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/make_fa_3.png)

assign文の部分を半加算器モジュールに置き換えます。

```verilog
module FullAdder(
  input i_a,
  input i_b,
  input i_x,
  output o_c,
  output o_s
);

  wire w_c_ab;
  wire w_s_ab;
  wire w_c_abx;

  HalfAdder HA1 (
    .i_a    (i_a    ),
    .i_b    (i_b    ),
    .o_c    (w_c_ab ),
    .o_s    (w_s_ab )
  );

  HalfAdder HA2 (
    .i_a    (w_s_ab ),
    .i_b    (i_x    ),
    .o_c    (w_c_abx),
    .o_s    (o_s    )
  );

  assign o_c = w_c_ab | w_c_abx;

endmodule
```
<p style="text-align:center"><b>FullAdder.v</b></p>

これにて半加算器を用いて全加算器を作成することが出来ました。先程に比べてイケてる見た目になりましたね。

#### テストベンチでモジュールを呼び出す

作成した全加算器が正しく動作することをテストベンチで確認しましょう。テストベンチでも全加算器を作った時と同じくモジュールを呼び出す事が出来ます。`Basic.v`を開き、`w_add`を消して再び`w_c`と`w_s`を定義し、モジュール呼び出しを用いて`Basic.v`から`FullAdder`を呼び出して入出力を接続しましょう。

```verilog
  // --- おまじないここまで ---
  
  wire w_c;
  wire w_s;

  FullAdder FA (
    .i_a    (r_a    ),
    .i_b    (r_b    ),
    .i_x    (r_x    ),
    .o_c    (w_c    ),
    .o_s    (w_s    )
  );

  // --- おまじないここから ---
```

それでは以下のコマンドを打ち、シミュレータで正しく動作できている事を確認しましょう。iverilogのコマンドに`Basic.v`, `FullAdder.v`, `HalfAdder.v`を忘れずに指定してください。

```bash
iverilog Basic.v FullAdder.v HalfAdder.v
vvp a.out
gtkwave wave.vcd
```

`r_a`, `r_b`, `r_x`, `w_c`, `w_s`の波形を覗いてみると、作成したモジュールが正しく動作していることが確認できると思います。

### ディジタル回路と時間

今まで学んだ回路は、現在の入力から出力が決定する**組み合わせ回路**と呼ばれるものでした。この次は、現在に加え過去の状態から出力が決定する**順序回路**について学んでいきます。

#### D-FF

さてD-FF、謎の物体が出てきました。高校数学でANDやORをなんか見たことあるなあ、という人もD-FFは高校数学で学びません。これは**D型フリップフロップ**という情報を記憶するディジタル回路です。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/dff.png)

突然ですが、回路が情報を記憶する為には何が必須だと思いますか？それは**時間**です。時間の概念が無ければ記憶は意味を持ちえません。という訳で回路に時間の概念を導入します。
ディジタル回路における時間の概念、それは**クロック**です。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/clock.png)

クロックとは一定の周波数で0と1を行ったり来たりする信号を指します。クロックに関係する用語として、0から1になる**立ち上がりエッジ(posedge)**と、1から0になる**立ち下がりエッジ(negedge)**がありますので覚えておきましょう。

クロックを理解した所でこのD-FF、これは信号を記憶する論理素子です。具体的な動作としては、クロックの立ち上がりエッジで入力を出力し、その他のタイミングでは出力を維持します。図にすると分かりやすいです。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/dff_wave.png)

このフリップフロップ、別名レジスタ(register)とも呼ばれる事があります、覚えておいて損はないです。


#### reg型

`wire`は信号線を作成するためのキーワードでしたが、信号線はデータを保持する事が出来ません。先程説明した通り、ディジタル回路がデータを保持するためには**フリップフロップ**(以後レジスタ)が必要でしたね。

Verilogでは`reg`というキーワードでレジスタを生成することが可能です。`reg`の使い方は以下のように。`reg サイズ レジスタ名;`の順で記述します。行の最後にセミコロン`;`を忘れずに付けてください。

```verilog
reg [31:0] test_32reg;
```

また以下のようにサイズの記述を省略すると1bitの幅を持ったレジスタになります。

```verilog
reg test_1reg;
```

#### always文

`reg`で作られたレジスタにはalways文というものを用いて信号を入力することが可能です。

先程述べたように、情報の記録とはクロックという信号に密接に関係してします。そこでalways文は殆どの場合においてクロック信号と一緒に用います。

always文は`always @(posedge クロック信号) begin ~ end`または`always @(negedge クロック信号) begin ~ end`という文法で記述し、`begin ~ end`の間に回路を記述します。`posedge`の場合はクロック信号が立ち上がりエッジのタイミングで処理が実行され、`negedge`の場合はクロック信号の立ち下がりのタイミングで処理が実行されます。

以下の例ではクロック信号として`i_clk`を定義しており、`i_clk`の立ち上がりで`i_data_a`と`i_data_b`の加算結果がレジスタ`r_data`に格納されるようにしています。そして`r_data`の値を`o_data`に出力しています。

```verilog
module test_module(
  input i_clk,
  input [31:0] i_data_a,
  input [31:0] i_data_b,
  output o_data
);

  reg [31:0] r_data;

  assign o_data = r_data;
  always @(posedge i_clk) begin
    r_data <= i_data_a + i_data_b;
  end

endmoudle
```

ここで注目してほしいのがalways文内の`<=`です。Verilogではalways文内において、代入演算子として`=`と`<=`を使うことが可能であり、それぞれ異なった動作をします。`<=`はノンブロッキング代入と呼ばれ、`=`はブロッキング代入と呼ばれています。なんか２種類ありますが混在させるとバグの温床になりますので、本記事では`<=`のノンブロッキング代入しか用いません。always文で代入は`<=`です！そういう事にしましょう。

#### MUXによるD-FFの改良

最後にこのD-FFをMUXで少し改良して、データを記憶するタイミングを制御できるようにしましょう。

以下ではD-FFとMUXを組み合わせ、`i_en`という信号でD-FFの保持するデータを書き換えるタイミングを決められるようにしてます。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/dff_improve.png)

実際にVerilogで作成してみましょう。`AdvancedDff.v`というファイルを新たに作り、そこに作成していきます。

まずはモジュールと入出力ポートを定義します。今回は実装を楽にするためDFFと`o_data`を同じものにします。よって、`output`で出力ポートをする際に`reg`と組み合わせて出力ポート兼レジスタの`o_data`を定義しています。

```verilog
module AdvancedDff(
  input i_clk,
  input i_data,
  input i_en,
  output reg o_data
);
endmodule
```

続いて、`i_clk`をクロックとしたalways文を書き、三項演算子を用いて`i_en`が1の場合に`o_data`に`i_data`の値が入力されるようにします。

```verilog
  always @(posedge i_clk) begin
    o_data <= i_en ? i_data : o_data;
  end
```

こうして完成したマルチプレクサで改良したDFFのコードが以下の通りです。

```verilog
module AdvancedDff(
  input i_clk,
  input i_data,
  input i_en,
  output reg o_data
);
  
  always @(posedge i_clk) begin
    o_data <= i_en ? i_data : o_data;
  end

endmodule
```
<p style="text-align:center"><b>AdvancedDff.v</b></p>

動作を確認するため以下のテストベンチを`AdvancedDff_tb.v`という名前で保存してください。

```verilog
module AdvancedDff_tb;

  // --- おまじないここから ---
  initial begin
    $dumpfile("wave.vcd");
    $dumpvars(0, AD);
  end

  reg i_clk = 1'b0;
  reg i_data = 1'b0;
  reg i_en = 1'b0;
  wire o_data;

  always #1 begin
    i_clk <= ~i_clk;
  end
  // --- おまじないここまで ---
  
  AdvancedDff AD(
    .i_clk  (i_clk  ),
    .i_data (i_data ),
    .i_en   (i_en   ),
    .o_data (o_data )
  );
  
  // --- おまじないここから ---
  initial begin
    // 初期状態
    i_data  = 1'b0;
    i_en    = 1'b0;
    #2
    // データ入力
    i_data  = 1'b1;
    i_en    = 1'b0;
    #6
    // 書き込みを有効化
    i_data  = 1'b1;
    i_en    = 1'b1;
    #2
    // 書き込みを無効化
    i_data  = 1'b1;
    i_en    = 1'b0;
    #4
    // データ0に
    i_data  = 1'b0;
    i_en    = 1'b0;
    #4
    $finish;
  end
  // --- おまじないここまで ---

endmodule
```
<p style="text-align:center"><b>AdvancedDff_tb.v</b></p>

そして以下のコマンドで動作を確認します。

```bash
iverilog AdvancedDff_tb.v AdvancedDff.v
vvp a.out
gtkwave wave.vcd
```

`i_clk`, `i_data`, `i_en`, `o_data`の波形を覗いてみると、D-FFに対する書き込みタイミングを`i_en`で制御出来ている事が確認出来ると思います。`o_data`の値が赤くなっていると思いますが、これは**不定値**と呼ばれ、最初は`o_data`の値が決まっていませんのでこの値になります。正しい動作です。

以上で我々はディジタル回路の基礎を完全に理解することが出来ました。やったね。

### Verilogのシミュレーションを学ぶ

今までは特に説明なしにテストベンチを出しました。このタイミングでテストベンチの書き方を説明します。

以下のテストベンチのコードを例にテストベンチ用のVerilogの構文を学んでいきましょう。

```verilog
module AdvancedDff_tb;

  // --- おまじないここから ---
  initial begin
    $dumpfile("wave.vcd");
    $dumpvars(0, AD);
  end

  reg i_clk = 1'b0;
  reg i_data = 1'b0;
  reg i_en = 1'b0;
  wire o_data;

  always #1 begin
    i_clk <= ~i_clk;
  end
  // --- おまじないここまで ---
  
  AdvancedDff AD(
    .i_clk  (i_clk  ),
    .i_data (i_data ),
    .i_en   (i_en   ),
    .o_data (o_data )
  );
  
  // --- おまじないここから ---
  initial begin
    // 初期状態
    i_data  = 1'b0;
    i_en    = 1'b0;
    #2
    // データ入力
    i_data  = 1'b1;
    i_en    = 1'b0;
    #6
    // 書き込みを有効化
    i_data  = 1'b1;
    i_en    = 1'b1;
    #2
    // 書き込みを無効化
    i_data  = 1'b1;
    i_en    = 1'b0;
    #4
    // データ0に
    i_data  = 1'b0;
    i_en    = 1'b0;
    #4
    $finish;
  end
  // --- おまじないここまで ---

endmodule
```

#### 入出力用信号の定義

作成した回路の動作を検証するためには、回路に様々な信号を入力し、出力を検査する必要があります。テストベンチにおいて回路に入力する信号は`reg ビット幅 信号名;`で定義します。また回路からの出力を受ける信号は`wire ビット幅 信号名;`で定義します。両方ともビット幅を省略した場合は1bitの幅になります。

実際に入出力用信号を定義した例が以下になります。この場合、`i_clk`、`i_data`、`i_en`が入力用の信号で、`o_data`が出力を受ける信号になります。

```verilog
  reg i_clk = 1'b0;
  reg i_data = 1'b0;
  reg i_en = 1'b0;
  wire o_data;
```

#### テスト用回路の生成

テストベンチで回路をテストするためにはテストベンチ内で回路を生成する必要があります。そこで行うのがテスト用回路の生成です。

Verilogにおいて、回路は`モジュール名 インスタンス名`で生成する事が可能です。以下の例では`AdvancedDff`という名前のモジュールから`AD`という名前の回路(インスタンス)を生成しています。
また入出力信号は`.インターフェイス名(信号線名)`で接続します。以下の例では`AdvancedDff`は`i_clk`, `i_data`, `i_en`,`o_data`というインターフェイスを持っていますので、先程定義した入出力信号をそれぞれのインターフェイスに接続しています。

カンマの有無に注意してください。

```verilog
  AdvancedDff AD(
    .i_clk  (i_clk  ),
    .i_data (i_data ),
    .i_en   (i_en   ),
    .o_data (o_data )
  );
```

ちなみにインスタンス名としてDUTが使われる事がありますが、これはDevice Under Testの略です。

#### システムタスク

上記のテストベンチには、`$`で始まる名前の関数がいくつか使われていますね。これらの関数を**システムタスク**と呼びます。システムタスクとは回路の検証に使うための関数であり、Verilog HDLには様々なシステムタスクが存在してます。

シミュレーション時に最低限使うべきシステムタスクは以下の４つです。

* `$dumpfile` : 波形ファイルの名前設定
* `$dumpvars` : 内部信号を波形に出力するモジュールを設定
* `$display`  : ターミナルに文字列を表示
* `$finish` : シミュレーションを終了する(必須)

必要な時にそれぞれ説明していきます。

#### 波形ファイルの生成

作成した回路が正しく動作しているか確かめる際に重要となるのが**波形ファイル**です。`$dumpfile("wave.vcd")`で`wave.vcd`という名前の波形ファイルを生成出来ます。また`$dumpvars()`でどの回路の波形を出力するか指定できます。例では`$dumpvars(0, DUT)`としており、`DUT`という名前の回路の波形を得ています。

実際には以下のように`initial begin ~ end`で囲った部分で利用します。

```verilog
  initial begin
    $dumpfile("wave.vcd");  
    $dumpvars(0, AD);      
  end
```

この記述は回路の生成の前に記述してください。

#### クロックの生成

基礎知識のD-FFの部分で説明しましたが、回路が情報を記録するためには**クロック**という信号がしばしば必要になります。先程の練習問題ではクロックを用いませんでしたが、今後は殆どの場合においてクロックを必要とする回路を作ることになりますので、今のうちにテストベンチでクロックを生成する方法を学んでおきましょう。

テストベンチにおいてはクロック信号の生成に以下の記述を用います。以下の記述では先程定義した入力用信号である`i_clk`の値を1秒毎にNOT演算子で反転し、再び`i_clk`に入力しています。この結果、`i_clk`の値は0, 1, 0, 1, 0, 1と振動するようになります。これをクロック信号として利用しています。

```verilog
  always #1 begin
    i_clk <= ~i_clk;
  end
```

#### 入力信号を制御

テスト対象の回路に対してずっと同じ信号を入力するのはあまり効果的なテストとは言えません。そこで時間に応じて入力信号を変化させる必要があります。そのために以下の記述を用います。

以下の`initial begin ~ end`で囲まれた記述では、入力信号の`data_i`に値を格納してから`#2`という記述をしています。この`#`は遅延と呼び、`#数値`で指定した秒数だけ待機する事が出来ます。待っている間にもクロックは動作します。

その後何回か信号を変化させ、最終的にシステムタスクの`$finish`が実行されます。この`$finish`が実行されるとシミュレーションが終了します。テストベンチには`$finish`を必ず記述する必要があります。

```verilog
  initial begin
    // 初期状態
    i_data  = 1'b0;
    i_en    = 1'b0;
    #2
    // データ入力
    i_data  = 1'b1;
    i_en    = 1'b0;
    #6
    // 書き込みを有効化
    i_data  = 1'b1;
    i_en    = 1'b1;
    #2
    // 書き込みを無効化
    i_data  = 1'b1;
    i_en    = 1'b0;
    #4
    // データ0に
    i_data  = 1'b0;
    i_en    = 1'b0;
    #4
    $finish;
  end
```

なぜ2の倍数の秒数だけ待機しているのか一応説明すると、先程のクロック生成で`clk`は`#1`で0 -> 1と変化しますが、これでは半クロック分しか待機できていません。そこで`#2`とすると`clk`は0 -> 1 -> 0と変化し、1クロック待機することが可能となるためです。

#### シミュレーションを実行

シミュレーションを実行するためにいくつかコマンドを実行しました。これらのコマンドの使い方とシミュレーションの実行方法をここで説明します。

シミュレーションにおいて主に`iverilog`、`vvp`、`gtkwave`の３つのコマンドを利用します。

* iverilog : シミュレータ用のVerilogコンパイラ
* vvp : シミュレーション用バーチャルマシン(実行環境)
* gtkwave : オープンソースの波形ビューワー

これらのコマンドを使う流れとしては、書いたVerilogファイルとテストベンチを`iverilog`に与えて`a.out`を生成します。そして出力された`a.out`を`vvp`に与えて波形ファイルを生成し、生成された波形ファイルを`gtkwave`で閲覧します。

それぞれのコマンドの使い方は以下の通りです。

```bash
iverilog テストベンチファイル名 Verilogファイル名
vvp a.out
gtkwave 波形ファイル名
```

実際には以下のように使います。

```bash
iverilog test_tb.v test.v 
vvp a.out
gtkwave wave.vcd
```

テストベンチを書くのは非常に面倒で退屈ですが、Verilog HDLでディジタル回路を設計する事とテストベンチを書くことは切っても切れない関係にあります。今後何度も書くことになりますので適宜参照するようにしてください。

### Verilogの便利な文法

さて、次はより進んだ回路を作るためのVerilogの便利な文法を学んでいきましょう。

#### 定数

Verilogでもプログラミング言語と同じように、数値に別名を付けたい場合が存在します。そこで使うのが定数です。Verilogには定数として`define`と`parameter`という２つの方法が存在しますが、`define`はキモいので`parameter`を使うことを推奨します。

`parameter`は`parameter 定数名 = 数値;`のように使います。最後にセミコロン`;`を忘れずに付けてください。

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

#### スライス

Verilogにおいて、複数ビットある信号の特定のビットを取り出したい場合が多く存在します。そこで使うのが**スライス**という操作です。

スライスは`信号名[先頭ビット:末尾ビット]`か`信号名[ビット番号]`で行います。以下の例では`w_signal`という8bit幅の信号線が存在するとして、`w_slice1`という4bit幅の信号線へ6から3bit目までを取り出して入力しており、また`w_slice2`へは7bit目のみを取り出して入力しています。

```verilog
wire [7:0] w_signal;
wire [4:0] w_slice1;
wire       w_slice2;

assign w_slice1 = w_signal[6:3];
assign w_slice2 = w_signal[7];
```

このスライスはwire型とreg型両方に対して行うことが可能です。

#### 二次元配列

Verilogではreg型を用いて自由な長さのレジスタを定義することが出来ました。例えば`reg [7:0] r_line;`と宣言することで、8bit幅の`r_line`という名前のレジスタを定義できました。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/reg_line.png)
<p style="text-align:center"><b>8bit幅のレジスタ</b></p>

しかし、複数の同じ長さのレジスタを定義したい時に、`reg [7:0] r_line1;`, `reg [7:0] r_line2`, ... のように名前を変えて何度も定義する必要があり、効率的ではありません。そこで使うのが**二次元配列**です。

Verilogではレジスタの宣言の後ろに`[最後の番号:最初の番号]`を付けることで、同じ長さの複数のレジスタを定義することが可能です。以下の例では、`[3:0]`を付けており、0番から3番の計4つの8bitのレジスタが定義されています。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/reg_array.png)
<p style="text-align:center"><b>8bit幅のレジスタ x 4</b></p>

二次元のレジスタにアクセスする際は`レジスタ名[レジスタ番号][ビット番号:ビット番号]`でアクセス出来ます。今回の例では`r_array[0][1:0]`で`r_array`の0番の1bit目から0bit目にアクセスでき、`r_array[1][7]`で1番のレジスタの7bit目にアクセス出来ます。また、ビット番号を省略することも可能であり、例えば`r_array[3]`で3番のレジスタの全体にアクセスが可能です。

またwire型でも同様の事が可能です。二次元配列は非常によく使いますので覚えておきましょう。

#### if文

if文とは条件に応じて処理を変える文法であり、always文と後述するfunction文で記述可能です。

if文の構文は以下の通りです。`if(条件1) begin ~ end`とすると条件1が真の場合に`begin ~ end`で囲まれた部分が実行されます。またこの後ろに`else if(条件2) begin ~ end`を加えると条件1が偽で条件2が真の場合に`begin ~ end`で囲まれた部分が実行されます。そして`else begin ~ end`を付け加えると、どの条件にも合致しない場合に`else begin ~ end`で囲まれた部分が実行されます。

```verilog
if(条件1) begin
  処理A
end else if(条件2) begin
  処理B
end else begin
  処理C
end
```

またif文で記述する条件には**比較演算子**と呼ばれるものを使います。

| 演算子 | 例 | 意味 |
| --- | --- | --- |
| `==` | `A == B` | AとBが同じ |
| `!=` | `A != B` | AとBが異なる |
| `<=` | `A <= B` | AはB以下 |
| `>=` | `A >= B` | AはB以上 |
| `<` | `A < B` | AはB未満 |
| `>` | `A > B` | AはBより大きい |

<p style="text-align:center"><b>Verilogの比較演算子</b></p>

実際にif文を使った例が以下になります。以下の例では`i_condition`の値が`4'h1`の場合は加算が実行され、`4'hE`の場合は減算が実行され、それ以外の場合はOR演算が実行されます。

```verilog
module test_module(
  input [3:0] i_condition,
  input [7:0] i_data_a,
  input [7:0] i_data_b,
  output reg [7:0] o_data
);

  always @(posedge clk) begin
    if(i_condition == 4'h1) begin
      o_data <= i_data_a + i_data_b;
    end else if(i_condition == 4'hE) begin
      o_data <= i_data_a - i_data_b;
    end else begin
      o_data <= i_data_a | i_data_b;
    end
  end

endmodule
```

また[MUXによるD-FFの改良](#muxによるd-ffの改良)で作った回路をif文で書き直したものが以下になります。

```verilog
module AdvancedDff(
  input i_clk,
  input i_data,
  input i_en,
  output reg o_data
);
  
  always @(posedge i_clk) begin
    if(i_en == 1'b1) begin
      o_data <= i_data;
    end else begin
      o_data <= o_data;
    end
  end

endmodule
```

if文は非常によく構文ですので覚えておきましょう。

#### case文

case文は便利なif文のような構文であり、always文と後述するfunction文内で記述可能です。`case()`の中に条件分岐に利用する信号名を記述し、`case() ~ endcase`で囲んで利用します。case文の内部では`数値 : 処理;`と記述し、`case(信号名)`の信号の値が数値と一致した場合にその処理が実行されます。また`default : 処理`と記述することで、どの数値にも当てはまらない場合はその処理が実行されるようになります。この`default`を記述していない場合、処理系によってはエラーが発生します。

if文の説明で用いた回路をcase文で書き直したものが以下になります。

```verilog
module test_module(
  input [3:0] i_condition,
  input [7:0] i_data_a,
  input [7:0] i_data_b,
  output reg [7:0] o_data
);

  always @(posedge clk) begin
    case(i_condition)
      4'h1 : o_data <= i_data_a + i_data_b;
      4'hE : o_data <= i_data_a - i_data_b;
      default : o_data <= i_data_a | i_data_b;
    endcase
  end

endmodule
```

#### function文

always文では気軽にif文やcase文を使うことが出来ましたが、assign文ではそうもいきません。assign文でif文やcase文を使いたい場合はfunction文で関数を作成しましょう。

function文では`function 返り値の幅 関数名;`と記述して定義し、`function ~ endfunction`で囲まれた部分で処理を記述します。入力信号は関数の定義後に`input`で定義し、関数の返り値はfunction名に入力する形で記述します。また返り値の幅を省略すると1bitになります。

以下の例では、`test_function()`という名前の関数を新たに定義し、関数の処理結果を`o_data`に入力しています。

```verilog
module test_module(
  input [3:0] i_condition,
  input [7:0] i_data,
  output [7:0] o_data,
);

  assign o_data = test_function(i_condition, i_data);

  function [7:0] test_function;
    input [3:0]  i_ctrl; 
    input [15:0] i_data;

  begin
    case(i_ctrl)
      4'b0000 : test_function = i_data + 16'h0001;
      4'b0000 : test_function = i_data & 16'h0505;
      4'b0000 : test_function = i_data & 16'hff00;
      default : test_function = 16'h0000;
    endcase
  end
  endfunction

endmodule
```

function文もそれなりに使いますので覚えておきましょう。

#### 連結演算子

便利な演算子をもう一つ紹介します。連結演算子です。これは以下のようにビットを結合する演算子です。

```
{4'h0011, 4'b1100} -> 8'b00111100
```

この連結演算子はassign文やalways文やfunction文で利用可能です。

```verilog
module test_module(
  input [7:0] i_data_a,
  input [7:0] i_data_b,
  output reg [15:0] o_data
);

  always @(posedge clk) begin
    o_data <= {i_data_a, i_data_b};
  end

endmodule
```

#### ビットの繰り返し

`{数字{値}}`で値を数字の数だけ並べる事が可能です。

```
{10{1'b0}} -> 10'b0000000000
{10{1'b1}} -> 10'11111111111
```

### 練習問題

実際に手を動かしてVerilogに慣れましょう。

#### 練習問題１

以下の回路と同等のモジュール`problem1`を`problem1.v`という名前のファイルに作成してください。入出力は画像の通りです。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/problem1.png)

またテストベンチとテスト用のコマンドは以下のものを使ってください。

テストベンチ、これを`problem1_tb.v`というファイル名で保存する。

```verilog
module problem1_tb;
  reg [7:0] i_p0;
  reg [7:0] i_p1;
  reg [7:0] i_p2;
  wire [7:0] o_p;

  initial begin
    $dumpfile("wave.vcd");
    $dumpvars(0, DUT);
  end
    
  problem1 DUT(
    .i_p0   (i_p0   ),
    .i_p1   (i_p1   ),
    .i_p2   (i_p2   ),
    .o_p    (o_p    )
  );
    
  initial begin
    i_p0 = 8'h00;
    i_p1 = 8'h00;
    i_p2 = 8'h00;
    #2
    $display("o_p = %02x", o_p);
    i_p0 = 8'h0f;
    i_p1 = 8'h55;
    i_p2 = 8'h88;
    #2
    $display("o_p = %02x", o_p);
    i_p0 = 8'h74;
    i_p1 = 8'h81;
    i_p2 = 8'h11;
    #2
    $display("o_p = %02x", o_p);
    $finish;
  end
endmodule
```

テスト用コマンド

```bash
iverilog problem1_tb.v problem1.v
vvp a.out
```

以下の出力が得られたら正解です。

```bash
VCD info: dumpfile wave.vcd opened for output.
o_p = 00
o_p = 8d
o_p = 11
problem1_tb.v:35: $finish called at 6 (1s)
```

#### 練習問題２

以下の回路と同等のモジュール`problem2`を作成してください。入出力は画像の通りです。名前の書かれていない信号線は各自で定義してください。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/problem2.png)

またテストベンチとテスト用のコマンドは以下のものを使ってください。

テストベンチ、これを`problem2_tb.v`というファイル名で保存する。

```verilog
module problem2_tb;
  reg [7:0] i_p0;
  reg [7:0] i_p1;
  reg [7:0] i_p2;
  wire [7:0] o_p;

  initial begin
    $dumpfile("wave.vcd");
    $dumpvars(0, DUT);
  end
    
  problem2 DUT(
    .i_p0   (i_p0   ),
    .i_p1   (i_p1   ),
    .i_p2   (i_p2   ),
    .o_p    (o_p    )
  );
    
  initial begin
    i_p0 = 8'h55;
    i_p1 = 8'h77;
    i_p2 = 8'h01;
    #2
    $display("o_p = %02x", o_p);
    i_p0 = 8'hf0;
    i_p1 = 8'h55;
    i_p2 = 8'h5a;
    #2
    $display("o_p = %02x", o_p);
    i_p0 = 8'hff;
    i_p1 = 8'h88;
    i_p2 = 8'h11;
    #2
    $display("o_p = %02x", o_p);
    $finish;
  end
endmodule
```

テスト用コマンド

```bash
iverilog problem2_tb.v problem2.v
vvp a.out
```

以下の出力が得られたら正解です。

```bash
VCD info: dumpfile wave.vcd opened for output.
o_p = 56
o_p = 40
o_p = ff
problem2_tb.v:35: $finish called at 6 (1s)
```

#### 練習問題３

以下の回路と同等のモジュール`problem3`を作成してください。入出力は画像の通りです。名前の書かれていない信号線は各自で定義してください。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/problem3.png)

またテストベンチとテスト用のコマンドは以下のものを使ってください。

テストベンチ、これを`problem3_tb.v`というファイル名で保存する。

```verilog
module problem3_tb;
  reg [15:0] i_p0;
  reg [15:0] i_p1;
  wire [15:0] o_p;

  initial begin
    $dumpfile("wave.vcd");
    $dumpvars(0, DUT);
  end
    
  problem3 DUT(
    .i_p0   (i_p0   ),
    .i_p1   (i_p1   ),
    .o_p    (o_p    )
  );
    
  initial begin
    i_p0 = 16'h0f0f;
    i_p1 = 16'h0f0f;
    #2
    $display("o_p = %04x", o_p);
    i_p0 = 16'h3366;
    i_p1 = 16'h6633;
    #2
    $display("o_p = %04x", o_p);
    i_p0 = 16'h1234;
    i_p1 = 16'h5678;
    #2
    $display("o_p = %04x", o_p);
    $finish;
  end
endmodule
```

テスト用コマンド

```bash
iverilog problem3_tb.v problem3.v
vvp a.out
```

以下の出力が得られたら正解です。

```bash
VCD info: dumpfile wave.vcd opened for output.
o_p = 0000
o_p = 4411
o_p = 4448
problem3_tb.v:30: $finish called at 6 (1s)
```

#### 練習問題４

以下の回路と同等のモジュール`problem4`を`problem4.v`という名前のファイルに作成してください。以下の回路は16bitのレジスタを持っており、クロック毎に1つづ値が増えていきます。また`i_rst`はリセット信号であり、これが1になるとレジスタの値が0に戻ります。

入出力は画像の通りです。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/problem4.png)

またテストベンチとテスト用のコマンドは以下のものを使ってください。

テストベンチ、これを`problem4_tb.v`というファイル名で保存する。

```verilog
module problem4_tb;
  reg           i_clk     = 0;
  reg           i_rst     = 0;
  wire  [15:0]  o_p;

  initial begin
    $dumpfile("wave.vcd");
    $dumpvars(0, DUT);
  end

  problem4 DUT(
    .i_clk  (i_clk  ),
    .i_rst  (i_rst  ),
    .o_p    (o_p    )
  );

  always #1 begin    
    i_clk <= ~i_clk;
  end

  initial begin
    i_rst   = 1'b1;
    #10
    i_rst   = 1'b0;
    #100
    $finish;
  end
endmodule
```

テスト用コマンド

```bash
iverilog problem4_tb.v problem4.v
vvp a.out
gtkwave wave.vcd
```

波形ファイルを開き、`i_rst`の値が0になってから`o_p`の値が1づつ増えていけば正しい動作です。

#### 練習問題５

以下の回路と同等のモジュール`problem5`を`problem5.v`という名前のファイルに作成してください。以下の回路は16bitのレジスタを持っており、`i_ctrl`によって格納するデータを選択できます。

入出力は画像の通りです。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/problem5.png)

テストベンチは各自で作成してください。

#### 練習問題６

以下の回路と同等のモジュール`problem6`を`problem6.v`という名前のファイルに作成してください。以下の回路は3bitのレジスタを持っており、`i_data`の下位2bitによって格納するデータを選択できます。また`i_rst`はリセット信号であり、これが1になるとレジスタの値が0に戻ります。

入出力は画像の通りです。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/problem6.png)

テストベンチは各自で作成してください。

### 模範解答

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

#### 模範解答２

```verilog
module problem2(
  input       [7:0] i_p0,
  input       [7:0] i_p1,
  input       [7:0] i_p2,
  output wire [7:0] o_p
);

  wire [7:0] w_p;

  assign w_p = i_p1 & i_p2;
  assign o_p = i_p0 + w_p;

endmodule
```

#### 模範解答３

```verilog
module problem3(
  input       [15:0] i_p0,
  input       [15:0] i_p1,
  output wire [15:0] o_p
);

  wire [15:0] w_p;

  assign w_p = ~i_p0;
  assign o_p = w_p & i_p1;

endmodule
```

#### 模範解答４

```verilog
module problem4(
  input             i_clk,
  input             i_rst,
  output reg [15:0] o_p
);

  always @(posedge i_clk) begin
    if(i_rst) begin
      o_p   <= 16'h0000;
    end else begin
      o_p   <= o_p + 16'h0001;
    end
  end

endmodule
```

#### 模範解答５

```verilog
module problem5(
  input         i_clk,
  input [15:0]  i_data_0,
  input [15:0]  i_data_1,
  input [15:0]  i_data_2,
  input [15:0]  i_data_3,
  input [1:0]   i_ctrl,
  output reg [15:0] o_data
);

  always @(posedge i_clk) begin
    case(i_ctrl)
      2'b00 : o_data <= i_data_0;
      2'b01 : o_data <= i_data_1;
      2'b10 : o_data <= i_data_2;
      2'b11 : o_data <= i_data_3;
    endcase
  end

endmodule
```

#### 模範解答６

```verilog
module problem6(
  input             i_clk,
  input             i_rst,
  input      [15:0] i_data,
  output reg [2:0]  o_data
);

  always @(posedge i_clk) begin
    if(i_rst) begin
      o_data <= 3'b000;
    end else begin
      case(i_data[1:0])
        2'b00 : o_data <= i_data[4:2];
        2'b01 : o_data <= i_data[7:5];
        2'b10 : o_data <= i_data[10:8];
        2'b11 : o_data <= i_data[13:11];
      endcase
    end
  end

endmodule
```


以上でディジタル回路の基礎とVerilogの基礎は終了です。これでCPUを作る為の技能を身につける事が出来ました。お疲れ様でした。

次はVerilogで書いた回路を実機で動かしてみるか、一番下のコンピュータアーキテクチャ入門へ進みましょう。

## 実機向け：FPGA入門

本章ではVerilog HDLで書いたディジタル回路を実機で動かす方法を学んでいきます。自分で設計した回路が現実世界で動くというのはプログラミングとはまた違った喜びを味わえます。開発用ボードを積んでる変態を除けば、実際にボードを買い、動かすというのは非常にハードルが高く感じるかもしれません。一度買ってしまえば慣れます。ブレーキを外していきましょう。

### FPGAボードを購入する

ではFPGAボードを購入しましょう。本記事では**Tang Nano 9K**というFPGAボードを使って開発を進めて行きます。Tang Nano 9kはGowin社のFPGAが搭載されているSipeed社が開発したFPGAボードです。FPGAボードは基本的に余裕で1万円を越しますが、このTang Nano 9kは非常に安価なFPGAボードであり、その値段はなんと2,500円です。チャイナパワーを感じますね。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/tangnano_board.png)
<center>
Tang Nano 9K
<br>
<br>
</center>

このTang Nano 9kを秋月電子通商で購入してください。秋葉に買いに行ってもいいですしネット通販でも大丈夫です。ネット通販の場合は秋月のページに行ってTang Nano 9kを検索してください。通販コードはM-17448です。

送料が500円くらい掛かるのでまとめて買うといいです。

[https://akizukidenshi.com/catalog/default.aspx](https://akizukidenshi.com/catalog/default.aspx)

購入しましたらワクワクしながら待ちましょう。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/akitsuki_tangnano.png)
<center>
筆者が店舗で買った際の画像
<br>
<br>
</center>

### 開発環境構築

次にTang Nano 9kで開発を行うためのソフトウェアをインストールしていきましょう。

まずは以下のGowin社のサイトに飛び、アカウントを作成してください。知らんサイトで知らんアカウントを作るのは抵抗があるかもしれませんが、諦めてください。

[https://www.gowinsemi.com/ja/support/home/](https://www.gowinsemi.com/ja/support/home/)

アカウントを作成しましたら、「Gowin® EDAのダウンロード」からGowin EDAのEducation Editionをダウンロードします。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/gowin_download.png)

そしてダウンロードしたファイルを解凍して、中のインストーラを実行してください。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/gowin_doubleclick.png)

インストーラを実行するとなんかスゲー怖い画面が出ます。臆せず`詳細情報`から`実行`をクリックしましょう。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/gowin_defender1.png)
![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/gowin_defender2.png)

インストールが開始した後は適当にNextなりYesなりポチポチしてけばいいです。たまに以下のようなスゲー怖い画面が出ますが`はい`を押しとけばいいです。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/gowin_defender3.png)

インストールが成功したらデスクトップにGowin EDAのアイコンが出来ます。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/gowin_icon.png)

### プロジェクト作成

Gowin EDAのインストールが完了したので、次はプロジェクト作成します。Gowin EDAを起動し、New Projectをクリックしてください。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/gowin_startpage.png)

そうしましたら、`Projects > FPGA Design Project`を選択し、OKをクリックします。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/gowin_makeproject.png)

次にプロジェクトの名前を入力します。Nameにプロジェクト名、Create inではプロジェクトを作成する場所を選択します。今回は`tangnano_test`という名前のプロジェクトをデスクトップに作成しましょう。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/gowin_makeprojectname.png)

プロジェクトの名前を決定したら、次はプロジェクトで使うFPGAを選択します。

Tang Nano 9Kに載っているFPGAをよく見ると、チップの上に`GW1NR-LV9QN88PC6/I5`と書いてあります。これがFPGAの型番です。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/tangnano_fpga.png)

よってSelect DeviceではSeriesを`GW1NR`にし、下に出てくる`GW1NR-LV9QN88PC6/I5`を選択してからNextをクリックしてください。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/gowin_selectfpga.png)

全ての設定が完了したのを確認してからFinishをクリックしてください。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/gowin_selectfpgasummary.png)

これでプロジェクトが作成されました。

### ディジタル回路を書く

まずはプロジェクト内にVerilogファイルを作成します。

画像のように`Design`のプロジェクト名の部分を右クリックし、`New File..`をクリックします。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/gowin_newfile.png)

出てきたウィンドウの`Verilog File`を選択してOKを押します。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/gowin_select_verilog.png)

そしてファイル名として今回は`test_module`と入力し、OKをクリックしてください。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/gowin_make_verilog.png)

そうしましたらVerilogを書く画面が出てきますので、ここではボタンを押すとLEDが光る簡単な回路を作成してみましょう。インターフェイスとしてボタン入力の`i_button`、LED出力の`o_led`を定義し、assign文で`i_button`の入力を`o_led`に出力します。

```verilog
module test_module(
  input wire i_button,
  output wire o_led
);

  assign o_led = i_button;

endmodule
```

これを書いた状態が以下の画像です。書き終わりましたら、必ず`Ctrl + S`を押して上書き保存を行ってください。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/gowin_write_verilog.png)


### 論理合成

Verilogが書き終わりましたら、**論理合成**を実行します。論理合成とはHDLをディジタル回路に変換する処理の事でしたね。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/learn_lsi/synthesis.png)

この論理合成、EDAではボタン一つで行うことが可能です。以下の画像のように右から３番目のボタンを押すことで、論理合成が開始されます。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/gowin_run_synth.png)

論理合成が完了したら、下の`Console`にGowinSynthesis Finishと表示されます。また書いたVerilogに記述ミスがある場合はここで赤文字で表示されます。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/gowin_synth_finish.png)

ちなみに`Tool > Schematic Viewer > RTL Design Viewer`を押すと論理合成で生成された回路図を見ることが可能です。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/gowin_rtl_schematic.png)

今回は`i_button`と`o_led`を直結させているだけですので、非常にシンプルな回路図となっています。

### ピンアサイン

論理合成が完了しましたら、次は**ピンアサイン**を行います。ピンアサインとは、FPGAの入出力ピンをVerilogで定義したモジュールの入出力に割り当てる作業の事を指します。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/tangnano_fpga.png)
<center>
チップの周りにピンが並んでいる
<br>
<br>
</center>

Gowin EDAでピンアサインを行うには、`Process`から`FloorPlanner`をクリックします。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/gowin_run_florplan.png)

そしたらピンアサインを行うためにツールである`FloorPlanner`が起動します。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/gowin_pin_assigning.png)

FloorPlannerの下にあるタブで`I/O Constraints`を選択してください。すると表が出てきます。

この`I/O Constraints`の表で見るべき部分は`Port`と`Location`２つです。`Port`にはVerilogで書いたモジュールの入出力ポートが書かれており、`Location`に数字を入力することでその行の入出力ポートにピンを割り当てることが可能です。

今回はボタンとLEDの割り当てが必要です。Tang Nano 9Kにおいて、ボタンはピン3に繋がっていますので、`i_button`には`3`を割り当て、またLEDはピン１０に繋がっていますので`o_led`には`10`を割り当てます。

`Location`への入力が完了したら、必ず`Ctrl + S`を押して変更を保存してください。

Tang Nano 9KにはボタンやLEDの他にも、LCDやUARTなど様々なピンが存在しています。詳しくはSiPEED社のページにある`Tang_Nano_9k_3672_Schematic.pdf`を参照してください。

[https://dl.sipeed.com/shareURL/TANG/Nano%209K/2_Schematic](https://dl.sipeed.com/shareURL/TANG/Nano%209K/2_Schematic)

また以下の表に本記事で使うピンと、その番号を載せてきます。

| ピン | ピン番号 |
| ---- | -------- |
| クロック(27MHz) | `52` |
| ボタン(S1) | `4` |
| ボタン(S2) | `3` |
| LED0 | `10` |
| LED1 | `11` |
| LED2 | `13` |
| LED3 | `14` |
| LED4 | `15` |
| LED5 | `16` |

<center>
Tang Nano 9Kのピンとピン番号(一部)
<br>
<br>
</center>

ピンアサインが完了したら、`FloorPlanner`は閉じて構いません。

### 配置配線

ピンアサインの次は配置配線を行います。配置配線とは加算器やAND回路、FFといった回路をFPGA内に配置し、それらを繋ぎ合わせる配線を行う処理です。

Gowin EDAで配置配線を行うためには上にある、右から二番目の`Run Place & Route`というボタンをクリックします。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/gowin_run_pnr.png)

配置配線が成功すると、下の`Console`にBitstream generation completedと出力されます。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/gowin_pnr_finish.png)

この配置配線は非常に重い処理です。今回は簡単な回路ですので短い時間で完了しましたが、複雑な回路を作ると30分以上掛かることも珍しくありません。

### 書き込み

配置配線が完了した次はFPGAに回路を書き込みましょう。ここはWindows限定です。Linuxの方はOpenFPGALoarderを使ってください。

まずはUSBケーブルでTang Nano 9KとPCを接続してください。その後、上のボタンから`Programmer`を起動します。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/gowin_programmer_run.png)

正常に起動すれば以下の画像のようにCable Settingのウィンドウが開きますので、Saveをクリックします。

たまにNo USB Cable Connectionと出る場合があります。その場合はとりあえずOKを押して、`Query/Detect Cable`を押してください。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/gowin_programmer_start.png)

そして`Program/Configure`をクリックするとFPGAへ書き込みが開始されます。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/gowin_programmer_write.png)

書き込み完了後、Tang Nano 9KのS2ボタンを押すとLEDが光ります。回路が正常に書き込めていますね。

## コンピュータアーキテクチャ入門へ

これで**ディジタル回路とVerilog入門**は終わりです。次からは今まで学んだ技術を使い、CPUを自作していきます。

<center>
<a href="/Computer_Architecture.html#コンピュータアーキテクチャ入門"><img src="https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/Computer_Architecture.png" style="border-radius: 30px;"></a>
<a href="/Computer_Architecture.html#コンピュータアーキテクチャ入門"> VLSI.JP/Computer_Architecture.html </a>
</center>

---

Copyright (C) Cra2yPierr0t, ALL RIGHTS RESERVED
