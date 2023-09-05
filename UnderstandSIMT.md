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

SIMTの概要をまずは知る。

> "All current-generation GPUs operate according to the SIMT (Single Instruction, Multiple Threads) execution model. From the programmer’s and compiler’s point of view, this model is similar to SPMD (Single Program, Multiple Data). The programmer writes a single program or *kernel*. A large number of instances of the kernel (or *threads*) are then run in parallel. "

> *"(訳) プログラマの観点からはSIMTはSPMDに近く、プログラマはカーネルとも呼ぶ単一のプログラムを書き、そのカーネルが大量に並列に実行される。"*
<p style="text-align:right">- <a href="https://hal.science/hal-00622654/document">Stack-less SIMT reconvergence at low cost</a></p>

これは理解できる。vaddを実行するんじゃなくてaddを大量に並列に実行するんですね。OpenCLで見た。[軽率にGPUを使っていこう、OpenCL入門](https://vlsi.jp/tsukaouOpenCL.html)

> "During execution on a GPU, transparent hardware mechanisms group threads in convoys named warps to execute their instructions on SIMD units."

> *"(訳)ハードウェアがスレッドをまとめてwarpと呼ばれる塊にして、命令をSIMDユニットで実行する。"* 
<p style="text-align:right">- <a href="https://hal.science/hal-00622654/document">Stack-less SIMT reconvergence at low cost</a></p>

warpはスレッドの塊、OK。warpをSIMDユニットで実行、OK。

ちなみに、CPUの文脈のSIMDユニットとGPUの文脈のSIMDユニットは全く別物であることを知っておかないと大変沼る(一敗)。AVX-512に代表されるようなCPUにおけるSIMD命令は、演算器が横にズラッと並んだSIMD演算器とも呼ぶべき実行ユニットで実行される。対してGPUにおけるSIMDは、PEと呼ばれる比較的汎用的で小規模なプロセッサを並べて実現している。

Simty: generalized SIMT execution on RISC-V の方では

> "The Single Instruction, Multiple Threads (SIMT) execution model as implemented in NVIDIA Graphics Processing Units (GPUs) associates a multi-thread programming model with an SIMD execution model."

> *"(訳)SIMT実行モデルはマルチスレッドプログラミングモデルをSIMD実行モデルに関連付けている。"*
<p style="text-align:right">- <a href="https://carrv.github.io/2017/papers/collange-simty-carrv2017.pdf">Simty: generalized SIMT execution on RISC-V</a></p>

とか

> "The SIMT execution model consists in assembling vector instructions across different scalar threads of SPMD programs at the microarchitectural level."

> *"(訳)SIMT実行モデルはマイクロアーキテクチャレベルでSPMDプログラムのスレッド群からベクトル命令を構成する。"* 
<p style="text-align:right">- <a href="https://carrv.github.io/2017/papers/collange-simty-carrv2017.pdf">Simty: generalized SIMT execution on RISC-V</a></p>

とか言ってる。わからん。

多分SPMDなプログラムをそのまま実行できる計算機がSIMTなんだと思います。

## SIMTの利点と欠点

とりあえずSIMTの概要を知ることが出来た。次はSIMTの何が嬉しいのか、そして何が辛いのか理解していく。

SIMTの利点と欠点は一言でまとめられている。

> "Although this model offers a good balance of performance and programmability compared to classic SIMD instruction sets, handling thread divergence dynamically has a cost in area, power and hardware complexity."

> *"(訳) 古典的なSIMD命令セットと比べると、SIMT実行モデルはパフォーマンスとプログラミングのしやすさのバランスが優れているが、スレッドの動的なdivergenceの扱いに回路面積・消費電力・複雑さにおいてコストを払う事になる。"* 
<p style="text-align:right">- <a href="https://hal.science/hal-00622654/document">Stack-less SIMT reconvergence at low cost</a></p>

プログラミングはしやすくなるけど分岐の実装がダルいんですね。

divergenceとか謎の単語が出てきたが、divergenceは最初は足並みを揃えてたwarpのスレッド達が、分岐命令を実行することでプログラムカウンタの値が異なってくることだと理解してる。convergenceはその逆。

このdivergenceとconvergenceを管理する特別な命令を持つ命令セットが存在してないので、SIMT実行モデルの汎用プロセッサの採用が辛いみたいな事も書いてあるが、これをRISC-Vで解決するのがSIMTYである。

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

> *"(訳) 古典的な並列アーキテクチャではwarpで単一のスタックポインタを共有する。これで異なるスレッドがコールスタックへ同時にアクセスしても同期が常に保たれる。"* 
<p style="text-align:right">- <a href="https://hal.science/hal-00622654/document">Stack-less SIMT reconvergence at low cost</a></p>

まあスタックポインタが１つだけならどのスレッドも関数呼び出し時は完全に全てのスレッドで同期される気がする、divergenceとの関係はよくわからない。

> *"(訳) 一方で、これはSIMTが実行できるスレッドを、同じスタックポインタかつ同じ命令ポインタを共有しているスレッドのみに制限する。関数コールのネストの深さが異なるスレッドを同時に実行することは出来ない。"* 

これは、この実装だと命令ポインタが同じでもスタックポインタが違う場合、例えばスレッド0とスレッド1で再帰の深さが違うような場合に、実行する関数は同じにも関わらずスレッド0の実行が完了してからスレッド1を実行しなくてはならず、これは明らかに効率が悪い。そういう意味だと思う。

> "The other solution consists in maintaining an independent stack pointer per thread. In case of recursion, control divergence is reduced. It is replaced by memory divergence: accesses to the stack become irregular."

> *"(訳) その他の手法としてスタックポインタをスレッド毎に持つ手法があり、この場合はcontrol divergenceが減少する代わりにスタックへのアクセスが異なるmemory divergenceに置き換わる。"* 
<p style="text-align:right">- <a href="https://hal.science/hal-00622654/document">Stack-less SIMT reconvergence at low cost</a></p>


難しい、control divergenceは多分分岐命令時にプログラムの流れが分裂する事を指してる、memory divergenceは何だろう。スレッド0とスレッド1で深さの異なる再帰を実行していったらスタックポインタの値だけが異なって行きそう、多分これがmemory divergenceかな。

確かに再帰の対処って大事っすね。

### 明示的な方法, スタックベース

#### Pixar Chap

Pixar ChapなるPixar製の古の画像処理用プロセッサは、命令セットに`if`, `else`, `fi`, `while-do`, `done`といった、プログラミング言語によくある制御構文を反映した制御命令を持っている。`if-else-then`といった条件分岐はcurrent prediction maskが扱い、ループはtwo mask stackというものが扱うらしい。

Pixar Chapでは制御フローグラフの走査順序はISA側で明示して、ReconvergenceはISAに加えてmask stackで検知するらしい。謎。

謎すぎるのでCaroline Collange先生の資料を読んでみる。23ページあたりから読むとアツい。

[GPU architecture part 2: SIMT control flow management](http://www.irisa.fr/alf/downloads/collange/cours/ada2020_gpu_2.pdf)

あー完全に理解した。ありがとうCaroline Collange先生。

まずはシンプルな実行マスクを考える。このマスクはi-bit目が1ならi番目のスレッドが実行されるものであり、その値は分岐命令が実行されたときに変化する。以下の例では、最初のif文までは全てのスレッドが実行されるが、if文の条件式ではスレッド番号が2未満の場合に真となるため、実行マスクの値は`[1100]`となり、T0とT1しか実行されなくなる。その後if文を抜け、実行マスクの値は`[1111]`となるが、次のif文の条件式はスレッド番号が5より大きい場合に真となるため、実行マスクの値は`[0000]`となり、if文のブロックの最後まで、処理がスキップされる。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/UnderstandSIMT/single_mask.png)
<p style="text-align:center"> <b>分岐命令時の実行マスクの変化</b><br>(<a href="http://www.irisa.fr/alf/downloads/collange/cours/ada2020_gpu_2.pdf">GPU architecture part 2: SIMT control flow management</a>より作成)</p>

次に、以下のようなif文がネストされ、またelseの存在するプログラムの場合に、実行マスクの値がどのように変化するか考えてみる。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/UnderstandSIMT/nest_kernel.png)
<p style="text-align:center"> <b>if文がネストされたプログラム</b><br>(<a href="http://www.irisa.fr/alf/downloads/collange/cours/ada2020_gpu_2.pdf">GPU architecture part 2: SIMT control flow management</a>より作成)</p>

単一の実行マスクの実装の場合、else以前までは正常に処理されるが、elseの実行時点でアクティブになっているスレッドはT0のみであるため、T0のelseの評価は偽となり実行マスクの値は`[0000]`となる。その結果if文のブロックの最後まで、処理がスキップされる。結果的に本来ならばT1が実行するはずのelseの処理が実行されなくなる。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/UnderstandSIMT/nest_kernel_single_mask.png)
<p style="text-align:center"> <b>if文がネストされたプログラムの実行マスクの変化</b><br>(<a href="http://www.irisa.fr/alf/downloads/collange/cours/ada2020_gpu_2.pdf">GPU architecture part 2: SIMT control flow management</a>より作成)</p>

そこで実行マスクを保存するスタック、マスクスタックを用意する。このスタックの先頭に存在する実行マスクを有効な実行マスクとする。マスクスタックには、分岐時に新たに生成された実行マスクをpushし、コードブロックの終了時にpopする。

マスクスタックにより、２番目のif文の終了時に`[1000]`がpopされ`[1100]`が先頭となり、T0とT1がアクティブになる。そしてelseでT1では真となるため、`[0100]`がpushされてelse内がT1で実行される。結果的に全てのスレッドが正しく動作する。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/UnderstandSIMT/mask_stack.png)
<p style="text-align:center"> <b>マスクスタック</b><br>(<a href="http://www.irisa.fr/alf/downloads/collange/cours/ada2020_gpu_2.pdf">GPU architecture part 2: SIMT control flow management</a>より作成)</p>

このmask stackの導入によりreconvergenceが実現できている。賢い。

まとめるとPixar Chapでは制御フローグラフの走査はISAで行い、Reconvergenceはmask stackで行っている。

現代のGPUではAMDが似たアプローチを取っている。実際にAMD Evergreenの命令セットを眺めてみると`LOOP_START`命令とかある。

[Evergreen Family Instruction Set Architecture: Instructions and Microcode](https://www.amd.com/content/dam/amd/en/documents/radeon-tech-docs/instruction-set-architectures/AMD_Evergreen-Family_Instruction_Set_Architecture.pdf)

#### POMP

POMPはKeryll氏とParis氏が提案したPOMP並列計算機プロジェクトの産物であり、スタックに保存されているマスクは常にヒストグラムの形状を取る事から着想を得た。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/UnderstandSIMT/stack_histogram.png)
<p style="text-align:center"> <b>マスクスタックの推移</b><br>(<a href="http://www.irisa.fr/alf/downloads/collange/cours/ada2020_gpu_2.pdf">GPU architecture part 2: SIMT control flow management</a>より作成)</p>

スタックのdepth $$d$$で非アクティブのスレッドはスタックのdepth $$d + 1$$でも必ず非アクティブのままとなる。つまり先頭が`[1100]`の状態で`[1101]`のようなマスクがpushされることは絶対に無い。

この性質のため、マスクスタックは各スレッドの最後にアクティブになったネストの深さを記録する、アクティビティカウンタに置き換えられる。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/UnderstandSIMT/activity_counter.png)
<p style="text-align:center"> <b>アクティビティカウンタ</b><br>(<a href="http://www.irisa.fr/alf/downloads/collange/cours/ada2020_gpu_2.pdf">GPU architecture part 2: SIMT control flow management</a>より作成)</p>

この実装により$$N$$をネストの深さとして、保存するデータ量をマスクスタックの$$O(N)$$から$$O(\log(N))$$に落とせる。

このアクティビティカウンタの実装はintelのSandy Bridge以前の内蔵GPUで使用されている。

#### NVIDIA Tesla

>  "The Tesla instruction set includes conditional branch instructions resembling those of scalar processors, rather than structured control instructions as other GPUs. It is complemented by annotations pointing at divergence and reconvergence points."

> *"(訳) NVIDIA Teslaでは汎用性を高めるために、命令セットはGPUの構造的な制御命令というよりスカラプロセッサの条件分岐命令に似たものが入っている。そして足りない情報はdivergenceとconvergenceの位置情報で補う"* 
<p style="text-align:right">- <a href="https://hal.science/hal-00622654/document">Stack-less SIMT reconvergence at low cost</a></p>

言いたい事はわかる。divergenceとconvergenceを明示する命令より、CPUの普通の分岐命令に寄った命令セットにして汎用性は高めてるけど、スレッドを管理するための情報が足りなくなるのでdivergenceとconvergenceが発生した時の命令アドレスをどこかに保存してよしなに使うんだと思う。具体的な事は全くわからん。

> "In particular, the matching between divergence pointsand reconvergence points is not explicit in the binary. Hence, the divergence stack needs to store the address of reconvergence points in addition to masks. This excludes an implementation based on activity counters."
>
> *"(訳) 特に、divergenceとreconvergence地点の一致はバイナリに明記されない。これにより、divergence stackはマスクに加えてreconvergenceの地点を保存する必要がある。そのためアクティビティカウンタに基づいた実装は行えない。"*
<p style="text-align:right">- <a href="https://hal.science/hal-00622654/document">Stack-less SIMT reconvergence at low cost</a></p>

divergenceとreconvergenceの地点の一致って何？ループ？なんか知らんけど分岐命令のアドレスをスタックに保存する必要があるみたいです。


### 暗黙的な方法, スタックベース

### 明示的な方法, スタックレス

#### Sandy Bridge
#### Lorie-Strong

### 暗黙的な方法, アクティビティビット
