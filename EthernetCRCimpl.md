---
layout: defualt
title: EthernetのCRCを実装した
image: "https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/CRC/CRC32.png"
---

# EthernetのCRCを実装した

最近EthernetのMAC部(NICのロジック部みたいなもん)を作っていてそれなりに大変だったので、大学卒業前の最後の記事としてここに残す。

## Ethernet frameの構造

LANケーブルを流れるEthernet frameは、DST MAC ADDR、SRC MAC ADDR、EtherType、Payload、FCSが並んだ構造をしている。(図1)

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/CRC/ethernet_frame.png)
<p style="text-align:center"> <b>図1. Ethernet frameの構造</b></p>

それぞれのフィールドの役割は、DST MAC ADDRが送信先MACアドレス、SRC MAC ADDRは送信元アドレス、EtheTypeはペイロードの種別またはペイロード長、FCSはFrame Check Sequenceの略で受信エラー検出用の値となっている。(表1)

| フィールド名 | サイズ | 説明 |
| --- | ---- | ---- |
| DST MAC ADDR | 6 Byte | 送信先MACアドレス |
| SRC MAC ADDR | 6 Byte | 送信元MACアドレス |
| EtherType | 2 Byte | ペイロードの種別またはペイロード長 |
| Payload | 46 ~ 1500 Byte | データ |
| FCS | 4 Byte | 受信エラー検出用の値 |

表1. Ethernet frameの各フィールドの説明

## FCS

Ethernet frameにおけるFCS (Frame Check Sequence)は受信エラー検出の役割を担っており、受信したEthernet frameからFCSを除いた部分に対してCRC32と呼ばれる計算を適用し、その計算結果とFCSの値を比較することで正しく受信できているか確認する。

このFCSはEthernet frameに32bitの値として格納され、MSBから順に送信される。(図2)

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/CRC/FCS.png)
<p style="text-align:center"> <b>図2. FCSの送信方向</b></p>

Ethernetからデータを受信するだけならFCSを検証しなくてもなんとかなるかもしれないが、データを送信する場合は適切なFCSを計算してEthernet frameに付けてやらないと、企業が真面目に作ったNICに送信したEthernet frameが弾かれてしまう。

今回は送信もしたかったのとASICにするつもりだったので、受信回路でも送信回路でも真面目にFCSを計算することにした。

## CRC

前述した通り、FCSの算出にはCRC32を使う。CRC (Cyclic Redundancy Check)は誤り検出符号の一種であり、CRCで誤り検出できる仕組みの説明はWikipediaの記事に譲る。というか筆者は数学の素養が無いので説明できない。

[https://ja.wikipedia.org/wiki/巡回冗長検査](https://ja.wikipedia.org/wiki/%E5%B7%A1%E5%9B%9E%E5%86%97%E9%95%B7%E6%A4%9C%E6%9F%BB)

CRCを算出するアルゴリズムは単純であり、入力データの先頭ビットから生成多項式をXORし、その結果の先頭の1まで生成多項式を右シフトしてから再びXORを、生成多項式が入力の左端に到達するまでの繰り返しとなっている。(図3)

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/CRC/CRC.png)
<p style="text-align:center"> <b>図3. CRCの計算方法</b></p>

図3の例では、4bit幅の生成多項式から3bit幅のCRC値を計算しており、33bitの生成多項式から32bitの生成多項式を計算するのがCRC32である。

## CRC32

FCSとCRCの概要が分かった所で、CRC32について詳しく見ていく。WikipediaではCRC32に使う生成多項式は`0x04C11DB7`とある。32bitの生成多項式を出されて若干混乱したが、記事の特化したCRCの項にあるように先頭の1bitを省略しているので、生成多項式は正しくは`0x104C11DB7`である。

生成多項式が分かったので早速実装に移りたい所だが、その前にリファレンス実装を探すとブラウザ上でCRCを計算できる[crccal.com](crccal.com)というサイトを見つけた。このサイトを覗いてみるとCRC32とひとえに言っても様々な種類がある事がわかる。(図4)

知ってましたか、CRC32にも種類があるんですよ。私は知りませんでした。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/CRC/crccal_com.png)
図4. [https://crccalc.com](https://crccalc.com)

サイトの表を見ていくと、Init, RefIn, RefOut, XorOutなどのパラメータによってCRC32の種類が決まるらしい。これらのパラメータの説明は七誌さんの資料が一番分かりやすい。

[https://www.slideshare.net/7shi/crc32](https://www.slideshare.net/7shi/crc32)

というかこの資料が無かったら私は実装できてない。ありがとう七誌さん、こうやって知識を残してくれる先達のお陰で人類社会は発展していくのだなあと思いました。(小並感)

標準的なCRC32は、CRCを計算以前・以後でいくつか処理が存在している。まずは計算の前処理として、入力データの各バイトのビット順序を反転させる。この操作はRefInに対応している。その後、入力データの先頭4バイトのみビット反転を行う。この前処理を行った後にCRCを計算する。CRC値を算出後CRC値のビット順序を反転する。この操作はRefOutに対応している。そして最後にビット反転を行う。この処理はXorOutの`9xFFFFFFFF`に対応している。そうして得られた値をCRC値とする。(図5)

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/CRC/CRCalgo.png)
<p style="text-align:center"> <b>図5. CRC32の計算順序</b></p>

図5にはInitが書かれていないが、図4のInitの`0xFFFFFFFF`が先頭4バイトの反転を意味していると思われる。

先頭4バイトのビット反転と、XorOutの操作についてはIEEE Standard for Ethernetの3.2.9に記述が存在する。(図6) 反転を行う理由も七誌さんの資料にある通り、入力データの先頭に0が続いていた場合、その0が無視されてしまう事への対策で正しいと思われる。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/CRC/Standard4Ether.png)
<p style="text-align:center"> <b>図6. IEEE 802.3にあるFCSの説明</b></p>

ここで気になるのがRefInとRefOutである。ソフトウェア屋さんからすると何故こんな処理が入るのか不思議に思うかもしれないが、Ethernetの仕様を知っているとそこまで不自然ではない。

Ethernet frameはDST MAC ADDRから順に送られていくが(図7)、各フィールドの各バイトはFCSを除いてLSBから最初に送信される。受信したデータをそのままシフトレジスタに入力していくとレジスタにはビット順序が反転された値が格納されることになる。この理由から、バイト毎にビット順序を反転した値を利用する決まりにした方が回路を実装する上では都合がいい。ただソフトウェアでCRCを実装するとなるとこの仕様は自然であるため、RefIn, RefOutで区別を付けている。

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/CRC/frame_sending.png)
<p style="text-align:center"> <b>図7. Ethernet frameの送信方向</b></p>


![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/CRC/frame_sending_real.png)
<p style="text-align:center"> <b>図8. 各バイトの送信方向</b></p>

ただRefIn, RefOutについてはIEEE Standard for Ethernetに書いてないので本当に勘弁してほしい。

## CRC32のハードウェア実装

CRC32の値を計算するにはEthernet frameの全てのデータが必要だが、そのためにわざわざSRAMを用意して受信データを全て格納してからCRCを計算するのは筋が良いとは言い難い。理想的には受信したEthernet frameのストリームを流し込んで、流し込まれたデータの部分までのCRC値を出力するようなハードウェアを実装したい。(図9)

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/CRC/CRC32_calculator.png)
<p style="text-align:center"> <b>図9. CRC32計算機</b></p>

これを作成する上で、Ethernetのデータレートが問題になった。筆者の実装ではPHYから50MHzで2-bitのデータをRMIIの仕様に則って受け取り、それを12.5MHzで8-bitのデータとして次のモジュールへ送信していた。(図10)

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/CRC/datarate.png)
<p style="text-align:center"> <b>図10. 各データレート</b></p>

つまり、CRC32の計算を行うモジュールには毎クロックに8-bitづつデータが送られる。

前述のCRCの計算方法を考えると、最初の4クロックまでは32-bitのデータがレジスタに溜まるまで待ち、レジスタに対して生成多項式をXORするだけでいいが、その後の5クロック目からは8-bitづつデータが増えていくため、１クロックで最大でも8回のCRCの計算を行わなくてはならない。

CRC32のままだと考えるのが面倒であるため、問題のスケールを変えてCRC8で毎クロック4-bitづつデータが来る場合を考える。

この場合、入力の先頭ビットが1の場合はCRC8と、0の場合は0とXORして、その計算結果を未XORの入力データの先頭ビットと結合して下の段に送る回路を４段にすれば、1クロックで4回CRCの計算を行う回路となる。(図11)

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/CRC/CRC8_arch.png)
<p style="text-align:center"> <b>図11. CRC8の計算回路</b></p>

そして入力データの4-bitと、出力の8-bitを結合して毎クロックCRC8の計算回路に送り込めば、データ入力が終わる頃にはCRC8の値が得られる事になる。(図12)

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/CRC/CRC_arch_steam.png)
<p style="text-align:center"> <b>図12. CRC8計算回路のデータフロー</b></p>

これを拡大して32-bitにすればCRC32を計算する回路となる。

実際にはCRC32の生成多項式(33-bit)とXORする時の入力データの先頭ビットは必ず1であり、その結果は必ず0であるので、33-bitの生成多項式のうち先頭を除く32-bitを保持しておき、前段の計算結果の先頭ビットを見て0またはCRC-32とXORするか選択している。(図13)

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/CRC/CRC32.png)
<p style="text-align:center"> <b>図13. CRC32の計算回路</b></p>

この回路では32個のXOR回路を通る経路がクリティカルパスとなるが、Tang Primer 20K向けに合成してみると300MHzくらいまでは余裕があるのでこのアーキテクチャで大して問題ない。

この回路の前後にInit, RefIn, RefOut, XorOutに対応した回路を付ければCRC32の計算回路は完成する。

Verilogでの実装を以下のGitHubに置いておく。

[https://github.com/Cra2yPierr0t/INTERNET_SAKEBI/blob/master/src/sakebi_crc32_calculator.v](https://github.com/Cra2yPierr0t/INTERNET_SAKEBI/blob/master/src/sakebi_crc32_calculator.v)

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/CRC/sim.png)
<p style="text-align:center"> <b>図14. Verilogで実装した回路のシミュレーション</b></p>

## まとめ

これで私は世界が滅んでもEthernetから誤ったEthernet frameを検出できるようになりました。

## 参考文献

1. 巡回冗長検査, [https://ja.wikipedia.org/wiki/巡回冗長検査](https://ja.wikipedia.org/wiki/%E5%B7%A1%E5%9B%9E%E5%86%97%E9%95%B7%E6%A4%9C%E6%9F%BB)
2. [https://crccalc.com](https://crccalc.com)
3. 七誌, CRC32, [https://www.slideshare.net/7shi/crc32](https://www.slideshare.net/7shi/crc32)
4. IEEE Standard for Ethernet, [https://ieeexplore.ieee.org/document/9844436](https://ieeexplore.ieee.org/document/9844436)
