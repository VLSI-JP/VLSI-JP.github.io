---
layout: default
title: コンピュータアーキテクチャ入門
image: "https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/Computer_Architecture.png"
---

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/Computer_Architecture.png)

# コンピュータアーキテクチャ入門

さあ始めましょう。

## プログラムが動く流れ

CPUはプログラムをどのように実行しているのでしょうか？CPUはプログラムを動かす物体ですので、一度ここで学んでおきましょう。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/prog_flow.png)

CPUはプログラムをそのまま実行している訳ではありません。

現代では、プログラムは**コンパイラ**というソフトウェアによって**アセンブリ**という中間言語に変換され、そのアセンブリは**アセンブラ**というソフトウェアによって`011010101...`のようなバイナリに変換されます。
CPUとってプログラムはこのバイナリになって初めて実行可能な形式になるというわけです。

- プログラム
![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/helloworld.png)
- アセンブリ
![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/assembly.png)
- バイナリ
![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/binary.png)


## 命令セットアーキテクチャ 

先程、プログラムはバイナリに変換されて初めてCPUが実行可能な形になると言いましたね、プログラミング言語がこの世に沢山あるのと同じように、実は変換後のバイナリもこの世にはいくつか種類が存在しています。このバイナリの種類の事を命令セットアーキテクチャ(Instruction Set Architecture)、**ISA**と呼びます。基本的に、CPUはISAが異なるバイナリを実行できません。例えばMacのソフトウェアはMacのCPU以外では動きませんし、PCゲームのファイルをスマホに突っ込んでも動きませんね、動かそうとするな。

具体的なISAとしては、x86_64, AArch64, RISC-Vなどが存在しています。

本記事では筆者が定義した独自のISA、**Z16**のCPUを作ります。Z16のCPUを作る前に一度、本項でZ16のアセンブリに親しみを持っておきましょう。

### Z16の概要

Z16は筆者が独自に定義したISAであり、16-bitの固定長命令セットです。16個の16-bitレジスタと16個の命令を持ちます。酒に酔いながら設計しました。

RISC-Vみたいなイケてるロゴを持ってるISAに憧れがあるのでZ16のロゴを以下に置いておきます。ライセンスフリーです。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/z16_logo.png)

### Z16のレジスタ

Z16は16bitのレジスタを16個持っています。その内訳は以下の通り、アドレス0のレジスタ`ZR`は常に値が0な特殊なレジスタ、レジスタ`B1`~`B3`は後述しますが分岐命令の対象にすることが可能なレジスタです。残りのレジスタ`G0`~`G11`は自由に使えるレジスタとなっています。

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

レジスタ関連の用語で、値が書き込まれるレジスタを**ディスティネーションレジスタ**、値が読み出されるレジスタを**ソースレジスタ**と呼びます。覚えておきましょう。

### Z16の命令

次はZ16の命令を見ていきましょう。

#### 演算命令

演算命令のビットフィールドの形式は以下の通り。4bit区切りになっており、下位4bitから順にオペコード、ディスティネーションレジスタ、ソースレジスタ１、ソースレジスタ２となっている。

| instr | `[15:12]` | `[11:8]` | `[7:4]` | `[3:0]` |
| ----- | ------- | ------ | ----- | ----- |
| 演算命令 | `rs2[3:0]` | `rs1[3:0]` | `rd[3:0]` | `opcode[3:0]` |

##### ADD

加算命令。レジスタ同士の足し算を行う命令であり、`ADD RS2 RS1 RD`と記述し、RS2とRS1の値を加算してRDに格納する。オペコードは`4'h0`。例として`ADD G3 G2 G1`は`16'h7650`となる。また、RS2にゼロレジスタZRを指定することで、RS1からRDに値を移動させるだけの命令として使うことが可能です。

<p>
$$
\text{RS2} + \text{RS1} \rightarrow \text{RD}
$$
</p>

実際にZ16のエミュレータでADD命令を使ってみましょう。以下のページにアクセスし、エミュレータにアクセスします。

[Z16エミュレータ](/Z16Emulator.html)

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/z16emu.png)

エミュレータの赤で囲われた部分に`ADD G3 G2 G1`と入力し、またレジスタの表のG3とG2に数字を入力します。そしてassembleボタンを押して命令をバイナリに変換し、runボタンを押して命令を実行します。

するとG1の値がG2とG3の加算結果が格納されます。

##### SUB

減算命令。レジスタ同士の引き算を行う命令であり、`SUB RS2 RS1 RD`と記述し、RS2からRS1の値を減算してRDに格納する。オペコードは`4'h1`。例として`SUB G3 G2 G1`は`16'h7651`となる。

<p>
$$
\text{RS2} - \text{RS1} \rightarrow \text{RD}
$$
</p>

##### MUL

乗算命令。レジスタ同士の掛け算を行う命令であり、`MUL RS2 RS1 RD`と記述し、RS2とRS1の値を乗算してRDに格納する。オペコードは`4'h2`。例として`MUL G3 G2 G1`は`16'h7652`となる。

<p>
$$
\text{RS2}\times \text{RS1} \rightarrow \text{RD}
$$
</p>

##### DIV

除算命令。レジスタ同士の割り算を行う命令であり、`DIV RS2 RS1 RD`と記述し、RS2をRS1で割った商をRDに格納する。オペコードは`4'h3`。例として`DIV G3 G2 G1`は`16'h7653`となる。

<p>
$$
\text{RS2} \div \text{RS1} \rightarrow \text{RD}
$$
</p>

##### OR

OR命令。レジスタ同士のOR演算を行う命令であり、`OR RS2 RS1 RD`と記述し、RS2とRS1の値のORをRDに格納する。オペコードは`4'h4`。例として`OR G3 G2 G1`は`16'h7654`となる。

<p>
$$
\text{RS2} \parallel \text{RS1} \rightarrow \text{RD}
$$
</p>

##### AND

AND命令。レジスタ同士のAND演算を行う命令であり、`AND RS2 RS1 RD`と記述し、RS2とRS1の値のANDをRDに格納する。オペコードは`4'h5`。例として`AND G3 G2 G1`は`16'h7655`となる。

<p>
$$
\text{RS2} \& \text{RS1} \rightarrow \text{RD}
$$
</p>

##### XOR

XOR命令。レジスタ同士のXOR演算を行う命令であり、`XOR RS2 RS1 RD`と記述し、RS2とRS1の値のXORをRDに格納する。オペコードは`4'h6`。例として`XOR G3 G2 G1`は`16'h7656`となる。

<p>
$$
\text{RS2} \oplus \text{RS1} \rightarrow \text{RD}
$$
</p>

##### SLL

論理左シフト命令。レジスタの値の左シフトを行う命令であり、`SLL RS2 RS1 RD`と記述し、RS1の値をRS2の値だけ論理左シフトしてRDに格納する。オペコードは`4'h7`。例として`SLL G3 G2 G1`は`16'h7657`となる。

<p>
$$
\text{RS1} << \text{RS2} \rightarrow \text{RD}
$$
</p>

##### SRL

論理右シフト命令。レジスタの値の右シフトを行う命令であり、`SRL RS2 RS1 RD`と記述し、RS1の値をRS2の値だけ論理右シフトしてRDに格納する。オペコードは`4'h8`。例として`SRL G3 G2 G1`は`16'h7658`となる。

$$
\text{RS1} >> \text{RS2} \rightarrow \text{RD}
$$

##### 演算命令まとめ

以下の計算を行いたい場合、アセンブリとそのバイナリは以下の通りになります。

<p>
$$
f(x, y) = x\times y + x - y
$$
</p>

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

エミュレータで実際に上記のアセンブリを動かしてみましょう。初期値はG0とG1に入力してください。

[Z16エミュレータ](/Z16Emulator.html)

演算命令の一覧表

| instr | `[15:12]` | `[11:8]` | `[7:4]` | `[3:0]` |
| ----- | ------- | ------ | ----- | ----- |
| 演算命令 | `rs2[3:0]` | `rs1[3:0]` | `rd[3:0]` | `opcode[3:0]` |

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

#### 即値命令

先で挙げた演算命令だけでは、G0に100を足す、G7から1を引くなどの操作を行うことが出来ないため、命令に値を直接埋め込みたい需要が存在します。そのような、命令に直接埋め込まれている値を**即値(immediate)**と呼び、即値を含む命令を**即値命令**と呼びます。

即値命令のビットフィールドの形式は以下の通り、LSBから下位4bitがオペコード、そこから4bitがディスティネーションレジスタ、上位8bitが即値となっています。

| instr | `[15:8]` | `[7:4]` | `[3:0]` |
| ----- | ------- | ------ | ----- |
| 即値命令 | `imm[7:0]` | `rd[3:0]` | `opcode[3:0]` |

##### ADDI

即値加算命令。即値とレジスタで足し算を行う命令であり、`ADDI IMM RD`と記述し、RDにIMMを加算してRDに格納します。IMMが取れる値の範囲は符号付き8bitなので-128~127。オペコードは`4'h9`。例として`ADDI 100 G0`は`16'h6449`、`ADDI -1 G2`は`16'hFF69`となります。

<p>
$$
\text{IMM} + \text{RD} \rightarrow \text{RD}
$$
</p>

実際に即値命令を動かしてみましょう。エミュレータにアクセスし、`ADDI 57 G0`と入力して実行してみてください。

[Z16エミュレータ](/Z16Emulator.html)

するとG0に57が格納されます。

##### 即値命令まとめ

以下の計算を行いたい場合、アセンブリとそのバイナリは以下の通りになります。

<p>
$$
f(x, y) = 3\times x + 100 + y
$$
</p>

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

| instr | `[15:8]` | `[7:4]` | `[3:0]` |
| ----- | ------- | ------ | ----- |
| 即値命令 | `imm[7:0]` | `rd[3:0]` | `opcode[3:0]` |

| opcode[3:0] | name | operation | assembly | binary |
| ----------- | ---- | --------- | ------- | ------ |
| `4'h9` | ADDI | `imm + rd ➜ rd` | `ADDI 100 G0` | `16'h6449` |

#### メモリ命令

Z16ではレジスタは16個あると先に説明しましたが、このままではCPUはデータを16個しか持つ事が出来ません。これではあまり実用的なCPUとは言えません。そこで**メモリ**という記憶装置を作り、扱えるデータの量を増やします。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/data_mem.png)

CPUがメモリを扱えるようにするため、メモリからデータを読み書きする専用の命令を定義します。

##### LOAD

Loadはメモリからデータを取り出す操作であり、Z16ではLOADというLoadを行う命令を一つ持っています。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/data_mem_load.png)

LOAD命令のビットフィールドの形式は以下の通り、LSBから下位4bitはオペコード、そこから4bitずつにディスティネーションレジスタ、ソースレジスタ１、即値となっている。

| instr | `[15:12]` | `[11:8]` | `[7:4]` | `[3:0]` |
| ----- | ------- | ------ | ----- | ---- |
| LOAD命令 | `imm[3:0]` | `rs1[3:0]` | `rd[3:0]` | `opcode[3:0]` |

LOAD命令。`LOAD IMM RS1 RD`と記述し、IMMとRS1の値の和をメモリアドレスとし、メモリのそのアドレスにあるデータをRDに格納する。IMMが取れる値の範囲は符号付き4bitであるため-8~7。オペコードは`4'hA`であるため、仮に`LOAD 0 G0 G1`は`16'h045A`となる。

<p>
$$
\text{Memory}[\text{IMM} + \text{RS1}] \rightarrow \text{RD}
$$
</p>

##### STORE

Storeはメモリにデータを書き込む操作であり、Z16ではSTOREというStoreを行う命令を一つ持っている。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/data_mem_store.png)

STORE命令のビットフィールドは以下の通り、LSBから下位4bitがオペコード、そこから4bitずつに即値、ソースレジスタ１、ソースレジスタ２となっている。

| instr | `[15:12]` | `[11:8]` | `[7:4]` | `[3:0]` |
| ----- | ------- | ------ | ----- | ---- |
| STORE命令 | `rs2[3:0]` | `rs1[3:0]` | `imm[3:0]` | `opcode[3:0]` |

STORE命令。`STORE RS2 RS1 IMM`と記述し、IMMとRS1の値の和をメモリアドレスとし、RS2の値をメモリのそのアドレスに格納する。IMMが取れる値の範囲は符号付き4bitであるため-8~7。オペコードは`4'hB`であるため、仮に`STORE G1 G0 0`は`16'h540B`となる。

<p>
$$
\text{RS2} \rightarrow \text{Memory}[\text{RS1} + \text{IMM}]
$$
</p>

##### メモリ命令まとめ

エミュレータにアクセスし、実際にメモリ命令を動かしてみましょう。

[Z16エミュレータ](/Z16Emulator.html)

以下の命令をエミュレータにコピペし、またG0に適当な数字を入れます。

```
STORE G0 ZR 0
LOAD 0 ZR G1
```

その後実行すると、G0のデータがメモリを介してG1に入力されます。

| instr | `[15:12]` | `[11:8]` | `[7:4]` | `[3:0]` |
| ----- | ------- | ------ | ----- | ---- |
| LOAD命令 | `imm[3:0]` | `rs1[3:0]` | `rd[3:0]` | `opcode[3:0]` |
| STORE命令 | `rs2[3:0]` | `rs1[3:0]` | `imm[3:0]` | `opcode[3:0]` |

| opcode[3:0] | name | operation | assembly | binary |
| ---- | --------- | ------------- | ------- |
| `4'hA` | LOAD | `[imm + rs1] ➜ rd` | `LOAD 4 G5 G6` | `16'h49AA` |
| `4'hB` | STORE | `rs2 ➜ [rs1 + imm]` | `STORE G5 G6 2` | `16'h9A2B` |

#### ジャンプ命令

ジャンプ命令とは文字通りジャンプする命令であり、プログラムの流れを強制的に変更します。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/jump.png)

ジャンプ命令のビットフィールドは以下の通り、LSBから下位4bitがオペコード、そこから4bitずつにディスティネーションレジスタ、ソースレジスタ１、即値となっている。

| instr | `[15:12]` | `[11:8]` | `[7:4]` | `[3:0]` |
| ----- | ------- | ------ | ----- | ---- |
| ジャンプ命令 | `imm[3:0]` | `rs1[3:0]` | `rd[3:0]` | `opcode[3:0]` |

##### JAL

**JAL**はJump Absolute and Link の略であり、`JAL IMM RS1 RD`と記述する。動作としてはIMMとRS1の値の和にジャンプすると同時に、JALの次の命令のアドレスをRDに格納する。

即値のフィールドは符号付き4bitであり、-8~7の値を取る。

<p>
$$
\begin{align}
\text{PC} + 2 &\rightarrow \text{RD} \\
\text{IMM} + \text{RS1} &\rightarrow \text{PC}
\end{align}
$$
</p>

実際にJAL命令を使ってみましょう。エミュレータにアクセスし、以下の命令列を実行します。

[Z16エミュレータ](/Z16Emulator.html)

```
ADD ZR ZR ZR
JAL 6 ZR G0
XOR ZR ZR ZR
SUB ZR ZR ZR
```

next cycleボタンを押していくと、JAL命令が実行された際に、実行中の命令のアドレス(PC)が6になり、XOR命令を飛ばしてSUB命令が実行されます。またJAL命令の次の命令のアドレスがG0に格納されます。

##### JRL

**JRL**はJump Relative and Linkの略であり、`JRL IMM RS1 RD`と記述する。動作としてはIMMとRS1の値とJRLが存在するアドレスの和にジャンプすると同時に、JRLの次の命令のアドレスをRDに格納する。

即値のフィールドは符号付き4bitであり、-8~7の値を取る。

<p>
$$
\begin{align}
\text{PC} + 2 &\rightarrow \text{RD} \\
\text{IMM} + \text{RS1} + \text{PC} &\rightarrow \text{PC}
\end{align}
$$
</p>

特殊な使い方として、`JRL 0 ZR ZR`とすることでプログラムを停止させる事が可能である。

実際にJAL命令を使ってみましょう。エミュレータにアクセスし、以下の命令列を実行します。

[Z16エミュレータ](/Z16Emulator.html)

```
ADD ZR ZR ZR
SUB ZR ZR ZR
JRL -4 ZR G0
AND ZR ZR ZR
XOR ZR ZR ZR
```

next cycleボタンをクリックしていき、JRL命令が実行されると実行中のアドレスから4が引かれ、2つ前の命令にジャンプします。またJRL命令の次の命令のアドレスがG0に格納されます。

またプログラム停止させる命令である`JRL 0 ZR ZR`も試してみましょう。以下のプログラムをエミュレータで実行すると、JRLが実行された後は実行中のアドレス(PC)がnext cycleボタンを押しても動かなくなります。

```
ADD ZR ZR ZR
SUB ZR ZR ZR
AND ZR ZR ZR
XOR ZR ZR ZR
JRL 0 ZR ZR
```

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

| instr | `[15:12]` | `[11:8]` | `[7:4]` | `[3:0]` |
| ----- | ------- | ------ | ----- | ---- |
| ジャンプ命令 | `imm[3:0]` | `rs1[3:0]` | `rd[3:0]` | `opcode[3:0]` |

| opcode[3:0] | name | operation | assembly | binary |
| ----------- | ---- | --------- | -------- | ------ |
| `4'hC` | JAL | `pc + 2 ➜ rd` <br>`imm + rs1 ➜ pc` | `JAL 4 G7 G8` | `16'h4BCC` |
| `4'hD` | JRL | `pc + 2 ➜ rd` <br>`imm + rs1 + pc ➜ pc` | `JRL 4 G7 G8` | `16'h4BCD` |

#### 分岐命令

分岐命令とは文字通り分岐を行う命令であり、強制的にプログラムの流れを変えるジャンプ命令とは異なり、条件に合致した場合にプログラムの流れを変える。

分岐命令のビットフィールドは以下の通りであり、LSBから下位4bitがオペコード、そこから2bitがソースレジスタ１、続いて2bitがソースレジスタ２、MSBから8bitが即値となっている。

| instr | `[15:8]` | `[7:6]` | `[5:4]` | `[3:0]` |
| ----- | ------- | ------ | ----- | ---- |
| 分岐命令 | `imm[7:0]` | `rs2[1:0]` | `rs1[1:0]` | `opcode[3:0]` |

レジスタを指定するフィールドの幅がそれぞれ2bitであるため、0~3番目のレジスタ`ZR, B0, B1, B2`のみをソースレジスタに指定できる。

##### BEQ

**BEQ**とは、Branch Equalの略であり、`BEQ RS2 RS1 IMM`と記述する。動作としてはRS1とRS2の値が等しい場合に自身のアドレスに即値を加えたアドレスにジャンプする。オペコードは`4'hE`であるため、仮に`BEQ B2 B1 100`は`16'h649E`となる。

即値のフィールドは符号付き8bitであり、-128~127の値を取る。

<p>
$$
\begin{align}
\text{if}\ \text{RS2}\ &== \text{RS1} \\
\text{then}\ \text{imm} &+ \text{PC} \rightarrow \text{PC}
\end{align}
$$
</p>

実際にBEQ命令を使ってみましょう。エミュレータにアクセスし、以下の命令列を実行します。

[Z16エミュレータ](/Z16Emulator.html)

```
BEQ B1 B2 4
ADD G1 G2 G3
SUB G1 G2 G3
JRL 0 ZR ZR
```

next cycleボタンを押していくと、B1とB2の値が異なる場合はBEQから順に通常通りに実行され、B1とB2の値が同じ場合はADD命令が飛ばれます。B1とB2の値を変えて試してみてください。

##### BLT

**BLT**とは、Branch Less Thanの略であり、`BLT RS2 RS1 IMM`と記述する。動作としてはRS2の値がRS1の値より大きい場合に自身のアドレスに即値を加えたアドレスにジャンプする。オペコードは`4'hF`であるため、仮に`BLT B2 ZR 100`は`16'h644F`となる。

即値のフィールドは符号付き8bitであり、-128~127の値を取る。

<p>
$$
\begin{align}
\text{if}\ \text{RS2}\ &> \text{RS1} \\
\text{then}\ \text{imm} &+ \text{PC} \rightarrow \text{PC}
\end{align}
$$
</p>

実際にBEQ命令を使ってみましょう。エミュレータにアクセスし、以下の命令列を実行します。

[Z16エミュレータ](/Z16Emulator.html)

```
ADDI -4 B1
ADDI 10 B2
BLT B2 B1 4
ADD ZR ZR ZR
SUB G7 G7 G7
```

next cycleボタンを押していくとBLT命令で`10 > -4`が評価され、真であるため分岐が発生し、実行している命令アドレスに4が加算されADD命令が飛ばされSUB命令が実行されます。

##### 分岐命令まとめ

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

このアセンブリをエミュレータで実際に動かしてみましょう。エミュレータにアクセスし、上記のアセンブリをコピペして実行してみてください。

[Z16エミュレータ](/Z16Emulator.html)

PCが停止するまでnext cycleボタンを押していくと、B2に1~10の総和が格納されます。

分岐命令一覧表

| instr | `[15:8]` | `[7:6]` | `[5:4]` | `[3:0]` |
| ----- | ------- | ------ | ----- | ---- |
| 分岐命令 | `imm[7:0]` | `rs2[1:0]` | `rs1[1:0]` | `opcode[3:0]` |

| opcode[3:0] | name | operation | assembly | binary |
| ----------- | ---- | --------- | -------- | ------ |
| `4'hE` | Branch equal | `if rs2 == rs1 `<br>`then imm + pc ➜ pc` | `BEQ B1 B2 -4` | `16'hFC5E` |
| `4'hF` | Branch Less than | `if rs2 > rs1 `<br>`then imm + pc ➜ pc` | `BLT B1 B2 -12` | `16'hF45F` |

### Z16のエミュレータ

[Z16Emulator](/Z16Emulator.html)

アセンブラ及びエミュレータとしてご利用ください。

## ディジタルビルディングブロック

座学は終わりです。実際に手を動かしていきましょう。

これはCPUの大まかな内部構造です。ご覧の通りCPUは多くの部品で構成されていますが、各部品はそこまで複雑ではありません。本章ではCPUの各部品について説明し、実際に実装していきます。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/computer_abst.png)

### Register File

**Register File**、これは**レジスタファイル**と呼び、CPUが少量のデータを保持するのに使います。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/regfile.png)

動作としてはアドレスを入力するとデータを出力する**読み出し**と、

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/regfile_read.png)

アドレスとデータを入力するとレジスタファイルに書き込まれる**書き込み**を行います。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/regfile_write.png)

#### 仕様策定

レジスタファイルの働きを理解したところで、我々が実装するレジスタファイルの仕様を決めましょう。

Z16は16bitのレジスタを16個持っていると説明しました。よって16bitのレジスタが16個必要です。そのまんまですね。

- 16bitのレジスタ x 16

またZ16の演算命令を思い出してください。Z16の演算命令は`ADD RS2 RS1 RD`のように、レジスタファイルから２つのデータを読み出し、演算結果である１つのデータをレジスタファイルへ書き込みます。つまり読み出すためのポートは２つ必要で、書き込むためのポートは１つ必要です。

- 読み出しポート x 2
- 書き込みポート x 1

そしてZ16には特殊なレジスタとして、`ZR`が定義されていました。これは常に値が0のレジスタでしたね。読み出しても書き込んでも常に値が0のレジスタは、値を読み出すために入力されたアドレスが0の場合に0を出力する機構を作れば実装が可能です。この`ZR`の事を便宜上ゼロレジスタと呼びます。

- ゼロレジスタ

レジスタファイルの仕様はこんなところですかね。仕様をまとめると以下になります。今後もモジュールを作る時は仕様をまず決めましょう。

- 16bitのレジスタ x 16
- 読み出しポート x 2
- 書き込みポート x 1
- ゼロレジスタ

#### 実装

では実装に移ります。`Z16RegisterFile.v`という名前のファイルを作成して、以下の記述を書き写してください。コピペではダメです。最初はインターフェイスから作成しましょう。

読み出しポートを２つ作ります。`i_rs1_addr`と`o_rs1_data`、`i_rs2_addr`と`o_rs2_data`は両方とも4bitのアドレス入力を受け、16bitのデータを出力する読み出しポートです。

次に書き込みポートを１つ作ります。`i_rd_addr`は書き込み先アドレス入力、`i_rd_wen`が書き込みの有効化、`i_rd_data`が書き込みデータとなっています。Z16にはレジスタに値を書き込まない命令もありますので、`i_rd_wen`で書き込むか否かを制御出来るようにしています。

```verilog
module Z16RegisterFile(
  input  wire           i_clk,      // クロック

  input  wire   [3:0]   i_rs1_addr, // RS1アドレス
  output wire   [15:0]  o_rs1_data, // RS1読み出しデータ

  input  wire   [3:0]   i_rs2_addr, // RS2アドレス
  output wire   [15:0]  o_rs2_data, // RS2読み出しデータ

  input  wire   [3:0]   i_rd_addr,  // RDアドレス
  input  wire           i_rd_wen,   // RD書き込み有効化
  input  wire   [15:0]  i_rd_data   // RD書き込みデータ
);

    // ここに回路を記述
endmodule
```

次に16bitのレジスタを16個作成しましょう。`reg`の二次元配列を使えば一撃です。

```verilog
  // レジスタファイル本体
  reg [15:0] mem[15:0];
```

次にレジスタの読み出し機構を作ります。三項演算子を使っており一見トリッキーに見えますがその実シンプルです。`i_rs1_addr == 4'h0`で入力されたアドレスが0の場合は`16'h0000`を出力し、それ以外の場合は`mem[i_rs1_addr]`でレジスタファイルから値を取り出し、`o_rs1_data`へデータを出力しています。RS2の場合も同様です。

ここのアドレスが0の場合に0を出力する機構が、ゼロレジスタの実装となっています。

```verilog
  // Read
  // アドレスが0なら0を出力
  assign o_rs1_data = (i_rs1_addr == 4'h0) ? 16'h0000 : mem[i_rs1_addr];
  assign o_rs2_data = (i_rs2_addr == 4'h0) ? 16'h0000 : mem[i_rs2_addr];
```

最語にレジスタへの書き込み機構を作ります。こちらは簡単です。`i_rd_wen`が1になり、書き込みが有効化された時にのみ`mem[i_rd_addr]`で指定したレジスタに対して`i_rd_data`の値を書き込んでいます。それ以外の場合は値が更新されないようにしています。

```verilog
  // Write
  always @(posedge i_clk) begin
    if(i_rd_wen) begin
      mem[i_rd_addr]  <= i_rd_data;
    end else begin
      mem[i_rd_addr]  <= mem[i_rd_addr];
    end
  end
```

以上の記述をまとめたものが以下になります。書き写した内容と見比べるのに使ってください。

```verilog
module Z16RegisterFile(
  input  wire           i_clk,      // クロック

  input  wire   [3:0]   i_rs1_addr, // RS1アドレス
  output wire   [15:0]  o_rs1_data, // RS1読み出しデータ

  input  wire   [3:0]   i_rs2_addr, // RS2アドレス
  output wire   [15:0]  o_rs2_data, // RS2読み出しデータ

  input  wire   [3:0]   i_rd_addr,  // RDアドレス
  input  wire           i_rd_wen,   // RD書き込み有効化
  input  wire   [15:0]  i_rd_data   // RD書き込みデータ
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

#### 動作の確認

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

### Data Mem

**Data Mem**、これは**データメモリ**と呼び。CPUがレジスタファイルに収まらない大量のデータを保存するのに使います。CPUはデータメモリ対してデータの書き込みと読み出しを行います。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/data_mem.png)

メモリに対してのデータ書き込みと読み出しにはそれぞれ名前があり、メモリへのデータ書き込みを**ストア(Store)**、

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/data_mem_store.png)

メモリからのデータ読み出しを**ロード(Load)**と呼びます。Store命令、Load命令とよく使われる単語ですので覚えておきましょう。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/data_mem_load.png)

#### 仕様策定

データメモリの役割を理解したところで、我々が作成するデータメモリの仕様を決めましょう。

まずはデータメモリのサイズを決めます。Z16ではメモリアドレスとして16bitの値が使えるので、最大65536(2^16) x 16bitのサイズを持つデータメモリを作成することが出来ます。ですが、本記事では非常に効率の悪いデータメモリの実装を行うので、そこまで大きなメモリを作ることは出来ません。そこで本記事では1024 x 16bitのデータメモリを作ることにします。

- 1024 x 16bit

次にインターフェイスの仕様を決めましょう。Z16においてLOAD命令はデータメモリから１つのデータを取り出し、STORE目入れはデータメモリに１つのデータを書き込みます。よってZ16のメモリは１つの読み出しポートと１つの書き込みのポートがあれば十分です。またLOAD命令とSTORE命令が同時に実行されることはありませんので、アドレス入力はLOADとSTOREで共通にします。

- 読み出しポート x 1
- 書き込みポート x 1
- アドレス入力は共通

ここまでで決めたデータメモリの仕様を以下にまとめます。

- 1024 x 16bit
- 読み出しポート x 1
- 書き込みポート x 1
- アドレス入力は共通

データメモリを実装するのに必要な仕様はこんなもんでしょう。実装に移ります。

#### 実装

ではデータメモリの実装をしていきます。`Z16DataMemory.v`という名前のファイルを作成して以下の記述を書き写してください。最初はインターフェイスから作成しましょう。

まずは読み出しと書き込み先を指定するアドレス入力である`i_addr`を定義し、続いて書き込み有効化信号の`i_wen`、書き込みデータの`i_data`、読み出しデータの`o_data`を定義しています。

```verilog
module Z16DataMemory(
  input  wire           i_clk,  // クロック

  input  wire   [15:0]  i_addr, // アドレス
  input  wire           i_wen,  // 書き込み制御
  input  wire   [15:0]  i_data, // 書き込みデータ
  output wire   [15:0]  o_data  // 読み出しデータ
);

    // ここに回路を記述
endmodule
```

次に16bit x 1024サイズのメモリを作成しましょう。こちらもレジスタファイルと同じく`reg`の二次元配列を使えば一撃です。

```verilog
  reg [15:0] mem[1023:0];
```

続いてLOADを行う機構を作成しましょう。`i_addr`で指定されたメモリアドレスを用いて`mem[i_addr[10:1]]`でメモリからデータを取り出し、`o_data`へデータを出力しています。

```verilog
  // Load
  assign o_data = mem[i_addr[10:1]];
```

最後にSTOREを行う機構を作成しましょう。これは非常に単純です。`i_wen`でデータ書き込みが有効な場合に、`i_addr`で指定されたメモリアドレスを用いて`mem[i_addr[10:1]]`でメモリを指定し、`i_data`のデータを入力しています。

```verilog
  always @(posedge i_clk) begin
    // Store
    if(i_wen) begin
      mem[i_addr[10:1]] <= i_data;
    end
  end
```

これでデータメモリは完成です。以上の記述をまとめたものが以下になります。書き写した内容と見比べるのに使ってください。

```verilog
module Z16DataMemory(
  input  wire           i_clk,  // クロック

  input  wire   [15:0]  i_addr, // アドレス
  input  wire           i_wen,  // 書き込み制御
  input  wire   [15:0]  i_data, // 書き込みデータ
  output wire   [15:0]  o_data  // 読み出しデータ
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

#### 動作の確認

データメモリが完成しましたら、テストベンチで動作を確認しましょう。以下のテストベンチでは、`0x100`番目のアドレスに`16'h5555`をストアした後、ロードしています。

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

以下のコマンドで動作を確認できます。

```bash
iverilog Z16DataMemory_tb.v Z16DataMemory.v
vvp a.out
gtkwave wave.vcd
```

### Instr Mem

**Instr Mem**、これは**命令メモリ(Instruction Memory)**と呼び、データメモリに似ていますが中には命令が入っています。命令メモリ内の各命令にはアドレスが振られており、命令メモリにアドレスを入力すると命令が出力されます。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/instr_mem.png)

#### 仕様策定

命令メモリの役割を理解したところで、我々が作成する命令メモリの仕様を決めましょう。

命令メモリは読み出ししか行えないメモリですので、読み出しポートが１つあれば十分です。

- 読み出しポート x 1

また命令メモリのサイズですが、これは今の所は格納する命令列と同じサイズにしておきます。つまりサイズは命令長 x 16bitです。

- 命令長 x 16bit

決定した命令メモリの仕様を以下にまとめます。

- 読み出しポート x 1
- 命令長 x 16bit

短いですね。実装に移ります。

#### 実装

では命令メモリの実装をしていきます。`Z16InstrMemory.v`という名前のファイルを作成して以下の記述を書き写してください。最初はインターフェイスから作成しましょう。

命令メモリはアドレスを受け取って命令を出力する回路ですので、定義すべきインターフェイスはアドレス入力である`i_addr`と命令出力の`o_instr`だけです。

```verilog
module Z16InstrMemory(
  input  wire   [15:0]  i_addr, // アドレス入力
  output wire   [15:0]  o_instr // 命令出力
);

    // ここに回路を記述
endmodule
```

次に16bit x 命令長サイズの命令メモリを作成しましょう。こちらは`wire`の二次元配列を使います。また格納する命令長は5とします。

```verilog
  wire [15:0] mem[4:0];
```

続いて命令を読み出す機構を作成しましょう。`i_addr`で指定されたメモリアドレスを用いて`mem[i_addr[10:1]]`でメモリからデータを取り出し、`o_instr`へデータを出力しています。

```verilog
  assign o_instr = mem[i_addr[15:1]];
```

最後に命令メモリに書き換えが不可能な形で命令を書き込みます。`mem`は`wire`で定義されていますので、書き込む時は`assign`です。`mem[]`でアドレスを指定し、命令を入力しています。ここでは初期値として1~10の総和を求めるプログラムを格納してあります。

```verilog
  assign mem[0] = 16'h0A19;
  assign mem[1] = 16'h1220;
  assign mem[2] = 16'hFF19;
  assign mem[3] = 16'hFC4F;
  assign mem[4] = 16'h00FD;
```

これで命令メモリは完成です。以上の記述をまとめたものが以下になります。書き写した内容と見比べるのに使ってください。

```verilog
module Z16InstrMemory(
  input  wire   [15:0]  i_addr, // アドレス入力
  output wire   [15:0]  o_instr // 命令出力
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

#### 動作の確認

命令メモリが完成しましたら、テストベンチで動作を確認しましょう。以下のテストベンチでは全ての命令をクロック毎に読み出しています。

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
    $finish;
  end

endmodule
```

以下のコマンドで動作を確認できます。

```bash
iverilog Z16InstrMemory_tb.v Z16InstrMemory.v
vvp a.out
gtkwave wave.vcd
```

### ALU

次はALUです。これは**算術論理演算ユニット(Arithmetic Logic Unit)**の略称で、論理演算(and, or, xor, シフト, etc..)と算術演算(加算, 減算, etc...)を行うモジュールです。CPUにおいて"計算"はこのALUが行っていると考えていただいて構いません。ALUは制御信号と２つのデータを受け取り、１つのデータを出力します。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/alu.png)

#### 仕様策定

ALUの役割を理解したところで、我々が作成するALUの仕様を決めましょう。

ALUは２つのデータを受け取って演算を行い、演算結果を出力するモジュールですので、データ入力ポートを２つとデータ出力ポートが１つ必要です。

- データ入力ポート x 2
- データ出力ポート x 1

またALUがどの演算を行うのか制御出来るように制御信号入力ポートが１つ必要です。

- 制御信号入力ポート x 1

次に制御信号がどの演算に対応するのか決めましょう。Z16では、演算命令においては多彩な演算が求められますが、それ以外の命令に関しては基本的に加算さえ行えば十分となっています。例えばLOAD命令は`Mem[IMM + RS1] -> RD`という操作を行う命令であり、アドレス計算においてALUによる`IMM + RS1`の加算が必要です。また即値命令にはADDI命令しかありません。よって演算命令以外では加算を行えば十分です。

そこで、制御信号と演算の対応を演算命令のオペコードと一致させ、演算命令以外のオペコードでは加算を行わせましょう。ややこしく感じるかもしれません。そういう時は実装と演算命令まとめの表を見比べてください。

- 演算と演算命令のオペコードを一致させる、演算命令以外は加算

ここまでで決めたデータメモリの仕様を以下にまとめます。

- データ入力ポート x 2
- データ出力ポート x 1
- 制御信号入力ポート x 1
- 演算と演算命令のオペコードを一致させる、演算命令以外は加算

#### 実装

ではALUの実装をしていきます。`Z16ALU.v`という名前のファイルを作成して以下の記述を書き写してください。最初はインターフェイスから作成しましょう。

まずはデータ入力ポートを２つ、`i_data_a`と`i_data_b`を定義します。続いて制御信号入力の`i_ctrl`、そしてデータ出力ポートの`o_data`を定義します。

```verilog
module Z16ALU(
  input  wire   [15:0]  i_data_a, // データ入力１
  input  wire   [15:0]  i_data_b, // データ入力２
  input  wire   [3:0]   i_ctrl,   // 制御信号入力
  output wire   [15:0]  o_data    // データ出力
);

    // ここに回路を記述
endmodule
```

続いて演算を行う機構を作成します。ALUの演算は入力があると即座に結果が出力される、組み合わせ回路で行って欲しいのでassign文を使いますが、`i_ctrl`による演算の制御を行いたいので、ここは複雑な記述が出来るfunction文を使います。

よって`alu`という名前の関数をfunction文で作成します。これはcase文で`i_ctrl`の値に応じて演算を切り替えるようにしています。例えば`i_ctrl`の値が5の場合は、オペコードが5の命令は演算命令のAND命令ですのでAND命令を行います。また制御信号が演算命令のオペコードの範囲外の場合は加算を行います。

```verilog
  function [15:0] alu;
    input [15:0]    i_data_a;
    input [15:0]    i_data_b;
    input [3:0]     i_ctrl;

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
        default: alu = i_data_b + i_data_a;
      endcase
    end
  endfunction
```

そして先程作成した関数`alu()`を用いて、計算結果を`o_data`に入力します。

```verilog
  assign o_data = alu(i_data_a, i_data_b, i_ctrl);
```

これでALUは完成です。以上の記述をまとめたものが以下になります。書き写した内容と見比べるのに使ってください。

```verilog
module Z16ALU(
  input  wire   [15:0]  i_data_a,
  input  wire   [15:0]  i_data_b,
  input  wire   [3:0]   i_ctrl,
  output wire   [15:0]  o_data
);

  assign o_data = alu(i_data_a, i_data_b, i_ctrl);

  function [15:0] alu;
    input [15:0]    i_data_a;
    input [15:0]    i_data_b;
    input [3:0]     i_ctrl;

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
        default: alu = i_data_b + i_data_a;
      endcase
    end
  endfunction

endmodule
```

#### 動作の確認

ALUが完成しましたら、テストベンチで動作を確認しましょう。以下のテストベンチでは、`i_data_a`に`0x4`を入力し、また`i_data_b`に`0x8`を入力してから加算、減算、乗算、除算、ORを行っています。

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

以下のコマンドで動作を確認できます。

```bash
iverilog Z16ALU_tb.v Z16ALU.v
vvp a.out
gtkwave wave.vcd
```

### PC

このCPUの中にある**PC**は**プログラムカウンタ(Program Counter)**と呼び、命令メモリにアドレスを供給します。通常、プログラムは上から下に実行されるのでプログラムカウンタが出す値は毎クロック増えていきます。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/pc_explain.png)

プログラムカウンタ・命令メモリ・CPUの関係をまとめると、プログラムカウンタがアドレスを命令メモリに入力し、命令メモリが命令をCPUへを出力するという流れになります。このデータの流れを**フェッチ(Fetch)**と呼びます。覚えておきましょう。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/fetch.png)

プログラムカウンタは後ほど実装します。

### Decoder

最後は**Decoder**です。これは**デコーダ**と呼び、命令から各種制御信号を生成します。具体的な動作の説明はCPUを実際に作成する際に説明しますので、とりあえず今は命令に応じて各部品を制御するモジュールだと認識しておいてください。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/decoder.png)

## マイクロアーキテクチャ

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

### Load命令

最初に実装する命令はLOAD命令です。LOAD命令はメモリからデータを取り出してレジスタに保存する操作でしたね、このLOAD命令の動作を箇条書きにするとこのようになります。

1. 命令メモリから命令をフェッチ
2. デコーダが命令を解釈
3. レジスタからRS1の値を取り出す
4. ALUでアドレス計算
5. データメモリからデータを取り出す
6. レジスタにデータを書き込む

かなり手順が多いですね、実際LOAD命令はZ16の命令の中で最もデータの流れが長い命令となっています。CPUを実装する際はこのLOAD命令のようなデータの流れ、つまりデータパスが最も長い命令を最初に実装した方が楽です。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/path_load.png)

#### 命令フェッチ

ではまずは命令メモリから命令をフェッチする機構を作りましょう。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/path_load_fetch.png)

`Z16CPU.v`に書き込んでいきます。

まずはプログラムカウンタの役割を担う`r_pc`というレジスタと、フェッチした命令が入力される信号線である`w_instr`を新たに定義します。

```verilog
  // プログラムカウンタ
  reg   [15:0]  r_pc;
  wire  [15:0]  w_instr;
```

次にこのプログラムカウンタを更新する機構を作ります。リセット入力が有効な場合にはこのプログラムカウンタの値を0にし、通常時は2づつカウントアップさせます。

```verilog
  always @(posedge i_clk) begin
    if(i_rst) begin
      // リセット
      r_pc  <= 16'h0000;
    end else begin
      r_pc  <= r_pc + 16'h0002;
    end
  end
```

続いてプログラムカウンタの値を命令メモリのアドレス入力に接続し、`w_instr`を命令メモリの出力に接続します。

```verilog
  Z16InstrMem InstrMem(
    .i_addr     (r_pc   ), // プログラムカウンタを命令メモリに接続
    .o_instr    (w_instr)  
  );
```

これで命令フェッチのデータパスは完成です。以下に`Z16CPU.v`の全体のコードを載せておきます。

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

#### デコーダの作成

これでフェッチ機構を実装できました。次はフェッチした命令を解釈するデコーダを作成していきましょう。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/path_decoder.png)

LOAD命令のビットフィールドは以下のように、LSBから順にオペコード、RDのアドレス、RS1のアドレス、即値となっていましたね。

| instr | `[15:12]` | `[11:8]` | `[7:4]` | `[3:0]` |
| ----- | ------- | ------ | ----- | ---- |
| LOAD命令 | `imm[3:0]` | `rs1[3:0]` | `rd[3:0]` | `opcode[3:0]` |

オペコード及びRDとRS1のアドレスはそのまま使えるのでスライスを使って取り出しましょう。`o_opcode`がオペコード、`o_rd_addr`がRDのアドレス、`o_rs1_addr`がRS1のアドレスとなっています。

```verilog
module Z16Decoder(
  input  wire   [15:0]  i_instr,    // 命令入力
  output wire   [3:0]   o_opcode,   // オペコード出力
  output wire   [3:0]   o_rd_addr,  // RDアドレス出力
  output wire   [3:0]   o_rs1_addr  // RS1アドレス出力
);

  assign o_opcode   = i_instr[3:0];
  assign o_rd_addr  = i_instr[7:4];
  assign o_rs1_addr = i_instr[11:8];

endmodule
```

次に即値ですが、これは命令内で符号付き4bitとなっています。ALUは符号付き16bitの値しか扱えませんので、4bitの値を16bitに変換する必要があります。しかし、4bitの値に12bitの0を結合するだけでは、負の値のおいて以下のような変換ミスが発生します。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/imm_expand_be.png)

よって正の値と負の値で結合する12bitの値を変える符号拡張という処理が必要になります。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/imm_expand_af.png)

そこでデコーダにおいて、命令のオペコードがLOAD命令の場合に即値の符号拡張を行う関数`get_imm()`を作成します。この関数の中では、連結演算子とビットの繰り返しを組み合わせる事で、符号拡張を実現しています。

```verilog
function [15:0] get_imm;
  input [15:0] i_instr;
begin
  case(i_instr[3:0])
    4'hA    : get_imm = { {12{i_instr[15]}}, i_instr[15:12] };
    default : get_imm = 16'h0000;
  endcase
end
endfunction
```

こうして作成した関数を使い、即値のデコード結果を即値出力である`o_imm`へ入力しましょう。

```verilog
module Z16Decoder(
  input  wire   [15:0]  i_instr,
  output wire   [3:0]   o_opcode,
  output wire   [3:0]   o_rd_addr,
  output wire   [3:0]   o_rs1_addr,
  output wire   [15:0]  o_imm       // 即値出力
);

  assign o_opcode   = i_instr[3:0];
  assign o_rd_addr  = i_instr[7:4];
  assign o_rs1_addr = i_instr[11:8];
  assign o_imm      = get_imm(i_instr);

  // 符号拡張
  function [15:0] get_imm;
    input [15:0] i_instr;
  begin
    case(i_instr[3:0])
      4'hA    : get_imm = { {12{i_instr[15]}}, i_instr[15:12] };
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
  output wire           o_rd_wen,   // レジスタ書き込み有効化信号
  output wire           o_mem_wen   // メモリ書き込み有効化信号
);
```

LOAD命令の場合、`o_rd_wen`は`1`を出力し、`o_mem_wen`は`0`を出力する必要がありますね。そのための関数である`get_rd_wen()`と`get_mem_wen()`を作成しましょう。

```verilog
function get_rd_wen;
  input [15:0] i_instr;
begin
  if(4'hA == i_instr[3:0]) begin
    get_rd_wen    = 1'b1;
  end else begin
    get_rd_wen    = 1'b0;
  end
end
endfunction

function get_mem_wen;
  input [15:0] i_instr;
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
  function [15:0] get_imm;
    input [15:0] i_instr;
  begin
    case(i_instr[3:0])
      4'hA    : get_imm = { {12{i_instr[15]}}, i_instr[15:12]};
      default : get_imm = 16'h0000;
    endcase
  end
  endfunction

  function get_rd_wen;
    input [15:0] i_instr;
  begin
    if(4'hA == i_instr[3:0]) begin
     get_rd_wen    = 1'b1;
    end else begin
      get_rd_wen    = 1'b0;
    end
  end
  endfunction

  function get_mem_wen;
    input [15:0] i_instr;
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
  output wire   [3:0]   o_alu_ctrl  // ALU演算制御信号
);
```

LOAD命令では、即値とRS1の値を加算してアドレスを作っていましたね。

<p>
$$
\text{Memory}[\text{IMM} + \text{RS1}] \rightarrow \text{RD}
$$
</p>

よってALUには加算を行ってもらう必要があります。ALUの実装を見返すと`i_ctrl`に`4'h0`を入力すると加算を行ってくれるように実装してありますね。よってLOAD命令のオペコードの場合は`4'h0`を出力する関数を作成します。

```verilog
function [3:0] get_alu_ctrl;
  input [15:0] i_instr;
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
  function [15:0] get_imm;
    input [15:0] i_instr;
  begin
    case(i_instr[3:0])
      4'hA    : get_imm = { {12{i_instr[15]}}, i_instr[15:12]};
      default : get_imm = 16'h0000;
    endcase
  end
  endfunction

  function get_rd_wen;
    input [15:0] i_instr;
  begin
      if(4'hA == i_instr[3:0]) begin
     get_rd_wen    = 1'b1;
    end else begin
      get_rd_wen    = 1'b0;
    end
  end
  endfunction

  function get_mem_wen;
    input [15:0] i_instr;
  begin
    if(4'hA == i_instr[3:0]) begin
      get_mem_wen = 1'b0;
    end else begin
      get_mem_wen = 1'b0;
    end
  end
  endfunction

  function [3:0] get_alu_ctrl;
    input [15:0] i_instr;
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
  wire [15:0]   w_rd_addr;  // RDアドレス信号線
  wire [15:0]   w_rs1_addr; // RS1アドレス信号線
  wire [15:0]   w_imm;      // 即値信号線
  wire          w_rd_wen;   // レジスタ書き込み有効化信号線
  wire          w_mem_wen;  // メモリ書き込み有効化信号線
  wire [3:0]    w_alu_ctrl; // ALU演算制御信号線


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

#### レジスタファイルからの値読み出し

次はRS1の値をレジスタから取り出すデータパスを作成しましょう。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/path_load_readreg.png)

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

  // レジスタファイル
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

#### アドレス計算

次はアドレス計算を行うデータパスを実装しましょう。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/path_load_caladdr.png)

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

  // ALU
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

#### メモリからのデータ読み出し

アドレス計算のデータパスが完成したら、次はそのアドレスをメモリに入力し、データを取り出すデータパスを作りましょう。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/path_load_readmem.png)

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

#### レジスタファイルへのデータ書き込み

最後にメモリから読み出したデータをレジスタファイルに書き込むデータパスを作成しましょう。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/path_load_writereg.png)

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

#### 動作の確認

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

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/wave_load.png)

これで、LOAD命令のデータパスが完成した事が確認できました。

### Store命令

次はSTORE命令のデータパスを作成し、CPUがSTORE命令を実行できるようにしましょう。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/path_store.png)

STORE命令の動作を箇条書きすると以下の通りになります。

1. 命令メモリから命令をフェッチ
2. デコーダが命令を解釈
3. レジスタからRS1, RS2の値を取り出す
4. ALUでアドレス計算
5. データメモリにRS2の値を書き込む

1の命令フェッチの機構はLOAD命令を実装した時に作りましたね。2のデコーダには少し手を加える必要があります。

#### デコーダを改造

ではデコーダの改造に着手しましょう。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/path_decoder.png)

STORE命令のビットフィールドは以下のように、LSBから順にオペコード、即値、RS1のアドレス、RS2のアドレスとなっていましたね。

| instr | `[15:12]` | `[11:8]` | `[7:4]` | `[3:0]` |
| ----- | ------- | ------ | ----- | ---- |
| STORE命令 | `rs2[3:0]` | `rs1[3:0]` | `imm[3:0]` | `opcode[3:0]` |

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
function [15:0] get_imm;
  input [15:0] i_instr;
begin
  case(i_instr[3:0])
    4'hA  : get_imm   = { {12{i_instr[15]}}, i_instr[15:12]};
    4'hB  : get_imm   = { {12{i_instr[7]}}, i_instr[7:4]};
    default : get_imm = 16'h0000;
  endcase
end
endfunction
```

次にSTORE命令はメモリに値を書き込む命令ですので、STORE命令が来た時はメモリ書き込み有効化信号である`o_mem_wen`を1にする必要があります。Z16において、メモリに値を書き込むのはSTORE命令の場合のみですので、`get_mem_wen`を改造し、オペコードがSTORE命令の`4'hB`の場合にのみ1になるようにします。

```verilog
function get_mem_wen;
  input [15:0] i_instr;
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

<p>
$$
\text{RS2} \rightarrow \text{Memory}[\text{RS1} + \text{IMM}]
$$
</p>

これはLOAD命令と同じですので、とりあえず今はALUには常に加算をやってもらうように改造します。

```verilog
function [3:0] get_alu_ctrl;
  input [15:0] i_instr;
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
  function [15:0] get_imm;
    input [15:0] i_instr;
  begin
    case(i_instr[3:0])
      4'hA  : get_imm   = { {12{i_instr[15]}}, i_instr[15:12]}; 
      4'hB  : get_imm   = { {12{i_instr[7]}}, i_instr[7:4]};
      default : get_imm = 16'h0000;
    endcase
  end
  endfunction

  function get_rd_wen;
    input [15:0] i_instr;
  begin
    if(4'hA == i_instr[3:0]) begin
      get_rd_wen    = 1'b1;
    end else begin
      get_rd_wen    = 1'b0;
    end
  end
  endfunction

  function get_mem_wen;
    input [15:0] i_instr;
  begin
    // STORE命令の場合にのみ1
    if(4'hB == i_instr[3:0]) begin
      get_mem_wen = 1'b1;
    end else begin
      get_mem_wen = 1'b0;
    end
  end
  endfunction

  function [3:0] get_alu_ctrl;
    input [15:0] i_instr;
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

#### レジスタファイルからの値読み出し

次はレジスタファイルからRS2の値を読み出すデータパスを作成しましょう。これで読み出されたRS2の値がメモリに書き込まれる値になります。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/path_store_readreg.png)

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

#### メモリへの値書き込み

最後に、メモリへデータを書き込むデータパスを作成していきます。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/path_store.png)

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

#### 動作の確認

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

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/wave_store.png)

これで、STORE命令のデータパスが完成した事が確認できました。

### 演算命令

さて、次は演算命令のデータパスを実装し、ADDやSUB、MUL命令などを実行できるようにしましょう。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/path_exe.png)

演算命令の動作を箇条書すると以下の通りになります。

1. 命令メモリから命令をフェッチ
2. デコーダが命令を解釈
3. レジスタからRS1, RS2の値を取り出す
4. ALUで演算
5. レジスタに演算結果を書き込む

命令フェッチは既に実装してありますので、デコーダの改造に着手しましょう。

#### デコーダを改造

ではデコーダの改造に着手します。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/path_decoder.png)

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
function get_rd_wen;
  input [15:0] i_instr;
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
function [3:0] get_alu_ctrl;
  input [15:0] i_instr;
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
  function [15:0] get_imm;
    input [15:0] i_instr;
  begin
    case(i_instr[3:0])
      4'hA  : get_imm   = { {12{i_instr[15]}}, i_instr[15:12]}; 
      4'hB  : get_imm   = { {12{i_instr[7]}}, i_instr[7:4]};
      default : get_imm = 16'h0000;
    endcase
  end
  endfunction

  function get_rd_wen
    input [15:0] i_instr;
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

  function get_mem_wen;
    input [15:0] i_instr;
  begin
    // STORE命令の場合にのみ1
    if(4'hB == i_instr[3:0]) begin
      get_mem_wen = 1'b1;
    end else begin
      get_mem_wen = 1'b0;
    end
  end
  endfunction

  function [3:0] get_alu_ctrl;
    input [15:0] i_instr;
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

#### ALUで演算

デコーダの改造が完了したら次はALUで演算を行うデータパスを実装しましょう。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/path_exe_alu.png)

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

#### レジスタへの演算結果書き込み

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

### 即値命令

続いて即値命令のデータパスを実装しましょう。

即値命令の動作を箇条書きすると以下の通りになります。

1. 命令メモリから命令をフェッチ
2. デコーダが命令を解釈
3. レジスタからRDの値を取り出す
4. ALUで演算
5. レジスタに演算結果を書き込む

#### デコーダを改造

ではデコーダの改造に着手します。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/path_decoder.png)

Z16の即値命令であるADDIは、RDの値に即値を加算してRDに格納する命令でした。

<p>
$$
\text{IMM} + \text{RD} \rightarrow \text{RD}
$$
</p>

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

  function [3:0] get_rs1_addr;
    input [15:0] i_instr;
  begin
    case(i_instr[3:0])
      4'h9  :   get_rs1_addr    = i_instr[7:4];
      default : get_rs1_addr    = i_instr[11:8];
    endcase
  end
  endfunction
```

次に即値に対処します。ADDIのビットフィールドには、`[15:8]`に符号付き8bitの即値が存在していました。

| instr | `[15:8]` | `[7:4]` | `[3:0]` |
| ----- | ------- | ------ | ----- |
| 即値命令 | `imm[7:0]` | `rd[3:0]` | `opcode[3:0]` |

デコーダの`get_imm()`を改造し、ADDI命令の場合は符号付き8bitの即値を16bitに符号拡張するようにしましょう。

```verilog
// 符号拡張
function [15:0] get_imm;
  input [15:0] i_instr;
begin
  case(i_instr[3:0])
    4'h9  : get_imm   = { {8{i_instr[15]}}, i_instr[15:8]};
    4'hA  : get_imm   = { {12{i_instr[15]}}, i_instr[15:12]}; 
    4'hB  : get_imm   = { {12{i_instr[7]}}, i_instr[7:4]};
    default : get_imm = 16'h0000;
  endcase
end
endfunction
```

そして、ADDI命令は演算命令と同じくレジスタファイルに値を書き込む命令ですので、`get_rd_wen()`を改造し、ADDI命令の場合は`o_rd_wen`を1にするようにします。

```verilog
function get_rd_wen;
  input [15:0] i_instr;
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
function get_rd_wen;
  input [15:0] i_instr;
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
function [3:0] get_alu_ctrl;
  input [15:0] i_instr;
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

  function [3:0] get_rs1_addr;
    input [15:0] i_instr;
  begin
    case(i_instr[3:0])
      4'h9  :   get_rs1_addr    = i_instr[7:4];
      default : get_rs1_addr    = i_instr[11:8];
    endcase
  end
  endfunction

  // 符号拡張
  function [15:0] get_imm;
    input [15:0] i_instr;
  begin
    case(i_instr[3:0])
      4'h9  : get_imm   = { {8{i_instr[15]}}, i_instr[15:8]};
      4'hA  : get_imm   = { {12{i_instr[15]}}, i_instr[15:12]}; 
      4'hB  : get_imm   = { {12{i_instr[7]}}, i_instr[7:4]};
      default : get_imm = 16'h0000;
    endcase
  end
  endfunction

  function get_rd_wen;
    input [15:0] i_instr;
  begin
    if(i_instr[3:0] <= 4'hA) begin
      get_rd_wen    = 1'b1;
    end else begin
      get_rd_wen    = 1'b0;
    end
  end
  endfunction

  function get_mem_wen;
    input [15:0] i_instr;
  begin
    // STORE命令の場合にのみ1
    if(4'hB == i_instr[3:0]) begin
      get_mem_wen = 1'b1;
    end else begin
      get_mem_wen = 1'b0;
    end
  end
  endfunction

  function [3:0] get_alu_ctrl;
    input [15:0] i_instr;
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

#### 動作の確認

それでは先程実装した演算命令と即値命令の動作確認を行っていきましょう。

動作確認用のプログラムとして、ADDIの説明の際に用いたサンプルプログラムを改造して使いましょう。

<p>
$$
\begin{align}
x &= 120 \\
y &= 87 \\
f(x, y) &= 3\times x + 100 + y
\end{align}
$$
</p>

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

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/wave_imm.png)

これを10進数に直すと547であり、上記の数式の計算結果と一致しています。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/cal_imm.png)

これで我々のCPUは演算命令と即値命令を実行できることが確認できました。やったね。

### ジャンプ命令

どんどん行きましょう。次はジャンプ命令のデータパスを作成していきます。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/path_jump.png)

ジャンプ命令の動作を箇条書すると以下の通りになります。

1. 命令メモリから命令をフェッチ
2. デコーダが命令を解釈
3. レジスタからRS1の値を読み出し
4. ALUでジャンプ先アドレスを計算
5. レジスタにPC + 2の値を書き込み
6. PCに書き込み

やることは変わりません。まずはデコーダを改造しましょう。

#### デコーダの改造

ではデコーダの改造に着手します。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/path_decoder.png)

ジャンプ命令のビットフィールドは以下のように、MSBから4bitに符号付き4bit即値が存在しています。

| instr | `[15:12]` | `[11:8]` | `[7:4]` | `[3:0]` |
| ----- | ------- | ------ | ----- | ---- |
| ジャンプ命令 | `imm[3:0]` | `rs1[3:0]` | `rd[3:0]` | `opcode[3:0]` |

この符号付き4bitを符号付き16bitに符号拡張してやる必要があります。という訳でデコーダの`get_imm()`を改造してジャンプ命令の場合に`w_instr[15:12]`を符号拡張するようにしましょう。

```verilog
// 符号拡張
function [15:0] get_imm;
  input [15:0] i_instr;
begin
  case(i_instr[3:0])
    4'h9  : get_imm   = { {8{i_instr[15]}}, i_instr[15:8]};
    4'hA  : get_imm   = { {12{i_instr[15]}}, i_instr[15:12]}; 
    4'hB  : get_imm   = { {12{i_instr[7]}}, i_instr[7:4]};
    4'hC  : get_imm   = { {12{i_instr[15]}}, i_instr[15:12]};
    4'hD  : get_imm   = { {12{i_instr[15]}}, i_instr[15:12]};
    default : get_imm = 16'h0000;
  endcase
end
endfunction
```

ジャンプ命令のJALとJRLの両方ともRDにPC + 2の値を格納する動作をしますので、ジャンプ命令のオペコードの場合は`w_rd_wen`を1にする必要があります。

- JAL

<p>
$$
\begin{align}
\text{PC} + 2 &\rightarrow \text{RD} \\
\text{IMM} + \text{RS1} &\rightarrow \text{PC}
\end{align}
$$
</p>

- JRL

<p>
$$
\begin{align}
\text{PC} + 2 &\rightarrow \text{RD} \\
\text{IMM} + \text{RS1} + \text{PC} &\rightarrow \text{PC}
\end{align}
$$
</p>

よって`get_rd_wen()`を改造して対応します。

```verilog
function get_rd_wen;
  input [15:0] i_instr;
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

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/path_jump_caladdr.png)

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

  function [3:0] get_rs1_addr;
    input [15:0] i_instr;
  begin
    case(i_instr[3:0])
      4'h9  :   get_rs1_addr    = i_instr[7:4];
      default : get_rs1_addr    = i_instr[11:8];
    endcase
  end
  endfunction

  // 符号拡張
  function [15:0] get_imm;
    input [15:0] i_instr;
  begin
    case(i_instr[3:0])
      4'h9  : get_imm   = { {8{i_instr[15]}}, i_instr[15:8]};
      4'hA  : get_imm   = { {12{i_instr[15]}}, i_instr[15:12]}; 
      4'hB  : get_imm   = { {12{i_instr[7]}}, i_instr[7:4]};
      4'hC  : get_imm   = { {12{i_instr[15]}}, i_instr[15:12]};
      4'hD  : get_imm   = { {12{i_instr[15]}}, i_instr[15:12]};
      default : get_imm = 16'h0000;
    endcase
  end
  endfunction

  function get_rd_wen;
    input [15:0] i_instr;
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

  function get_mem_wen;
    input [15:0] i_instr;
  begin
    // STORE命令の場合にのみ1
    if(4'hB == i_instr[3:0]) begin
      get_mem_wen = 1'b1;
    end else begin
      get_mem_wen = 1'b0;
    end
  end
  endfunction

  function [3:0] get_alu_ctrl;
    input [15:0] i_instr;
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

#### レジスタにPC + 2の値を格納

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

  function [15:0] select_rd_data;
    input   [3:0]   i_opcode;
    input   [15:0]  i_mem_rdata;
    input   [15:0]  i_pc;
    input   [15:0]  i_alu_data;
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

#### PCにジャンプ先のアドレス書き込み

次にプログラムカウンタにジャンプ先のアドレスを書き込む機構を実装します。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/path_jump.png)

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

  function [15:0] select_rd_data;
    input   [3:0]   i_opcode;
    input   [15:0]  i_mem_rdata;
    input   [15:0]  i_pc;
    input   [15:0]  i_alu_data;
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

#### 動作の確認

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

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/wave_jump.png)

`r_pc`の値を見てみると0 -> 2 -> 8 -> 0とループしており、またレジスタにはジャンプ命令の次の命令のアドレスが格納されている事が確認できます。

これでジャンプ命令のデータパスが完成しました。

### 分岐命令

さて最後です。分岐命令のデータパスを実装しましょう。

分岐命令の動作を箇条書すると以下の通りになります。

1. 命令メモリから命令をフェッチ
2. デコーダが命令を解釈
3. レジスタからRS1, RS2の値を読み出し
4. 条件に合致する場合はPCに書き込み

意外と少ないですね。ご多分に漏れずデコーダの改造に着手しましょう。

#### デコーダの改造

ではデコーダの改造に着手します。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/path_decoder.png)

分岐命令のビットフィールドは他の命令と比べてかなり特異なものでした。

| instr | `[15:8]` | `[7:6]` | `[5:4]` | `[3:0]` |
| ----- | ------- | ------ | ----- | ---- |
| 分岐命令 | `imm[7:0]` | `rs2[1:0]` | `rs1[1:0]` | `opcode[3:0]` |

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

  function [3:0] get_rs1_addr;
    input [15:0] i_instr;
  begin
    case(i_instr[3:0])
      4'h9  :   get_rs1_addr    = i_instr[7:4];
      4'hE  :   get_rs1_addr    = {2'b00, i_instr[5:4]};
      4'hF  :   get_rs1_addr    = {2'b00, i_instr[5:4]};
      default : get_rs1_addr    = i_instr[11:8];
    endcase
  end
  endfunction

  function [3:0] get_rs2_addr;
    input [15:0] i_instr;
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
function [15:0] get_imm;
  input [15:0] i_instr;
begin
  case(i_instr[3:0])
    4'h9  : get_imm   = { {8{i_instr[15]}}, i_instr[15:8]};
    4'hA  : get_imm   = { {12{i_instr[15]}}, i_instr[15:12]}; 
    4'hB  : get_imm   = { {12{i_instr[7]}}, i_instr[7:4]};
    4'hC  : get_imm   = { {12{i_instr[15]}}, i_instr[15:12]};
    4'hD  : get_imm   = { {12{i_instr[15]}}, i_instr[15:12]};
    4'hE  : get_imm   = { {8{i_instr[15]}}, i_instr[15:8]};
    4'hF  : get_imm   = { {8{i_instr[15]}}, i_instr[15:8]};
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

  function [3:0] get_rs1_addr;
    input [15:0] i_instr;
  begin
    case(i_instr[3:0])
      4'h9  :   get_rs1_addr    = i_instr[7:4];
      4'hE  :   get_rs1_addr    = {2'b00, i_instr[5:4]};
      4'hF  :   get_rs1_addr    = {2'b00, i_instr[5:4]};
      default : get_rs1_addr    = i_instr[11:8];
    endcase
  end
  endfunction

  function [3:0] get_rs2_addr;
    input [15:0] i_instr;
  begin
    case(i_instr[3:0])
      4'hE  :   get_rs2_addr    = {2'b00, i_instr[7:6]};
      4'hF  :   get_rs2_addr    = {2'b00, i_instr[7:6]};
      default : get_rs2_addr    = i_instr[15:12];
    endcase
  end
  endfunction

  // 符号拡張
  function [15:0] get_imm;
    input [15:0] i_instr;
  begin
    case(i_instr[3:0])
      4'h9  : get_imm   = { {8{i_instr[15]}}, i_instr[15:8]};
      4'hA  : get_imm   = { {12{i_instr[15]}}, i_instr[15:12]}; 
      4'hB  : get_imm   = { {12{i_instr[7]}}, i_instr[7:4]};
      4'hC  : get_imm   = { {12{i_instr[15]}}, i_instr[15:12]};
      4'hD  : get_imm   = { {12{i_instr[15]}}, i_instr[15:12]};
      4'hE  : get_imm   = { {8{i_instr[15]}}, i_instr[15:8]};
      4'hF  : get_imm   = { {8{i_instr[15]}}, i_instr[15:8]};
      default : get_imm = 16'h0000;
    endcase
  end
  endfunction

  function get_rd_wen;
    input [15:0] i_instr;
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

  function get_mem_wen;
    input [15:0] i_instr;
  begin
    // STORE命令の場合にのみ1
    if(4'hB == i_instr[3:0]) begin
      get_mem_wen = 1'b1;
    end else begin
      get_mem_wen = 1'b0;
    end
  end
  endfunction

  function [3:0] get_alu_ctrl;
    input [15:0] i_instr;
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

#### 条件に合致する場合はPCに書き込み

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

  function [15:0] select_rd_data;
    input   [3:0]   i_opcode;
    input   [15:0]  i_mem_rdata;
    input   [15:0]  i_pc;
    input   [15:0]  i_alu_data;
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

#### 動作の確認

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

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/wave_branch.png)

波形を見てみると、最後の方でレジスタB2に16進数で`16'h0037`が格納されています。これは10進数で55であり、1~10の総和です。そして最終的に`r_pc`の値が更新されなくなっています。これはJRLによってCPUを停止出来たという事です。理想通りの動作をしていますね。

**おめでとうございます！** 分岐命令の実装が完了しました！これにてZ16のCPUは完成です。お疲れ様でした。これにてあなたは友達に「俺はCPU自作したけど、お前は？」と言えるようになりました。言いましょう。

## CPUを実用的にしよう

我々は動作するCPUを自作できたわけですが、現状ではお世辞にも「使える」CPUとは言い難いです。そこでこの自作CPUを実用的にするための努力をしてみましょう。

### MMIO

先程1~10の総和を求めた際のように、現状では計算結果を確認するにはGTKWaveでレジスタファイルの内部信号を覗いてやる必要があります。これはあまりスマートとは言えません。そこでCPUの信号を外の世界に出力する仕組みを作りましょう。

このような場合に使える仕組みが**MMIO**、メモリマップドIOです。MMIOとはデータメモリのアドレスの一部に外部入出力用のレジスタを割り当てる手法の事を指します。メモリアドレスに割り当てる事により、外部にデータを出力したい場合はSTORE命令を用い、外部からのデータを得たい場合はLOAD命令を用いる事が可能となります。下図の例ではメモリの`0x007A`にLEDの出力が割り当てられており、ここに`0x0013`をストアすると1番目と2番目と5番目のLEDが点灯します。また`0x007C`がボタンの入力に割り当てられており、ここから値をロードした際に1が得られた場合はボタンが押されており、0の場合はボタンが押されていない事を情報として得られます。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/data_mem_mmio.png)

では実際にMMIOを実装してみましょう。今回はLEDとButtonをMMIOに割り当ててみます。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/cpu_mmio.png)

#### LED

まず仕様を決めます。LEDをメモリアドレスの`0x007A`に割り当て、書き込む16bitのデータの内訳を下位6bitをLEDのオンオフ、上位10bitを未使用とします。

| `0x007A` | `[15:6]` | `[5:0]` |
| -------- | -------- | ------- |
| LED | `未使用[9:0]` | `LED[5:0]` |

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

#### Button

まず仕様を決定します。ボタンのMMIOをメモリアドレスの`0x007C`に割り当て、読み出す16bitのデータの内訳を下位1bitをボタンの状態、上位15bitを未使用とします。

| `0x007C` | `[15:1]` | `[0]` |
| -------- | -------- | ------- |
| Button | `未使用[14:0]` | `Button[0]` |

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

  function [15:0] select_rd_data;
    input   [3:0]   i_opcode;
    input           i_button;
    input   [15:0]  i_mem_rdata;
    input   [15:0]  i_pc;
    input   [15:0]  i_alu_data;
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

  function [15:0] select_rd_data;
    input   [3:0]   i_opcode;
    input           i_button;
    input   [15:0]  i_mem_rdata;
    input   [15:0]  i_pc;
    input   [15:0]  i_alu_data;
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

#### 動作の確認

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

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/wave_mmio_led.png)

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

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/wave_mmio_button.png)

これで我々の自作CPUはボタン入力とLED出力を兼ね備えた物となりました。素晴らしい。

### 実機向け：自作CPUを実機で動かす

では我々の作り出したCPUを現実に存在させましょう。

#### ラッパーの作成

現在、命令メモリである`Z16InstrMemory.v`の中にはLEDの値をカウントアップするプログラムが入っています。よって今までに我々が作ったCPUをFPGAに書き込むと、FPGAボードに載っているLEDが`000000 -> 000001 -> 000010 -> ...`と光る筈です。 

ただ問題があり、それは我々が使うFPGAボードであるTang Nano 9Kは、デフォルトでは27MHzのクロックしか使えない事です。27MHzのクロックとは1秒間に27000000回振動する信号であり、これを我々のCPUの`i_clk`に接続すると、LEDが1秒間に2700万回変化する事になります。これでは正しく動作しているか目視で確認する事が出来ません。

CPUは27MHzの速度に耐えられますが、LEDの変化速度に人間の動体視力が耐えられないという訳ですね。

またもう一つ問題というか対処すべきTang Nano 9Kの仕様があります。Tang Nano 9KのLEDとボタンは両方とも負論理という仕様です。これにより、LEDのピンに0を出力するとLEDが光り、1を出力するとLEDの光が消えます。またボタンが押されている時はボタンのピンから0が入力され、押されていない時は1が入力されます。我々が作成したZ16CPUの`i_rst`と`i_button`は、それぞれTang Nano 9Kの２つあるボタンに繋げる予定ですので、これをそのまま接続するとZ16CPUの`i_button`と`i_rst`が、ボタンが押されていない状態では１が入力される事になってしまいます。これでは使い辛いです。

以上で挙げたように、CPUを実装するFPGAボード由来の解決すべき問題がいくつか存在します。

そこで、Z16CPUを包み込むモジュールである`Z16CPU_wrapper.v`を新たに作り、そこで先で挙げた問題に対処することにしましょう。

まずは`Z16CPU_wrapper.v`という名前のVerilogファイルを作成してください。

そして`Z16CPU.v`と同じ入出力を定義します。

```verilog
module Z16CPU_wrapper(
  input  wire   i_clk,
  input  wire   i_rst,
  input  wire   i_button,
  output wire   [5:0]   o_led
);

    // ここに回路を記述
endmodule
```

ではまずはクロック速度に対処します。今回は27MHzのクロックから10Hzのクロックを生成します。`r_clk`と`r_counter`というレジスタを新たに定義し、0で初期化します。

```verilog
  reg r_clk = 1'b0;
  reg [31:0] r_counter = 32'h00000000;
```

そして、入力クロックの立ち上がりエッジの数を`r_counter`でカウントし、135万回になったら`r_clk`の値を反転する事で、`r_clk`の値が10MHzのクロックになります。

```verilog
  always @(posedge i_clk) begin
    if(r_counter == 32'd1350000) begin
      r_counter <= 32'h00000000;
      r_clk     <= ~r_clk;
    end else begin
      r_counter <= r_counter + 32'h00000001;
      r_clk     <= r_clk;
    end
  end
```

次にLEDとボタンの負論理に対処します。まずはLED x 2とボタンの信号を中継するための信号線の`w_rst`、`w_button`、`w_led`を定義します。

```verilog
  wire w_rst;
  wire w_button;
  wire [5:0] w_led;
```

そしてLEDへの出力と、ボタンの入力を全て反転します。

```verilog
  assign w_rst = ~i_rst;
  assign w_button = ~i_button;
  assign o_led = ~w_led;
```

最後に作成したCPUに対して各種信号を接続します。これで全ての問題に対処できました。

```verilog
  Z16CPU CPU(
    .i_clk      (r_clk      ),
    .i_rst      (w_rst      ),
    .i_button   (w_button   ),
    .o_led      (w_led      )
  );
```

`Z16CPU_wrapper.v`のコード全体を以下に載せておきます。

```verilog
module Z16CPU_wrapper(
  input  wire   i_clk,
  input  wire   i_rst,
  input  wire   i_button,
  output wire   [5:0]   o_led
);

  wire w_rst;
  wire w_button;
  wire [5:0] w_led;
  reg [31:0] r_counter = 32'h00000000;
  reg r_clk = 1'b0;

  always @(posedge i_clk) begin
    if(r_counter == 32'd1350000) begin
      r_counter <= 32'h00000000;
      r_clk     <= ~r_clk;
    end else begin
      r_counter <= r_counter + 32'h00000001;
      r_clk     <= r_clk;
    end
  end

  assign w_rst = ~i_rst;
  assign w_button = ~i_button;
  assign o_led = ~w_led;

  Z16CPU CPU(
    .i_clk      (r_clk      ),
    .i_rst      (w_rst      ),
    .i_button   (w_button   ),
    .o_led      (w_led      )
  );

endmodule
```

次はEDAでFPGAに書き込んでいきます。

#### EDAを使う

Gowin EDAを起動し`Z16CPU`という名前でプロジェクトを作成します。Gowin EDAの使い方は[ディジタル回路とVerilog入門#プロジェクト作成](https://vlsi.jp/DigitalCircuit_And_Verilog.html#プロジェクト作成)を参照してください。

プロジェクトを作成しましたら、`Design`のプロジェクトを右クリックし、`Add Files...`から以下の今まで作成したVerilogファイルをプロジェクトに追加します。

- Z16CPU_wrapper.v
- Z16CPU.v
- Z16RegisterFile.v
- Z16DataMemory.v
- Z16InstrMemory.v
- Z16ALU.v
- Z16Decoder.v

そして論理合成を実行しましょう。10秒程で終わるはずです。

`Tools > Schematic Viewer > RTL Design Viewer`から回路図を開き、黄色で示されているCPUをダブルクリックしてCPUの回路図を覗くとCPUっぽい何かが見られて嬉しいです。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/Z16_Schematic.png)

次にピンアサインを行います。FloorPlannerを起動し、`I/O Constraints`から以下の表に従ってピンアサインを行ってください。

| ピン | インターフェイス |ピン番号 |
| ---- | ---------------- | -------- |
| クロック(27MHz) | `i_clk` | `52` |
| ボタン(S1) | `i_rst` | `4` |
| ボタン(S2) | `i_button` | `3` |
| LED0 | `o_led[0]` | `10` |
| LED1 | `o_led[1]` | `11` |
| LED2 | `o_led[2]` | `13` |
| LED3 | `o_led[3]` | `14` |
| LED4 | `o_led[4]` | `15` |
| LED5 | `o_led[5]` | `16` |

完了したら、`Ctrl + S`で上書き保存を忘れないでください。

次に配置配線を実行しましょう。これはかなり時間が掛かります。筆者の環境では3分掛かりました。

たかが3分と言っても書いたVerilogにバグがあり、実機でのデバッグを行わなければならない時にこの3分は非常にストレスになります。修正をしてから結果が出るまでにカップラーメンを作れる時間が掛かると一日なんて平気で溶けますので、今までのようにシミュレータで消せるバグは先に消しておくのが賢い選択です。

配置配線が終わりましたら、Programmerを起動してFPGAに書き込みましょう。書き込みが完了するとTang Nano 9K上で我々が作成したCPUが生成されます。

ボタン(S1)を押してリセットをしてみたり、ボタン(S2)を押してカウントを停止させてみたりしてください。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/Z16CPU_Lchika.jpg)

お疲れ様でした。これで自作CPUを現実世界に顕現させる事が出来ました。おめでとうございます。

## CPUを高速にしよう

実用的でないCPUはカスですが、遅いCPUもカスです。ここでは偉大な先人たちが作り出した高速なCPUを作るための技術を紹介いたします。実装はしません。

### パイプライン化

演算命令において、我々の作ったCPUは主に4つのステージに分けることが出来ます。まずは命令をフェッチする**IF(Instruction Fetch)ステージ**、そして命令をデコードする**ID(Instruction Decode)ステージ**、次に命令を実行する**EX(Execute)ステージ**、最後に計算結果をレジスタに書き戻す**WB(Write Back)ステージ**です。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/pipelined_cpu.png)

我々が作成したZ16 CPUは、この4つのステージの処理を１クロックで行うシングルサイクルプロセッサでした。しかしながら１クロックで全てのステージの処理を行うため、１クロックの間にデータが通るべきデータパスが長くなり、クロックの周波数を上げて高速に動作させるのが非常に難しいという欠点があります。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/single_cycle_processor.png)

そこで、各ステージの境目にレジスタを追加し、１クロックで１ステージだけ処理を行うようにします。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/insert_buffer_path.png)

そうすると、１クロックの間にデータが通るべきデータパスが短くなり、容易に動作周波数を上げる事が出来ます。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/multi_cycle_processor.png)

ただこの場合、動作周波数を４倍に出来ても１つの命令を実行するのに４クロック必要になり、結果的に処理速度は変わりません。これでは意味がありません。

そこで更に改造を加え、各ステージで並列に処理を行うようにします。つまり命令１をデコードしている間に命令２をフェッチし、次のクロックでは命令１が実行されている間に命令２がデコードされ、それと同時に命令３がフェッチされます。

このようにCPUのデータパスを各ステージに分割し、それぞれのステージを並列に動作させる技法を**パイプライン化**と呼びます。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/pipelined_processor.png)

マルチサイクルの例では、命令１から命令３が完了するまでに12クロック掛かっていました。これをパイプライン化した結果、6クロックで処理が完了しています。なんと2倍の性能向上です。ヤバいですね。

残念ながらこれは理想的な場合です。実際に素朴なパイプライン化を行うと、データハザードや構造ハザード、制御ハザードといった様々な問題が発生します。ただこれらの問題もフォワーディングなどの技法を通して解決が可能です。最高ですね。

### OoO実行

OoO実行という高速化技法を説明します。これはOut-of-Order実行の略で、簡単に言うと命令の順序を変えて高速に実行できるようにする高速化技法です。例として以下の命令列を考えます。１つの命令の実行に１サイクル掛かると仮定して、この命令列は実行するのに計6サイクル掛かります。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/OoO_0.png)

サイクル０とサイクル１の命令を見ると、この２つの命令の間にはG2についてデータ依存が存在しています。つまりSUB命令はADD命令が計算するG2の値が必要であるため、当然SUB命令はADD命令より先に実行することは不可能です。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/OoO_1.png)

しかし、その下のサイクル２とサイクル３の命令を見てみると、この２つの命令の間にはデータ依存は存在してません。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/OoO_2.png)

つまりこの２つの命令は同時に実行することが可能です。してしまいましょう。この操作によって１サイクルだけ高速に実行できるようになりました。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/OoO_baka.png)

ただ更によく見てみると、一番下のDIV命令は命令列のどの命令に対してもデータ依存を持っていません。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/OoO_3.png)

よって一番最初に実行しても何も問題ありません。してしまいましょう。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/OoO_4.png)

命令の実行順序を変えることで、結果的に２サイクル分も高速化することが出来ました。嬉しいですね。これを**Out-of-Order実行**と呼びます。

そもそもどうやって２つの命令を同時に実行するのか、レジスタファイルは２読み出し１書き込みの構造ではないのか、など様々な疑問が浮かんでいると思います。答えは簡単で、ALUの数を増やしたりVerilogを書き換えてレジスタファイルの構造を変えてやれば良いです。更に具体的な知識が欲しければ**Tomasuloのアルゴリズム**や**Scoreboarding**で調べれば出てきます。出てこない気がしてきた。頑張ってください。

### 更なる速さを求める

コンピュータアーキテクチャの世界においてパイプライン化やOut-of-Order実行は氷山の一角に過ぎず、計算機の高速化を求めた先人たちの様々な高速化技法が存在しています。シングルサイクルの16bitのCPUを作って満足ですか？更なる速さを求めたくはありませんか？加速しましょう。

Hisa Ando大先生の最強の本がオススメです。重版してくれんかな。

<iframe src="https://book.mynavi.jp/ec_products_detail_html/id=22500" frameborder="0" width="220" height="250" scrolling="yes"></iframe>

## この次へ

**無から始める自作CPU**の最後では、自作CPUのその後にどんな事がやれるのか少しだけ紹介します。

[VLSI.JP/LetsMakeCPU.html#この次へ](https://VLSI.JP/LetsMakeCPU.html#この次へ)

---

Copyright (C) Cra2yPierr0t, ALL RIGHTS RESERVED
