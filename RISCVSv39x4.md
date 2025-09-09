---
layout: default
title: RISC-V Sv39x4 非完全解説
image: "https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/RISCVSv39x4/Sv39x4.png"
---

# RISC-V Sv39x4 非完全解説

注意：RISC-V のアドレス変換をご存じ無い方は、先に以下の記事を眺めることを推奨します。

[https://vlsi.jp/UnderstandMMU.html#risc-v-sv32sv39を理解する](https://vlsi.jp/UnderstandMMU.html#risc-v-sv32sv39%E3%82%92%E7%90%86%E8%A7%A3%E3%81%99%E3%82%8B)

## イントロ

RISC-V H拡張では、Two-Stage Address Translation をサポートしている。これはアプリケーションの 仮想アドレス(GVA) を VM の 物理アドレス(GPA) へ変換し、更に GPA をハイパーバイザの物理アドレス(HPA) に 変換する２段階のアドレス変換である。

![image.png](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/RISCVSv39x4/VSGStage.png)

GVA → GPA の変換は、従来の RISC-V のアドレス変換方式と同一だが、GPA → HPA の変換は従来のアドレス変換方式と若干、本当に若干異なる。腹立つ。

ちなみに GVA → GPA は VS-Stage と呼び、GPA → HPA は G-Stage と呼びます(上図参照)。

本記事では、RISC-V H拡張で追加されたアドレス変換方式が、どの程度若干異なるのか解説する。

## おさらい：RISC-Vのアドレス変換方式

RISC-V のデフォルトのアドレス変換方式として、Sv32, Sv39, Sv48, Sv57 が定義されている。これは VA → PA のアドレス変換の仕様であり、例えば Sv39 では 39-bit の仮想アドレスを 55-bit の物理アドレスに変換する。

![image.png](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/RISCVSv39x4/VAPA.png)

RISC-V H拡張において、VS-Stage のアドレス変換では、上記のアドレス変換方式が使われるが、G-Stage のアドレス変換では新たに定義された Sv32x4, Sv39x4, Sv48x4, Sv57x4 が使われる。例えば Sv39x4 では 41-bit(！)の GPA を 55-bit の HPA に変換する。

![image.png](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/RISCVSv39x4/GVAHPA.png)

この x4 が付いてるか付いてないかでアルゴリズムが若干異なる。どの程度若干異なるか説明するために、まず Sv39 をざっと説明する。

## アドレス変換方式：Sv39

Sv39 では、アドレス変換は下図のように行う。とりあえず図を眺めて頂いて、眺め終わったら説明する。

![image.png](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/RISCVSv39x4/Sv39.png)

上から順に説明する。

まず前提として Sv39 では、39-bit の仮想アドレスは 四つのフィールド(`vpn[2]` , `vpn[1]` , `vpn[0]` , `offset`)に分けられ、区別される。 `vpn[]` は 9-bit であり、 `offset` は 12-bit である。

1. メモリアクセスに用いられる仮想アドレスが Canonical Address になっているかチェックされる。なっていない場合は Page Fault 例外が発生する。
2.  `satp` レジスタの `ppn` フィールドに `vpn[2]` と 3-bit の `000` を結合し、これが L3 Page Table  の エントリ のアドレスとなる
3. 1. で作ったアドレスでメモリアクセス、L3 Page Table のエントリを手に入れる
4. L3 Page Table エントリの `ppn` フィールドに `vpn[1]` と 3-bit の `000` を結合し、これが L2 Page Table の エントリ のアドレスとなる
5. 3. で作ったアドレスでメモリアクセス、L2 Page Table のエントリを手に入れる
6. L2 Page Table エントリの `ppn` フィールドに `vpn[0]` と 3-bit の `000` を結合し、これが L1 Page Table の エントリ のアドレスとなる
7. 5. で作ったアドレスでメモリアクセス、L1 Page Table のエントリを手に入れる
8. L1 Page Table エントリの `ppn` フィールドに `offset` を結合し、これが物理アドレスとなる

まあ普通のアドレス変換といった感じだ。無論、Gigapage や Megapage の利用によっては手順が早期に終わる場合もある。詳しくは下記の記事を参照。

[https://vlsi.jp/UnderstandMMU.html#risc-v-sv32sv39を理解する](https://vlsi.jp/UnderstandMMU.html#risc-v-sv32sv39%E3%82%92%E7%90%86%E8%A7%A3%E3%81%99%E3%82%8B)

## アドレス変換方式：Sv39x4

それでは本命の Sv39x4 について説明する。以下の図が、Sv39x4 のアドレス変換を表しているが、まあご覧の通り Sv39 の図と間違い探しと見紛うほど酷似している。

どこが異なっているのか、先に説明しておくと、まず `vpn[2]` が 2-bit 拡張され、9-bit から 11-bit になっている。これにより Sv39x4 で扱う 仮想アドレス(ゲスト物理アドレス) は 41-bit となる。

また L3 Page Table エントリのアドレスを作る際に、 `hgatp` レジスタの `ppn` フィールドの値のうち、下 2-bit が使われず、代わりに `vpn[2]` の拡張分が使われている。

残りは Sv39 と同じである。

ちなみに、 `vpn[2]` が 12-bit であるため、L3 Page Table のサイズは 16 KiB になる( $2^{16} \times 8$ Byte(PTESIZE) なので )。

では図をざっとご覧頂いた後に、アドレス変換の流れを説明する。

![image.png](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/RISCVSv39x4/Sv39x4.png)

上から順に説明する。41-bit の ゲスト物理アドレス は 四つのフィールド(`vpn[2]` , `vpn[1]` , `vpn[0]` , `offset`)に分けられ、区別される。 `vpn[2]` は 11-bit、 `vpn[1]` , `vpn[0]` は 9-bit であり、 `offset` は 12-bit である。

1. メモリアクセスに用いられる ゲスト物理アドレス が ゼロ拡張されているかチェックされる。なっていない場合は Page Fault 例外が発生する。
2.  `hgatp` レジスタの `ppn` フィールドの `[43:2]` に、 `vpn[2]` と 3-bit の `000` を結合し、これが L3 Page Table の エントリ のアドレスとなる
3. 1. で作ったアドレスでメモリアクセス、L3 Page Table のエントリを手に入れる
4. L3 Page Table エントリの `ppn` フィールドに `vpn[1]` と 3-bit の `000` を結合し、これが L2 Page Table の エントリ のアドレスとなる
5. 3. で作ったアドレスでメモリアクセス、L2 Page Table のエントリを手に入れる
6. L2 Page Table エントリの `ppn` フィールドに `vpn[0]` と 3-bit の `000` を結合し、これが L1 Page Table の エントリ のアドレスとなる
7. 5. で作ったアドレスでメモリアクセス、L1 Page Table のエントリを手に入れる
8. L1 Page Table エントリの `ppn` フィールドに `offset` を結合し、これがホスト物理アドレスとなる

本当に若干異なる。

## x4 の理由

なぜ G-Stage のアドレス変換方式では、VS-Stage の仮想アドレスを 2-bit 拡張した長さのものを 変換前のアドレスとするのか、この理由は以下のスレッドに書いてある。

[https://lists.riscv.org/g/sig-hypervisors/topic/90950271#msg83](https://lists.riscv.org/g/sig-hypervisors/topic/90950271#msg83)

Sv32 は 32-bit 仮想アドレスを 34-bit 物理アドレスに変換する方式なので、VS-Stage で Sv32 を使ったら、G-Stage が受け取るアドレスは 当然 34-bit になるので(G-Stage の出力は知らん)、G-Stage のアドレス変換方式は Sv32 を 2-bit 拡張したものになり、Sv32x4 と呼ぶことにしたと。そして RV64 で定義されてるアドレス変換方式でも一貫性を保つために、G-Stageでは x4 したものを使う事にした。という感じっぽい。解釈が間違ってたらごめんね。それとなく後で教えてください。

あとそもそも Sv32 が 32-bit 仮想アドレスを 34-bit 物理アドレスに変換する方式なのは、RISC-V定義勢の勘っぽい。

おわり。意見・訂正は @Cra2yPierr0t マデ