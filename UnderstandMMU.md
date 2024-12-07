---
layout: default
title: RISC-V Sv32,Sv39を理解する
image: "https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/refs/heads/main/images/UnderstandMMU/Sv39_Translation.png"
---

# RISC-V Sv32,Sv39を理解する

MMUなんもわからん。本記事ではRISC-VにおけるMMUの仕様であるSv32, Sv39について、なんもわかりませんが、わからないなりに解説します。

## Sv32のアドレス変換

Sv32ではS-mode, U-modeでのデータアクセス・命令フェッチにおいて、メモリアクセス前に **仮想アドレス(32 bit) -> 物理アドレス(34 bit)** の変換を行う。またその際に、許可されていない領域にアクセスしようとした場合や、許可されていない操作を行おうとした場合は**ページフォルト**を発生させる。

このアドレス変換は**ページテーブル**と呼ばれる表を引いて行う。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/refs/heads/main/images/UnderstandMMU/AddressTranslation.png)

Sv32において、アドレス変換はページテーブルを２回引くことで物理アドレスを得る２段ページテーブルとなっている。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/refs/heads/main/images/UnderstandMMU/AddressTranslation_2L.png)

便宜上、一段目のページテーブルをL2 Page Table、二段目のページテーブルをL1 Page Tableと表記する。


### Sv32の仮想アドレス

Sv32では、32 bitの仮想アドレスに対して31 bit ~ 22 bitを`VPN[1]`, 21 bit ~ 12 bitを`VPN[0]`, 11 bit ~ 0 bitを`page offset`という名前を付けている。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/refs/heads/main/images/UnderstandMMU/Sv32_va.png)

## Sv32のページとメガページ

Sv32のページサイズは4KiB、メガページ(Megapage)のサイズは2MiB

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

Sv32のアドレス変換の手順は以下の通り。なお原文はRISC-V Specification Volume2の10.3.2. Virtual Address Translation Processにある。



![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/refs/heads/main/images/UnderstandMMU/Sv32_Translation.png)

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/refs/heads/main/images/UnderstandMMU/Sv32_Megapage.png)

## Sv39のアドレス変換

### Sv39の仮想アドレス

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/refs/heads/main/images/UnderstandMMU/Sv39_vapa.png)

### Sv39のページとメガページとギガページ

Sv39のページサイズは4KiB、メガページ(Megapage)のサイズは2MiB、ギガページ(Gigapage)のサイズは1GiB。

### Sv39のsatpレジスタ

Sv39のsatpレジスタはSv32の各フィールドを拡張した形になっている。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/refs/heads/main/images/UnderstandMMU/Sv39_satp.png)

`satp.MODE`は現在のアドレス変換モードを示しており、`0`の場合はアドレス変換を行わないBare、`8`の場合はSv39を意味している。

### Sv39のPTE

Sv39においてPTEのサイズはSv39では64 bitとなっている。PTEのフィールドの内訳は以下の図の通り。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/refs/heads/main/images/UnderstandMMU/Sv39_pte.png)

各フィールドの働きはSv32と同じ。

### Sv39のアドレス変換手順

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/refs/heads/main/images/UnderstandMMU/Sv39_Translation.png)

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/refs/heads/main/images/UnderstandMMU/Sv39_Megapage.png)

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/refs/heads/main/images/UnderstandMMU/Sv39_Gigapage.png)

### Sv39を試してみる

```c
typedef unsigned long uint64_t;

#define M_MODE 0x11
#define S_MODE 0x01
#define U_MODE 0x00

#define PTE_V 0x1
#define PTE_R 0x2
#define PTE_W 0x4
#define PTE_X 0x8
#define PTE_A 0x40
#define PTE_D 0x80

#define MMU_EN 0x8000000000000000

#define TEXT_ADDR 0x80000000
#define L3_PAGETABLE 0x80010000 // under .text
#define L2_PAGETABLE 0x80020000
#define L1_PAGETABLE 0x80030000
#define L2_PAGETABLE_D 0x80080000
#define L1_PAGETABLE_D 0x80090000

void init_pmp() {
  //S-mode can access 0x00000000 ~ 0xffffffff range
  asm volatile("li    t0      , 0x0000001f");
  asm volatile("csrw  pmpcfg0 , t0");
  asm volatile("li    t0      , 0xffffffff");
  asm volatile("csrw  pmpaddr0, t0");
  return;
}

void enter_smode() {
  uint64_t mstatus;
  asm volatile("csrr  %0      , mstatus" : "=r" (mstatus));
  mstatus = mstatus | (S_MODE << 11);
  asm volatile("csrw  mstatus , %0" : : "r" (mstatus));
  asm volatile("la    t0      , _now_smode");
  asm volatile("csrw  mepc    , t0");
  asm volatile("mret");
  asm volatile("_now_smode:"); // here is in S-mode
  return;
}

void init_mmu() {
  uint64_t satp = 0;
  uint64_t *pte;
  // L3 Page Table Head is 0x80010000
  satp = (L3_PAGETABLE >> 12) & 0x00000fffffffffff;

  // direct mapping for instruction fetch
  // 0x80000000 -> 0x80000000
  // 0x80001000 -> 0x80000000
  // ...
  // 0x80040000 -> 0x80040000 0x41 Pages 260 KiB

  // VPN[2],0b000 = (0x80000000 >> 30) << 3
  pte = L3_PAGETABLE | (0x80000000 >> 30) << 3;
  *pte = ((L2_PAGETABLE >> 12) << 10) | PTE_V;

  // VPN[1],0b000 = ((0x80000000 & 0x3fe00000) >> 21) << 3
  pte = L2_PAGETABLE | ((0x80000000 & 0x3fe00000) >> 21) << 3;
  *pte = ((L1_PAGETABLE >> 12) << 10) | PTE_V;

  // VPN[0],0b000 = ((0x80000000 & 0x1ff000) >> 12) << 3
  for(int i = 0; i < 0x41; i++) {
    pte = L1_PAGETABLE | (((0x80000000 + (i * 0x1000)) & 0x1ff000) >> 12) << 3;
    *pte = (((0x80000000 + (i * 0x1000)) >> 12) << 10) | PTE_V | PTE_R | PTE_W | PTE_X | PTE_A | PTE_D;
  }

  // mapping for data access
  // VA : 0x00000000 -> PA: 0x80003000
  // VA : 0x00001000 -> PA: 0x80004000
  // VA : 0x00002000 -> PA: 0x80005000
  // VA : 0x00003000 -> PA: 0x80006000 0x4 Pages 16 KiB

  // VPN[2],0b000 = (0x00000000 >> 30) << 3
  pte = L3_PAGETABLE | (0x00000000 >> 30) << 3;
  *pte = ((L2_PAGETABLE_D >> 12) << 10) | PTE_V;

  // VPN[1],0b000 = ((0x00000000 & 0x3fe00000) >> 21) << 3
  pte = L2_PAGETABLE_D | ((0x00000000 & 0x3fe00000) >> 21) << 3;
  *pte = ((L1_PAGETABLE_D >> 12) << 10) | PTE_V;

  // VPN[0],0b000 = ((0x00000000 & 0x1ff000) >> 12) << 3
  for(int i = 0; i < 0x4; i++) {
    pte = L1_PAGETABLE_D | (((0x00000000 + (i * 0x1000)) & 0x1ff000) >> 12) << 3;
    *pte = (((0x80003000 + (i * 0x1000)) >> 12) << 10) | PTE_V | PTE_R | PTE_W | PTE_X | PTE_A | PTE_D;
  }

  // activate MMU
  satp |= MMU_EN;
  asm volatile("csrw satp, %0" : : "r" (satp));
  asm volatile("sfence.vma zero, zero");
}

int main() {
  init_pmp();

  enter_smode();

  init_mmu();

  int a = 0x114514;
  uint64_t *mem;
  mem = (uint64_t *)0x00000040;
  *mem = a;

  return 0;
}
```

## 参考

- デイビッド, パターソン & アンドリュー, ウォーターマン著. *"RISC-V原典: オープンアーキテクチャのススメ(成田 光彰訳)"*. 日経BP(2018)
- RISC-V Foundation. *"The RISC-V Instruction Set Manual: Volume II."* [https://drive.google.com/file/d/17GeetSnT5wW3xNuAHI95-SI1gPGd5sJ_/view](https://drive.google.com/file/d/17GeetSnT5wW3xNuAHI95-SI1gPGd5sJ_/view). (2024/05/24閲覧)
