---
layout: default
title: 無から始める自作CPU
image: "https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/letsmakecpu.png"
---

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/letsmakecpu.png)

# 無から始める自作CPU

**CPUは作れる！！！！！！！！ご存知でしたか！！！？？？？？？**

魔法の石、究極のブラックボックス、人類の英知たるCPUは不世出の天才にしか作れないとお思いですか？ **違います**！適切な知識さえあれば割と凡庸な人間でも作ることが出来ます！！！作りましょう！！

この記事は半導体やコンピュータサイエンスやプログラミングの知識が無くても、CPUを自作出来るレベルまでお前を持っていく事を目的としています。やっていきましょう。

## 基礎知識

ここではCPUを作るのに必要な知識を説明します。いくつかは既にご存知かもしれません、もしくはどれも初めて見る単語かもしれません。ここで全て覚えろとは言いません。とりあえずざっと読んで必要になったら読み返してください。

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

ですが考えてみてください、そのプログラム、一体どこで動いているのでしょうか？説明できますか？ その答えが**CPU**です。

CPUとはCentral Processing Unit、中央演算装置の略で、石みたいなガラスみたいな物質で出来ています。見た目は大体こんな感じです。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/cpu_image.jpeg)

この世に存在するプログラムは全てCPUの上で動いています。このCPUが無ければプログラムは動かせませんし、いくらプログラムを書いても意味がありません。我々の生活基盤はこの石に支えられているという訳ですね、ありがとうCPU。消えないでCPU。本記事ではそんなCPUを作ります。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/world_of_cpu.png)

### ディジタル回路

CPUは**ディジタル回路**というもので構成されています。ディジタル回路、難しそうな単語ですね。安心してください、ディジタル回路は単純なルールの集まりに過ぎません。少しずつ学んでいきましょう。

#### ディジタル回路が扱う値

ディジタル**回路**と呼ぶからには、電気的な何かを扱う物体だと思われるかもしれません。正解です。ディジタル回路は電圧の高低を扱う回路です。具体的には0Vと5Vや、0Vと1.2Vなどを入力に取りますが、実際に何Vなのかは回路によってまちまちなので入力をHigh, Lowとするか、それすら見づらいし面倒なので電圧の高い入力を1、電圧の低い入力を0とする表記が一般的です。

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

次は**ORゲート**です。これは入力の論理和、入力の少なくともどちらか一方が1なら1を出力します。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/or.png)

真理値表は以下の通りです。

| `A` | `B` | `A OR B` |
| - | - | ------- |
| `0` | `0` | `0` |
| `0` | `1` | `1` |
| `1` | `0` | `1` |
| `1` | `1` | `1` |

記号としては`a | b`が使われがちです。

#### AND

次は**ANDゲート**です。これは入力の論理積、入力の両方が1なら1を出力します。

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

次は**NANDゲート**、これはANDゲートの出力にNOTしたものです、入力の少なくともどちらか一方が0なら1を出力します。それ以外なら1を出力します。

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

論理回路ラスト、**XORゲート**です。これは**排他的論理和**と呼び、入力が異なる場合に1を出力し、同じ場合は0を出力します。

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

ちなみに上記の３つ以外でCPUを作っている人を見かけたらそいつは異常な変態ですので近づきましょう。

ロジックICは秋葉で以下の黒いムカデみたいな奴を沢山買ってきて無限に半田付けするやつですね。

![](https://2.bp.blogspot.com/-khfR1UvFhbw/XDXbZILRDOI/AAAAAAABQ_k/FVT0GCYVkV4PgTySWyA63UIkk29pP0DkwCLcBGAs/s800/computer_ic_syusekikairo_long.png)

楽しいですが骨が折れるので今回はパス。

次のLSIですが、これは工場に大金を払って生産してもらう奴ですね。そんな金は無いので今回はパス。

![](https://user-images.githubusercontent.com/48832611/205153031-033603ce-ccac-4f02-b91a-920f15859ff1.png)

↑ Googleの金を使った例([詳細](https://vlsi.jp/OpenMPW.html))

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

開発は**シミュレーション**と**実機検証**に分かれています。基本的に書いたコードの動作確認は上段のシミュレーションで行い、現実世界での動作の確認は下段の実機検証の流れで行います。

シミュレーションにおいて、回路を記述したVerilog HDLファイルとテストベンチを書き、それをシミュレータ上で動かし動作を検証します。実機検証では、利用するソフトウェアはEDA(Electronic Design Automation)ツールのみです。シミュレーションで検証したVerilog HDLファイルをEDAに渡し、EDAでビルドを行いFPGA等の実機で動作を確認します。

本書の大半の作業はシミュレーションで行いますので基本はPCを触ってればいいです。実機検証に関しては環境構築も作業も非常に煩雑であるため、本記事の後半で解説します。

### 開発環境構築

初めに開発環境を構築していきましょう。大半の初心者は環境構築で躓く事が経験的に知られています。心を強く持ちましょう。

以下ではWindows環境向けに解説を進めていきますが、LinuxやMacをお使いの方でも同名のソフトウェアをインストールして頂ければ本記事の内容を進めることが出来ます。

#### テキストエディタのインストール

テキストエディタとは文字を書く為のソフトウェアです。メモ帳でも自作CPUを出来なくはないですが、メモ帳は使い辛いのでVisual Studio Codeをインストールしましょう。もしVSCode以外にお気に入りのテキストエディタがありましたら(vimとか)、そのままお使いいただいて構いません。

このページ([https://code.visualstudio.com](https://code.visualstudio.com))からダウンロードを行ってください。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/vscode_download.png)

[https://code.visualstudio.com](https://code.visualstudio.com)

#### Verilog HDLシミュレータのインストール

本記事ではシミュレータとしてIcarus Verilogを利用します。

このページ([http://bleyer.org/icarus/](http://bleyer.org/icarus/))の**Downloadの一番上のリンク**からダウンロードしてください。v12がv13とかv14とかになってるかもしれませんが、とりあえず**Downloadの一番上のリンク**です。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/verilog_download.png)

[http://bleyer.org/icarus/](http://bleyer.org/icarus/)

またインストール時の注意点として**１つだけ絶対に押すボタンが存在します。** 以下の画像のSelect Additional TasksのAdd executable folder(s) to the user PATHです。**絶対にチェックを入れてください。** それ以外は適当にNextなりFinishなりポチポチ押してけばいいです。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/verilog_install.png)

インストールが終わりましたら、正しくインストール出来ているか確かめるためにソフトウェアの検索欄に`cmd`と入力してコマンドプロンプトを起動してください。起動方法をご存知ならPowerShellでもターミナルでも構いません。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/run_cmd.png)

そして`iverlog`コマンドと`vvp`コマンドを実行してください。以下のような表示が出たら成功です。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/check_iverilog.png)

また`gtkwave`コマンドを実行してください。何かウィンドウが出たら成功です。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/check_gtkwave.png)

### 開発環境に慣れる

環境構築が完了しましたら、次は実際に開発環境を触って開発の流れを体感しましょう。

まずは`verilog_test`という名前で新たにフォルダを作成してください。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/make_folder.png)

#### Verilogファイルとテストベンチ

そしてVSCodeを開き、`New File..`から先程作成したフォルダに`adder8.v`という名前のファイルを作成します。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/vscode_newfile.png)
![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/vscode_newfile_adder8.png)

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

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/vscode_write_adder8.png)

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

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/vscode_write_adder8_tb.png)

さて、今しがた`adder8.v`と`adder8_tb.v`というファイルを作りました。`adder8.v`はVerilog HDLで書いたディジタル回路です。また`adder8_tb.v`というファイルも書きましたね、これは**テストベンチ**というものです。Verilogをシミュレータ上で動かす為には、ディジタル回路を記述したVerilogファイルの他に、**テストベンチ**と呼ばれるVerilogファイルが必要です。

テストベンチとは文字通り回路のテストを行うためのVerilogファイルです。例えば、あなたが何か回路を作成したとして、その回路が意図した通りに動作するか確かめたい場合、回路に入力を与え、その出力を調べる必要がありますね？テストベンチはそういった回路へ入力を与えたり、出力を確認したり、内部信号を検査したり、回路内の波形ファイルを出力したりするのに使います。

具体的なテストベンチの書き方は後々解説しますので、今はとりあえず手元でシミュレーションを動かしてみましょう。

#### シミュレーションを実行

回路とテストベンチが書き終わりましたら、作成したファイルがあるフォルダを開き、白い部分をShiftを押しながら右クリックをして「ターミナルで開く」をクリックしてください。これで`verilog_test`フォルダ内でターミナルを開くことができます。起動したターミナルで`ls`コマンドを実行すると、フォルダ内のファイルの一覧を見ることが可能です。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/open_terminal.png)

以下のコマンドを実行してください。

```bash
iverilog adder8_tb.v adder8.v
vvp a.out
```

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/run_iverilog.png)

実行が完了したらフォルダ内に`wave.vcd`というファイルが生成されている筈です。これは波形ファイルです。波形ファイルは以下のコマンドで開きます。

```bash
gtkwave wave.vcd
```

このコマンドでGTKwaveが起動します。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/run_gtkwave.png)

GTKWaveでは信号名をダブルクリックか、信号名を選択してAppendボタンを押すと波形を見ることができます。適当に触ってみて気合で慣れてください。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/use_gtkwave.png)

以上のVerilog HDLでディジタル回路とテストベンチを記述し、シミュレーションを実行。波形を見てデバッグが開発の流れになります。この一連の流れは今後何度も繰り返します。今全てを覚える必要はありませんので大丈夫です、そのうち手癖で開発を回せるようになります。

1. Verilog HDLで回路を書く
2. Verilog HDLでテストベンチを書く
3. シミュレーションを実行
4. 波形を見る

### Verilogの文法を学ぼう(基礎１)

では本格的にVerilog HDLを学んでいきましょう。最初は必須の構文です。

#### module

Verilog HDLではシステム全体をモジュールという単位で分割して構成します。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/module.png)

モジュールのイメージ

Verilogでモジュールを作成するには`module`というキーワードを用います。例えば`test_module`という名前のモジュールを作成したい場合は、`module test_module();`と記述し、`module`の後ろにモジュール名を記述します。そして回路の内容を`module モジュール名();` ~ `endmodule`の間に記述します。

```verilog
module test_module();
 // ここに回路を記述
endmodule
```

#### 入出力ポート

回路には入出力が必要です、回路の入出力ポートは`input`, `output`というキーワードで定義します。入力信号名は`input`の後ろに宣言し、出力信号名は`output`の後ろに宣言します。IOの個数や順番に決まりはありませんが、最後の宣言にはカンマが必要ないことに注意しましょう。

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

回路の上には配線がありましたね、あの配線は信号を伝達する信号線です。Verilogにおいて、`wire`というキーワードを用いることで信号線を定義する事が出来ます。

`wire`の使い方は以下のように、`wire 信号幅 信号線名;`の順で記述します。

```verilog
wire [31:0] 信号線名;
```

`wire`の後ろにある`[31:0]`は信号線の幅です。この場合は32本の幅を持った信号線になります。`[15:0]`にすれば16本の幅を持った信号線になりますし、`[3:0]`にすると4本の幅を持った信号線になります。

また、以下のように信号線の幅の記述を省略すると1本の幅を持った信号線になります。

```verilog
wire 信号線名;
```

#### 演算子

Verilogでは信号線同士で演算を行うことが可能です。Verilogには様々な演算子が存在し、これらの演算子は複数ビットを纏めて演算が可能です。以下によく使う演算子を載せておきますので適宜参照してください。

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

#### assign文

`wire`で作られた信号線にはキーワード`assign`を用いて入力できます。`assign`を用いて書かれた回路は、入力が変化すると即座に出力が変化する**組み合わせ回路**となります。

この`assign`を用いた信号の入力時に、演算子を用いて演算を行うことが可能です。以下の例では`test_wire0`と`test_wire1`の信号を加算し、`test_output`に入力しています。

```verilog
module test_module (
  input [15:0] test_input,
  output [7:0] test_output
);
    
  wire [7:0] test_wire0;
  wire [7:0]  test_wire1;

  assign test_wire0 = 16'h0505;
  assign test_wire1 = test_input[7:0];

  assign test_output = test_wire0 + test_wire1;
endmodule
```

また`test_wire1`に入力している`test_input`の直後に書いてある`[7:0]`は**スライス**という操作であり、`test_input`の7ビット目から0ビット目の計8ビットを抜き出して、`test_wire1`に入力しています。スライスはよく使う操作ですので覚えておきましょう。

### Verilogの文法を学ぼう(練習１)

覚えてばかりもつまらないでしょうし、ここで少し手を動かしてみましょう。

#### 練習問題１

以下の回路と同等のモジュール`problem1`を`problem1.v`という名前のファイルに作成してください。入出力は画像の通りです。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/problem1.png)

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

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/problem2.png)

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

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/problem3.png)

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
o_p = ffff
o_p = eebb
o_p = fffb
problem3_tb.v:30: $finish called at 6 (1s)
```

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
  assign o_p = w_p | i_p1;

endmodule
```

お疲れ様です。

### Verilogのシミュレーションを学ぶ

練習問題では特に説明なしにテストベンチを出しました。このタイミングでテストベンチの書き方を説明します。

以下のテストベンチのコードを例にテストベンチ用のVerilogの構文を学んでいきましょう。

```verilog
module test_sim;
  reg           clk     = 0;
  reg   [7:0]   data_i  = 8'h00;
  wire  [7:0]   data_o;

  initial begin
    $dumpfile("wave.vcd");  // 波形ファイル
    $dumpvars(0, DUT);      // DUTインスタンスの全信号を出力
  end

  test_module DUT(
    .clk        (clk        ), 
    .input_data (data_i     ),
    .output_data(data_o     )
  );

  always #1 begin    // クロックの生成 1秒毎に反転
    clk <= ~clk;
  end

  initial begin
    data_i  = 8'h11;
    #2     // 2秒待機
    data_i  = 8'h20;
    #2
    data_i  = 8'h30;
    #10    // 10秒待機
    $finish;    // 終了
  end
endmodule
```

#### 入出力用信号の定義

作成した回路の動作を検証するためには、回路に様々な信号を入力し、出力を検査する必要があります。テストベンチにおいて回路に入力する信号は`reg ビット幅 信号名;`で定義します。また回路からの出力を受ける信号は`wire ビット幅 信号名;`で定義します。両方ともビット幅を省略した場合は1bitの幅になります。

実際に入出力用信号を定義した例が以下になります。この場合、`clk`、`data_i`が入力用の信号で、`data_o`が出力を受ける信号になります。

```verilog
  reg           clk     = 0;
  reg   [7:0]   data_i  = 8'h00;
  wire  [7:0]   data_o;
```

#### テスト用回路の生成

テストベンチで回路をテストするためにはテストベンチ内で回路を生成する必要があります。そこで行うのがテスト用回路の生成です。

Verilogにおいて、回路は`モジュール名 インスタンス名`で生成する事が可能です。以下の例では`test_module`という名前のモジュールから`DUT`という名前の回路(インスタンス)を生成しています。
また入出力信号は`.インターフェイス名(信号線名)`で接続します。以下の例では`test_module`が`clk`, `input_data`, `output_data`というインターフェイスを持っているとして、先程定義した入出力信号をそれぞれのインターフェイスに接続しています。

カンマの有無に注意してください。

```verilog
  test_module DUT(
    .clk        (clk        ), 
    .input_data (data_i     ),
    .output_data(data_o     )
  );
```

ちなみにDUTはDevice Under Testの略です。

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
    $dumpfile("wave.vcd");  // 波形ファイル
    $dumpvars(0, DUT);      // DUTインスタンスの全信号を出力
  end
```

この記述は回路の生成の前に記述してください。

#### クロックの生成

基礎知識のD-FFの部分で説明しましたが、回路が情報を記録するためには**クロック**という信号がしばしば必要になります。先程の練習問題ではクロックを用いませんでしたが、今後は殆どの場合においてクロックを必要とする回路を作ることになりますので、今のうちにテストベンチでクロックを生成する方法を学んでおきましょう。

テストベンチにおいてはクロック信号の生成に以下の記述を用います。以下の記述では先程定義した入力用信号である`clk`の値を1秒毎にNOT演算子で反転し、再び`clk`に入力しています。この結果、`clk`の値は0, 1, 0, 1, 0, 1と振動するようになります。これをクロック信号として利用しています。

```verilog
  always #1 begin
    clk <= ~clk;
  end
```

#### 入力信号を制御

テスト対象の回路に対してずっと同じ信号を入力するのはあまり効果的なテストとは言えません。そこで時間に応じて入力信号を変化させる必要があります。そのために以下の記述を用います。

以下の`initial begin ~ end`で囲まれた記述では、入力信号の`data_i`に値を格納してから`#2`という記述をしています。この`#`は遅延と呼び、`#数値`で指定した秒数だけ待機する事が出来ます。待っている間にもクロックは動作します。つまり以下の例では回路に`8'h11`という値を入力して2秒待機しています。

その後何回か信号を変化させ、最終的にシステムタスクの`$finish`が実行されます。この`$finish`が実行されるとシミュレーションが終了します。テストベンチには`$finish`を必ず記述する必要があります。

```verilog
  initial begin
    data_i  = 8'h11;
    #2     // 2秒待機
    data_i  = 8'h20;
    #2
    data_i  = 8'h30;
    #10    // 10秒待機
    $finish;    // 終了
  end
```

なぜ2秒待機しているのか一応説明すると、先程のクロック生成で`clk`は`#1`で0 -> 1と変化しますが、これでは半クロック分しか待機できていません。そこで`#2`とすると`clk`は0 -> 1 -> 0と変化し、1クロック待機することが可能となるためです。

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

### Verilogの文法を学ぼう(基礎２)

さて、少し進んだ回路を作るための文法を学んでいきましょう。

#### 定数

Verilogでもプログラミング言語と同じように、数値に別名を付けたい場合が存在します。そこで使うのが定数です。Verilogには定数として`define`と`parameter`という２つの方法が存在しますが、`define`はキモいので`parameter`を使うことを推奨します。

`parameter`は`parameter 定数名 = 数値;`のように使います。

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

`wire`は信号線を作成するためのキーワードでしたが、信号線はデータを保持する事が出来ません。基礎知識でも書きましたが、ディジタル回路がデータを保持するためには**フリップフロップ**(以後レジスタ)が必要でしたね。

Verilogでは`reg`というキーワードでレジスタを生成することが可能です。`reg`の使い方は以下のように。`reg サイズ レジスタ名;`の順で記述します。

```verilog
reg [31:0] test_32reg;
```

また以下のようにサイズの記述を省略すると1bitの幅を持ったレジスタになります。

```verilog
reg test_1reg;
```

#### always文

`reg`で作られたレジスタにはalways文というものを用いて信号を入力することが可能です。

基礎知識で述べたように、情報の記録とはクロックという信号に密接に関係してします。そこでalways文は殆どの場合においてクロック信号と一緒に用います。

always文は`always @(posedge クロック信号) begin ~ end`または`always @(negedge クロック信号) begin ~ end`という文法で記述し、`begin ~ end`の間に回路を記述します。`posedge`の場合はクロック信号が立ち上がりエッジのタイミングで処理が実行され、`negedge`の場合はクロック信号の立ち下がりのタイミングで処理が実行されます。

以下の例ではクロック信号として`i_clk`を定義しており、`i_clk`の立ち上がりで`i_data_a`と`i_data_b`の加算結果がレジスタ`r_data`に格納されるようにしています。そして`r_data`の値を`o_data`に出力しています。

ここで注目してほしいのがalways文内の`<=`です。Verilogではalways文内において、代入演算子として`=`と`<=`を使うことが可能であり、それぞれ異なった動作をします。`<=`はノンブロッキング代入と呼ばれ、`=`はブロッキング代入と呼ばれています。なんか２種類ありますが混在させるとバグの温床になりますので、本記事では`<=`のノンブロッキング代入しか用いません。always文で代入は`<=`です！そういう事にしましょう。

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

#### if文

if文とは条件に応じて処理を変える文法であり、always文と後述するfunction文で記述可能です。

if文の構文は以下の通りです。`if(条件1) begin ~ end`とすると条件1が真の場合に`begin ~ end`で囲まれた部分が実行されます。またこの後ろに`else if(条件2) begin ~ end`を加えると条件1が偽で条件2が真の場合に、`begin ~ end`で囲まれた部分が実行されます。そして`else begin ~ end`を付け加えると、どの条件にも合致しない場合に`else begin ~ end`で囲まれた部分が実行されます。

```verilog
if(条件1) begin
  処理
end else if(条件2) begin
  処理
end else begin
  処理
end
```

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

function文では`function()`~`endfunction`で処理の内容を囲み、`function 返り値のビット幅 関数名(input ビット幅 入力信号)`と記述して定義します。そして関数の返り値はfunction名に入力する形で記述します。

以下の例では、`test_function()`という名前の関数を新たに定義し、関数の処理結果を`o_data`に入力しています。

```verilog
module test_module(
  input [3:0] i_condition,
  input [7:0] i_data,
  output [7:0] o_data
);

  assign o_data = test_function(i_condition, i_data);

  function [7:0] test_function(
    input [3:0]  i_ctrl, 
    input [15:0] i_data
  );
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

#### 三項演算子

Verilogには`+`や`&`など様々な演算子が存在してますが、その中でもある便利な演算子を紹介します。三項演算子です。

三項演算子は`?`と`:`の２つを使う演算子で、一行で書けるif文のような演算子であり、以下のような演算をします。

```
代入先 = (条件) ? (条件が真の時に代入する値) : (条件が偽の時に代入する値)
```

これは演算子ですのでassign文でもalways文でもfunction文でも使用可能です。

以下の例では`i_condition`の値が`2'b11`の場合は`o_data`に`i_data_a`の値が格納され、それ以外の場合は`i_data_b`が格納されます。

```verilog
module test_module(
  input [1:0] i_condition,
  input [7:0] i_data_a,
  input [7:0] i_data_b,
  output reg [7:0] o_data
);

  always @(posedge clk) begin
    o_data <= (i_condition == 2'b11) ? i_data_a : i_data_b ;
  end

endmodule
```

#### 連結演算子

三項演算子に続いて便利な演算子をもう一つ紹介します。連結演算子です。これは以下のようにビットを結合する演算子です。

```
{4'h0011, 4'b1100} -> 8'b00111100
```

この連結演算子も演算子ですのでassign文やalways文やfunction文で利用可能です。

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

#### モジュール呼び出し

Verilogを用いて作成したモジュールは`モジュール名 インスタンス名`で利用する事が可能です。またインターフェイスは`.インターフェイス名(信号線名)`で接続します。

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

### Verilogの文法を学ぼう(練習２)


## 実機向け：FPGA入門
### 論理合成
### ピンアサイン
### 書き込み

## 自作CPU入門

さあ始めましょう

### プログラムが動く流れ

CPUはプログラムをどのように実行しているのでしょうか？

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/prog_flow.png)

CPUはプログラムをそのまま実行している訳ではありません。

現代では、プログラムは**コンパイラ**というソフトウェアによって**アセンブリ**という中間言語に変換され、そのアセンブリは**アセンブラ**というソフトウェアによって`011010101...`のようなバイナリに変換されます。
CPUとってプログラムはこのバイナリになって初めて実行可能な形式になるというわけです。

- プログラム
![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/helloworld.png)
- アセンブリ
![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/assembly.png)
- バイナリ
![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/binary.png)


### 命令セットアーキテクチャ 

先程、プログラムはバイナリに変換されて初めてCPUが実行可能な形になると言いましたね、プログラミング言語がこの世に沢山あるのと同じように、実は変換後のバイナリもこの世にはいくつか種類が存在しています。このバイナリの種類の事を命令セットアーキテクチャ(Instruction Set Architecture)、**ISA**と呼びます。基本的に、CPUはISAが異なるバイナリを実行できません。例えばMacのソフトウェアはMacのCPU以外では動きませんし、PCゲームのファイルをスマホに突っ込んでも動きませんね、動かそうとするな。

具体的なISAとしては、x86_64, AArch64, RISC-Vなどが存在しています。

本記事では筆者が定義した独自のISA、**Z16**のCPUを作ります。Z16のCPUを作る前に一度、本項でZ16のアセンブリに親しみを持っておきましょう。

#### Z16の概要

Z16は筆者が独自に定義したISAであり、16-bitの固定長命令セットです。16個の16-bitレジスタと16個の命令を持ちます。酒に酔いながら設計しました。

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

加算命令。レジスタ同士の足し算を行う命令であり、`ADD RS2 RS1 RD`と記述し、RS2とRS1の値を加算してRDに格納する。オペコードは`4'h0`。例として`ADD G3 G2 G1`は`16'h7650`となる。また、RS2にゼロレジスタZRを指定することで、RS1からRDに値を移動させるだけの命令として使うことが可能です。

$$
\text{RS2} + \text{RS1} \rightarrow \text{RD}
$$

###### SUB

減算命令。レジスタ同士の引き算を行う命令であり、`SUB RS2 RS1 RD`と記述し、RS2からRS1の値を減算してRDに格納する。オペコードは`4'h1`。例として`SUB G3 G2 G1`は`16'h7651`となる。

$$
\text{RS2} - \text{RS1} \rightarrow \text{RD}
$$

###### MUL

乗算命令。レジスタ同士の掛け算を行う命令であり、`MUL RS2 RS1 RD`と記述し、RS2とRS1の値を乗算してRDに格納する。オペコードは`4'h2`。例として`MUL G3 G2 G1`は`16'h7652`となる。

$$
\text{RS2}\times \text{RS1} \rightarrow \text{RD}
$$

###### DIV

除算命令。レジスタ同士の割り算を行う命令であり、`DIV RS2 RS1 RD`と記述し、RS2をRS1で割った商をRDに格納する。オペコードは`4'h3`。例として`DIV G3 G2 G1`は`16'h7653`となる。

$$
\text{RS2} \div \text{RS1} \rightarrow \text{RD}
$$

###### OR

OR命令。レジスタ同士のOR演算を行う命令であり、`OR RS2 RS1 RD`と記述し、RS2とRS1の値のORをRDに格納する。オペコードは`4'h4`。例として`OR G3 G2 G1`は`16'h7654`となる。

$$
\text{RS2} \parallel \text{RS1} \rightarrow \text{RD}
$$

###### AND

AND命令。レジスタ同士のAND演算を行う命令であり、`AND RS2 RS1 RD`と記述し、RS2とRS1の値のANDをRDに格納する。オペコードは`4'h5`。例として`AND G3 G2 G1`は`16'h7655`となる。

$$
\text{RS2} \& \text{RS1} \rightarrow \text{RD}
$$

###### XOR

XOR命令。レジスタ同士のXOR演算を行う命令であり、`XOR RS2 RS1 RD`と記述し、RS2とRS1の値のXORをRDに格納する。オペコードは`4'h6`。例として`XOR G3 G2 G1`は`16'h7656`となる。

$$
\text{RS2} \oplus \text{RS1} \rightarrow \text{RD}
$$

###### SLL

論理左シフト命令。レジスタの値の左シフトを行う命令であり、`SLL RS2 RS1 RD`と記述し、RS1の値をRS2の値だけ論理左シフトしてRDに格納する。オペコードは`4'h7`。例として`SLL G3 G2 G1`は`16'h7657`となる。

$$
\text{RS1} << \text{RS2} \rightarrow \text{RD}
$$

###### SRL

論理右シフト命令。レジスタの値の右シフトを行う命令であり、`SRL RS2 RS1 RD`と記述し、RS1の値をRS2の値だけ論理右シフトしてRDに格納する。オペコードは`4'h8`。例として`SRL G3 G2 G1`は`16'h7658`となる。

$$
\text{RS1} >> \text{RS2} \rightarrow \text{RD}
$$

###### 演算命令まとめ

以下の計算を行いたい場合、アセンブリとそのバイナリは以下の通りになります。

$$
f(x, y) = x\times y + x - y
$$

xの値をG0、yの値をG1、結果をG7に格納する事にします。

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

即値加算命令。即値とレジスタで足し算を行う命令であり、`ADDI IMM RD`と記述し、RDにIMMを加算してRDに格納する。IMMが取れる値の範囲は符号付き8bitなので-128~127。オペコードは`4'h9`。例として`ADDI 100 G0`は`16'h6449`、`ADDI -1 G2`は`16'hFF69`となる。

$$
\text{IMM} + \text{RD} \rightarrow \text{RD}
$$

###### 即値命令まとめ

以下の計算を行いたい場合、アセンブリとそのバイナリは以下の通りになります。

$$
f(x, y) = 3\times x + 100 + y
$$

xの値をG0、yの値をG1、結果をG7に格納する事にします。

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

Z16ではレジスタは16個あると先に説明しましたが、このままではCPUはデータを16個しか持つ事が出来ません。これではあまり実用的なCPUとは言えません。そこで**メモリ**という記憶装置を作り、扱えるデータの量を増やします。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/data_mem.png)

CPUがメモリを扱えるようにするため、メモリからデータを読み書きする専用の命令を定義します。

###### LOAD

Loadはメモリからデータを取り出す操作であり、Z16ではLOADというLoadを行う命令を一つ持っている。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/data_mem_load.png)

LOAD命令のビットフィールドの形式は以下の通り、LSBから下位4bitはオペコード、そこから4bitずつにディスティネーションレジスタ、ソースレジスタ１、即値となっている。

`MSB | imm[3:0] | rs1[3:0] | rd[3:0] | opcode[3:0] | LSB`

LOAD命令。`LOAD IMM RS1 RD`と記述し、IMMとRS1の値の和をメモリアドレスとし、メモリのそのアドレスにあるデータをRDに格納する。IMMが取れる値の範囲は符号付き4bitであるため-8~7。オペコードは`4'hA`であるため、仮に`LOAD 0 G0 G1`は`16'h045A`となる。

$$
\text{Memory}[\text{IMM} + \text{RS1}] \rightarrow \text{RD}
$$

###### STORE

Storeはメモリにデータを書き込む操作であり、Z16ではSTOREというStoreを行う命令を一つ持っている。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/data_mem_store.png)

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

###### ジャンプ命令まとめ

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

CPUは多くの部品で構成されていますが、各部品はそこまで複雑ではありません。本章ではCPUの各部品について説明し、実際に実装していきます。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/computer_abst.png)

#### Register File

**Register File**、これは**レジスタファイル**と呼び、CPUが少量のデータを保持するのに使います。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/regfile.png)

動作としてはアドレスを入力するとデータを出力する**読み出し**と、

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/regfile_read.png)

アドレスとデータを入力するとレジスタファイルに書き込まれる**書き込み**を行います。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/regfile_write.png)

Z16において、レジスタファイルには2つの読み出しポートと1つの書き込みポートがあると都合が良いです。これはZ16では`ADD RS2 RS1 RD`のように、２つのソースレジスタと１つのディスティネーションレジスタを取る命令が多いためです。

以下が実際にレジスタファイルをVerilog HDLで実装したものです。Verilog HDL入門を終えているなら読めると思います。

```verilog
module Z16RegisterFile(
  input  wire           i_clk,
  input  wire   [3:0]   i_rs1_addr,
  input  wire   [3:0]   i_rs2_addr,
  input  wire   [3:0]   i_rd_addr,
  input  wire           i_rd_wen,
  input  wire   [15:0]  i_rd_data,
  output wire   [15:0]  o_rs1_data,
  output wire   [15:0]  o_rs2_data
);

  // レジスタファイル本体
  reg [15:0] mem[15:0];

  // Read
  // アドレスが0なら0を出力
  assign o_rs1_data = (i_rs1_addr == 4'h0) ? 16'h0000 : mem[i_rs1_addr];
  assign o_rs2_data = (i_rs2_addr == 4'h0) ? 16'h0000 : mem[i_rs2_addr];

  // Write
  always @(posedge i_clk) begin
    if(i_rd_wen) begin
      mem[i_rd_addr]  <= i_rd_data;
    end else begin
      mem[i_rd_addr]  <= mem[i_rd_addr];
    end
  end
endmodule
```

入力として、クロック入力の`i_clk`、RS1のアドレス入力の`i_rs1_addr`、RS2のアドレス入力の`i_rs2_addr`、RDのアドレス入力の`i_rd_addr`、RDへの書き込み有効化信号の`i_rd_wen`、RDへの書き込みデータの`i_rd_data`が存在し、出力としてはRS1の読み出しデータの`o_rs1_data`、RS2の読み出しデータの`o_rs2_data`が存在しています。

また常に値が0であるZRを実現するために、読み出し機構に入力されたアドレスが0なら0を出力するようにしています。

これでレジスタファイルが完成しました。テストベンチで動作を確認しましょう。以下のテストベンチでは、`0xA`番目のレジスタに`16'h5555`を書き込んだ後、RS1とRS2両方のポートから読み出しています。

```verilog
module Z16RegisterFile_tb;
  
  reg i_clk = 1'b0;

  reg [3:0]     i_rs1_addr;
  reg [3:0]     i_rs2_addr;
  reg [3:0]     i_rd_addr;
  reg           i_rd_wen;
  reg [15:0]    i_rd_data;
  wire [15:0]   o_rs1_data;
  wire [15:0]   o_rs2_data;

  always #1 begin
    i_clk <= ~i_clk;
  end
  
  initial begin
    $dumpfile("wave.vcd");
    $dumpvars(0, Z16RegisterFile_tb);
  end

  Z16RegisterFile Regfile(
    .i_clk      (i_clk      ),
    .i_rs1_addr (i_rs1_addr ),
    .i_rs2_addr (i_rs2_addr ),
    .i_rd_addr  (i_rd_addr  ),
    .i_rd_wen   (i_rd_wen   ),
    .i_rd_data  (i_rd_data  ),
    .o_rs1_data (o_rs1_data ),
    .o_rs2_data (o_rs2_data )
  );

  initial begin
    i_rd_wen = 1'b0;
    i_rs1_addr  = 4'h0;
    i_rs2_addr  = 4'h0;
    #2
    // 値を書き込み
    i_rd_wen = 1'b1;
    i_rd_addr = 4'hA;
    i_rd_data = 16'h5555;
    #2
    i_rd_wen = 1'b0;
    #2
    // RS1から値を読み出し
    i_rs1_addr  = 4'hA;
    #2
    // RS2から値を読み出し
    i_rs2_addr  = 4'hA;
    #2
    $finish;
  end

endmodule
```

以下のコマンドで動作を確認できます。

```bash
iverilog Z16RegisterFile_tb.v Z16RegisterFile.v
vvp a.out
gtkwave wave.vcd
```

#### Data Mem

**Data Mem**、これは**データメモリ**と呼び。CPUがレジスタファイルに収まらない大量のデータを保存するのに使います。CPUはデータメモリ対してデータの書き込みと読み出しを行います。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/data_mem.png)

メモリに対してのデータ書き込みと読み出しにはそれぞれ名前があり、メモリへのデータ書き込みを**ストア(Store)**、

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/data_mem_store.png)

メモリからのデータ読み出しを**ロード(Load)**と呼びます。Store命令、Load命令とよく使われる単語ですので覚えておきましょう。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/data_mem_load.png)

以下が実際にデータメモリをVerilog HDLで実装したものです。入力信号としてクロック入力の`i_clk`、アドレス入力の`i_addr`、書き込み有効化信号の`i_wen`、データ入力の`i_data`が存在し、出力信号としてはデータ出力の`o_data`が存在しています。

```verilog
module Z16DataMemory(
  input  wire           i_clk,
  input  wire   [15:0]  i_addr,
  input  wire           i_wen,
  input  wire   [15:0]  i_data,
  output wire   [15:0]  o_data
);

  reg [15:0] mem[1023:0];

    // Load
  assign o_data = mem[i_addr[10:1]];

  always @(posedge i_clk) begin
    // Store
    if(i_wen) begin
      mem[i_addr[10:1]] <= i_data;
    end
  end

endmodule
```

これでデータメモリが完成しました。テストベンチで動作を確認しましょう。以下のテストベンチでは、`0x100`番目のアドレスに`16'h5555`をストアした後、ロードしています。

```verilog
module Z16DataMemory_tb;

  reg i_clk = 1'b0;
  reg [15:0]    i_addr;
  reg           i_wen;
  reg [15:0]    i_data;
  wire [15:0]   o_data;

  always #1 begin
    i_clk <= ~i_clk;
  end

  initial begin
    $dumpfile("wave.vcd");
    $dumpvars(0, Z16DataMemory_tb);
  end

  Z16DataMemory DataMem(
    .i_clk  (i_clk  ),
    .i_addr (i_addr ),
    .i_wen  (i_wen  ),
    .i_data (i_data ),
    .o_data (o_data )
  );

  initial begin
    i_wen   = 1'b0;
    #2
    // ストア
    i_wen   = 1'b1;
    i_addr  = 16'h0100;
    i_data  = 16'h5555;
    #2
    i_wen   = 1'b0;
    i_addr  = 16'h0000;
    #2
    // ロード
    i_addr  = 16'h0100;
    #2
    $finish;
  end

endmodule
```

#### Instr Mem

**Instr Mem**、これは**命令メモリ(Instruction Memory)**と呼び、データメモリに似ていますが中には命令が入っています。命令メモリ内の各命令にはアドレスが振られており、命令メモリにアドレスを入力すると命令が出力されます。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/instr_mem.png)

以下が実際に命令メモリをVerilog HDLで実装したものです。ここでは初期値として1~10の総和を求めるプログラムを格納してあります。

```verilog
module Z16InstrMemory(
  input  wire   [15:0]  i_addr,
  output wire   [15:0]  o_instr
);

  wire [15:0] mem[4:0];

  assign o_instr = mem[i_addr[15:1]];

  assign mem[0] = 16'h0A19;
  assign mem[1] = 16'h1220;
  assign mem[2] = 16'hFF19;
  assign mem[3] = 16'hFC4F;
  assign mem[4] = 16'h00FD;

endmodule
```

```verilog
module Z16InstrMemory_tb;

  reg [15:0]    i_addr;
  wire [15:0]   o_instr;

  initial begin
    $dumpfile("wave.vcd");
    $dumpvars(0, Z16InstrMemory_tb);
  end

  Z16InstrMemory InstrMem(
    .i_addr (i_addr ),
    .o_instr(o_instr)
  );

  initial begin
    #2
    i_addr  = 16'h0000;
    #2
    i_addr  = 16'h0002;
    #2
    i_addr  = 16'h0004;
    #2
    i_addr  = 16'h0006;
    #2
    i_addr  = 16'h0008;
    #2
    i_addr  = 16'h000A;
    #2
    i_addr  = 16'h000C;
    #2
    $finish;
  end

endmodule
```

#### ALU

次はALUです。これは**算術論理演算ユニット(Arithmetic Logic Unit)**の略称で、文字通り論理演算(and, or, xor, シフト, etc..)と算術演算(加算, 減算, etc...)を行うモジュールです。CPUにおいて"計算"はこのALUが行っていると考えていただいて構いません。ALUは制御信号と２つのデータを受け取り、１つのデータを出力します。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/alu.png)

以下が実際にALUをVerilog HDLで実装したものです。このALUは入力として入力データ１の`i_data_a`、入力データ２の`i_data_b`、行う演算を制御する`i_ctrl`を持ち、出力として演算結果の`o_data`を持ちます。

```verilog
module Z16ALU(
  input  wire   [15:0]  i_data_a,
  input  wire   [15:0]  i_data_b,
  input  wire   [3:0]   i_ctrl,
  output wire   [15:0]  o_data
);

  assign o_data = alu(i_data_a, i_data_b, i_ctrl);

  function [15:0] alu(
    input [15:0]    i_data_a,
    input [15:0]    i_data_b,
    input [3:0]     i_ctrl
  );
    begin
      case(i_ctrl)
        4'h0 : alu  = i_data_b + i_data_a;
        4'h1 : alu  = i_data_b - i_data_a;
        4'h2 : alu  = i_data_b * i_data_a;
        4'h3 : alu  = i_data_b / i_data_a;
        4'h4 : alu  = i_data_b | i_data_a;
        4'h5 : alu  = i_data_b & i_data_a;
        4'h6 : alu  = i_data_b ^ i_data_a;
        4'h7 : alu  = i_data_a << i_data_a;
        4'h8 : alu  = i_data_a >> i_data_a;
        default: alu = 16'h0000;
      endcase
    end
  endfunction

endmodule
```

```verilog
module Z16ALU_tb;

  reg [15:0]    i_data_a;
  reg [15:0]    i_data_b;
  reg [3:0]     i_ctrl;
  wire [15:0]   o_data;

  initial begin
    $dumpfile("wave.vcd");
    $dumpvars(0, Z16ALU_tb);
  end

  Z16ALU ALU(
    .i_data_a   (i_data_a   ),
    .i_data_b   (i_data_b   ),
    .i_ctrl     (i_ctrl     ),
    .o_data     (o_data     )
  );

  initial begin
    i_data_a    = 16'h0004;
    i_data_b    = 16'h0008;
    i_ctrl      = 4'h0; // ADD
    #2
    i_ctrl      = 4'h1; // SUB
    #2
    i_ctrl      = 4'h2; // MUL
    #2
    i_ctrl      = 4'h3; // DIV
    #2
    i_ctrl      = 4'h4; // OR
    #2
    $finish;
  end

endmodule
```

#### PC

このCPUの中にある**PC**は**プログラムカウンタ(Program Counter)**と呼び、命令メモリにアドレスを供給します。通常、プログラムは上から下に実行されるのでプログラムカウンタが出す値は毎クロック増えていきます。

プログラムカウンタ・命令メモリ・CPUの関係をまとめると、プログラムカウンタがアドレスを命令メモリに入力し、命令メモリが命令をCPUへを出力するという流れになります。このデータの流れを**フェッチ(Fetch)**と呼びます。覚えておきましょう。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/fetch.png)

#### Decoder

最後は**Decoder**です。これは**デコーダ**と呼び、命令から各種制御信号を生成します。具体的な動作の説明はCPUを実際に作成する際に説明しますので、とりあえず今は命令に応じて各部品を制御するモジュールだと認識しておいてください。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/decoder.png)

### マイクロアーキテクチャ

さて、CPUの部品が一通り実装できましたね。本章では実装した部品を実際に組み立て、CPUとしての動作が出来るようにしていきましょう。

たたき台となるコードを以下に用意しました。以下のコードを`Z16CPU.v`という名前で保存してください。このたたき台には命令メモリとデータメモリが未接続の状態で置いてあります。

```verilog
module Z16CPU(
  input  wire   i_clk,
  input  wire   i_rst
);

  // 命令メモリ
  Z16InstrMem InstrMem(
    .i_addr     (),
    .o_instr    ()
  );

  // データメモリ
  Z16DataMem DataMem(
    .i_clk  (),
    .i_addr (),
    .i_wen  (),
    .i_data (),
    .o_data ()
  );

endmodule
```

#### Load命令

最初に実装する命令はLOAD命令です。LOAD命令はメモリからデータを取り出してレジスタに保存する操作でしたね、このLOAD命令の動作を箇条書きにするとこのようになります。

1. 命令メモリから命令をフェッチ
2. デコーダが命令を解釈
3. レジスタからRS1の値を取り出す
4. ALUでアドレス計算
5. データメモリからデータを取り出す
6. レジスタにデータを書き込む

かなり手順が多いですね、実際LOAD命令はZ16の命令の中で最もデータの流れが長い命令となっています。CPUを実装する際はこのLOAD命令のようなデータの流れ、つまりデータパスが最も長い命令を最初に実装した方が楽です。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/path_load.png)

##### 命令フェッチ

ではまずは命令メモリから命令をフェッチする機構を作りましょう。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/path_load_fetch.png)

`Z16CPU.v`に書き込んでいきます。

まずはプログラムカウンタの役割を担う`r_pc`というレジスタを新たに定義します。リセット入力が有効な場合にはこのプログラムカウンタの値を0にし、通常時は2づつカウントアップさせます。そしてプログラムカウンタの値を命令メモリのアドレス入力に接続します。また`w_instr`という名前の信号線を定義し、命令メモリの出力に接続します。

```verilog
module Z16CPU(
  input  wire   i_clk,
  input  wire   i_rst
);

  // プログラムカウンタ
  reg   [15:0]  r_pc;
  wire  [15:0]  w_instr;

  always @(posedge i_clk) begin
    if(i_rst) begin
      // リセット
      r_pc  <= 16'h0000;
    end else begin
      r_pc  <= r_pc + 16'h0002;
    end
  end

  Z16InstrMem InstrMem(
    .i_addr     (r_pc   ), // プログラムカウンタを命令メモリに接続
    .o_instr    (w_instr)  
  );

  Z16DataMem DataMem(
    .i_clk  (),
    .i_addr (),
    .i_wen  (),
    .i_data (),
    .o_data ()
  );

endmodule
```

##### デコーダの作成

これでフェッチ機構を実装できました。次はフェッチした命令を解釈するデコーダを作成していきましょう。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/path_decoder.png)

LOAD命令のビットフィールドは以下のように、LSBから順にオペコード、RDのアドレス、RS1のアドレス、即値となっていましたね。

`MSB | imm[3:0] | rs1[3:0] | rd[3:0] | opcode[3:0] | LSB`

オペコードに加え、RDとRS1のアドレスはそのまま使えるのでスライスを使って取り出しましょう。`o_opcode`がオペコード、`o_rd_addr`がRDのアドレス、`o_rs1_addr`がRS1のアドレスとなっています。

```verilog
module Z16Decoder(
  input  wire   [15:0]  i_instr,
  output wire   [3:0]   o_opcode,
  output wire   [3:0]   o_rd_addr,
  output wire   [3:0]   o_rs1_addr
);

  assign o_opcode   = i_instr[3:0];
  assign o_rd_addr  = i_instr[7:4];
  assign o_rs1_addr = i_instr[11:8];

endmodule
```

次に即値のフィールドですが、これは符号付き4bitとなっていますがALUは符号付き16bitの値しか扱えませんので、符号付き4bitの値を符号付き16bitの値に符号拡張する必要があります。そこで命令のオペコードがLOAD命令のもの場合にのみ、符号拡張を行う関数を作成します。

```verilog
function [15:0] get_imm(input [15:0] i_instr);
begin
  case(i_instr[3:0])
    4'hA    : get_imm = {i_instr[15] ? 12'hFFF : 12'h000, i_instr[15:12]};
    default : get_imm = 16'h0000;
  endcase
end
endfunction
```

(ここに符号拡張の説明を入力)

こうして作成した関数を使い、即値のデコード結果を`o_imm`から出力しましょう。

```verilog
module Z16Decoder(
  input  wire   [15:0]  i_instr,
  output wire   [3:0]   o_opcode,
  output wire   [3:0]   o_rd_addr,
  output wire   [3:0]   o_rs1_addr,
  output wire   [15:0]  o_imm
);

  assign o_opcode   = i_instr[3:0];
  assign o_rd_addr  = i_instr[7:4];
  assign o_rs1_addr = i_instr[11:8];
  assign o_imm      = get_imm(i_instr);

  // 符号拡張
  function [15:0] get_imm(input [15:0] i_instr);
  begin
    case(i_instr[3:0])
      4'hA  : get_imm   = {i_instr[15] ? 12'hFFF : 12'h000, i_instr[15:12]};
      default : get_imm = 16'h0000;
    endcase
  end
  endfunction

endmodule
```

この次は各モジュールを制御する為の信号を作成していきます。

LOAD命令はメモリから値を読み出し、レジスタファイルに値を書き込む命令でしたね。よってメモリは値を読み出す状態にし、レジスタファイルは値を書き込む状態にする必要があります。そのために、デコーダの出力信号にレジスタファイルの書き込み有効化信号である`o_rd_wen`とデータメモリの書き込み有効化信号である`o_mem_wen`を追加しましょう。

```verilog
module Z16Decoder(
  input  wire   [15:0]  i_instr,
  output wire   [3:0]   o_opcode,
  output wire   [3:0]   o_rd_addr,
  output wire   [3:0]   o_rs1_addr,
  output wire   [15:0]  o_imm,
  output wire           o_rd_wen,
  output wire           o_mem_wen
);
```

LOAD命令の場合、`o_rd_wen`は`1`を出力し、`o_mem_wen`は`0`を出力する必要がありますね。そのための関数である`get_rd_wen()`と`get_mem_wen()`を作成しましょう。

```verilog
function get_rd_wen(input [15:0] i_instr);
begin
  if(4'hA == i_instr[3:0]) begin
    get_rd_wen    = 1'b1;
  end else begin
    get_rd_wen    = 1'b0;
  end
end
endfunction

function get_mem_wen(input [15:0] i_instr);
begin
  if(4'hA == i_instr[3:0]) begin
    get_mem_wen = 1'b0;
  end else begin
    get_mem_wen = 1'b0;
  end
end
endfunction
```

こうして作成した関数をデコーダに追加すると以下の通りになります。

```verilog
module Z16Decoder(
  input  wire   [15:0]  i_instr,
  output wire   [3:0]   o_opcode,
  output wire   [3:0]   o_rd_addr,
  output wire   [3:0]   o_rs1_addr,
  output wire   [15:0]  o_imm,
  output wire           o_rd_wen,
  output wire           o_mem_wen
);

  assign o_opcode   = i_instr[3:0];
  assign o_rd_addr  = i_instr[7:4];
  assign o_rs1_addr = i_instr[11:8];
  assign o_imm      = get_imm(i_instr);
  assign o_rd_wen   = get_rd_wen(i_instr);
  assign o_mem_wen  = get_mem_wen(i_instr);

  // 符号拡張
  function [15:0] get_imm(input [15:0] i_instr);
  begin
    case(i_instr[3:0])
      4'hA  : get_imm   = {i_instr[15] ? 12'hFFF : 12'h000, i_instr[15:12]};
      default : get_imm = 16'h0000;
    endcase
  end
  endfunction

  function get_rd_wen(input [15:0] i_instr);
  begin
    if(4'hA == i_instr[3:0]) begin
     get_rd_wen    = 1'b1;
    end else begin
      get_rd_wen    = 1'b0;
    end
  end
  endfunction

  function get_mem_wen(input [15:0] i_instr);
  begin
    if(4'hA == i_instr[3:0]) begin
      get_mem_wen = 1'b0;
    end else begin
      get_mem_wen = 1'b0;
    end
  end
  endfunction

endmodule
```

次はALUにアドレス計算を行わせる為に、ALUの制御信号を作成しましょう。まずはALUの制御信号である`o_alu_ctrl`を追加します。

```verilog
module Z16Decoder(
  input  wire   [15:0]  i_instr,
  output wire   [3:0]   o_opcode,
  output wire   [3:0]   o_rd_addr,
  output wire   [3:0]   o_rs1_addr,
  output wire   [15:0]  o_imm,
  output wire           o_rd_wen,
  output wire           o_mem_wen,
  output wire   [3:0]   o_alu_ctrl
);
```

LOAD命令では、即値とRS1の値を加算してアドレスを作っていましたね。

$$
\text{Memory}[\text{IMM} + \text{RS1}] \rightarrow \text{RD}
$$

よってALUには加算を行ってもらう必要があります。ALUの実装を見返すと`i_ctrl`に`4'h0`を入力すると加算を行ってくれるように実装してありますね。よってLOAD命令のオペコードの場合は`4'h0`を出力する関数を作成します。

```verilog
function [3:0] get_alu_ctrl(input [15:0] i_instr);
begin
  if(4'hA == i_instr[3:0]) begin
    get_alu_ctrl  = 4'h0;
  end else begin
    get_alu_ctrl  = 4'h0;
  end
end
endfunction
```

こうして作成した関数をデコーダに追加すると以下の通りになります。

```verilog
module Z16Decoder(
  input  wire   [15:0]  i_instr,
  output wire   [3:0]   o_opcode,
  output wire   [3:0]   o_rd_addr,
  output wire   [3:0]   o_rs1_addr,
  output wire   [15:0]  o_imm,
  output wire           o_rd_wen,
  output wire           o_mem_wen,
  output wire   [3:0]   o_alu_ctrl
);

  assign o_opcode   = i_instr[3:0];
  assign o_rd_addr  = i_instr[7:4];
  assign o_rs1_addr = i_instr[11:8];
  assign o_imm      = get_imm(i_instr);
  assign o_rd_wen   = get_rd_wen(i_instr);
  assign o_mem_wen  = get_mem_wen(i_instr);
  assign o_alu_ctrl = get_alu_ctrl(i_instr);

  // 符号拡張
  function [15:0] get_imm(input [15:0] i_instr);
  begin
    case(i_instr[3:0])
      4'hA  : get_imm   = {i_instr[15] ? 12'hFFF : 12'h000, i_instr[15:12]};
      default : get_imm = 16'h0000;
    endcase
  end
  endfunction

  function get_rd_wen(input [15:0] i_instr);
  begin
    if(4'hA == i_instr[3:0]) begin
     get_rd_wen    = 1'b1;
    end else begin
      get_rd_wen    = 1'b0;
    end
  end
  endfunction

  function get_mem_wen(input [15:0] i_instr);
  begin
    if(4'hA == i_instr[3:0]) begin
      get_mem_wen = 1'b0;
    end else begin
      get_mem_wen = 1'b0;
    end
  end
  endfunction

  function [3:0] get_alu_ctrl(input [15:0] i_instr);
  begin
    if(4'hA == i_instr[3:0]) begin
      get_alu_ctrl  = 4'h0;
    end else begin
      get_alu_ctrl  = 4'h0;
    end
  end
  endfunction

endmodule
```

これでデコーダが完成しました。CPUに設置し、各種信号線を接続します。

```verilog
module Z16CPU(
  input  wire   i_clk,
  input  wire   i_rst
);

  // Program Counter
  reg   [15:0]  r_pc;

  wire  [15:0]  w_instr;
  wire [15:0]   w_rd_addr;
  wire [15:0]   w_rs1_addr;
  wire [15:0]   w_imm;
  wire          w_rd_wen;
  wire          w_mem_wen;
  wire [3:0]    w_alu_ctrl;


  always @(posedge i_clk) begin
    if(i_rst) begin
      r_pc  <= 16'h0000;
    end else begin
      r_pc  <= r_pc + 16'h0002;
    end
  end

  Z16InstrMemory InstrMem(
    .i_addr     (r_pc   ),
    .o_instr    (w_instr)
  );

  Z16Decoder Decoder(
    .i_instr    (w_instr    ),
    .o_rd_addr  (w_rd_addr  ),
    .o_rs1_addr (w_rs1_addr ),
    .o_imm      (w_imm      ),
    .o_rd_wen   (w_rd_wen   ),
    .o_mem_wen  (w_mem_wen  ),
    .o_alu_ctrl (w_alu_ctrl )
  );

  Z16DataMemory DataMem(
    .i_clk  (),
    .i_addr (),
    .i_wen  (),
    .i_data (),
    .o_data ()
  );

endmodule
```

これで命令をフェッチし、デコーダに命令を入力する所まで実装できました。

##### レジスタファイルからの値読み出し

次はRS1の値をレジスタから取り出すデータパスを作成しましょう。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/path_load_readreg.png)

以下ではCPUにレジスタファイルを設置し、デコーダが出力したRS1のアドレスである`w_rs1_addr`を接続しています。またRS1のデータの信号線である`w_rs1_data`を新たに定義し、レジスタファイルの出力に接続しています。

```verilog
module Z16CPU(
  input  wire   i_clk,
  input  wire   i_rst
);

  // Program Counter
  reg   [15:0]  r_pc;

  wire  [15:0]  w_instr;
  wire [3:0]    w_rd_addr;
  wire [3:0]    w_rs1_addr; // RS1のアドレス
  wire [15:0]   w_imm;
  wire          w_rd_wen;
  wire          w_mem_wen;
  wire [3:0]    w_alu_ctrl;

  wire [15:0]   w_rs1_data; // RS1のデータ


  always @(posedge i_clk) begin
    if(i_rst) begin
      r_pc  <= 16'h0000;
    end else begin
      r_pc  <= r_pc + 16'h0002;
    end
  end

  Z16InstrMemory InstrMem(
    .i_addr     (r_pc   ),
    .o_instr    (w_instr)
  );

  Z16Decoder Decoder(
    .i_instr    (w_instr    ),
    .o_rd_addr  (w_rd_addr  ),
    .o_rs1_addr (w_rs1_addr ),
    .o_imm      (w_imm      ),
    .o_rd_wen   (w_rd_wen   ),
    .o_mem_wen  (w_mem_wen  ),
    .o_alu_ctrl (w_alu_ctrl )
  );

  Z16RegisterFile RegFile(
    .i_clk      (i_clk      ),
    .i_rs1_addr (w_rs1_addr ),  // RS1のアドレスを接続
    .o_rs1_data (w_rs1_data ),  // RS1のデータを出力
    .i_rs2_addr (),
    .o_rs2_data (),
    .i_rd_data  (),
    .i_rd_addr  (),
    .i_rd_wen   ()
  );

  Z16DataMemory DataMem(
    .i_clk  (),
    .i_addr (),
    .i_wen  (),
    .i_data (),
    .o_data ()
  );

endmodule
```

##### アドレス計算

次はアドレス計算を行うデータパスを実装しましょう。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/path_load_caladdr.png)

ALUを設置し、`w_rs1_data`、`w_imm`、`w_alu_ctrl`を接続します。またALUの演算結果の信号である`w_alu_data`を新たに定義し、ALUの演算結果出力に接続します。

```verilog
module Z16CPU(
  input  wire   i_clk,
  input  wire   i_rst
);

  // Program Counter
  reg   [15:0]  r_pc;

  wire  [15:0]  w_instr;
  wire [3:0]    w_rd_addr;
  wire [3:0]    w_rs1_addr;
  wire [15:0]   w_imm;
  wire          w_rd_wen;
  wire          w_mem_wen;
  wire [3:0]    w_alu_ctrl;

  wire [15:0]   w_rs1_data;

  wire [15:0]   w_alu_data; // ALUの演算結果

  always @(posedge i_clk) begin
    if(i_rst) begin
      r_pc  <= 16'h0000;
    end else begin
      r_pc  <= r_pc + 16'h0002;
    end
  end

  Z16InstrMemory InstrMem(
    .i_addr     (r_pc   ),
    .o_instr    (w_instr)
  );

  Z16Decoder Decoder(
    .i_instr    (w_instr    ),
    .o_rd_addr  (w_rd_addr  ),
    .o_rs1_addr (w_rs1_addr ),
    .o_imm      (w_imm      ),
    .o_rd_wen   (w_rd_wen   ),
    .o_mem_wen  (w_mem_wen  ),
    .o_alu_ctrl (w_alu_ctrl )
  );

  Z16RegisterFile RegFile(
    .i_clk      (i_clk      ),
    .i_rs1_addr (w_rs1_addr ),
    .o_rs1_data (w_rs1_data ),
    .i_rs2_addr (),
    .o_rs2_data (),
    .i_rd_data  (),
    .i_rd_addr  (),
    .i_rd_wen   ()
  );

  Z16ALU ALU(
    .i_data_a   (w_rs1_data ),  // RS1のデータを入力
    .i_data_b   (w_imm      ),  // 即値を入力
    .i_ctrl     (w_alu_ctrl ),  // ALUの制御信号を入力
    .o_data     (w_alu_data )   // ALUの演算結果を出力
  );

  Z16DataMemory DataMem(
    .i_clk  (),
    .i_addr (),
    .i_wen  (),
    .i_data (),
    .o_data ()
  );

endmodule
```

##### メモリからのデータ読み出し

アドレス計算のデータパスが完成したら、次はそのアドレスをメモリに入力し、データを取り出すデータパスを作りましょう。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/path_load_readmem.png)

データメモリにALUが計算したアドレスである`w_alu_data`とデコーダからの書き込み有効化信号である`w_mem_wen`を接続します。また、メモリから読み出したデータの信号線である`w_mem_rdata`を新たに定義し、データメモリに接続します。

```verilog
module Z16CPU(
  input  wire   i_clk,
  input  wire   i_rst
);

  // Program Counter
  reg   [15:0]  r_pc;

  wire  [15:0]  w_instr;
  wire [3:0]    w_rd_addr;
  wire [3:0]    w_rs1_addr;
  wire [15:0]   w_imm;
  wire          w_rd_wen;
  wire          w_mem_wen;
  wire [3:0]    w_alu_ctrl;

  wire [15:0]   w_rs1_data;

  wire [15:0]   w_alu_data;
  wire [15:0]   w_mem_rdata; // メモリからの読み出しデータ

  always @(posedge i_clk) begin
    if(i_rst) begin
      r_pc  <= 16'h0000;
    end else begin
      r_pc  <= r_pc + 16'h0002;
    end
  end

  Z16InstrMemory InstrMem(
    .i_addr     (r_pc   ),
    .o_instr    (w_instr)
  );

  Z16Decoder Decoder(
    .i_instr    (w_instr    ),
    .o_rd_addr  (w_rd_addr  ),
    .o_rs1_addr (w_rs1_addr ),
    .o_imm      (w_imm      ),
    .o_rd_wen   (w_rd_wen   ),
    .o_mem_wen  (w_mem_wen  ),
    .o_alu_ctrl (w_alu_ctrl )
  );

  Z16RegisterFile RegFile(
    .i_clk      (i_clk      ),
    .i_rs1_addr (w_rs1_addr ),
    .o_rs1_data (w_rs1_data ),
    .i_rs2_addr (),
    .o_rs2_data (),
    .i_rd_data  (),
    .i_rd_addr  (),
    .i_rd_wen   ()
  );

  Z16ALU ALU(
    .i_data_a   (w_rs1_data ),
    .i_data_b   (w_imm      ),
    .i_ctrl     (w_alu_ctrl ),
    .o_data     (w_alu_data )
  );

  Z16DataMemory DataMem(
    .i_clk  (i_clk      ),
    .i_addr (w_alu_data ),  // アドレス入力
    .i_wen  (w_mem_wen  ),  // メモリ書き込み有効化信号
    .i_data (),
    .o_data (w_mem_rdata)   // メモリのデータ出力
  );

endmodule
```

##### レジスタファイルへのデータ書き込み

最後にメモリから読み出したデータをレジスタファイルに書き込むデータパスを作成しましょう。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/path_load_writereg.png)

メモリから取り出したデータの信号線である`w_mem_rdata`をレジスタファイルのデータ書き込みポートに接続し、またデコーダからのRDのアドレスと書き込み有効化信号を接続します。

```verilog
module Z16CPU(
  input  wire   i_clk,
  input  wire   i_rst
);

  // Program Counter
  reg   [15:0]  r_pc;

  wire  [15:0]  w_instr;
  wire [3:0]    w_rd_addr;
  wire [3:0]    w_rs1_addr;
  wire [15:0]   w_imm;
  wire          w_rd_wen;
  wire          w_mem_wen;
  wire [3:0]    w_alu_ctrl;

  wire [15:0]   w_rs1_data;

  wire [15:0]   w_alu_data;
  wire [15:0]   w_mem_rdata;

  always @(posedge i_clk) begin
    if(i_rst) begin
      r_pc  <= 16'h0000;
    end else begin
      r_pc  <= r_pc + 16'h0002;
    end
  end

  Z16InstrMemory InstrMem(
    .i_addr     (r_pc   ),
    .o_instr    (w_instr)
  );

  Z16Decoder Decoder(
    .i_instr    (w_instr    ),
    .o_rd_addr  (w_rd_addr  ),
    .o_rs1_addr (w_rs1_addr ),
    .o_imm      (w_imm      ),
    .o_rd_wen   (w_rd_wen   ),
    .o_mem_wen  (w_mem_wen  ),
    .o_alu_ctrl (w_alu_ctrl )
  );

  Z16RegisterFile RegFile(
    .i_clk      (i_clk      ),
    .i_rs1_addr (w_rs1_addr ),
    .o_rs1_data (w_rs1_data ),
    .i_rs2_addr (),
    .o_rs2_data (),
    .i_rd_data  (w_mem_rdata),
    .i_rd_addr  (w_rd_addr  ),
    .i_rd_wen   (w_rd_wen   )
  );

  Z16ALU ALU(
    .i_data_a   (w_rs1_data ),
    .i_data_b   (w_imm      ),
    .i_ctrl     (w_alu_ctrl ),
    .o_data     (w_alu_data )
  );

  Z16DataMemory DataMem(
    .i_clk  (i_clk      ),
    .i_addr (w_alu_data ),
    .i_wen  (w_mem_wen  ),
    .i_data (),
    .o_data (w_mem_rdata)
  );

endmodule
```

これでLOAD命令のデータパスが完成しました。流石に大変でしたね、LOAD命令が一番データパスが長い命令ですので残りはもっと簡単に実装できます。気を楽にしてください。

このCPUは現在LOAD命令のみを実行できます。シミュレーションで動作を確認してみましょう。

##### 動作の確認

動作を確認するために、まずは命令メモリにLOAD命令を書き込みましょう。以下では命令メモリのアドレス0とアドレス3(正しくは6)にLOAD命令を書き込んであります。それ以外は適当に0で埋めておきましょう。

```verilog
module Z16InstrMemory(
  input  wire           i_clk,
  input  wire   [15:0]  i_addr,
  output wire   [15:0]  o_instr
);

  wire [15:0] mem[4:0];

  assign o_instr = mem[i_addr[15:1]];

  assign mem[0] = 16'h406A; // LOAD 4 ZR G2 
  assign mem[1] = 16'h0000;
  assign mem[2] = 16'h0000;
  assign mem[3] = 16'h008A; // LOAD 0 ZR G4
  assign mem[4] = 16'h0000;

endmodule
```

またテストベンチも適当に作成します。最初はリセットをオンにし、2単位時間後にリセットを落としてプログラムの実行を開始するようにしています。

```verilog
module Z16CPU_tb;

  reg i_clk = 1'b0;
  reg i_rst = 1'b0;

  always #1 begin
    i_clk <= ~i_clk;
  end

  initial begin
    $dumpfile("wave.vcd");
    $dumpvars(0, Z16CPU_tb);
  end

  Z16CPU CPU(
    .i_clk  (i_clk  ),
    .i_rst  (i_rst  )
  );

  initial begin
    // リセット
    i_rst   = 1'b1;
    #2
    // リセットを落とし、実行開始
    i_rst   = 1'b0;
    #100
    $finish;
  end

endmodule
```

以下のコマンドでシミュレーションを実行します。

```bash
iverilog Z16CPU_tb.v Z16ALU.v Z16CPU.v Z16DataMemory.v Z16Decoder.sv Z16InstrMemory.v Z16RegisterFile.v
vvp a.out
gtkwave wave.vcd
```

実際にシミュレーションをした結果が以下の波形です。命令メモリから命令をフェッチ、デコーダが各種制御信号を生成、レジスタからの値取り出し、ALUによるアドレス計算、データメモリからの値のロード、レジスタへの値書き込みを行っている様子が分かります。

ただし、値が何も書き込まれていないメモリからデータをロードしているので、レジスタには不定値`16'hxxxx`が書き込まれている事に注意してください。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/wave_load.png)

これで、LOAD命令のデータパスが完成した事が確認できました。

#### Store命令

次はSTORE命令のデータパスを作成し、CPUがSTORE命令を実行できるようにしましょう。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/path_store.png)

STORE命令の動作を箇条書きすると以下の通りになります。

1. 命令メモリから命令をフェッチ
2. デコーダが命令を解釈
3. レジスタからRS1, RS2の値を取り出す
4. ALUでアドレス計算
5. データメモリにRS2の値を書き込む

1の命令フェッチの機構はLOAD命令を実装した時に作りましたね。2のデコーダには少し手を加える必要があります。

##### デコーダを改造

ではデコーダの改造に着手しましょう。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/path_decoder.png)

STORE命令のビットフィールドは以下のように、LSBから順にオペコード、即値、RS1のアドレス、RS2のアドレスとなっていましたね。

`MSB | rs2[3:0] | rs1[3:0] | imm[3:0] | opcode[3:0] | LSB`

LOAD命令のビットフィールドと比較すると、RS1のフィールドは共通ですが、RDのフィールドが即値になり、即値のフィールドがRS2になりました。

RS2のアドレスはそのまま使えるのでスライスを使って取り出しましょう。デコーダの出力として新たに`o_rs2_addr`を定義し、STORE命令のRS2のアドレスを出力します。

```verilog
module Z16Decoder(
  input  wire   [15:0]  i_instr,
  output wire   [3:0]   o_opcode,
  output wire   [3:0]   o_rd_addr,
  output wire   [3:0]   o_rs1_addr,
  output wire   [3:0]   o_rs2_addr,
  output wire   [15:0]  o_imm,
  output wire           o_rd_wen,
  output wire           o_mem_wen,
  output wire   [3:0]   o_alu_ctrl
);

  assign o_opcode   = i_instr[3:0];
  assign o_rd_addr  = i_instr[7:4];
  assign o_rs1_addr = i_instr[11:8];
  assign o_rs2_addr = i_instr[15:12];
  assign o_imm      = get_imm(i_instr);
  assign o_rd_wen   = get_rd_wen(i_instr);
  assign o_mem_wen  = get_mem_wen(i_instr);
  assign o_alu_ctrl = get_alu_ctrl(i_instr);
```

次に即値に対処しましょう。LOAD命令では`i_instr`の`[15:12]`の4bitを取り出し16bitに符号拡張していましたが、STORE命令では`i_instr`の`[7:4]`の4bitが即値となっています。そこで`get_imm`を改造してオペコードがSTORE命令の`4'hB`の場合は`i_instr[7:4]`の4bitを取り出して符号拡張するように改造します。

```verilog
// 符号拡張
function [15:0] get_imm(input [15:0] i_instr);
begin
  case(i_instr[3:0])
    4'hA  : get_imm   = {i_instr[15] ? 12'hFFF : 12'h000, i_instr[15:12]};
    4'hB  : get_imm   = {i_instr[7]  ? 12'hFFF : 12'h000, i_instr[7:4]};
    default : get_imm = 16'h0000;
  endcase
end
endfunction
```

次にSTORE命令はメモリに値を書き込む命令ですので、STORE命令が来た時はメモリ書き込み有効化信号である`o_mem_wen`を1にする必要があります。Z16において、メモリに値を書き込むのはSTORE命令の場合のみですので、`get_mem_wen`を改造し、オペコードがSTORE命令の`4'hB`の場合にのみ1になるようにします。

```verilog
function get_mem_wen(input [15:0] i_instr);
begin
  // STORE命令の場合にのみ1
  if(4'hB == i_instr[3:0]) begin
    get_mem_wen = 1'b1;
  end else begin
    get_mem_wen = 1'b0;
  end
end
endfunction
```

次にALUの制御信号ですが、STORE命令のアドレス計算もRS1と即値の加算ですので、ALUには加算をしてもらう必要があります。

$$
\text{RS2} \rightarrow \text{Memory}[\text{RS1} + \text{IMM}]
$$

これはLOAD命令と同じですので、とりあえず今はALUには常に加算をやってもらうように改造します。

```verilog
function [3:0] get_alu_ctrl(input [15:0] i_instr);
begin
  get_alu_ctrl  = 4'h0;
end
endfunction
```

改造が完了したデコーダは以下の通りです。

```verilog
module Z16Decoder(
  input  wire   [15:0]  i_instr,
  output wire   [3:0]   o_opcode,
  output wire   [3:0]   o_rd_addr,
  output wire   [3:0]   o_rs1_addr,
  output wire   [3:0]   o_rs2_addr,
  output wire   [15:0]  o_imm,
  output wire           o_rd_wen,
  output wire           o_mem_wen,
  output wire   [3:0]   o_alu_ctrl
);

  assign o_opcode   = i_instr[3:0];
  assign o_rd_addr  = i_instr[7:4];
  assign o_rs1_addr = i_instr[11:8];
  assign o_rs2_addr = i_instr[15:12];
  assign o_imm      = get_imm(i_instr);
  assign o_rd_wen   = get_rd_wen(i_instr);
  assign o_mem_wen  = get_mem_wen(i_instr);
  assign o_alu_ctrl = get_alu_ctrl(i_instr);

  // 符号拡張
  function [15:0] get_imm(input [15:0] i_instr);
  begin
    case(i_instr[3:0])
      4'hA  : get_imm   = {i_instr[15] ? 12'hFFF : 12'h000, i_instr[15:12]};
      4'hB  : get_imm   = {i_instr[7]  ? 12'hFFF : 12'h000, i_instr[7:4]};
      default : get_imm = 16'h0000;
    endcase
  end
  endfunction

  function get_rd_wen(input [15:0] i_instr);
  begin
    if(4'hA == i_instr[3:0]) begin
      get_rd_wen    = 1'b1;
    end else begin
      get_rd_wen    = 1'b0;
    end
  end
  endfunction

  function get_mem_wen(input [15:0] i_instr);
  begin
    // STORE命令の場合にのみ1
    if(4'hB == i_instr[3:0]) begin
      get_mem_wen = 1'b1;
    end else begin
      get_mem_wen = 1'b0;
    end
  end
  endfunction

  function [3:0] get_alu_ctrl(input [15:0] i_instr);
  begin
    get_alu_ctrl  = 4'h0;
  end
  endfunction

endmodule
```

デコーダの改造が完了しましたら、`Z16CPU.v`にRS2のアドレスの信号線である`w_rs2_addr`を追加し、デコーダに接続しましょう。

```verilog
  // Program Counter
  reg   [15:0]  r_pc;

  wire  [15:0]  w_instr;
  wire [3:0]    w_rd_addr;
  wire [3:0]    w_rs1_addr;
  wire [3:0]    w_rs2_addr;
  wire [15:0]   w_imm;
  wire          w_rd_wen;
  wire          w_mem_wen;
  wire [3:0]    w_alu_ctrl;
```

中略

```verilog
  Z16Decoder Decoder(
    .i_instr    (w_instr    ),
    .o_rd_addr  (w_rd_addr  ),
    .o_rs1_addr (w_rs1_addr ),
    .o_rs2_addr (w_rs2_addr ),
    .o_imm      (w_imm      ),
    .o_rd_wen   (w_rd_wen   ),
    .o_mem_wen  (w_mem_wen  ),
    .o_alu_ctrl (w_alu_ctrl )
  );
```

##### レジスタファイルからの値読み出し

次はレジスタファイルからRS2の値を読み出すデータパスを作成しましょう。これで読み出されたRS2の値がメモリに書き込まれる値になります。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/path_store_readreg.png)

まずはRS2の値の信号線である`w_rs2_data`を新たに定義します。

```verilog
  wire          w_mem_wen;
  wire [3:0]    w_alu_ctrl;

  wire [15:0]   w_rs1_data;
  wire [15:0]   w_rs2_data;

  wire [15:0]   w_alu_data;
  wire [15:0]   w_mem_rdata;
```

そしてレジスタファイルのRS2のアドレス入力にはデコーダが出力した`w_rs2_addr`を接続し、RS2のデータ出力には先程新たに定義した`w_rs2_data`を接続します。

```verilog
  Z16RegisterFile RegFile(
    .i_clk      (i_clk      ),
    .i_rs1_addr (w_rs1_addr ),
    .o_rs1_data (w_rs1_data ),
    .i_rs2_addr (w_rs2_addr ),
    .o_rs2_data (w_rs2_data ),
    .i_rd_data  (w_mem_rdata),
    .i_rd_addr  (w_rd_addr  ),
    .i_rd_wen   (w_rd_wen   )
  );
```

これでレジスタファイルからRS2の値を読み出すデータパスが実装できました。ここまでの改造内容をまとめたものが以下になります。

```verilog
module Z16CPU(
  input  wire   i_clk,
  input  wire   i_rst
);

  // Program Counter
  reg   [15:0]  r_pc;

  wire  [15:0]  w_instr;
  wire [3:0]    w_rd_addr;
  wire [3:0]    w_rs1_addr;
  wire [3:0]    w_rs2_addr;
  wire [15:0]   w_imm;
  wire          w_rd_wen;
  wire          w_mem_wen;
  wire [3:0]    w_alu_ctrl;

  wire [15:0]   w_rs1_data;
  wire [15:0]   w_rs2_data;

  wire [15:0]   w_alu_data;
  wire [15:0]   w_mem_rdata;

  always @(posedge i_clk) begin
    if(i_rst) begin
      r_pc  <= 16'h0000;
    end else begin
      r_pc  <= r_pc + 16'h0002;
    end
  end

  Z16InstrMemory InstrMem(
    .i_addr     (r_pc   ),
    .o_instr    (w_instr)
  );

  Z16Decoder Decoder(
    .i_instr    (w_instr    ),
    .o_rd_addr  (w_rd_addr  ),
    .o_rs1_addr (w_rs1_addr ),
    .o_rs2_addr (w_rs2_addr ),
    .o_imm      (w_imm      ),
    .o_rd_wen   (w_rd_wen   ),
    .o_mem_wen  (w_mem_wen  ),
    .o_alu_ctrl (w_alu_ctrl )
  );

  Z16RegisterFile RegFile(
    .i_clk      (i_clk      ),
    .i_rs1_addr (w_rs1_addr ),
    .o_rs1_data (w_rs1_data ),
    .i_rs2_addr (w_rs2_addr ),
    .o_rs2_data (w_rs2_data ),
    .i_rd_data  (w_mem_rdata),
    .i_rd_addr  (w_rd_addr  ),
    .i_rd_wen   (w_rd_wen   )
  );

  Z16ALU ALU(
    .i_data_a   (w_rs1_data ),
    .i_data_b   (w_imm      ),
    .i_ctrl     (w_alu_ctrl ),
    .o_data     (w_alu_data )
  );

  Z16DataMemory DataMem(
    .i_clk  (i_clk      ),
    .i_addr (w_alu_data ),
    .i_wen  (w_mem_wen  ),
    .i_data (),
    .o_data (w_mem_rdata)
  );

endmodule
```

この次のALUでアドレス計算を行うデータパスは、既にLOAD命令を実装する際に完成しているので手を加える必要はありません。

##### メモリへの値書き込み

最後に、メモリへデータを書き込むデータパスを作成していきます。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/path_store.png)

レジスタファイルのRS2のデータ出力をデータメモリの書き込みデータ入力に接続します。

```verilog
module Z16CPU(
  input  wire   i_clk,
  input  wire   i_rst
);

  // Program Counter
  reg   [15:0]  r_pc;

  wire  [15:0]  w_instr;
  wire [3:0]    w_rd_addr;
  wire [3:0]    w_rs1_addr;
  wire [3:0]    w_rd_addr;
  wire [15:0]   w_imm;
  wire          w_rd_wen;
  wire          w_mem_wen;
  wire [3:0]    w_alu_ctrl;

  wire [15:0]   w_rs1_data;
  wire [15:0]   w_rs2_data;

  wire [15:0]   w_alu_data;
  wire [15:0]   w_mem_rdata;

  always @(posedge i_clk) begin
    if(i_rst) begin
      r_pc  <= 16'h0000;
    end else begin
      r_pc  <= r_pc + 16'h0002;
    end
  end

  Z16InstrMemory InstrMem(
    .i_addr     (r_pc   ),
    .o_instr    (w_instr)
  );

  Z16Decoder Decoder(
    .i_instr    (w_instr    ),
    .o_rd_addr  (w_rd_addr  ),
    .o_rs1_addr (w_rs1_addr ),
    .o_rs2_addr (w_rs2_addr ),
    .o_imm      (w_imm      ),
    .o_rd_wen   (w_rd_wen   ),
    .o_mem_wen  (w_mem_wen  ),
    .o_alu_ctrl (w_alu_ctrl )
  );

  Z16RegisterFile RegFile(
    .i_clk      (i_clk      ),
    .i_rs1_addr (w_rs1_addr ),
    .o_rs1_data (w_rs1_data ),
    .i_rs2_addr (w_rs2_addr ),
    .o_rs2_data (w_rs2_data ),
    .i_rd_data  (w_mem_rdata),
    .i_rd_addr  (w_rd_addr  ),
    .i_rd_wen   (w_rd_wen   )
  );

  Z16ALU ALU(
    .i_data_a   (w_rs1_data ),
    .i_data_b   (w_imm      ),
    .i_ctrl     (w_alu_ctrl ),
    .o_data     (w_alu_data )
  );

  Z16DataMemory DataMem(
    .i_clk  (i_clk      ),
    .i_addr (w_alu_data ),
    .i_wen  (w_mem_wen  ),
    .i_data (w_rs2_data ),
    .o_data (w_mem_rdata)
  );

endmodule
```

これでSTORE命令のデータパスの実装が完了しました。割と簡単に出来ましたね。これで`Z16CPU`はLOAD命令とSTORE命令を実行できるようになりました。シミュレーションで動作を確認してみましょう。

##### 動作の確認

動作を確認するために、命令メモリのプログラムを変更しましょう。以下のプログラムはSTORE命令を用いてレジスタZRの値、つまり0をメモリのアドレス4に保存し、次にLOAD命令を用いてメモリのアドレス4に保存されている値をレジスタG1に格納します。つまり実行後に0がレジスタG1が格納されていれば成功という事です。

```verilog
module Z16InstrMemory(
  input  wire           i_clk,
  input  wire   [15:0]  i_addr,
  output wire   [15:0]  o_instr
);

  wire [15:0] mem[4:0];

  assign o_instr = mem[i_addr[15:1]];

  assign mem[0] = 16'h004B; // STORE ZR ZR 4
  assign mem[1] = 16'h405A; // LOAD 4 ZR G1
  assign mem[2] = 16'h0000;
  assign mem[3] = 16'h0000;
  assign mem[4] = 16'h0000;

endmodule
```

テストベンチに変更を行う必要はありません。以下のコマンドでシミュレーションを行い、波形を見ることが出来ます。

```bash
iverilog Z16CPU_tb.v Z16ALU.v Z16CPU.v Z16DataMemory.v Z16Decoder.sv Z16InstrMemory.v Z16RegisterFile.v
vvp a.out
gtkwave wave.vcd
```

実際にシミュレーションをした結果が以下の波形です。1サイクル目のSTORE命令でレジスタZRの値をメモリの`16'h0004`に書き込み、2サイクル目のLOAD命令でメモリの`16'h0004`の値をレジスタG1に書き込んでいる様子が分かります。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/wave_store.png)

これで、STORE命令のデータパスが完成した事が確認できました。

#### 演算命令

さて、次は演算命令のデータパスを実装し、ADDやSUB、MUL命令などを実行できるようにしましょう。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/path_exe.png)

演算命令の動作を箇条書すると以下の通りになります。

1. 命令メモリから命令をフェッチ
2. デコーダが命令を解釈
3. レジスタからRS1, RS2の値を取り出す
4. ALUで演算
5. レジスタに演算結果を書き込む

命令フェッチは既に実装してありますので、デコーダの改造に着手しましょう。

##### デコーダを改造

ではデコーダの改造に着手します。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/path_decoder.png)

以下はデコーダの入出力信号です。演算命令を実行するにあたり、変更を加える必要がある信号はレジスタファイルへの書き込み制御信号である`o_rd_wen`とALUの制御信号である`o_alu_ctrl`です。改造していきましょう。

```verilog
module Z16Decoder(
  input  wire   [15:0]  i_instr,
  output wire   [3:0]   o_opcode,
  output wire   [3:0]   o_rd_addr,
  output wire   [3:0]   o_rs1_addr,
  output wire   [3:0]   o_rs2_addr,
  output wire   [15:0]  o_imm,
  output wire           o_rd_wen,
  output wire           o_mem_wen,
  output wire   [3:0]   o_alu_ctrl
);
```

まずは`o_rd_wen`に手を加えてきます。Z16において、演算命令が割り当てられているオペコードは`4'h0`~`4'h8`でしたね。よってオペコードがこの範囲である場合も`o_rd_wen`が1になっていればよいという事です。`get_rd_wen()`を以下のように改造します。

```verilog
function get_rd_wen(input [15:0] i_instr);
begin
  if(i_instr[3:0] <= 4'h8) begin
    get_rd_wen    = 1'b1;
  end else if(4'hA == i_instr[3:0]) begin
    get_rd_wen    = 1'b1;
  end else begin
    get_rd_wen    = 1'b0;
  end
end
endfunction
```

次はALUの制御信号である`o_alu_ctrl`に手を加えます。ALUの実装を覚えていますでしょうか。本実装では、Z16の演算命令のオペコードとALUの制御信号の値を一致さています。つまり演算命令をALUに実行させたい場合はオペコードをALUに入力すればよいという事です。よって`get_alu_ctrl()`を改造し、オペコードが演算命令のものの場合は`o_alu_ctrl`にオペコードの値をそのまま入力するようにします。

```verilog
function [3:0] get_alu_ctrl(input [15:0] i_instr);
begin
  if(i_instr[3:0] <= 4'h8) begin
    get_alu_ctrl  = i_instr[3:0];
  end else begin
    get_alu_ctrl  = 4'h0;
  end
end
endfunction
```

以上でデコーダの改造は終わりです。RS1, RS2, RDのアドレスを取り出す機構もLOAD命令とSTORE命令を実装する際に実装できていますので、少しの改造で済みましたね。

以下に改造が完了したデコーダを載せておきます。

```verilog
module Z16Decoder(
  input  wire   [15:0]  i_instr,
  output wire   [3:0]   o_opcode,
  output wire   [3:0]   o_rd_addr,
  output wire   [3:0]   o_rs1_addr,
  output wire   [3:0]   o_rs2_addr,
  output wire   [15:0]  o_imm,
  output wire           o_rd_wen,
  output wire           o_mem_wen,
  output wire   [3:0]   o_alu_ctrl
);

  assign o_opcode   = i_instr[3:0];
  assign o_rd_addr  = i_instr[7:4];
  assign o_rs1_addr = i_instr[11:8];
  assign o_rs2_addr = i_instr[15:12];
  assign o_imm      = get_imm(i_instr);
  assign o_rd_wen   = get_rd_wen(i_instr);
  assign o_mem_wen  = get_mem_wen(i_instr);
  assign o_alu_ctrl = get_alu_ctrl(i_instr);

  // 符号拡張
  function [15:0] get_imm(input [15:0] i_instr);
  begin
    case(i_instr[3:0])
      4'hA  : get_imm   = {i_instr[15] ? 12'hFFF : 12'h000, i_instr[15:12]};
      4'hB  : get_imm   = {i_instr[7]  ? 12'hFFF : 12'h000, i_instr[7:4]};
      default : get_imm = 16'h0000;
    endcase
  end
  endfunction

  function get_rd_wen(input [15:0] i_instr);
  begin
    if(i_instr[3:0] <= 4'h8) begin
      get_rd_wen    = 1'b1;
    end else if(4'hA == i_instr[3:0]) begin
      get_rd_wen    = 1'b1;
    end else begin
      get_rd_wen    = 1'b0;
    end
  end
  endfunction

  function get_mem_wen(input [15:0] i_instr);
  begin
    // STORE命令の場合にのみ1
    if(4'hB == i_instr[3:0]) begin
      get_mem_wen = 1'b1;
    end else begin
      get_mem_wen = 1'b0;
    end
  end
  endfunction

  function [3:0] get_alu_ctrl(input [15:0] i_instr);
  begin
    if(i_instr[3:0] <= 4'h8) begin
      get_alu_ctrl  = i_instr[3:0];
    end else begin
      get_alu_ctrl  = 4'h0;
    end
  end
  endfunction

endmodule
```

##### ALUで演算

デコーダの改造が完了したら次はALUで演算を行うデータパスを実装しましょう。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/path_exe_alu.png)

現状の`Z16CPU.v`の実装を見てみると、ALUのデータ入力には`w_rs1_data`と`w_imm`が入力されています。演算命令は`w_rs1_data`と`w_rs2_data`の値で演算を行う命令ですので、ALUに`w_rs2_data`を接続する必要があります。しかしながら`w_imm`を`w_rs2_data`に置き換えるだけでは先程実装したLOAD命令とSTORE命令が実行できなくなります。そこで、フェッチされた命令によってALUの`i_data_b`に入力される信号が変わるようにしましょう。

```verilog
  Z16ALU ALU(
    .i_data_a   (w_rs1_data ),
    .i_data_b   (w_imm      ),
    .i_ctrl     (w_alu_ctrl ),
    .o_data     (w_alu_data )
  );
```

まずは`w_opcode`と`w_data_b`という信号線を`Z16CPU.v`に新たに定義します。

```verilog
  wire          w_mem_wen;
  wire [3:0]    w_alu_ctrl;
  wire [3:0]    w_opcode;

  wire [15:0]   w_rs1_data;
  wire [15:0]   w_rs2_data;
  
  wire [15:0]   w_data_b;
  wire [15:0]   w_alu_data;
```

そしてデコーダにある今まで使ってこなかった信号線である`o_opcode`の出力信号を`w_opcode`に接続します。

```verilog
  Z16Decoder Decoder(
    .i_instr    (w_instr    ),
    .o_opcode   (w_opcode   ),
    .o_rd_addr  (w_rd_addr  ),
    .o_rs1_addr (w_rs1_addr ),
    .o_rs2_addr (w_rs2_addr ),
    .o_imm      (w_imm      ),
    .o_rd_wen   (w_rd_wen   ),
    .o_mem_wen  (w_mem_wen  ),
    .o_alu_ctrl (w_alu_ctrl )
  );
```

そして`w_opcode`の値を利用して、`w_opcode`が演算命令の場合は`w_data_b`に`w_rs2_data`値が入力され、それ以外の場合は`w_imm`の値が入力されるようにします。

```verilog
  assign w_data_b = (w_opcode <= 8'h8) ? w_rs2_data : w_imm;
  Z16ALU ALU(
    .i_data_a   (w_rs1_data ),
    .i_data_b   (w_data_b   ),
    .i_ctrl     (w_alu_ctrl ),
    .o_data     (w_alu_data )
  );
```

##### レジスタへの演算結果書き込み

あと残っているのはレジスタファイルへの演算結果の書き込みのデータパスです。やっていきましょう。

レジスタファイルへの値書き込みですが、現状の`Z16CPU.v`の実装を見てみると、レジスタファイルのデータ入力である`i_rd_data`にはメモリからのデータ信号である`w_mem_rdata`が直接接続されていますね。これではALUの演算結果をレジスタに書き込めません。

```verilog
  Z16RegisterFile RegFile(
    .i_clk      (i_clk      ),
    .i_rs1_addr (w_rs1_addr ),
    .o_rs1_data (w_rs1_data ),
    .i_rs2_addr (w_rs2_addr ),
    .o_rs2_data (w_rs2_data ),
    .i_rd_data  (w_mem_rdata),
    .i_rd_addr  (w_rd_addr  ),
    .i_rd_wen   (w_rd_wen   )
  );
```

よって対応が必要になります。まずはレジスタファイルへの書き込みデータ信号線である`w_rd_data`を新たに定義し、レジスタファイルのデータ書き込みに接続しましょう。

```verilog
  wire [15:0]   w_rs1_data;
  wire [15:0]   w_rs2_data;
  wire [15:0]   w_rd_data;
  
  wire [15:0]   w_data_b;
  wire [15:0]   w_alu_data;
```

そしたら以下のように`i_rd_data`に`w_rd_data`を接続し、フェッチされた命令がLOAD命令の場合は`w_mem_rdata`が`w_rd_addr`に入力され、それ以外の場合は`w_alu_data`が入力されるようにします。

```verilog
  assign w_rd_data = (w_opcode[3:0] == 4'hA) ? w_mem_rdata : w_alu_data;
  Z16RegisterFile RegFile(
    .i_clk      (i_clk      ),
    .i_rs1_addr (w_rs1_addr ),
    .o_rs1_data (w_rs1_data ),
    .i_rs2_addr (w_rs2_addr ),
    .o_rs2_data (w_rs2_data ),
    .i_rd_data  (w_rd_data  ),
    .i_rd_addr  (w_rd_addr  ),
    .i_rd_wen   (w_rd_wen   )
  );
```

これで演算命令のデータバスの実装が完了しました。全体のコードが以下の通りです。

```verilog
module Z16CPU(
  input  wire   i_clk,
  input  wire   i_rst
);

  // Program Counter
  reg   [15:0]  r_pc;

  wire  [15:0]  w_instr;
  wire [3:0]    w_rd_addr;
  wire [3:0]    w_rs1_addr;
  wire [3:0]    w_rs2_addr;
  wire [15:0]   w_imm;
  wire          w_rd_wen;
  wire          w_mem_wen;
  wire [3:0]    w_alu_ctrl;
  wire [3:0]    w_opcode;

  wire [15:0]   w_rs1_data;
  wire [15:0]   w_rs2_data;
  wire [15:0]   w_rd_data;
  
  wire [15:0]   w_data_b;
  wire [15:0]   w_alu_data;

  wire [15:0]   w_mem_rdata;

  always @(posedge i_clk) begin
    if(i_rst) begin
      r_pc  <= 16'h0000;
    end else begin
      r_pc  <= r_pc + 16'h0002;
    end
  end

  Z16InstrMemory InstrMem(
    .i_addr     (r_pc   ),
    .o_instr    (w_instr)
  );

  Z16Decoder Decoder(
    .i_instr    (w_instr    ),
    .o_opcode   (w_opcode   ),
    .o_rd_addr  (w_rd_addr  ),
    .o_rs1_addr (w_rs1_addr ),
    .o_rs2_addr (w_rs2_addr ),
    .o_imm      (w_imm      ),
    .o_rd_wen   (w_rd_wen   ),
    .o_mem_wen  (w_mem_wen  ),
    .o_alu_ctrl (w_alu_ctrl )
  );

  assign w_rd_data = (w_opcode[3:0] == 4'hA) ? w_mem_rdata : w_alu_data;
  Z16RegisterFile RegFile(
    .i_clk      (i_clk      ),
    .i_rs1_addr (w_rs1_addr ),
    .o_rs1_data (w_rs1_data ),
    .i_rs2_addr (w_rs2_addr ),
    .o_rs2_data (w_rs2_data ),
    .i_rd_data  (w_rd_data  ),
    .i_rd_addr  (w_rd_addr  ),
    .i_rd_wen   (w_rd_wen   )
  );

  assign w_data_b = (w_opcode <= 8'h8) ? w_rs2_data : w_imm;
  Z16ALU ALU(
    .i_data_a   (w_rs1_data ),
    .i_data_b   (w_data_b   ),
    .i_ctrl     (w_alu_ctrl ),
    .o_data     (w_alu_data )
  );

  Z16DataMemory DataMem(
    .i_clk  (i_clk      ),
    .i_addr (w_alu_data ),
    .i_wen  (w_mem_wen  ),
    .i_data (w_rs2_data ),
    .o_data (w_mem_rdata)
  );

endmodule
```

動作の確認は次の即値命令を実装してからにしましょう

#### 即値命令

続いて即値命令のデータパスを実装しましょう。

即値命令の動作を箇条書きすると以下の通りになります。

1. 命令メモリから命令をフェッチ
2. デコーダが命令を解釈
3. レジスタからRDの値を取り出す
4. ALUで演算
5. レジスタに演算結果を書き込む

##### デコーダを改造

ではデコーダの改造に着手します。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/path_decoder.png)

Z16の即値命令であるADDIは、RDの値に即値を加算してRDに格納する命令でした。

$$
\text{IMM} + \text{RD} \rightarrow \text{RD}
$$

現状、レジスタファイルから値を読み出すには`o_rs1_addr`か`o_rs2_addr`にアドレスを入力する必要があります。そこで今回はRDの値をRS1のポートから取り出す事にします。

そのため、`get_rs1_addr()`という関数を新たに定義し、即値命令のオペコードの場合は`o_rs1_addr`にRDのアドレスを入力し、それ以外の場合はRS1のアドレスを入力するようにします。

```verilog
  assign o_opcode   = i_instr[3:0];
  assign o_rd_addr  = i_instr[7:4];
  assign o_rs1_addr = get_rs1_addr(i_instr);
  assign o_rs2_addr = i_instr[15:12];
  assign o_imm      = get_imm(i_instr);
  assign o_rd_wen   = get_rd_wen(i_instr);
  assign o_mem_wen  = get_mem_wen(i_instr);
  assign o_alu_ctrl = get_alu_ctrl(i_instr);

  function [3:0] get_rs1_addr(input [15:0] i_instr);
  begin
    case(i_instr[3:0])
      4'h9  :   get_rs1_addr    = i_instr[7:4];
      default : get_rs1_addr    = i_instr[11:8];
    endcase
  end
  endfunction
```

次に即値に対処します。ADDIのビットフィールドには、`[15:8]`に符号付き8bitの即値が存在していました。

`MSB | imm[7:0] | rd[3:0] | opcode[3:0] | LSB`

デコーダの`get_imm()`を改造し、ADDI命令の場合は符号付き8bitの即値を16bitに符号拡張するようにしましょう。

```verilog
// 符号拡張
function [15:0] get_imm(input [15:0] i_instr);
begin
  case(i_instr[3:0])
    4'h9  : get_imm   = {i_instr[15] ? 8'hFF : 8'h00, i_instr[15:8]};
    4'hA  : get_imm   = {i_instr[15] ? 12'hFFF : 12'h000, i_instr[15:12]};
    4'hB  : get_imm   = {i_instr[7]  ? 12'hFFF : 12'h000, i_instr[7:4]};
    default : get_imm = 16'h0000;
  endcase
end
endfunction
```

そして、ADDI命令は演算命令と同じくレジスタファイルに値を書き込む命令ですので、`get_rd_wen()`を改造し、ADDI命令の場合は`o_rd_wen`を1にするようにします。

```verilog
function get_rd_wen(input [15:0] i_instr);
begin
  if(i_instr[3:0] <= 4'h8) begin
    get_rd_wen    = 1'b1;
  end else if(4'h9 == i_instr[3:0]) begin
    get_rd_wen    = 1'b1;
  end else if(4'hA == i_instr[3:0]) begin
    get_rd_wen    = 1'b1;
  end else begin
    get_rd_wen    = 1'b0;
  end
end
endfunction
```

上記の記述だと冗長ですので、以下のように纏めてしまいましょう。

```verilog
function get_rd_wen(input [15:0] i_instr);
begin
  if(i_instr[3:0] <= 4'hA) begin
    get_rd_wen    = 1'b1;
  end else begin
    get_rd_wen    = 1'b0;
  end
end
endfunction
```

またALUの制御信号ですが、即値命令はADDIだけであり、既に演算命令以外ではALUに加算を行わせるようにしていますので、`get_alu_ctrl()`の改造の必要はありません。

```verilog
function [3:0] get_alu_ctrl(input [15:0] i_instr);
begin
  if(i_instr[3:0] <= 4'h8) begin
    get_alu_ctrl  = i_instr[3:0];
  end else begin
    get_alu_ctrl  = 4'h0;
  end
end
endfunction
```

これでデコーダの改造が完了しました。改造が完了したデコーダを以下に載せておきます。

```verilog
module Z16Decoder(
  input  wire   [15:0]  i_instr,
  output wire   [3:0]   o_opcode,
  output wire   [3:0]   o_rd_addr,
  output wire   [3:0]   o_rs1_addr,
  output wire   [3:0]   o_rs2_addr,
  output wire   [15:0]  o_imm,
  output wire           o_rd_wen,
  output wire           o_mem_wen,
  output wire   [3:0]   o_alu_ctrl
);

  assign o_opcode   = i_instr[3:0];
  assign o_rd_addr  = i_instr[7:4];
  assign o_rs1_addr = get_rs1_addr(i_instr);
  assign o_rs2_addr = i_instr[15:12];
  assign o_imm      = get_imm(i_instr);
  assign o_rd_wen   = get_rd_wen(i_instr);
  assign o_mem_wen  = get_mem_wen(i_instr);
  assign o_alu_ctrl = get_alu_ctrl(i_instr);

  function [3:0] get_rs1_addr(input [15:0] i_instr);
  begin
    case(i_instr[3:0])
      4'h9  :   get_rs1_addr    = i_instr[7:4];
      default : get_rs1_addr    = i_instr[11:8];
    endcase
  end
  endfunction

  // 符号拡張
  function [15:0] get_imm(input [15:0] i_instr);
  begin
    case(i_instr[3:0])
      4'h9  : get_imm   = {i_instr[15] ? 8'hFF : 8'h00, i_instr[15:8]};
      4'hA  : get_imm   = {i_instr[15] ? 12'hFFF : 12'h000, i_instr[15:12]};
      4'hB  : get_imm   = {i_instr[7]  ? 12'hFFF : 12'h000, i_instr[7:4]};
      default : get_imm = 16'h0000;
    endcase
  end
  endfunction

  function get_rd_wen(input [15:0] i_instr);
  begin
    if(i_instr[3:0] <= 4'hA) begin
      get_rd_wen    = 1'b1;
    end else begin
      get_rd_wen    = 1'b0;
    end
  end
  endfunction

  function get_mem_wen(input [15:0] i_instr);
  begin
    // STORE命令の場合にのみ1
    if(4'hB == i_instr[3:0]) begin
      get_mem_wen = 1'b1;
    end else begin
      get_mem_wen = 1'b0;
    end
  end
  endfunction

  function [3:0] get_alu_ctrl(input [15:0] i_instr);
  begin
    if(i_instr[3:0] <= 4'h8) begin
      get_alu_ctrl  = i_instr[3:0];
    end else begin
      get_alu_ctrl  = 4'h0;
    end
  end
  endfunction

endmodule
```

実はデコーダの改造が完了した時点で、即値命令のデータパスの実装は完了しています。今までの実装が生きましたね。

##### 動作の確認

それでは先程実装した演算命令と即値命令の動作確認を行っていきましょう。

動作確認用のプログラムとして、ADDIの説明の際に用いたサンプルプログラムを改造して使いましょう。

$$
\begin{align}
x &= 120 \\
y &= 87 \\
f(x, y) &= 3\times x + 100 + y
\end{align}
$$

以下のプログラムはG0とG1とG4を0で初期化した後、G0に120を格納、G1に87を格納して計算を行い、計算結果をメモリのアドレス0に格納しています。

```
ADD ZR ZR G0
ADD ZR ZR G1
ADD ZR ZR G4
ADDI 120 G0
ADDI 87 G1
ADDI 3 G4
MUL G4 G0 G2
ADDI 100 G1
ADD G2 G1 G7
STORE G7 ZR 0
```

このプログラムをアセンブルしてバイナリを命令メモリに格納します。

```verilog
module Z16InstrMemory(
  input  wire           i_clk,
  input  wire   [15:0]  i_addr,
  output wire   [15:0]  o_instr
);

  wire [15:0] mem[10:0];

  assign o_instr = mem[i_addr[15:1]];

  assign mem[0] = 16'h0040; // ADD ZR ZR G0
  assign mem[1] = 16'h0050; // ADD ZR ZR G1
  assign mem[2] = 16'h0080; // ADD ZR ZR G4
  assign mem[3] = 16'h7849; // ADDI 120 G0
  assign mem[4] = 16'h5759; // ADDI 87 G1
  assign mem[5] = 16'h0389; // ADDI 3 G4
  assign mem[6] = 16'h8462; // MUL G4 G0 G2
  assign mem[7] = 16'h6459; // ADDI 100 G1
  assign mem[8] = 16'h65B0; // ADD G2 G1 G7
  assign mem[9] = 16'hB00B; // STORE G7 ZR 0
  assign mem[10] = 16'h0000; 

endmodule
```

テストベンチに変更を行う必要はありません。以下のコマンドでシミュレーションを行い、波形を見ることが出来ます。

```bash
iverilog Z16CPU_tb.v Z16ALU.v Z16CPU.v Z16DataMemory.v Z16Decoder.sv Z16InstrMemory.v Z16RegisterFile.v
vvp a.out
gtkwave wave.vcd
```

以下の波形が実際にシミュレーションを行った結果です。データメモリの信号を見てみると、後半にメモリのアドレス0に16進数で`16'h0223`が書き込まれています。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/wave_imm.png)

これを10進数に直すと547であり、上記の数式の計算結果と一致しています。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/cal_imm.png)

これで我々のCPUは演算命令と即値命令を実行できることが確認できました。やったね。

#### ジャンプ命令

どんどん行きましょう。次はジャンプ命令のデータパスを作成していきます。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/path_jump.png)

ジャンプ命令の動作を箇条書すると以下の通りになります。

1. 命令メモリから命令をフェッチ
2. デコーダが命令を解釈
3. レジスタからRS1の値を読み出し
4. ALUでジャンプ先アドレスを計算
5. レジスタにPC + 2の値を書き込み
6. PCに書き込み

やることは変わりません。まずはデコーダを改造しましょう。

##### デコーダの改造

ではデコーダの改造に着手します。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/path_decoder.png)

ジャンプ命令のビットフィールドは以下のように、MSBから4bitに符号付き4bit即値が存在しています。

`MSB | imm[3:0] | rs1[3:0] | rd[3:0] | opcode[3:0] | LSB`

この符号付き4bitを符号付き16bitに符号拡張してやる必要があります。という訳でデコーダの`get_imm()`を改造してジャンプ命令の場合に`w_instr[15:12]`を符号拡張するようにしましょう。

```verilog
// 符号拡張
function [15:0] get_imm(input [15:0] i_instr);
begin
  case(i_instr[3:0])
    4'h9  : get_imm   = {i_instr[15] ? 8'hFF : 8'h00, i_instr[15:8]};
    4'hA  : get_imm   = {i_instr[15] ? 12'hFFF : 12'h000, i_instr[15:12]};
    4'hB  : get_imm   = {i_instr[7]  ? 12'hFFF : 12'h000, i_instr[7:4]};
    4'hC  : get_imm   = {i_instr[15] ? 12'hFFF : 12'h000, i_instr[15:12]};
    4'hD  : get_imm   = {i_instr[15] ? 12'hFFF : 12'h000, i_instr[15:12]};
    default : get_imm = 16'h0000;
  endcase
end
endfunction
```

ジャンプ命令のJALとJRLの両方ともRDにPC + 2の値を格納する動作をしますので、ジャンプ命令のオペコードの場合は`w_rd_wen`を1にする必要があります。

- JAL

$$
\begin{align}
\text{PC} + 2 &\rightarrow \text{RD} \\
\text{IMM} + \text{RS1} &\rightarrow \text{PC}
\end{align}
$$

- JRL

$$
\begin{align}
\text{PC} + 2 &\rightarrow \text{RD} \\
\text{IMM} + \text{RS1} + \text{PC} &\rightarrow \text{PC}
\end{align}
$$

よって`get_rd_wen()`を改造して対応します。

```verilog
function get_rd_wen(input [15:0] i_instr);
begin
  if(i_instr[3:0] <= 4'hA) begin
    get_rd_wen    = 1'b1;
  end else if((i_instr[3:0] == 4'hC) || (i_instr[3:0] == 4'hD)) begin
    get_rd_wen    = 1'b1;
  end else begin
    get_rd_wen    = 1'b0;
  end
end
endfunction
```

`get_mem_wen()`に関しては、ジャンプ命令はメモリに値を書き込みませんので手を加える必要はありません。また`get_alu_ctrl()`に関しては、ジャンプ命令先のアドレス計算には加算しか用いませんので、従来の演算命令以外は0を返す実装に手を加える必要はありません。

よってALUでジャンプ先のアドレスを計算するデータパスまでは完成しました。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/path_jump_caladdr.png)

以下に改造が完了したデコーダの全体を載せておきます。

```verilog
module Z16Decoder(
  input  wire   [15:0]  i_instr,
  output wire   [3:0]   o_opcode,
  output wire   [3:0]   o_rd_addr,
  output wire   [3:0]   o_rs1_addr,
  output wire   [3:0]   o_rs2_addr,
  output wire   [15:0]  o_imm,
  output wire           o_rd_wen,
  output wire           o_mem_wen,
  output wire   [3:0]   o_alu_ctrl
);

  assign o_opcode   = i_instr[3:0];
  assign o_rd_addr  = i_instr[7:4];
  assign o_rs1_addr = get_rs1_addr(i_instr);
  assign o_rs2_addr = i_instr[15:12];
  assign o_imm      = get_imm(i_instr);
  assign o_rd_wen   = get_rd_wen(i_instr);
  assign o_mem_wen  = get_mem_wen(i_instr);
  assign o_alu_ctrl = get_alu_ctrl(i_instr);

  function [3:0] get_rs1_addr(input [15:0] i_instr);
  begin
    case(i_instr[3:0])
      4'h9  :   get_rs1_addr    = i_instr[7:4];
      default : get_rs1_addr    = i_instr[11:8];
    endcase
  end
  endfunction

  // 符号拡張
  function [15:0] get_imm(input [15:0] i_instr);
  begin
    case(i_instr[3:0])
      4'h9  : get_imm   = {i_instr[15] ? 8'hFF : 8'h00, i_instr[15:8]};
      4'hA  : get_imm   = {i_instr[15] ? 12'hFFF : 12'h000, i_instr[15:12]};
      4'hB  : get_imm   = {i_instr[7]  ? 12'hFFF : 12'h000, i_instr[7:4]};
      4'hC  : get_imm   = {i_instr[15] ? 12'hFFF : 12'h000, i_instr[15:12]};
      4'hD  : get_imm   = {i_instr[15] ? 12'hFFF : 12'h000, i_instr[15:12]};
      default : get_imm = 16'h0000;
    endcase
  end
  endfunction

  function get_rd_wen(input [15:0] i_instr);
  begin
    if(i_instr[3:0] <= 4'hA) begin
      get_rd_wen    = 1'b1;
    end else if((i_instr[3:0] == 4'hC) || (i_instr[3:0] == 4'hD)) begin
      get_rd_wen    = 1'b1;
    end else begin
      get_rd_wen    = 1'b0;
    end
  end
  endfunction

  function get_mem_wen(input [15:0] i_instr);
  begin
    // STORE命令の場合にのみ1
    if(4'hB == i_instr[3:0]) begin
      get_mem_wen = 1'b1;
    end else begin
      get_mem_wen = 1'b0;
    end
  end
  endfunction

  function [3:0] get_alu_ctrl(input [15:0] i_instr);
  begin
    if(i_instr[3:0] <= 4'h8) begin
      get_alu_ctrl  = i_instr[3:0];
    end else begin
      get_alu_ctrl  = 4'h0;
    end
  end
  endfunction

endmodule
```

##### レジスタにPC + 2の値を格納

次はレジスタのRDにPC + 2の値を格納するデータパスを実装しましょう。

従来のデータパスでは、RDに格納する値はLOAD命令でメモリから読み出したデータの`w_mem_rdata`か、演算命令か即値命令で計算した`w_alu_data`でした。ここにジャンプ命令のオペコードの場合はPC + 2の値を格納するようにする必要があります。

```verilog
  assign w_rd_data = (w_opcode == 4'hA) ? w_mem_rdata : w_alu_data;
  Z16RegisterFile RegFile(
    .i_clk      (i_clk      ),
    .i_rs1_addr (w_rs1_addr ),
    .o_rs1_data (w_rs1_data ),
    .i_rs2_addr (w_rs2_addr ),
    .o_rs2_data (w_rs2_data ),
    .i_rd_data  (w_rd_data  ),
    .i_rd_addr  (w_rd_addr  ),
    .i_rd_wen   (w_rd_wen   )
  );
```

そこで新たな関数である`select_rd_data()`を定義し、オペコードによってRDに格納するデータを決められるようにしましょう。

オペコードがLOAD命令の場合は`w_mem_rdata`を入力し、ジャンプ命令の場合は`r_pc + 16'h0002`の値を入力し、それ以外の場合は`w_alu_data`の値を入力するようにします。

```verilog
  assign w_rd_data = select_rd_data(w_opcode, w_mem_rdata, r_pc, w_alu_data);

  function [15:0] select_rd_data(
    input   [3:0]   i_opcode,
    input   [15:0]  i_mem_rdata,
    input   [15:0]  i_pc,
    input   [15:0]  i_alu_data
  );
  begin
    case(i_opcode)
      4'hA  : select_rd_data    = i_mem_rdata;
      4'hC  : select_rd_data    = i_pc + 16'h0002;
      4'hD  : select_rd_data    = i_pc + 16'h0002;
      default : select_rd_data  = i_alu_data;
    endcase
  end
  endfunction

  Z16RegisterFile RegFile(
    .i_clk      (i_clk      ),
    .i_rs1_addr (w_rs1_addr ),
    .o_rs1_data (w_rs1_data ),
    .i_rs2_addr (w_rs2_addr ),
    .o_rs2_data (w_rs2_data ),
    .i_rd_data  (w_rd_data  ),
    .i_rd_addr  (w_rd_addr  ),
    .i_rd_wen   (w_rd_wen   )
  );
```

##### PCにジャンプ先のアドレス書き込み

次にプログラムカウンタにジャンプ先のアドレスを書き込む機構を実装します。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/path_jump.png)

今までは`r_pc`に対してリセット時以外はひたすらカウントアップしていく機構が組まれていましたが、ここにジャンプ命令のオペコードに対応して`r_pc`の値を変更する機構を追加します。

```verilog
  always @(posedge i_clk) begin
    if(i_rst) begin
      r_pc  <= 16'h0000;
    end else if(w_opcode == 4'hC) begin // JAL
      r_pc  <= w_alu_data;
    end else if(w_opcode == 4'hD) begin // JRL
      r_pc  <= r_pc + w_alu_data;
    end else begin
      r_pc  <= r_pc + 16'h0002;
    end
  end
```

JALの倍は絶対ジャンプなのでALUの計算結果を直接入力し、JRLは相対ジャンプなのでALUの計算結果をPCの値に加算しています。

これでジャンプ命令のデータパスの実装が完了しました。全体のコードを以下に載せておきます。

```verilog
module Z16CPU(
  input  wire   i_clk,
  input  wire   i_rst
);

  // Program Counter
  reg   [15:0]  r_pc;

  wire  [15:0]  w_instr;
  wire [3:0]    w_rd_addr;
  wire [3:0]    w_rs1_addr;
  wire [3:0]    w_rs2_addr;
  wire [15:0]   w_imm;
  wire          w_rd_wen;
  wire          w_mem_wen;
  wire [3:0]    w_alu_ctrl;
  wire [3:0]    w_opcode;

  wire [15:0]   w_rs1_data;
  wire [15:0]   w_rs2_data;
  wire [15:0]   w_rd_data;
  
  wire [15:0]   w_data_b;
  wire [15:0]   w_alu_data;

  wire [15:0]   w_mem_rdata;

  always @(posedge i_clk) begin
    if(i_rst) begin
      r_pc  <= 16'h0000;
    end else if(w_opcode == 4'hC) begin // JAL
      r_pc  <= w_alu_data;
    end else if(w_opcode == 4'hD) begin // JRL
      r_pc  <= r_pc + w_alu_data;
    end else begin
      r_pc  <= r_pc + 16'h0002;
    end
  end

  Z16InstrMemory InstrMem(
    .i_addr     (r_pc   ),
    .o_instr    (w_instr)
  );

  Z16Decoder Decoder(
    .i_instr    (w_instr    ),
    .o_opcode   (w_opcode   ),
    .o_rd_addr  (w_rd_addr  ),
    .o_rs1_addr (w_rs1_addr ),
    .o_rs2_addr (w_rs2_addr ),
    .o_imm      (w_imm      ),
    .o_rd_wen   (w_rd_wen   ),
    .o_mem_wen  (w_mem_wen  ),
    .o_alu_ctrl (w_alu_ctrl )
  );

  assign w_rd_data = select_rd_data(w_opcode, w_mem_rdata, r_pc, w_alu_data);

  function [15:0] select_rd_data(
    input   [3:0]   i_opcode,
    input   [15:0]  i_mem_rdata,
    input   [15:0]  i_pc,
    input   [15:0]  i_alu_data
  );
  begin
    case(i_opcode)
      4'hA  : select_rd_data    = i_mem_rdata;
      4'hC  : select_rd_data    = r_pc + 16'h0002;
      4'hD  : select_rd_data    = r_pc + 16'h0002;
      default : select_rd_data  = i_alu_data;
    endcase
  end
  endfunction

  Z16RegisterFile RegFile(
    .i_clk      (i_clk      ),
    .i_rs1_addr (w_rs1_addr ),
    .o_rs1_data (w_rs1_data ),
    .i_rs2_addr (w_rs2_addr ),
    .o_rs2_data (w_rs2_data ),
    .i_rd_data  (w_rd_data  ),
    .i_rd_addr  (w_rd_addr  ),
    .i_rd_wen   (w_rd_wen   )
  );

  assign w_data_b = (w_opcode <= 8'h8) ? w_rs2_data : w_imm;
  Z16ALU ALU(
    .i_data_a   (w_rs1_data ),
    .i_data_b   (w_data_b   ),
    .i_ctrl     (w_alu_ctrl ),
    .o_data     (w_alu_data )
  );

  Z16DataMemory DataMem(
    .i_clk  (i_clk      ),
    .i_addr (w_alu_data ),
    .i_wen  (w_mem_wen  ),
    .i_data (w_rs2_data ),
    .o_data (w_mem_rdata)
  );

endmodule
```

動作の確認に移りましょう。

##### 動作の確認

動作の確認を行うための以下のテストプログラムを使います。

```
0: ADD ZR ZR G0
2: JRL 6 ZR G1
4: ADD ZR ZR ZR
6: ADD ZR ZR ZR
8: JAL 0 ZR G2
```

このプログラムではアドレス2の相対ジャンプでアドレス8の命令にジャンプし、アドレス8の絶対ジャンプでアドレス0の命令にジャンプします。つまりPCの値が0 -> 2 -> 8 -> 0とループすれば理想の動作ということです。
このプログラムをアセンブルし、実際に動かしてみましょう。

まずは命令メモリにアセンブルしたバイナリを格納します。

```verilog
module Z16InstrMemory(
  input  wire           i_clk,
  input  wire   [15:0]  i_addr,
  output wire   [15:0]  o_instr
);

  wire [15:0] mem[10:0];

  assign o_instr = mem[i_addr[15:1]];

  assign mem[0] = 16'h0040; // ADD ZR ZR G0
  assign mem[1] = 16'h605D; // JRL 6 ZR G1
  assign mem[2] = 16'h0000; // ADD ZR ZR ZR
  assign mem[3] = 16'h0000; // ADD ZR ZR ZR
  assign mem[4] = 16'h006C; // JAL 0 ZR G2

endmodule
```

そして以下のコマンドでシミュレーションを走らせます。

```bash
iverilog Z16CPU_tb.v Z16ALU.v Z16CPU.v Z16DataMemory.v Z16Decoder.sv Z16InstrMemory.v Z16RegisterFile.v
vvp a.out
gtkwave wave.vcd
```

以下の波形が実際にシミュレーションを行った結果です。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/wave_jump.png)

`r_pc`の値を見てみると0 -> 2 -> 8 -> 0とループしており、またレジスタにはジャンプ命令の次の命令のアドレスが格納されている事が確認できます。

これでジャンプ命令のデータパスが完成しました。

#### 分岐命令

さて最後です。分岐命令のデータパスを実装しましょう。

分岐命令の動作を箇条書すると以下の通りになります。

1. 命令メモリから命令をフェッチ
2. デコーダが命令を解釈
3. レジスタからRS1, RS2の値を読み出し
4. 条件に合致する場合はPCに書き込み

意外と少ないですね。ご多分に漏れずデコーダの改造に着手しましょう。

##### デコーダの改造

ではデコーダの改造に着手します。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/path_decoder.png)

分岐命令のビットフィールドは他の命令と比べてかなり特異なものでした。

`MSB | imm[7:0] | rs2[1:0] | rs1[1:0] | opcode[3:0] | LSB`

RS1とRS2のアドレスが2bitづつしか割り当てられていません。よって分岐命令のオペコードの場合はこの2bitからレジスタの抜き出す必要があります。そこで、関数`get_rs1_addr()`に手を加えると同時に、RS2のアドレスを抜き出す関数である`get_rs2_addr()`を新たに定義します。

そしてそれぞれの関数では、分岐命令の場合は抜き出した2bitのアドレスに`2'b00`を結合し、4bitのアドレスを返すようにします。新たに定義した`get_rs2_addr()`の結果を`o_rs2_addr`に入力するのを忘れないようにしましょう。

```verilog
  assign o_opcode   = i_instr[3:0];
  assign o_rd_addr  = i_instr[7:4];
  assign o_rs1_addr = get_rs1_addr(i_instr);
  assign o_rs2_addr = get_rs2_addr(i_instr);
  assign o_imm      = get_imm(i_instr);
  assign o_rd_wen   = get_rd_wen(i_instr);
  assign o_mem_wen  = get_mem_wen(i_instr);
  assign o_alu_ctrl = get_alu_ctrl(i_instr);

  function [3:0] get_rs1_addr(input [15:0] i_instr);
  begin
    case(i_instr[3:0])
      4'h9  :   get_rs1_addr    = i_instr[7:4];
      4'hE  :   get_rs1_addr    = {2'b00, i_instr[5:4]};
      4'hF  :   get_rs1_addr    = {2'b00, i_instr[5:4]};
      default : get_rs1_addr    = i_instr[11:8];
    endcase
  end
  endfunction

  function [3:0] get_rs2_addr(input [15:0] i_instr);
  begin
    case(i_instr[3:0])
      4'hE  :   get_rs2_addr    = {2'b00, i_instr[7:6]};
      4'hF  :   get_rs2_addr    = {2'b00, i_instr[7:6]};
      default : get_rs2_addr    = i_instr[15:12];
    endcase
  end
  endfunction
```

次にPCに加算する為の即値を命令から取り出すために、`get_imm()`に手を加えます。

分岐命令のビットフィールドでは`i_instr[15:8]`の8bitが即値ですので、これを符号付き16bitに符号拡張します。

```verilog
  // 符号拡張
function [15:0] get_imm(input [15:0] i_instr);
begin
  case(i_instr[3:0])
    4'h9  : get_imm   = {i_instr[15] ? 8'hFF : 8'h00, i_instr[15:8]};
    4'hA  : get_imm   = {i_instr[15] ? 12'hFFF : 12'h000, i_instr[15:12]};
    4'hB  : get_imm   = {i_instr[7]  ? 12'hFFF : 12'h000, i_instr[7:4]};
    4'hC  : get_imm   = {i_instr[15] ? 12'hFFF : 12'h000, i_instr[15:12]};
    4'hD  : get_imm   = {i_instr[15] ? 12'hFFF : 12'h000, i_instr[15:12]};
    4'hE  : get_imm   = {i_instr[15] ? 8'hFF : 8'h00, i_instr[15:8]};
    4'hF  : get_imm   = {i_instr[15] ? 8'hFF : 8'h00, i_instr[15:8]};
    default : get_imm = 16'h0000;
  endcase
end
endfunction
```

分岐命令ではレジスタにもメモリにも値を書き込みませんし、ALUも利用しませんので他の関数に手を加える必要はありません。これでデコーダの改造は完了です、

デコーダの全体を以下に載せておきます。

```verilog
module Z16Decoder(
  input  wire   [15:0]  i_instr,
  output wire   [3:0]   o_opcode,
  output wire   [3:0]   o_rd_addr,
  output wire   [3:0]   o_rs1_addr,
  output wire   [3:0]   o_rs2_addr,
  output wire   [15:0]  o_imm,
  output wire           o_rd_wen,
  output wire           o_mem_wen,
  output wire   [3:0]   o_alu_ctrl
);

  assign o_opcode   = i_instr[3:0];
  assign o_rd_addr  = i_instr[7:4];
  assign o_rs1_addr = get_rs1_addr(i_instr);
  assign o_rs2_addr = get_rs2_addr(i_instr);
  assign o_imm      = get_imm(i_instr);
  assign o_rd_wen   = get_rd_wen(i_instr);
  assign o_mem_wen  = get_mem_wen(i_instr);
  assign o_alu_ctrl = get_alu_ctrl(i_instr);

  function [3:0] get_rs1_addr(input [15:0] i_instr);
  begin
    case(i_instr[3:0])
      4'h9  :   get_rs1_addr    = i_instr[7:4];
      4'hE  :   get_rs1_addr    = {2'b00, i_instr[5:4]};
      4'hF  :   get_rs1_addr    = {2'b00, i_instr[5:4]};
      default : get_rs1_addr    = i_instr[11:8];
    endcase
  end
  endfunction

  function [3:0] get_rs2_addr(input [15:0] i_instr);
  begin
    case(i_instr[3:0])
      4'hE  :   get_rs2_addr    = {2'b00, i_instr[7:6]};
      4'hF  :   get_rs2_addr    = {2'b00, i_instr[7:6]};
      default : get_rs2_addr    = i_instr[15:12];
    endcase
  end
  endfunction

  // 符号拡張
  function [15:0] get_imm(input [15:0] i_instr);
  begin
    case(i_instr[3:0])
      4'h9  : get_imm   = {i_instr[15] ? 8'hFF : 8'h00, i_instr[15:8]};
      4'hA  : get_imm   = {i_instr[15] ? 12'hFFF : 12'h000, i_instr[15:12]};
      4'hB  : get_imm   = {i_instr[7]  ? 12'hFFF : 12'h000, i_instr[7:4]};
      4'hC  : get_imm   = {i_instr[15] ? 12'hFFF : 12'h000, i_instr[15:12]};
      4'hD  : get_imm   = {i_instr[15] ? 12'hFFF : 12'h000, i_instr[15:12]};
      4'hE  : get_imm   = {i_instr[15] ? 8'hFF : 8'h00, i_instr[15:8]};
      4'hF  : get_imm   = {i_instr[15] ? 8'hFF : 8'h00, i_instr[15:8]};
      default : get_imm = 16'h0000;
    endcase
  end
  endfunction

  function get_rd_wen(input [15:0] i_instr);
  begin
    if(i_instr[3:0] <= 4'hA) begin
      get_rd_wen    = 1'b1;
    end else if((i_instr[3:0] == 4'hC) || (i_instr[3:0] == 4'hD)) begin
      get_rd_wen    = 1'b1;
    end else begin
      get_rd_wen    = 1'b0;
    end
  end
  endfunction

  function get_mem_wen(input [15:0] i_instr);
  begin
    // STORE命令の場合にのみ1
    if(4'hB == i_instr[3:0]) begin
      get_mem_wen = 1'b1;
    end else begin
      get_mem_wen = 1'b0;
    end
  end
  endfunction

  function [3:0] get_alu_ctrl(input [15:0] i_instr);
  begin
    if(i_instr[3:0] <= 4'h8) begin
      get_alu_ctrl  = i_instr[3:0];
    end else begin
      get_alu_ctrl  = 4'h0;
    end
  end
  endfunction

endmodule
```

##### 条件に合致する場合はPCに書き込み

デコーダが完成しましたら、次はPCの機構に手を加えていきます。

以下のように分岐命令のオペコードの場合はRS1とRS2の値を比較し、条件に合致する場合はプログラムカウンタに即値の値を加算します。

```verilog
  always @(posedge i_clk) begin
    if(i_rst) begin
      r_pc  <= 16'h0000;
    end else if(w_opcode == 4'hC) begin // JAL
      r_pc  <= w_alu_data;
    end else if(w_opcode == 4'hD) begin // JRL
      r_pc  <= r_pc + w_alu_data;
    end else if((w_opcode == 4'hE) && (w_rs2_data == w_rs1_data)) begin // BEQ
      r_pc  <= r_pc + w_imm;
    end else if((w_opcode == 4'hF) && (w_rs2_data > w_rs1_data)) begin // BLT
      r_pc  <= r_pc + w_imm;
    end else begin
      r_pc  <= r_pc + 16'h0002;
    end
  end
```

やることは以上です。完成したCPUのコードを以下に載せておきます。

```verilog
module Z16CPU(
  input  wire   i_clk,
  input  wire   i_rst
);

  // Program Counter
  reg   [15:0]  r_pc;

  wire  [15:0]  w_instr;
  wire [3:0]    w_rd_addr;
  wire [3:0]    w_rs1_addr;
  wire [3:0]    w_rs2_addr;
  wire [15:0]   w_imm;
  wire          w_rd_wen;
  wire          w_mem_wen;
  wire [3:0]    w_alu_ctrl;
  wire [3:0]    w_opcode;

  wire [15:0]   w_rs1_data;
  wire [15:0]   w_rs2_data;
  wire [15:0]   w_rd_data;
  
  wire [15:0]   w_data_b;
  wire [15:0]   w_alu_data;

  wire [15:0]   w_mem_rdata;

  always @(posedge i_clk) begin
    if(i_rst) begin
      r_pc  <= 16'h0000;
    end else if(w_opcode == 4'hC) begin // JAL
      r_pc  <= w_alu_data;
    end else if(w_opcode == 4'hD) begin // JRL
      r_pc  <= r_pc + w_alu_data;
    end else if((w_opcode == 4'hE) && (w_rs2_data == w_rs1_data)) begin // BEQ
      r_pc  <= r_pc + w_imm;
    end else if((w_opcode == 4'hF) && (w_rs2_data > w_rs1_data)) begin // BLT
      r_pc  <= r_pc + w_imm;
    end else begin
      r_pc  <= r_pc + 16'h0002;
    end
  end

  Z16InstrMemory InstrMem(
    .i_addr     (r_pc   ),
    .o_instr    (w_instr)
  );

  Z16Decoder Decoder(
    .i_instr    (w_instr    ),
    .o_opcode   (w_opcode   ),
    .o_rd_addr  (w_rd_addr  ),
    .o_rs1_addr (w_rs1_addr ),
    .o_rs2_addr (w_rs2_addr ),
    .o_imm      (w_imm      ),
    .o_rd_wen   (w_rd_wen   ),
    .o_mem_wen  (w_mem_wen  ),
    .o_alu_ctrl (w_alu_ctrl )
  );

  assign w_rd_data = select_rd_data(w_opcode, w_mem_rdata, r_pc, w_alu_data);

  function [15:0] select_rd_data(
    input   [3:0]   i_opcode,
    input   [15:0]  i_mem_rdata,
    input   [15:0]  i_pc,
    input   [15:0]  i_alu_data
  );
  begin
    case(i_opcode)
      4'hA  : select_rd_data    = i_mem_rdata;
      4'hC  : select_rd_data    = r_pc + 16'h0002;
      4'hD  : select_rd_data    = r_pc + 16'h0002;
      default : select_rd_data  = i_alu_data;
    endcase
  end
  endfunction

  Z16RegisterFile RegFile(
    .i_clk      (i_clk      ),
    .i_rs1_addr (w_rs1_addr ),
    .o_rs1_data (w_rs1_data ),
    .i_rs2_addr (w_rs2_addr ),
    .o_rs2_data (w_rs2_data ),
    .i_rd_data  (w_rd_data  ),
    .i_rd_addr  (w_rd_addr  ),
    .i_rd_wen   (w_rd_wen   )
  );

  assign w_data_b = (w_opcode <= 8'h8) ? w_rs2_data : w_imm;
  Z16ALU ALU(
    .i_data_a   (w_rs1_data ),
    .i_data_b   (w_data_b   ),
    .i_ctrl     (w_alu_ctrl ),
    .o_data     (w_alu_data )
  );

  Z16DataMemory DataMem(
    .i_clk  (i_clk      ),
    .i_addr (w_alu_data ),
    .i_wen  (w_mem_wen  ),
    .i_data (w_rs2_data ),
    .o_data (w_mem_rdata)
  );

endmodule
```

これでZ16のCPUは完成しました。しかしながら安心してはいけません。動作の確認が完了して初めて完成と言えます。検証してきましょう。

##### 動作の確認

分岐命令の動作の確認の為に、以下の1~10の総和を求めるプログラムを用いましょう。

以下のプログラムでは、最初にB1とB2を0で初期化し、総和を求めた後アドレスCでジャンプ命令を用いて停止します。

```
0: ADD ZR ZR B1
2: ADD ZR ZR B2
4: ADDI 10 B1
6: ADD B1 B2 B2
8: ADDI -1 B1
A: BLT B1 ZR -4
C: JRL 0 ZR G11
```

アセンブルしてバイナリを命令メモリに書き込みます。

```verilog
module Z16InstrMemory(
  input  wire           i_clk,
  input  wire   [15:0]  i_addr,
  output wire   [15:0]  o_instr
);

  wire [15:0] mem[10:0];

  assign o_instr = mem[i_addr[15:1]];

  assign mem[0] = 16'h0010; // ADD ZR ZR B1
  assign mem[1] = 16'h0020; // ADD ZR ZR B2
  assign mem[2] = 16'h0A19; // ADDI 10 B1
  assign mem[3] = 16'h1220; // ADD B1 B2 B2
  assign mem[4] = 16'hFF19; // ADDI -1 B1
  assign mem[5] = 16'hFC4F; // BLT B1 ZR -4
  assign mem[6] = 16'h00FD; // JRL 0 ZR G11

endmodule
```

そして以下のコマンドでシミュレーションを実行します。

```bash
iverilog Z16CPU_tb.v Z16ALU.v Z16CPU.v Z16DataMemory.v Z16Decoder.sv Z16InstrMemory.v Z16RegisterFile.v
vvp a.out
gtkwave wave.vcd
```

以下の波形が実際にシミュレーションを行った結果です。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/wave_branch.png)

波形を見てみると、最後の方でレジスタB2に16進数で`16'h0037`が格納されています。これは10進数で55であり、1~10の総和です。そして最終的に`r_pc`の値が更新されなくなっています。これはJRLによってCPUを停止出来たという事です。理想通りの動作をしていますね。

**おめでとうございます！** 分岐命令の実装が完了しました！これにてZ16のCPUは完成です。お疲れ様でした。これにてあなたは友達に「俺はCPU自作したけど、お前は？」と言えるようになりました。言いましょう。

### CPUを実用的にしよう

我々は動作するCPUを自作できたわけですが、現状ではお世辞にも「使える」CPUとは言い難いです。そこでこの自作CPUを実用的にするための努力をしてみましょう。

#### MMIO

先程1~10の総和を求めた際のように、現状では計算結果を確認するにはGTKWaveでレジスタファイルの内部信号を覗いてやる必要があります。これはあまりスマートとは言えません。そこでCPUの信号を外の世界に出力する仕組みを作りましょう。

このような場合に使える仕組みが**MMIO**、メモリマップドIOです。MMIOとはデータメモリのアドレスの一部に外部入出力用のレジスタを割り当てる手法の事を指します。メモリアドレスに割り当てる事により、外部にデータを出力したい場合はSTORE命令を用い、外部からのデータを得たい場合はLOAD命令を用いる事が可能となります。下図の例ではメモリの`0x007A`にLEDの出力が割り当てられており、ここに`0x0013`をストアすると1番目と2番目と5番目のLEDが点灯します。また`0x007C`がボタンの入力に割り当てられており、ここから値をロードした際に1が得られた場合はボタンが押されており、0の場合はボタンが押されていない事を情報として得られます。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/data_mem_mmio.png)

では実際にMMIOを実装してみましょう。今回はLEDとButtonをMMIOに割り当ててみます。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/cpu_mmio.png)

##### LED

まず仕様を決めます。LEDをメモリアドレスの`0x007A`に割り当て、書き込む16bitのデータの内訳を下位6bitをLEDのオンオフ、上位10bitを未使用とします。

`0x007A : [MSB | 未使用[9:0] | LED[5:0] | LSB]`

実装に移りましょう。まずは`Z16CPU.v`のインターフェイスに新たに`o_led`を追加します。

```verilog
module Z16CPU(
  input  wire   i_clk,
  input  wire   i_rst,
  output reg    [5:0]   o_led
);
```

そして`o_led`の値を制御する新たな機構を追加します。これはメモリの書き込み制御信号`w_mem_wen`が1になっており、また書き込み先のアドレスが`16'h007A`の場合に、`o_led`の値を更新する機構になっています。

```verilog
  Z16DataMemory DataMem(
    .i_clk  (i_clk      ),
    .i_addr (w_alu_data ),
    .i_wen  (w_mem_wen  ),
    .i_data (w_rs2_data ),
    .o_data (w_mem_rdata)
  );

  // MMIO
  always @(posedge i_clk) begin
    // LED
    if(i_rst) begin
      o_led <= 6'b000000;
    end else if(w_mem_wen && (w_alu_data == 16'h007A)) begin
      o_led <= w_rs2_data[5:0];
    end else begin
      o_led <= o_led;
    end
  end
```

これでLEDのMMIOの実装が完了しました。Buttonの実装に移りましょう。

##### Button

まず仕様を決定します。ボタンのMMIOをメモリアドレスの`0x007C`に割り当て、読み出す16bitのデータの内訳を下位1bitをボタンの状態、上位15bitを未使用とします。

`0x007C : [MSB | 未使用[14:0] | Button[0] | LSB]`

実装に移りましょう。まずは`Z16CPU.v`のインターフェイスに新たに`i_button`を追加します。

```verilog
module Z16CPU(
  input  wire   i_clk,
  input  wire   i_rst,
  input  wire   i_button,
  output reg    [5:0]   o_led
);
```

MMIOで値を読み込む場合はLOAD命令を用いますので、読み込んだデータの書き込み先はレジスタファイルになります。そこでレジスタファイルに書き込むデータを選択する関数である`select_rd_data()`を改造します。

ここではオペコードがLOAD命令の場合であり、また計算されたアドレス`w_alu_data`の値が`16'h007C`の場合に`i_button`の値を`w_rd_data`に接続して、レジスタファイルにボタンの状態が入力されるようにします。また`i_button`のビット幅は1bitですので、15bitの0を結合して16bitにしています。

```verilog
  assign w_rd_data = select_rd_data(w_opcode, i_button, w_mem_rdata, r_pc, w_alu_data);

  function [15:0] select_rd_data(
    input   [3:0]   i_opcode,
    input           i_button,
    input   [15:0]  i_mem_rdata,
    input   [15:0]  i_pc,
    input   [15:0]  i_alu_data
  );
  begin
    case(i_opcode)
      4'hA  : begin
        if(w_alu_data == 16'h007C) begin // MMIO Button
          select_rd_data    = {15'b000_0000_0000_0000, i_button};
        end else begin
          select_rd_data    = i_mem_rdata;
        end
      end 
      4'hC  : select_rd_data    = r_pc + 16'h0002;
      4'hD  : select_rd_data    = r_pc + 16'h0002;
      default : select_rd_data  = i_alu_data;
    endcase
  end
  endfunction

  Z16RegisterFile RegFile(
    .i_clk      (i_clk      ),
    .i_rs1_addr (w_rs1_addr ),
    .o_rs1_data (w_rs1_data ),
    .i_rs2_addr (w_rs2_addr ),
    .o_rs2_data (w_rs2_data ),
    .i_rd_data  (w_rd_data  ),
    .i_rd_addr  (w_rd_addr  ),
    .i_rd_wen   (w_rd_wen   )
  );
```

以上でMMIOの実装が完了しました。全体のコードを以下に載せておきます。

```verilog
module Z16CPU(
  input  wire   i_clk,
  input  wire   i_rst,
  input  wire   i_button,
  output reg    [5:0]   o_led
);

  // Program Counter
  reg   [15:0]  r_pc;

  wire  [15:0]  w_instr;
  wire [3:0]    w_rd_addr;
  wire [3:0]    w_rs1_addr;
  wire [3:0]    w_rs2_addr;
  wire [15:0]   w_imm;
  wire          w_rd_wen;
  wire          w_mem_wen;
  wire [3:0]    w_alu_ctrl;
  wire [3:0]    w_opcode;

  wire [15:0]   w_rs1_data;
  wire [15:0]   w_rs2_data;
  wire [15:0]   w_rd_data;
  
  wire [15:0]   w_data_b;
  wire [15:0]   w_alu_data;

  wire [15:0]   w_mem_rdata;

  always @(posedge i_clk) begin
    if(i_rst) begin
      r_pc  <= 16'h0000;
    end else if(w_opcode == 4'hC) begin // JAL
      r_pc  <= w_alu_data;
    end else if(w_opcode == 4'hD) begin // JRL
      r_pc  <= r_pc + w_alu_data;
    end else if((w_opcode == 4'hE) && (w_rs2_data == w_rs1_data)) begin // BEQ
      r_pc  <= r_pc + w_imm;
    end else if((w_opcode == 4'hF) && (w_rs2_data > w_rs1_data)) begin // BLT
      r_pc  <= r_pc + w_imm;
    end else begin
      r_pc  <= r_pc + 16'h0002;
    end
  end

  Z16InstrMemory InstrMem(
    .i_addr     (r_pc   ),
    .o_instr    (w_instr)
  );

  Z16Decoder Decoder(
    .i_instr    (w_instr    ),
    .o_opcode   (w_opcode   ),
    .o_rd_addr  (w_rd_addr  ),
    .o_rs1_addr (w_rs1_addr ),
    .o_rs2_addr (w_rs2_addr ),
    .o_imm      (w_imm      ),
    .o_rd_wen   (w_rd_wen   ),
    .o_mem_wen  (w_mem_wen  ),
    .o_alu_ctrl (w_alu_ctrl )
  );

  assign w_rd_data = select_rd_data(w_opcode, i_button, w_mem_rdata, r_pc, w_alu_data);

  function [15:0] select_rd_data(
    input   [3:0]   i_opcode,
    input           i_button,
    input   [15:0]  i_mem_rdata,
    input   [15:0]  i_pc,
    input   [15:0]  i_alu_data
  );
  begin
    case(i_opcode)
      4'hA  : begin
        if(w_alu_data == 16'h007C) begin // MMIO Button
          select_rd_data    = {15'b000_0000_0000_0000, i_button};
        end else begin
          select_rd_data    = i_mem_rdata;
        end
      end 
      4'hC  : select_rd_data    = r_pc + 16'h0002;
      4'hD  : select_rd_data    = r_pc + 16'h0002;
      default : select_rd_data  = i_alu_data;
    endcase
  end
  endfunction

  Z16RegisterFile RegFile(
    .i_clk      (i_clk      ),
    .i_rs1_addr (w_rs1_addr ),
    .o_rs1_data (w_rs1_data ),
    .i_rs2_addr (w_rs2_addr ),
    .o_rs2_data (w_rs2_data ),
    .i_rd_data  (w_rd_data  ),
    .i_rd_addr  (w_rd_addr  ),
    .i_rd_wen   (w_rd_wen   )
  );

  assign w_data_b = (w_opcode <= 8'h8) ? w_rs2_data : w_imm;
  Z16ALU ALU(
    .i_data_a   (w_rs1_data ),
    .i_data_b   (w_data_b   ),
    .i_ctrl     (w_alu_ctrl ),
    .o_data     (w_alu_data )
  );

  Z16DataMemory DataMem(
    .i_clk  (i_clk      ),
    .i_addr (w_alu_data ),
    .i_wen  (w_mem_wen  ),
    .i_data (w_rs2_data ),
    .o_data (w_mem_rdata)
  );

  // MMIO
  always @(posedge i_clk) begin
    // LED
    if(i_rst) begin
      o_led <= 6'b000000;
    end else if(w_mem_wen && (w_alu_data == 16'h007A)) begin
      o_led <= w_rs2_data[5:0];
    end else begin
      o_led <= o_led;
    end
  end

endmodule
```

##### 動作の確認

動作の確認に移りましょう。まずはLEDの動作を確認します。以下のテスト用のプログラムでは1~5の総和を求めた後、LEDのMMIOのアドレスである122(0x007A)に計算結果をストアし、停止しています。

```
00: ADD ZR ZR B1
02: ADD ZR ZR B2
04: ADDI 5 B1
06: ADD B1 B2 B2
08: ADDI -1 B1
0A: BLT B1 ZR -4
0C: ADD ZR ZR G0
0E: ADDI 122 G0
10: STORE B2 G0 0
12: JRL 0 ZR G11
```

上記のプログラムをアセンブルし、バイナリを命令メモリに書き込みます。

```verilog
module Z16InstrMemory(
  input  wire           i_clk,
  input  wire   [15:0]  i_addr,
  output wire   [15:0]  o_instr
);

  wire [15:0] mem[10:0];

  assign o_instr = mem[i_addr[15:1]];

  assign mem[0] = 16'h0010; // ADD ZR ZR B1
  assign mem[1] = 16'h0020; // ADD ZR ZR B2
  assign mem[2] = 16'h0519; // ADDI 5 B1
  assign mem[3] = 16'h1220; // ADD B1 B2 B2
  assign mem[4] = 16'hFF19; // ADDI -1 B1
  assign mem[5] = 16'hFC4F; // BLT B1 ZR -4
  assign mem[6] = 16'h0040; // ADD ZR ZR G0
  assign mem[7] = 16'h7A49; // ADDI 122 G0
  assign mem[8] = 16'h240B; // STORE B2 G0 0
  assign mem[9] = 16'h00FD; // JRL 0 ZR G11

endmodule
```

そしてテストベンチ`Z16CPU_tb.v`を改造して`o_led`を接続します。

```verilog
module Z16CPU_tb;

  reg i_clk = 1'b0;
  reg i_rst = 1'b0;
  wire [5:0] o_led;

  always #1 begin
    i_clk <= ~i_clk;
  end

  initial begin
    $dumpfile("wave.vcd");
    $dumpvars(0, Z16CPU_tb);
  end

  Z16CPU CPU(
    .i_clk  (i_clk  ),
    .i_rst  (i_rst  ),
    .o_led  (o_led  )
  );

  initial begin
    // リセット
    i_rst   = 1'b1;
    #2
    // リセットを落とし、実行開始
    i_rst   = 1'b0;
    #100
    $finish;
  end

endmodule
```

そして以下のコマンドでシミュレーションを実行します。

```bash
iverilog Z16CPU_tb.v Z16ALU.v Z16CPU.v Z16DataMemory.v Z16Decoder.sv Z16InstrMemory.v Z16RegisterFile.v
vvp a.out
gtkwave wave.vcd
```

実際にシミュレーションを走らせた結果が以下の波形です。数クロックの後、`o_led`に16進数で`0xF`が出力されています。これは10進数で15であり、1~5の総和に一致しています。これで計算結果を外部に出力出来るようになりました。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/wave_mmio_led.png)

続いてボタンのMMIOの動作確認を行いましょう。今回はテスト用プログラムとして、ボタンが押されるまではLEDに出力される値をひたすらカウントアップしていくプログラムを使いましょう。

動作を理解しやすくするために疑似コードを書きます。

```c
x = 0;
led = 122;
button = 124;
while(1){
  x = x + 1;
  Mem[led] = x;
  if(Mem[button] == 1) {
    stop();
  }
}
```

そのあとこの疑似コードをZ16のアセンブリに人力コンパイルします。解読しなくて結構です、適当に斜め読みしてコピペしてください。コンパイラ欲しいですね。

```
00: ADD ZR ZR B1    // B1 <- 0
02: ADDI 1 B1       // B1 <- B1 + 1
04: ADD ZR ZR G0    // G0 <- 0
06: ADDI 122 G0     // G0 <- G0 + 122
08: ADD ZR ZR G1    // G1 <- 0
0A: ADDI 124 G1     // G1 <- G1 + 124
0C: ADD ZR ZR G3    // G3 <- 0
0E: ADDI -8 G3      // G3 <- -8
10: ADD ZR ZR G2    // G2 <- 0
12: ADDI 1 G2       // G2 <- G2 + 1
14: STORE G2 G0 0   // [LED] <- G2
16: LOAD 0 G1 B2    // B2 <- [Button]
18: BEQ B1 B2 4     // if B1 == B2, then PC <- PC + 4
1A: JRL 0 G3 ZR     // PC <- PC + G3
1C: JRL 0 ZR G11    // STOP
```

そしてこれをアセンブルし、バイナリを命令メモリに書き込みます。

```verilog
module Z16InstrMemory(
  input  wire           i_clk,
  input  wire   [15:0]  i_addr,
  output wire   [15:0]  o_instr
);

  wire [15:0] mem[14:0];

  assign o_instr = mem[i_addr[15:1]];

  assign mem[0] = 16'h0010; // ADD ZR ZR B1    // B1 <- 0
  assign mem[1] = 16'h0119; // ADDI 1 B1       // B1 <- B1 + 1
  assign mem[2] = 16'h0040; // ADD ZR ZR G0    // G0 <- 0
  assign mem[3] = 16'h7A49; // ADDI 122 G0     // G0 <- G0 + 122
  assign mem[4] = 16'h0050; // ADD ZR ZR G1    // G1 <- 0
  assign mem[5] = 16'h7C59; // ADDI 124 G1     // G1 <- G1 + 124
  assign mem[6] = 16'h0070; // ADD ZR ZR G3    // G3 <- 0
  assign mem[7] = 16'hF879; // ADDI -8 G3      // G3 <- -8
  assign mem[8] = 16'h0060; // ADD ZR ZR G2    // G2 <- 0
  assign mem[9] = 16'h0169; // ADDI 1 G2       // G2 <- G2 + 1
  assign mem[10] = 16'h640B; // STORE G2 G0 0   // [LED] <- G2
  assign mem[11] = 16'h052A; // LOAD 0 G1 B2    // B2 <- [Button]
  assign mem[12] = 16'h046E; // BEQ B1 B2 4     // if B1 == B2, then PC <- PC + 4
  assign mem[13] = 16'h070D; // JRL 0 G3 ZR     // PC <- PC + G3
  assign mem[14] = 16'h00FD; // JRL 0 ZR G11    // STOP

endmodule
```

次にテストベンチを改造し、`i_button`を定義してCPUに接続します。そしてCPUを動かし始めてから100クロック後に`i_button`に1を入力し、20クロック後に0にするように信号を制御します。

```verilog
module Z16CPU_tb;

  reg i_clk = 1'b0;
  reg i_rst = 1'b0;
  reg i_button = 1'b0;
  wire [5:0] o_led;

  always #1 begin
    i_clk <= ~i_clk;
  end

  initial begin
    $dumpfile("wave.vcd");
    $dumpvars(0, Z16CPU_tb);
  end

  Z16CPU CPU(
    .i_clk      (i_clk      ),
    .i_rst      (i_rst      ),
    .i_button   (i_button   ),
    .o_led      (o_led      )
  );

  initial begin
    // リセット
    i_rst   = 1'b1;
    #2
    // リセットを落とし、実行開始
    i_rst   = 1'b0;
    #100
    // 100クロック後にボタンを押す
    i_button = 1'b1;
    #20
    // 20クロック後にボタンを離す
    i_button = 1'b0;
    #100
    $finish;
  end

endmodule
```

後はいつものコマンドでシミュレーションを実行します。

```bash
iverilog Z16CPU_tb.v Z16ALU.v Z16CPU.v Z16DataMemory.v Z16Decoder.sv Z16InstrMemory.v Z16RegisterFile.v
vvp a.out
gtkwave wave.vcd
```

以下の波形が実際にシミュレーションを行った結果です。

プログラムの実行開始時からLED出力がカウントアップされ、ボタンが押された途端にカウントが停止しました。自作CPU上でプログラムが意図した通りに動きましたね、最高の気分です。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/master/images/LetsMakeCPU/wave_mmio_button.png)

これで我々の自作CPUはボタン入力とLED出力を兼ね備えた物となりました。素晴らしい。

#### 割り込み

## CPUを高速にしよう
### パイプライン化
### キャッシュ
### より高度なアーキテクチャ

## この先へ

---

Copyright (C) Cra2yPierr0t, ALL RIGHTS RESERVED
