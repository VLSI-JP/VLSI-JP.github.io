---
layout: default
title: GMII 仕様
---
# GMII 仕様

これの超下位互換です。
[https://ieeexplore.ieee.org/document/8457469](https://ieeexplore.ieee.org/document/8457469)

## Interface
GMIIの各種信号の働きを説明する。後半に連れて仕様書を読む気が無くなっていて雑になっている。

特にエラー用の信号とデータの有効性用の信号の組み合わせで便利な事が出来るらしいが、今は面倒なので書かない。LPIとか知らない。

### GTX_CLK (1000 Mb/s transmit clock)
TX_EN, TX_ER, TXD用のクロック。基本的に125MHz。TX_EN, TX_ER, TXDはPHYにGTX_CLKの立ち上がりエッジで取り込まれる。

### RX_CLK (receive clock)
RX_DV, RX_ER, RXD用のクロック。基本的に125MHz±0.01%。RX_DV, RX_ER, RXDはMAC層にRX_CLKの立ち上がりエッジで取り込まれる。

### TX_EN (transmit enable)
TXDのデータが有効である事を示す。プリアンブルの最初と同期してアサートし、フレームの最後のGTX_CLKの立ち上がりエッジでネゲートする。この信号はGTX_CLKと同期して動く。

### TXD (transmit data)
8bitの送信データ用信号。TX_ENがアサートされ、TX_ERがデアサートされている時、TXDは有効なデータとする。この信号はGTX_CLKと同期して動く。

### TX_ER (transmit coding error)
エラーが発生している事を示す。TX_ENがアサートされている時にTX_ERが1クロック以上アサートされている場合、その部分は有効なデータではない。

あとなんか色々あるけど見なかったことにしよう。

### RX_DV (receive data valid)
RXDのデータが有効である事を示す。プリアンブルの最初と同期してアサートされ、フレームの最後のRX_CLKの立ち上がりエッジでデアサートされる。正しくフレームを得るために、RX_DVはSFDよりは早くアサートされ、End-of-Frameデリミタまでにデアサートされなければならない。

### RXD (receive data)
8bitの受信データ用信号。

なんか色々ある。

### RX_ER (receive error)
エラー用の信号。TX_ERをほぼ同じ。

### CRS (carrier sense)
全二重モードの時は使わない。半二重なんて2022年に使ってる奴いる？？？？？

### COL (collision detected)
全二重モードの時は使わない。

### MDC (management data clock)
MDIO用のクロック、160ns~400nsの範囲ならよい。立ち上がりエッジでMDIOを取り込む。

### MDIO (management data input/output)
データ線。シリアル通信を行う。
通信プロトコルは後述。

## GMIIデータストリーム
GMIIのデータストリームは以下の形式になっている。
```
<inter-frame><preamble><sfd><data><efd><extend>
```

フレームのビット順を間違えると厄介な事になるのでここに記しておく。
以降で、SFDの値は`0b10101011`である、と書いてあるが、これはTXD<0>が`1`、TXD<1>が`0`、TXD<6>が`1`、TXD<7>が`1`ということである。verilog風に言うと数値リテラルは`[0:7]`になってるんですね。

### \<inter-frame\>
\<inter-frame\>はデータパスに何もデータが無い状態である。RXDではRX_DVとRX_ERの両方がデアサートされているか、RX_DVがデアサートされておりRXDが0x00の場合に\<inter-frame\>となる。TXDではTX_ENとTX_ERの療法がデアサートされている場合に\<inter-frame\>となる。

バースト転送の仕様は気が向いたら。

### \<preamble\>と\<sfd\>
プリアンブル\<preamble\>はフレーム転送の開始を示す。サイズは7byteであり、以下の値である。

> 10101010 10101010 10101010 10101010 10101010 10101010 10101010

\<sfd\>はStart Frame Delimiter(フレーム開始デリミタ)の略であり、フレームの開始を示す。これはプリアンブルの直後にあり、以下の値である。

> 10101011

プリアンブルとSFDの転送はTX_ENのアサートと同時に始めるべき。RX_DVも多分同じ。

### \<data\>
データだよ。

### \<efd\>
\<efd\>はEnd-of-Frame Delimiter(フレーム終了デリミタ)の略であり、フレームの終了を示す。TX_ENのデアサート時のTXDかRX_DVのデアサート時のRXDが\<efd\>となる。

### \<extend\>
TX_ENが0でTX_ERが1で使える何かだけど無くても使えるので割愛。

## 管理フレームストリーム
MDCとMDIOはPHYのコンフィグに用いる信号線である。MDIOは1bitでI2Cっぽい双方向なシリアル通信を行う。
MDIOがPHYを通信する際に受送信されるデータフレームはManagement frameと呼ばれている。

データのREAD、WRITEに用いるManagement Frameのフォーマットは以下の通り。


|       |   PRE   |  ST  |  OP  |  PHYAD  |  REGAD  |  TA  |   DATA    | IDLE |
| ----- | ------- | ---- | ---- | ------- | ------- | ---- | --------- | ---- |
| READ  | `1...1` | `01` | `10` | `AAAAA` | `RRRRR` | `Z0` | `D[15:0]` |  `Z` |
| WRITE | `1...1` | `01` | `01` | `AAAAA` | `RRRRR` | `10` | `D[15:0]` |  `Z` |

### IDLE
IDLE時にはMDIOをハイ・インピーダンスにする。

### PRE (Preamble)
通信を開始する際に、MDCの32クロック分MDIOを1にする。つまり`0xffff_ffff`を送る。PHYによっては省略できる。

### ST (start of frame)
フレームの開始を`01`を送ることで知らせる。

### OP (operation code)
PHYのレジスタを読み出す場合は`10`を送り、レジスタに書き込む場合は`01`を送る。

### PHYAD (Register Address)
5bitのPHYのアドレスを指定する。アドレスはMSBから送る。PHY単体が22.6にあるようなインターフェース(謎)に接続されている場合、アドレスは`00000`でよい。複数のPHYが接続されている場合は、各PHYについてアドレスを調べる必要がある。

### REGAD (Register Address)
5bitのレジスタのアドレスを指定する。PHYは32個のレジスタを持つことが出来る。アドレスはMSBから送る。`00000`はコントロールレジスタ、`00001`はステータスレジスタとIEEE 802.3-2018に定められている。

### TA (turnaround)
2bitで次から誰がデータを送るか決める。OPでREADを指定した場合、MACはTAの最初のビットをハイ・インピーダンスにし、次のクロックでPHYが0を送ってくる。WRITEを指定した場合、MACはTAの最初のビットを1にし、次のビットでMACが0を送る。

### DATA (data)
16bitのデータ。ビット15(MSBか？)から受送信する。
