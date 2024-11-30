---
layout: default
title: RISC-V Sv32,Sv39を理解する
---

# RISC-V Sv32,Sv39を理解する

MMUなんもわからん。本記事ではRISC-VにおけるMMUの仕様であるSv32, Sv39について、なんもわかりませんが、わからないなりに解説します。

## Sv32のアドレス変換

Sv32ではS-mode, U-modeでのデータアクセス・命令フェッチにおいて、メモリアクセス前に **仮想アドレス(32 bit) -> 物理アドレス(34 bit)** の変換を行う。またその際に、許可されていない領域にアクセスしようとした場合や、許可されていない操作を行おうとした場合は**ページフォルト**を発生させる。

このアドレス変換は**ページテーブル**と呼ばれる表を引いて行う。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/refs/heads/main/images/UnderstandMMU/AddressTranslation.png)

Sv32において、アドレス変換はページテーブルを２回引くことで物理アドレスを得る２段ページテーブルとなっている。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/refs/heads/main/images/UnderstandMMU/AddressTranslation_2L.png)


### Sv32の仮想アドレス

Sv32では、32 bitの仮想アドレスに対して31 bit ~ 22 bitを`VPN[1]`, 21 bit ~ 12 bitを`VPN[0]`, 11 bit ~ 0 bitを`page offset`という名前を付けている。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/refs/heads/main/images/UnderstandMMU/Sv32_va.png)

### Sv32のsatpレジスタ

satpレジスタはSupervisor address translation and Protectionの略で、現在のアドレス変換モードを示す値(`satp.MODE`)と、アドレス空間の識別子(`satp.ASID`)と、アドレス変換の最初の手掛かりとなるアドレス(`satp.PPN`)を保持している。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/refs/heads/main/images/UnderstandMMU/Sv32_satp.png)

`satp.MODE`は現在のアドレス変換モードを示しており、`0`の場合はアドレス変換を行わないBare、`1`の場合はSv32を意味している。

satpのフィールドで最も重要なものが`satp.PPN`であり、これは１段目のページテーブルのアドレスを指す。

### Sv32のPTE

ページテーブルは**ページテーブルエントリ(Page Table Entry, PTE)**の集合であり、そのサイズはSv32では32 bitとなっている。PTEのフィールドの内訳は以下の図の通り。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/refs/heads/main/images/UnderstandMMU/Sv32_pte.png)

PTEのフィールドの内、最も重要なものが`PTE.V`であり、そのPTEが有効であることを示すValidビットを意味する。`PTE.R`, `PTE.W`, `PTE.X`はパーミッションである。


RWXの全てが0の場合、`PTE.PPN`は次の段のページテーブルの先頭アドレスを意味する。RWXの少なくとも１つが1の場合、`PTE.PPN`は物理アドレスを意味する。


`PTE.U`はU-modeはアクセス可能かを示す。U-modeによるメモリアクセスは、このビットが1になってるページにしかアクセスできない。
`PTE.G`はグローバルマッピングを意味する。多分グローバル変数的ななんかだけどよくわかんないです。`PTE.A`と`PTE.D`もよくわかんないです。

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
