---
layout: default
title: 無から始める自作CPU
image: "https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/letsmakecpu.png"
---

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/letsmakecpu.png)
[クレイジーピエロ](https://x.com/Cra2yPierr0t) 著

# 無から始める自作CPU

**CPUは作れる！！！！！！！！ご存知でしたか！！！？？？？？？**

**CPU**、それは我々が暮らす情報社会の基盤となる魔法の石です。

世に存在する全てのソフトウェア、例えばゲーム、AI、Webサーバ、OS、これらは全てCPUが無ければ動きませんし、今や車や飛行機、家電にも全てCPUが入っている時代です。

そんな誰もがCPUに依存している時代にも関わらず、CPUについて理解を持っている人間は余りにも僅か、というのが現状です。

そんな今こそ**CPUを作りましょう**。

CPUを作り、完全に理解する事で、CPUによって成り立つ技術を学ぶ上での、揺るぎない自信と確証を身につける事が出来るでしょう。

本記事ではCPUという究極のブラックボックスに光を当て、半導体やプログラミングの知識が無の状態から、CPUを作る事を目標としています。

## 必要な物

本記事の内容の99%はWindows、Linux、Macのパソコンがあれば行うことが可能です。残り1%はMacを除くWindowsかLinuxのパソコンに加え、2,500円程の基板を買って頂く必要がありますが、まあその部分は飛ばして頂いても力はつくと思います。こんな変な記事の内容に従って2,500円の基板を買うだけの心の余裕がある方は買ってください。楽しいので。


## ディジタル回路とVerilog入門

**ディジタル回路とVerilog入門**では、CPUを作る前に必要な基礎知識、そして作るために必要な道具の使い方を学んでいきます。もしVerilogを知っている、使った経験がある、という方は飛ばして頂いても構いません。

<center>
<a href="/DigitalCircuit_And_Verilog.html#ディジタル回路とverilog入門"><img src="https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/DigitalCircuit_And_Verilog.png" style="border-radius: 30px;"></a>
<a href="/DigitalCircuit_And_Verilog.html#ディジタル回路とverilog入門"> VLSI.JP/DigitalCircuit_And_Verilog.html </a>
</center>

## コンピュータアーキテクチャ入門

我々はついにCPUを作る道具を手に入れました。**コンピュータアーキテクチャ入門**ではその道具を用いて、CPUを作る方法を学んでいきます。

<center>
<a href="/Computer_Architecture.html#コンピュータアーキテクチャ入門"><img src="https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/LetsMakeCPU/Computer_Architecture.png" style="border-radius: 30px;"></a>
<a href="/Computer_Architecture.html#コンピュータアーキテクチャ入門"> VLSI.JP/Computer_Architecture.html </a>
</center>

## この次へ

次なにやる？

### RISC-V CPUを作る

正直言ってZ16 CPUとかクソの役にも立ちません。何？Z16って。無いISAを出さないで欲しい。

時代は**RISC-V**です。世の天才が作り、業界標準となりつつあるISAです。RISC-V CPUを作ったその瞬間、お前は本物のコンピュータアーキテクトとなります。この記事を読んだ奴しか知らないZ16と違ってRISC-Vは誰にでも通じます。大学に突撃して教授に「自分RISC-V CPU作ったんすよ〜」と言うだけでよしよししてくれます。コンパイラもソフトウェア資源も豊富にあり、家電や車やPCで動いている本気で実用的なCPUを作るのも夢じゃありません。RISC-V CPUを作れ。お前の未来はRISC-Vにあります。

![](https://riscv.org/wp-content/uploads/2020/06/riscv-color.svg)
[https://riscv.org/japan/](https://riscv.org/japan/)

### 半導体を作る

焼きたくないか、半導体を

[OpenMPW入門](https://vlsi.jp/OpenMPW.html)

### コンパイラを作る

自作コンパイラが吐いたバイナリが自作CPUで動いたら嬉しいですよね。

Rui Ueyama大先生が最強の本を書いてくれましたので全人類やりましょう。

[低レイヤを知りたい人のためのCコンパイラ作成入門](https://www.sigbus.info/compilerbook)

### OSを作る

いっぱい本出てるよ。自作CPUで自作OSを動かせ。

## 謝辞

校閲にご協力していただいた方々に感謝いたします。

- sksat ([@sksat_tty](https://x.com/@sksat_tty))
- いしたに ([@taichi600730](https://x.com/@taichi600730))
- IDA Kenta ([@ciniml](https://x.com/@ciniml))

## Contributers

Pull request, Issueで記事に貢献して頂いた皆様に感謝いたします。

- [@kekemoto](https://github.com/kekemoto)
- [@funera1](https://github.com/funera1)
- [@azure17581](https://github.com/azure17581)
- [@RZYN2020](https://github.com/RZYN2020)

<div align="right"> おわり </div>
---
Copyright (C) Cra2yPierr0t, ALL RIGHTS RESERVED
