---
layout: default
title:  RMII 仕様
---
# RMII 仕様

英語なんて誰も読みたくないよな。

RMIIのMAC作ろうとしたら実装できる程度に仕様を教えてくれるサイトが無かったので備忘録をここに残す。間違っている点があったらVLSI.JPの管理者まで適当に連絡してほしい。

[http://ebook.pldworld.com/_eBook/-Telecommunications,Networks-/TCPIP/RMII/rmii_rev12.pdf](http://ebook.pldworld.com/_eBook/-Telecommunications,Networks-/TCPIP/RMII/rmii_rev12.pdf)

## RMIIとは

RMIIはReduced Media Independent Interfaceの略称であり、動作周波数を二倍にする事でピン数を大体半分に減らしたMIIです。今は影も形も無いRMIIコンソーシアムという団体が独自に策定しました。

## 信号一覧

一撃で理解できる図は[仕様書](http://ebook.pldworld.com/_eBook/-Telecommunications,Networks-/TCPIP/RMII/rmii_rev12.pdf)のFigure 1.参照

| 信号名 | 方向 | 機能 |
| ----- | --- | --- |
| REF_CLK | PHY <- XTAL -> MAC <br>or PHY <- MAC | クロック |
| CRS_DV | PHY -> MAC | Carrier Sense/Receive Data Valid |
| RXD[1:0] | PHY -> MAC  | 受信データ |
| TX_EN | PHY <- MAC | 送信イネーブル |
| TXD[1:0] | PHY <- MAC | 送信データ |
| RX_ER(Optional)  | PHY -> MAC  | 受信エラー |

## 信号詳解

### REF_CLK

REF_CLKはリファレンスクロックであり、CRS_DV, RXD, TX_EN, TXD, RX_ERはこのREF_CLKを参照して動作する。PHYにとってはREF_CLKは必ず外部から入力される信号であり、信号源はMACか外部ソースの場合がある。

REF_CLKは50MHz±50ppmでありduty比は35%から65%までが許容される。

### CRS_DV

CRS_DVはCarrier Sense/Receive Data Validであり、MIIにおけるCRSとRX_DV信号を兼ねている。

大前提として、MIIは25MHz, 4bitでデータを受信する仕様になっており、RMIIでは50MHz, 2bitでデータを受信することで帯域を維持しながらピン数を削減している。MIIでは、1サイクルに来る4bitのデータの事を**ニブル**と呼ぶ。RMIIではこのニブルを2bitに分け、2サイクルかけて一つのニブルを受信する。

CRS_DVでは、RXDから先に来る２ビットではCRSとして扱い、後に来る２ビットではRX_DVとして扱う。この仕様により、キャリアは消失しているもののPHYのFIFOにデータが残っているような場合では、CRSとして扱うタイミングではCRS_DVがデアサートされ、RX_DVとして扱われるタイミングではCRS_DVがアサートされる事になり、結果的にCRS_DVの値が振動する。これはRMIIのv1.2で追加された仕様であり、物理層パケットの末尾でよく起こる。

また、CRS_DVがアサートされるまではRXD[1:0]の値は00として扱う。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/RMII/CRS_DV.png)

### RXD[1:0]

RXDは受信データであり、CRS_DVがアサートされている各クロックで2bitづつデータを受信する。

CRS_DVがデアサートされている時はRXD[1:0]の値は00になる。もし、CRS_DVがデアサートされている時にRXD[1:0]の値が00以外になる場合、MACはそれを無視して00として扱う。

一撃で理解できる図は[仕様書](http://ebook.pldworld.com/_eBook/-Telecommunications,Networks-/TCPIP/RMII/rmii_rev12.pdf)のFigure 2.参照

#### RXD[1:0] in 100 Mb/s mode

CRS_DVがアサートされた直後は、RXD[1:0]は00である必要がある。その後プリアンブル(01)が現れ、SFDの検出後データの格納が行われる。PHYが誤ったキャリアを検出した場合、RXD[1:0]は10という値が受信イベントの終了まで入力される。

一撃で理解できる図は[仕様書](http://ebook.pldworld.com/_eBook/-Telecommunications,Networks-/TCPIP/RMII/rmii_rev12.pdf)のFigure 3.参照

#### RXD[1:0] in 10 Mb/s mode

REF_CLKがデータレートの10倍になるため、RXDは10サイクル毎にサンプリングされる。

#### RXD[1:0]と受信エラー検知

RX_ERはOptionalであり、RX_ERを実装しない場合は受信エラー発生時にRXDの値を強制的に01に置き換え、CRCエラーを引き起こすことで受信エラー検知を可能にしている。本物のCRCエラーと区別するには、PHYが持つMIIレジスタの受信エラーカウンタを確認する。

### TX_EN

TX_ENは送信イネーブルであり、MACがTXDの2bitで有効なデータを出力していることを意味する。

TX_ENはREF_CLKに同期してアサートされ、プリアンブルの最初から送信データが送信されている間はアサートされ続ける。そして、フレームの最後の2bitが送信されるREF_CLKの立ち上がりエッジの前にデアサートされる。

一撃で理解できる図は[仕様書](http://ebook.pldworld.com/_eBook/-Telecommunications,Networks-/TCPIP/RMII/rmii_rev12.pdf)のFigure 4.参照

### TXD[1:0]

TXDは送信データであり、REF_CLKに応じて駆動する。

TX_ENがデアサートされている時にTXDが00の場合にはidleを意味する。TX_ENがデアサートされておりTXDが00以外の場合は予約されている。多分未定義。

RXDの送信版。

#### TXD[1:0] in 100 Mb/s mode

特に言うことなし

#### TXD[1:0] in 10 Mb/s mode

REF_CLKがデータレートの10倍なため、10サイクル毎にTXDの値がサンプリングされる。この10サイクルの間は同じ値を保つ必要がある。

### RX_ER

RX_ERは受信エラーであり、PHYはエラーが起きたREF_CLKの周期でRX_ERをアサートする。CRS_DVがデアサートされている間はRX_ERはMACに影響を与えない。

## フレームの構造

フレームの構造は以下のようになっている。

```
<inter-frame><preamble><sfd><data><efd>
```

一撃で理解できる図は[仕様書](http://ebook.pldworld.com/_eBook/-Telecommunications,Networks-/TCPIP/RMII/rmii_rev12.pdf)のFigure 5.参照
