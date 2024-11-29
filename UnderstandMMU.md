---
layout: default
title: RISC-V Sv32,Sv39を理解する
---

# RISC-V Sv32,Sv39を理解する

MMUなんもわからん。本記事ではRISC-VにおけるMMUの仕様であるSv32, Sv39について、なんもわかりませんが、わからないなりに解説します。

## アドレス変換とは

なんか書く

## Sv32とSv39の違い

なんか書く

## Sv32のアドレス変換

Sv32ではS-mode, U-modeでのデータアクセス・命令フェッチにおいて、メモリアクセス前に 仮想アドレス -> 物理アドレス の変換を行いますが、

### Sv32の仮想アドレス

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/refs/heads/main/images/UnderstandMMU/Sv32_va.png)

### Sv32のsatpレジスタ

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/refs/heads/main/images/UnderstandMMU/Sv32_satp.png)

### Sv32のPTE

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/refs/heads/main/images/UnderstandMMU/Sv32_pte.png)

### Sv32のアドレス変換手順

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/refs/heads/main/images/UnderstandMMU/Sv32_Translation.png)

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/refs/heads/main/images/UnderstandMMU/Sv32_Megapage.png)

## Sv39のアドレス変換

### Sv39の仮想アドレス

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/refs/heads/main/images/UnderstandMMU/Sv39_vapa.png)

### Sv39のsatpレジスタ

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/refs/heads/main/images/UnderstandMMU/Sv39_satp.png)

### Sv39のPTE

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/refs/heads/main/images/UnderstandMMU/Sv39_pte.png)

### Sv39のアドレス変換手順

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/refs/heads/main/images/UnderstandMMU/Sv39_Translation.png)

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/refs/heads/main/images/UnderstandMMU/Sv39_Megapage.png)

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/refs/heads/main/images/UnderstandMMU/Sv39_Gigapage.png)

### Sv39を試してみる

## 参考

- デイビッド, パターソン & アンドリュー, ウォーターマン著. *"RISC-V原典: オープンアーキテクチャのススメ(成田 光彰訳)"*. 日経BP(2018)
- RISC-V Foundation. *"The RISC-V Instruction Set Manual: Volume II."* [https://drive.google.com/file/d/17GeetSnT5wW3xNuAHI95-SI1gPGd5sJ_/view](https://drive.google.com/file/d/17GeetSnT5wW3xNuAHI95-SI1gPGd5sJ_/view). (2024/05/24閲覧)
