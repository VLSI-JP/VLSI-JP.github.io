---
layout: default
title: RISC-V H拡張でHS-modeからVS-modeに遷移する
image: "https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/refs/heads/main/images/RISCV_H_extension/h_priviledge.png"
---

# RISC-V H拡張でHS-modeからVS-modeに遷移する

RISC-VのM-modeからS-modeへ遷移する方法はネット上に割とあるが、H拡張のHS-modeからVS-modeへ遷移する方法はあまり無かったので書くことにする。こんな記事を読まなくても仕様書を読めば分かるっちゃ分かるんですが、とりあえずググって方法が出てきたら嬉しいよね。

## RISC-V H拡張における特権モード

RISC-V H拡張では、従来の特権モード(M-mode, S-mode, U-mode)に対して、S-modeをHS-modeに改め、そしてVirtualizedされたモード(VS-mode, VU-mode)を追加する。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/refs/heads/main/images/RISCV_H_extension/h_priviledge.png)

## 1. M-mode to HS-mode

手始めにM-modeからHS-modeへ遷移する。これは通常通り、`mstatus.MPP`に`0x01`を書き込み、MRETをするだけで遷移できる。ただしM-modeでPMPを設定しとかないと`instruction access fault`で死ぬ(一敗)。

```c
#define HS_MODE 0x01
#define MSTATUS_MPP 11

void init_pmp() {
  //S-mode can access 0x00000000 ~ 0xffffffff range
  asm volatile("li    t0      , 0x0000001f");
  asm volatile("csrw  pmpcfg0 , t0");
  asm volatile("li    t0      , 0xffffffff");
  asm volatile("csrw  pmpaddr0, t0");
  return;
}

void enter_hsmode() {
  uint64_t mstatus;
  asm volatile("csrr  %0      , mstatus" : "=r" (mstatus));
  mstatus = mstatus | (HS_MODE << MSTATUS_MPP);
  asm volatile("csrw  mstatus , %0" : : "r" (mstatus));
  asm volatile("la    t0      , _now_hsmode");
  asm volatile("csrw  mepc    , t0");
  asm volatile("mret");
  asm volatile("_now_hsmode:"); // here is in HS-mode
  return;
}
```

## 2. HS-mode to VS-mode

RISC-Vの仕様によると、どこにあるかもわからない`V`っちゅうパラメータが1だとVirtualizedなモードらしい。これは`hstatus.SPV`で操作できる。

### HS-modeのCSR

HS-modeでは、S-modeのCSRである `sstatus`, `sepc`等に加え、`hstatus`, `htvec`等の`h~`で始まるCSRが追加されている。前述したとおり、HS-modeからVS-modeに遷移するには、`hstatus`を操作する。

#### `hstatus`の各フィールド

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/refs/heads/main/images/RISCV_H_extension/hstatus.png)

- `hstatus.VTSR` , `hstatus.VTW` , `hstatus.VTVM` : `mstatus.TSR` , `mstatus.TW` , `mstatus.TVM` と同様。だたしVS-modeにしか影響を与えない
- `hstatus.VGEIN` : よくわかんないけどあんま重要じゃない
- `hstatus.HU` : U-mode(VU-mode?)でHLV, HLVX, HSV命令が使えるか設定
- `hstatus.SPVP` : VS-modeかVU-modeから帰ってきたかを判別する？1ならVSで0ならVUかな？
- `hstatus.SPV` : V専用の `mstatus.MPP` みたいなやつ。これを1に設定してSRETするとV = 1になる
- `hstatus.GVA` : `stval` にGuest Virtual Addressが書き込まれるような例外の場合は1が書き込まれ、それ以外なら0が書き込まれる。
- `hstatus.VSBE` : VS-modeによるメモリアクセスのエンディアンを制御する。0ならリトルエンディアン、1ならビッグエンディアン。これいる？

### VS-modeへの遷移手順

手順をまとめると以下の通り。`sstatus.SPP`でSRET後の特権モードをS-modeに設定し、`hstatus.SPV`でSRET後のVを1に設定し、`sepc`でSRET後のプログラムカウンタを設定している。

1. `sstatus.SPP` に1を書き込む
2. `hstatus.SPV` に1を書き込む
3. `sepc` にアドレスを書き込む
4. SRET

```c
#define V_MODE 0x1
#define VS_MODE 0x01
#define HSTATUS_SPV 7
#define SSTATUS_SPP 8

void enter_vsmode() {
  uint64_t hstatus, sstatus;
  asm volatile("csrr  %0      , hstatus" : "=r" (hstatus));
  asm volatile("csrr  %0      , sstatus" : "=r" (sstatus));
  hstatus = hstatus | (V_MODE << HSTATUS_SPV);
  sstatus = sstatus | (VS_MODE << SSTATUS_SPP);
  asm volatile("csrw  hstatus , %0" : : "r" (hstatus));
  asm volatile("csrw  sstatus , %0" : : "r" (sstatus));
  asm volatile("la    t0      , _now_vsmode");
  asm volatile("csrw  sepc    , t0");
  asm volatile("sret");
  asm volatile("_now_vsmode:"); // here is in VS-mode
  return;
}
```

M-mode -> HS-mode -> VS-mode と遷移するコード全体は以下の通り。

```c
typedef unsigned long uint64_t;

#define M_MODE 0x11
#define HS_MODE 0x01
#define VS_MODE 0x01
#define VU_MODE 0x00

#define V_MODE 0x1

#define MSTATUS_MPP 11
#define HSTATUS_SPV 7
#define SSTATUS_SPP 8

void init_pmp() {
  //S-mode can access 0x00000000 ~ 0xffffffff range
  asm volatile("li    t0      , 0x0000001f");
  asm volatile("csrw  pmpcfg0 , t0");
  asm volatile("li    t0      , 0xffffffff");
  asm volatile("csrw  pmpaddr0, t0");
  return;
}

void enter_hsmode() {
  uint64_t mstatus;
  asm volatile("csrr  %0      , mstatus" : "=r" (mstatus));
  mstatus = mstatus | (HS_MODE << MSTATUS_MPP);
  asm volatile("csrw  mstatus , %0" : : "r" (mstatus));
  asm volatile("la    t0      , _now_hsmode");
  asm volatile("csrw  mepc    , t0");
  asm volatile("mret");
  asm volatile("_now_hsmode:"); // here is in HS-mode
  return;
}

void enter_vsmode() {
  uint64_t hstatus, sstatus;
  asm volatile("csrr  %0      , hstatus" : "=r" (hstatus));
  asm volatile("csrr  %0      , sstatus" : "=r" (sstatus));
  hstatus = hstatus | (V_MODE << HSTATUS_SPV);
  sstatus = sstatus | (VS_MODE << SSTATUS_SPP);
  asm volatile("csrw  hstatus , %0" : : "r" (hstatus));
  asm volatile("csrw  sstatus , %0" : : "r" (sstatus));
  asm volatile("la    t0      , _now_vsmode");
  asm volatile("csrw  sepc    , t0");
  asm volatile("sret");
  asm volatile("_now_vsmode:"); // here is in VS-mode
  return;
}

int main() {
  init_pmp();

  enter_hsmode();

  enter_vsmode();

  return 0;
}
```
