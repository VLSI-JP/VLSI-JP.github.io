---
layout: default
title: 現代コンピュータアーキテクチャとPRF方式まとめ
---

# 現代コンピュータアーキテクチャとPRF方式まとめ

本記事は作成途中というか執筆メモ及び雑翻訳です。Googleの検索エンジンさんが掘り当てない限りはこのページは見えないはず。というかとりあえず自分向けに翻訳して頭が冴えてる時の自分がいい感じに記事にします。

後輩及び友達と Reservation Station についての話が噛み合わなくて尋問したら現代コンピューターアーキテクチャの新常識を教えてもらった。いたく感動したのでここにまとめることにする。

なお話をまとめるにあたり、Australian National UniversityのShoaib Akram先生の講義資料を大いに参考にした。英語に抵抗が無いならこちらを読んだ方が早い。

[https://comp.anu.edu.au/courses/comp3710-uarch/assets/lectures/week7.pdf](https://comp.anu.edu.au/courses/comp3710-uarch/assets/lectures/week7.pdf)

## ARF+ROB方式

![](https://raw.githubusercontent.com/VLSI-JP/VLSI-JP.github.io/main/images/PRF/ARFROB.png)
<p style="text-align:center"> <b>図1. 概ね正しいARF+ROB方式のデータフロー</b></p>

## ARF+ROB方式の欠点

- Issueステージ前の Register Read
    - Register Read が Issueステージの前に存在しており、これを Issueステージの前にすることは出来ない。
    - リネーミング時にデータが利用可能な場合、
    - Issue queue は全てのオペランドが利用可能になるまで、オペランドを保持しなくてはならない。
    - もし Issue Queue が 値へのポインタだけを持っていれば、ROBからARFへ命令がIssueされる前にデータを動かされ、ポインタが古くなりうる。

- レジスタの値のコミットはデータの移動を要する
    - ROB から ARF へのデータ移動は余計なサイクルと電力を要する

## Physical Register File方式

### ARF+ROBとの比較

- モノリシックなPhysical Regsiter Fileがリネーミングのためのレジスタの拡張セットを提供する
- レジスタのサブセットがアーキテクチャ的な状態を表す
- Rename Map Table がアーキテクチャと物理レジスタのマッピングを提供する
- 利点：Commiting と Freeing にデータの移動を必要としない
- 欠点：RMP の復元はビットのクリアのようにシンプルではない(コンセプトは同じ)
- Issueステージの後に Register Readステージが存在する
- Issue Queueは値を保持しない。タグのみを保持する

### 論理レジスタと物理レジスタ

- 論理レジスタ
    - プログラムレジスタとも呼ばれる
    - MIPS : 32
    - Alpha : 32
    - 8080 : 8
    - x86-64(w/o AVX) : 16

- 物理レジスタ
    - ハードウェアレジスタとも呼ばれる
    - Intel Sandybridge : 160
    - Apple A14 : 300+


### リネーミング

- リネーミングの課題
    - ユニークな物理レジスタを論理レジスタの割り当て
    - 複数の論理レジスタの定義をユニークな物理レジスタに割り当て

- リネーミングの構造
    - Physical Register File(PRF)
    - Rename Map Table(RMP)
    - Free list(PRFの利用されていない物理レジスタのリスト)

### リネーミングの例

```asm
load r1, 16(r2)
add r3, r1, #1
load r1, 20(r2)
sub r4, r1, #1
```

- 論理 ソース レジスタ
    - マッピングを RMP から得る
- 論理 ディスティネーション レジスタ
    - フリーリストから物理レジスタアドレスを得る
    - 物理レジスタを論理レジスタに割り当てる
    - マッピングを反映するために RMP をアップデートする

```asm
load r1, 16(r2)
add r3, r1, #1
load r1, 20(r2)
sub r4, r1, #1
↓
load p67, 16(p11)
add p19, p67, #1
load p27, 20(p11)
sub p28, p27, #1
```

- いつ・どうレジスタを解放するか
- 分岐予測ミスと例外からどう回復するか
- アーキテクチャ的な状態をどう管理するか
    - アーキテクチャ的な状態とPRFの一部との対応

### アクティブリスト

- アクティブリスト
    - プログラム順にアクティブな命令を保持する
    - アクティブな命令：ディスパッチされたがまだリタイアされていない命令
    - RoBと同等
    - Active Window、Instruction Windowとも呼ばれる

### アクティブリストへの操作

- 命令ディスパッチ
    - Tailのエントリを予約する
    - エントリの初期化
        - 命令の現在のマッピングとPCを書く
        - Completed, misprediction, exceptionフラグをクリア
    - Tailポインタをインクリメント
- 命令リタイア
    - 命令リタイアは論理 ディスティネーション レジスタにより保持されていた物理レジスタを解放する
    - 命令リタイアは命令の論理 ディスティネーション レジスタの新しい値(物理レジスタ指定子(謎))をコミットする
        - コミット済みマッピングを保持する新たな構造物が必要
        - Architectural Map Table(AMT)
        - Intel は Retirement Register Alias Table(RRAT)と呼んでる

### Architectural Map Table(AMT)

- Architectural Map Table(AMT)
    - 論理レジスタから物理レジスタへのコミット済みマッピングを保持する
    - 注：AMTに対して、RMPは論理から物理への投機的なマッピングを保持している
    - RMTは乱雑で、汚れており、予測ミスで破棄される
    - AMTはクリーンであり、例外的な結果で保存されている必要がある(謎)
    - AMT/RRAT は予測ミスや例外からの回復を可能とする

### AMT は本当に必要か？

- RMT及びAMTを持つことはコストが掛かる
- AMTのないPRF方式が実はあるっぽい

### アクティブリストへの操作(with AMT)

- リタイア
    - 命令がヘッドに到達し、完了するまで待機
    - ヘッドの命令の論理 ディスティネーション レジスタの指定子を使ってAMTをインデックス
        - AMTが格納している前のマッピングを解放
        - 前のマッピングをフリーリストに突っ込む
        - 新バージョンを暗黙にコミット、古いバージョンはもはや潜在的に復元する必要がない
        - 現在のマッピングが示す物理レジスタはAMTを現在のマッピングにリライトすることでコミットされる
    - 若い命令をアクティブリストにコミットするためにヘッドポインタをインクリメント(deplete the instruction windowって書いてるけど意図がよくわかんないっす)

### 例外と予測ミス時の対応

- なんかこれムズいぞ
- ヤバすぎ
