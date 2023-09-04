---
layout: default
title: SIMTってやつを完全に理解したい
---

院落ちた＼(^o^)／

# SIMTってやつを完全に理解したい

最近(ここ20年くらい)、GPUでSIMTってのがアツい。アツいという事にしましょう。まあ興味を持って損するもんでもないでしょう。というわけで論文読んで理解しような。

良さそうな論文を用意したのでこいつらに疑問に答えてもらう。

- [Stack-less SIMT reconvergence at low cost](https://hal.science/hal-00622654/document)
- [Simty: generalized SIMT execution on RISC-V](https://carrv.github.io/2017/papers/collange-simty-carrv2017.pdf)

## SIMTとは

SIMTの概要をまずは理解する。

> "All current-generation GPUs operate according to the SIMT (Single Instruction, Multiple Threads) execution model. From the programmer’s and compiler’s point of view, this model is similar to SPMD (Single Program, Multiple Data). The programmer writes a single program or *kernel*. A large number of instances of the kernel (or *threads*) are then run in parallel. "
<p style="text-align:right">- <a href="https://hal.science/hal-00622654/document">Stack-less SIMT reconvergence at low cost</a></p>

*"(訳) プログラマの観点からはSIMTはSPMDに近く、プログラマはカーネルとも呼ぶ単一のプログラムを書き、そのカーネルが大量に並列に実行される。"*これは理解できる。vaddを実行するんじゃなくてaddを大量に並列に実行するんですね。OpenCLで見た。

[軽率にGPUを使っていこう、OpenCL入門](https://vlsi.jp/tsukaouOpenCL.html)

> "During execution on a GPU, transparent hardware mechanisms group threads in convoys named warps to execute their instructions on SIMD units."
<p style="text-align:right">- <a href="https://hal.science/hal-00622654/document">Stack-less SIMT reconvergence at low cost</a></p>

*"(訳)ハードウェアがスレッドをまとめてwarpと呼ばれる塊にして、命令をSIMDユニットで実行する。"* warpはスレッドの塊、OK。warpをSIMDユニットで実行、OK。

Simty: generalized SIMT execution on RISC-V の方でも

> "The Single Instruction, Multiple Threads (SIMT) execution model as implemented in NVIDIA Graphics Processing Units (GPUs) associates a multi-thread programming model with an SIMD execution model."
<p style="text-align:right">- <a href="https://carrv.github.io/2017/papers/collange-simty-carrv2017.pdf">Simty: generalized SIMT execution on RISC-V</a></p>

*"(訳)SIMT実行モデルはマルチスレッドプログラミングモデルをSIMD実行モデルに関連付けている。"*とか

> "The SIMT execution model consists in assembling vector instructions across different scalar threads of SPMD programs at the microarchitectural level."
<p style="text-align:right">- <a href="https://carrv.github.io/2017/papers/collange-simty-carrv2017.pdf">Simty: generalized SIMT execution on RISC-V</a></p>

*"(訳)SIMT実行モデルはマイクロアーキテクチャレベルでSPMDプログラムのスレッド群からベクトル命令を構成する。"* とか言っているので、大量のスレッドを合成してベクトル命令を作り、それをSIMDユニットで実行する。これをハードウェアでやるのがSIMTな計算機なんだと思う。

つまりSIMDとSIMTの違いはこんな感じ、信じるか信じないかはあなた次第です。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/UnderstandSIMT/SIMD.png)
<p style="text-align:center"> <b>SIMDの実行モデル</b> </p>
![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/UnderstandSIMT/SIMT.png)
<p style="text-align:center"> <b>SIMTの実行モデル</b> </p>

図にすると理解した気になれて嬉しい。


## SIMTの利点と欠点

とりあえずSIMTの実行方法は理解した。次はSIMTの何が嬉しいのか、そして何が辛いのか理解していく。

SIMTの利点と欠点は一言でまとめられている。

> "Although this model offers a good balance of performance and programmability compared to classic SIMD instruction sets, handling thread divergence dynamically has a cost in area, power and hardware complexity."
<p style="text-align:right">- <a href="https://hal.science/hal-00622654/document">Stack-less SIMT reconvergence at low cost</a></p>

*"(訳) 古典的なSIMD命令セットと比べると、SIMT実行モデルはパフォーマンスとプログラミングのしやすさのバランスが優れているが、スレッドの動的なdivergenceの扱いに回路面積・消費電力・複雑さにおいてコストを払う事になる。"* と書いてある。プログラミングはしやすくなるけど分岐の実装がダルいんですね。

divergenceとか謎の単語が出てきたが、divergenceは最初は足並みを揃えてたwarpのスレッド達が、分岐命令を実行することでプログラムカウンタの値が異なってくることだと理解してる。convergenceはその逆。

このdivergenceとconvergenceを管理する特別な命令を持つ命令セットが存在してないので、SIMT実行モデルの汎用プロセッサの採用が辛いみたいな事も書いてあるが、これをRISC-Vで解決するのがSIMTYである。

## SIMTにおける命令のベクトル化

ここまで知ってくると次は具体的な実装が気になってくる。分岐の扱いがSIMTの実装にとってのボトルネックなのは分かるが、それ以前にスレッド群からどうやってSIMD命令を作ってSIMDユニットで実行するのかが気になる。vaddの場合は全てのスレッドでadd命令が発行されているからデータだけSIMDユニットに突っ込むとかそんな感じだろうか、でもSPMDって`int b = a;`みたいな感じで普通に値のムーブとか行うけど、どうやって対処してるんだろうとか色々納得してない。

参考文献を眺めてみたらいい感じの論文があったので読んでみる。徳島大の御人が書いた論文らしい。名前をどこかで見たと思ったらB. ウィルキンソンの並列計算機設計技法を翻訳した御方だった。

- [A mechanism for SIMD execution of SPMD programs](https://ieeexplore.ieee.org/document/592203)



## SIMTにおける分岐の実装

SIMTにおける分岐の実装にはどんなものがあるのか、Stack-less SIMT reconvergence at low costの2. Related workに紹介があるので読んでみる。

> "We distinguish explicit methods, whose behavior is exposed at the architectural level and which require specific action from the compiler, from implicit methods, which are restricted to the micro-architectural level and can operate on standard scalar instruction sets. <br>
> Divergence control mechanisms address two distinct issues: <br>
> • In which order the control flow graph is traversed: which divergent branch will be executed first. <br>
> • How thread reconvergence is detected: when should divergent paths be re-united." 
<p style="text-align:right">- <a href="https://hal.science/hal-00622654/document">Stack-less SIMT reconvergence at low cost</a></p>

このdivergenceの実装には明示的な方法と暗黙的な方法の２つに分けられ、明示的な方法はコンパイラによるアクションが必要らしい。多分分岐命令の前後に特殊な命令でも挿入すると思う。暗黙的な方法はそういうのが必要無い。

スレッドのdivergenceには２つの問題が存在する。

- 制御フローグラフをどの順序で走査するのか、つまりどの分岐を先に実行するか
- どうやってconvergenceを検知するか、つまり分岐したパスをいつ結合するか

これはわかる。制御フローグラフの走査順序はパフォーマンスにかなり影響する。convergence検知はよくわからないです。

> "For techniques which allow function calls, the call stack can be implemented in two ways. The difference between the solutions appear in cases of recursion."
<p style="text-align:right">- <a href="https://hal.science/hal-00622654/document">Stack-less SIMT reconvergence at low cost</a></p>

関数呼び出しのためのコールスタックの実装は再帰の対処方法に着目すると二種類に分けられるらしい。

> "Traditional parallel architectures share a single stack pointer per warp. This ensures that concurrent accesses to the call stack from different threads are always kept synchronized. On the other hand, this restricts SIMT execution to threads that share the same stack pointer in addition to sharing the same instruction pointer. Threads that are at different call nesting levels cannot run together. "
<p style="text-align:right">- <a href="https://hal.science/hal-00622654/document">Stack-less SIMT reconvergence at low cost</a></p>

*"(訳) 古典的な並列アーキテクチャではwarpで単一のスタックポインタを共有する。これで異なるスレッドがコールスタックへ同時にアクセスしても同期が常に保たれる。"* それはそう、スタックポインタが１つだけなんだからどのスレッドも関数呼び出し時は完全に全てのスレッドで同期される、分岐とかは今は忘れる。

> "The other solution consists in maintaining an independent stack pointer per thread. In case of recursion, control divergence is reduced. It is replaced by memory divergence: accesses to the stack become irregular."
<p style="text-align:right">- <a href="https://hal.science/hal-00622654/document">Stack-less SIMT reconvergence at low cost</a></p>

### 明示的な方法, スタックベース

#### Pixar Chap
#### POMP
#### NVIDIA Tesla

### 暗黙的な方法, スタックベース

### 明示的な方法, スタックレス

#### Sandy Bridge
#### Lorie-Strong

### 暗黙的な方法, アクティビティビット
