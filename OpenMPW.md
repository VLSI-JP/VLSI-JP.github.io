---
layout: default
title: OpenMPW入門 改訂版
---
# OpenMPW入門 改訂版
質問、修正案、その他連絡は@Cra2yPierr0tマデ

久々に以前書いた入門記事を読んだら日本語は雑だし内容は古いしこの世の終わりみたいな出来だったので90nmに備えて書きます。

**オレオレLSIを焼きたいですよね？ 焼きましょう。** Skywater社がPDKを公開し、OSSなGDSIIコンパイラである**OpenLANE**も生まれました。そしてGoogleの出資によってEfablessが無料でLSIを作らせてくれるプログラム、**Open MPW Shuttle Program**をスタートしました。 今こそオレオレLSIを焼くチャンスです。あなたの作りたいチップをGoogleの金で作りましょう！


## OpenMPWとは
**OpenMPW**(Open Multi Project Wafer)は**Efabless**にGoogleが出資して生まれたシャトルプログラムであり、ホームページには次の文言が書かれています。

---

> The shuttle provides opportunities for designers to experiment and push the state-of-the-art without having to reconcile the risk associated with the cost of fabrication.
> The shuttle program is open to anyone, provided that their project is fully open source and meets the other program requirements.
> Costs for fabrication, packaging, evaluation boards and shipping are covered by Google for this program.

出典：[https://efabless.com/open_shuttle_program](https://efabless.com/open_shuttle_program])

> このシャトルはデザイナーに製造コストに纏わるリスクを負うことなく、実験し、最先端を追求する機会を提供します。
> シャトルプログラムは、プロジェクトが完全にオープンソースであり、一定の要件を満たしていれば、誰でも参加することができます。
> 製造、パッケージング、評価ボード、そして送料は全てGoogleが負担します。


---

つまりデザインをオープンソースにすれば**完全無料**で自分の半導体を作れるプログラムという事ですね。これは熱い、参加するしかない、Google最高一生ついていきます。

と言っても提出された全てのデザインを焼いてくれるという訳ではなく、いくつかの条件が存在しています。

* 提出された中からランダムに40のプロジェクトが選ばれる
* デザインをオープンソースにすること
* レイアウトをリポジトリから再現可能にすること
* ライセンス諸々
* Caravel(後述)の使用
* プリチェックツールをパスしていること

今すぐにでも自分のHDLをLSIにしたい気持ちは分かりますが、必要な事がちょっと多いので少しずつやっていきましょう。

この次は各ツールの概要に入りますが、既に知っておりインストール済みなら飛ばして頂いて構いません。

## 宣伝

高位合成ツール、RgGenを使ったWalkthorughが生えました！

[RgGen ✕ OpenMPWでLSIを焼こう！](https://vlsi.jp/RgGenxOpenMPW.html)


## 環境構築
まずはOpenMPWの開発を始めるための最低限の環境を構築しましょう。Linux/M1 Mac/Intel Mac/Winに対応しています。

最小要件はこちら

* GNU Make
* Python 3.6+ with pip and virtualenv
* Git 2.22+
* Docker 19.03.12+

以下のコマンドはarchlinuxを対象としたのもですが、pipとvirtualenvとdockerがインストールさえ出来ればどんなOSでも構いません。

pipとvirtualenvのインストール
```bash
sudo pacman -S python-virtualenv python-pip
```
dockerのインストール
```bash
sudo pacman -S docker
```

自分をdockerグループに追加しましょう。これはセキュリティ上大変よろしくありませんが、これを回避するためにpodmanを使ったり色々試したが結局無理だったので諦めました。回避出来るなら教えてほしい。
```bash
sudo usermod -a -G docker $(whoami)
```
一旦再起動する。


Dockerデーモンの起動
```bash
systemctl start docker
```

## OpenLANEの概要
OpenLANEとはOSSなRTL-to-GDSIIコンパイラであり、20個くらいのOSSを組み合わせて作られています。このOpenLANEにPDKとRTLとコンフィグを揃えて実行するとGDSIIが生成されます。凄いですね。

このGDSIIをファブに送りつけるとLSIを焼いてくれます。

OpenLANEはDockerコンテナが提供されており、Dockerでインストールするのが推奨されています。一応Dockerコンテナを使わずにインストールすることも可能ですが、It is more complexとか書いてあるので近づかない方が良いです。

### OpenLANEのインストール
次にOpenLANEをインストールしますが、後述するCaravel経由でインストールした方がなにかと都合が良いので先にcaravelをインストールします。

最初にcaravelをダウンロード。`<tag>`は次のページから最新のものを指定してください。執筆時点ではSkywaterの130nmを使うシャトルの場合は`mpw-8c`でGlobalFoundriesの180nmを使う場合は`gfmpw-0d`です。[https://github.com/efabless/caravel_user_project/tags](https://github.com/efabless/caravel_user_project/tags)
```bash
git clone -b <tag> https://github.com/efabless/caravel_user_project.git
```

そしてdependenciesディレクトリを作成、OpenLANEとPDKをインストールするディレクトリを作成し、環境変数を設定します。
```bash
cd caravel_user_project
mkdir dependencies
export OPENLANE_ROOT=$(pwd)/dependencies/openlane_src
export PDK_ROOT=$(pwd)/dependencies/pdks
```

次にインストールするPDKの種類を設定する。PDKの中身はOR回路とかAND回路とかFFとかの設計図とか物性情報です。

`<pdk>`にはインストールしたいPDKの種類を指定します。PDKの種類には`sky130A`, `sky130B`, `gf180mcuC`などがあり、シャトルで使うプロセスに応じて選択してください。
```bash
export PDK=<pdk>
```

`make setup`でインストールを開始する。
```bash
make setup
```

環境変数の`OPENLANE_ROOT`と`PDK_ROOT`と`PDK`は重要であり、これが設定されていなければOpenLANEもCaravelも動きませんので、必ず設定するようにしてください。
### OpenLANEの動作確認

OpenLANE以下に入り`make test`を実行してOpenLANEの動作確認をすることが出来ます。やってみましょう。
```bash
cd $OPENLANE_ROOT
make test
```

Basic test passedと出れば問題なくインストール出来ています。

![](https://i.imgur.com/mhYH6Wb.png)

既に用意されている`design/`以下のデザインをビルドしたい場合は、`make mount`してから`./flow.tcl -design ディレクトリ名`でビルド出来ます。

以下では試しにAPUをビルドしています。
```bash
cd $OPENLANE_ROOT
make mount
./flow.tcl -design APU
```

OpenLANEを直接使ってデザインをビルドした際に生成されたGDSIIは
`OpenLANE/designs/APU/runs/RUN_<date>/results/final/gds`以下に存在しており、klayoutを用いてレイアウトを見られます、見てみましょう。テンションが上がりますね。
```bash
klayout APU.gds
```
![](https://i.imgur.com/XJ0v0dL.png)

この画像は回路が生成されている感を感じるためにデキャップセルを非表示にしており、実際に見られるものとは少し異なります。

### OpenLANEの設定
OpenLANEを使う上で、HDLを書くのと同時に、そのデザインに応じてOpenLANE用の設定ファイルを書く必要があります。

試しにデフォルトで用意されているデザインの設定ファイルを見てみましょう。デフォルトのデザインは`OpenLANE/designs/`以下に存在しており、usbやpicorv32a等色々ありますがそれらのディレクトリ内の**config.tcl**がOpenLANEの設定ファイルとなっています。

設定ファイルはtclファイルの**config.tcl**かjsonファイルの**config.json**で記述可能です。

設定ファイルの基本的な書き方の説明はここにあります、後で解説します。
[https://openlane.readthedocs.io/en/latest/usage/hardening_macros.html](https://openlane.readthedocs.io/en/latest/usage/hardening_macros.html)

また利用可能な変数の一覧はここにあり、その中で必須な変数を後で解説します。
[https://openlane.readthedocs.io/en/latest/reference/configuration.html](https://openlane.readthedocs.io/en/latest/reference/configuration.html)

## Caravelの概要
今までさんざん出てきたCaravelですが、これはOpenMPWの半導体設計用テンプレートです。
このCaravel一つでデザインのビルド、プリチェック、テストベンチをすることが可能となっています。凄いですね。

CaravelはManagement AreaとUser Project Areaに分かれています。Management Areaは変更不可能な領域であり、軽量なRISC-Vコアと各種IOからなります。またUser Project Areaは自由に変更できる領域であり、開発者はここに自分のデザインを挿入します。

Management Areaのサイズは`2.920um x 3.520um`であり、大体`10mm^2`です。

![caravelの概要](https://i.imgur.com/CQLoSjf.png)

User Project Areaから外部には38本のGPIO線が伸びており、ここから直接自分のデザインにアクセス出来ます。またManagement Areaとは32bitのwishboneインターフェースと128bitのLogic Analyzer線で繋がっており、RISC-Vコア上で走っているプログラムがMMIOでアクセスを行えます。

### Caravelのインストール
ここの作業は、上記のOpenLANEのインストールで既にCaravelのインストールを完了している場合はやる必要はありません。**むしろやらない方が良い。**

ただOpenLANEもpdkもCaravelも全部消してもう一度OpenMPWで必要なOSSをインストールしたいならここを見るのが手っ取り早いです。

caravelをダウンロード。`<tag>`は次のページから最新のものを指定してください。執筆時点ではSkywaterの130nmを使うシャトルの場合は`mpw-8c`でGlobalFoundriesの180nmを使う場合は`gfmpw-0d`です。[https://github.com/efabless/caravel_user_project/tags](https://github.com/efabless/caravel_user_project/tags)
```bash
git clone -b <tag> https://github.com/efabless/caravel_user_project.git
```

dependenciesディレクトリを作成、OpenLANEとPDKをインストールするディレクトリを作成し、環境変数を設定する。
```bash
cd caravel_user_project
mkdir dependencies
export OPENLANE_ROOT=$(pwd)/dependencies/openlane_src
export PDK_ROOT=$(pwd)/dependencies/pdks
```

次にインストールするPDKの種類を設定する。PDKにはOR回路とかAND回路とかFFとかの設計図が入っている。

`<pdk>`にはインストールしたいPDKの種類を指定する。PDKの種類には`sky130A`, `sky130B`, `gf180mcuC`などがありますので、シャトルで使うプロセスに応じて選択してください。
```bash
export PDK=<pdk>
```

インストールを開始。
```bash
make setup
```

### Caravelの動作確認
Caravelが正しくインストールされている事を確認するために、既に用意されているデザインをビルドしてみましょう。`caravel_user_project`のルートに入り、以下のコマンドでビルドを開始します。
```bash
make user_proj_example
```

GDSIIファイルは`gds/`以下に生成されます。klayoutで見てみましょう。

### Caravelのファイル構造
Caravelには多くのディレクトリがありますが、基本的に以下の4つのディレクトリの使い途を知っていれば十分です。

* `gds/`：ここにGDSIIファイルが生成される
* `openlane/`：OpenLANEの設定ファイルをここに置く
* `verilog/dv/`：シミュレーションとテストベンチ用ファイルをここに置く
* `verilog/rtl/`：自分のデザインのverilogファイルをここに置く

Caravelの全てのディレクトリの説明はここにあります。気になりましたら見てみてください。

[https://caravel-harness.readthedocs.io/en/latest/getting-started.html#required-directory-structure](https://caravel-harness.readthedocs.io/en/latest/getting-started.html#required-directory-structure)

### Caravelにおけるデザインのビルド
Caravelにおいて、デザインのビルドは二段階に分かれています。第一段階では自分のVerilogのGDSIIへのビルドであり、第二段階がそのGDSIIを内包したラッパーのビルドです。OpenMPWではこのラッパーの方を提出します。

ここで言うラッパーとは自分のデザインを1つ以上包んだデザインの事であり、`user_project_wrapper`という名前です。

ビルドされた`user_project_wrapper`は通常、以下の画像のようにラッパーにデザインが挿入された形になります。

![](https://i.imgur.com/qMfFIJS.png)

この画像も回路が生成されている感を出すためデキャップセルを非表示にしています。

開発者は自分のデザイン用と`user_project_wrapper`用の2つのOpenLANEの設定ファイルを編集する必要がある、という事です。後者の方は少ししか編集しないので安心してください。

Caravelのインストール直後では`user_project_wrapper`は`user_proj_example`を含むようになっているため、`user_proj_wrapper`をビルドした後`user_project_wrapper`をビルドすれば上の画像のようなGDSIIが生成されます。

```bash
make user_proj_example
make user_project_wrapper

klayout gds/user_project_wrapper.gds
```

### Caravelのドキュメント

Caravelのドキュメントはここにあり、後で詳細な使い方を説明します。

[https://caravel-harness.readthedocs.io/en/latest/index.html](https://caravel-harness.readthedocs.io/en/latest/index.html)

## OpenMPWで自分のデザインを焼こう！
ここからは本格的に自分のデザインをビルドし、OpenMPWに提出する方法を説明する。なお、Verilogの書き方は説明しないのでデザインは自分で用意してほしい。

必要なものはこちら。上3つに関しては先の章で既に済ませている。

* Caravel_user_project : これのUser Project Areaに自分のデザインを加える
* Skywater PDK : Process Design Kit、素材の情報みたいななやつ
* OpenLANE : RTLをGDSIIに変換するコンパイラ(Synopsysとか持ってるのなら必要ない)
* 自分のデザイン
* 時間、精神力 : いっぱい

### 1. 焼きたいデザインを用意する
まずはデザインを用意しよう。今回は例としてEthernet MACを用います。

https://github.com/Cra2yPierr0t/Vthernet-SoC/tree/main/verilog/rtl/Vthernet_MAC

貴方のために作りました。

### 2. 自分のデザインにCaravel用のインターフェースを生やす
自分のデザインをCaravelのMGMT Coreに接続すれば、ファームウェアでレジスタの設定やデータの受送信を行う事が出来るようになる。また、自分のデザインを外部ピンに接続すれば、直接外部とデータを受送信が行えるようになる。

ここでは、自分のデザインをCaravelに対応させるために必要な事項を説明する。

Management Coreと接続する方法は2通り存在し、**Wishbone**と**Logic Analyzer**という名前である。外部と接続する方法は1通りであり、**mprj_io**という名前が付いている。

まとめるとこう。

|   信号名  | バス幅 | 方向 |
| -------- | ----- | ---- |
| Wishbone | 32bit | user_design <-> Management Core |
| Logic Analyzer | 128bit |  user_design <-> Management Core |
| mprj_io | 38bit | user_design <-> Outside |

また、自分のデザインからMGMT Coreへ割り込みを掛けることも可能である。割り込みについては上の３つの信号の説明の後、扱い方を解説する。

#### WishboneとLogic Analyzerの違い
自分のデザインとMGMT Core間の通信を行う方法として、32bit幅のWishboneと128bit幅のLogic Analyzerの2通りが用意されていると書いたが、この２つの何が異なるのか疑問に思う方もいるだろう。なのでここでは、この２つの概要をさっくり説明する。

まずWishboneだが、これは通信プロトコルでありAXI4とかAMBAの友達である。10種類の信号線をプロトコルに沿って操作することでREADやWRITEが可能となる。Wishboneは32bitのデータ線とアドレス線に分かれており、アドレスで指定したデータの読み書きが行える。Caravelにおいては`0x30000000`~`0x80000000-1`までのアドレスがWishbone用に確保されており、この範囲でなら自由に利用できる。

次にLogic Analyzerだが、これはただの信号線名にすぎない。バス幅は128bitとWishboneの4倍あり、データ線もアドレス線も分かれておらず、3種類の信号線があるだけとなっている。MGMT Coreからは128bitを4分割した`0x250000000`, `0x250000004`, `0x250000008`, `0x25000000c`の4つのレジスタとして見えている。

HDLでの実装しやすさで言ったらLogic Analyzerの方が実装しやすいが、アドレスは4つしか割り当てられない。一方WishboneはLogic Analyzerの単純さに比べれば多少複雑だが、広範囲のアドレスを扱える。まあ用途によってどちらが適しているか変わるので、本節では両方の使い方を説明する。

#### Wishbone interfaceを作る
Wishboneでレジスタのデータを読み出したり、レジスタにデータを書き込んだりする機構の作り方を解説する。これを作ることでRISC-VプロセッサからMMIOで自分のデザインとデータの受送信が出来るようになる。

Wishboneの信号線は以下の通り。

| 信号名             | 方向 | 働き                                                                       |
| ----------------- | --- | -------------------------------------------------------------------------- |
| `wb_clk_i`        | MGMT Core -> user_design | マスターから入力されるクロック。<br>この立ち上がりのタイミングで値が読まれる。 |
| `wb_rst_i`        | MGMT Core -> user_design | リセット信号。                                                             |
| `wbs_stb_i`       | MGMT Core -> user_design | ストローブ入力。<br>スレーブが選択されている事を示す。                         |
| `wbs_cyc_i`       | MGMT Core -> user_design | サイクル入力。<br>有効なバスサイクルが進行している事を示す。                   |
| `wbs_we_i`        | MGMT Core -> user_design | ライトイネーブル入力。<br>1でWrite、0でRead。                                  |
| `wbs_sel_i[3:0]`  | MGMT Core -> user_design | セレクタ入力。<br>`wbs_dat_i[31:0]`の有効なバイトがどれか示す。8ビット粒度。   |
| `wbs_dat_i[31:0]` | MGMT Core -> user_design | データ入力。                                                               |
| `wbs_adr_i[31:0]` | MGMT Core -> user_design | アドレス入力。                                                             |
| `wbs_ack_o`       | MGMT Core <- user_design | アクノリッジ出力。<br>1にすると正常なバスサイクルの終了を示す。                |
| `wbs_dat_o[31:0]` | MGMT Core <- user_design | データ出力。                                                               |

信号線が10本もあり、AXI4やAMBA等の通信プロトコルをあまり扱わなかった方は怯むかもしれないが、扱い方は非常に単純である。まずREADとWRITEの波形を以下に示す。これはWishboneの仕様書から拝借した。

READの波形
![](https://i.imgur.com/42WsBN4.png)

READはMGMT Coreにデータを送る操作(MGMT CoreがMaster)であり、2サイクルで完了する。

WRITEの波形
![](https://i.imgur.com/ae7Rm82.png)

WRITEはMGMT Coreからデータを受け取る操作であり、同様に2サイクルで完了する。
以降プロトコルの説明を行う。

まず大前提として知ってほしいのが`adr_o`, `dat_i`, `dat_o`の３つの信号線である。それぞれ`adr_o`はアドレス線であり、`dat_i`はMGMT Coreへのデータ(READ用)であり、`dat_o`はMGMT Coreからのデータ(WRITE用)である。

1サイクル目で最初に見るべきは`stb_o`と`cyc_o`である。この２つが立っていれば、自分が選択され、通信が開始された事になる。次に`we_o`が立っていればそのサイクルではWRITEを行い、立っていなければREADを行う事を意味する。WRITEなら`adr_o`と`dat_o`にはvalidな値が来ており、`adr_o`で指定されている先に`dat_o`の値を入れればよい。READなら`adr_o`は欲しいデータのアドレスとなっている。

2サイクル目で最初に見るべきは`dat_i`と`ack_i`である。`dat_i`はREADの場合のみに用い、`adr_o`で指定されたデータをここに入力する。`ack_i`はREADとWRITEの場合両方に用い、このサイクルで立てて通信が完了したことを示す。

グダグダ説明したが、まあ波形を見ればわかると思う。あと`wbs_sel_i`はバイトイネーブルです。

仕様書にはパイプライン化されたトランザクションとか色々書いてあるが、Caravelは**おそらく**連続したトランザクションを行わず、トランザクション同士の間隔は離れていると思われる。少なくともシミュレーションではそうだった。

以下に筆者が作ったWishbone interfaceを置いておくので、実装の参考にしてほしい。(ライセンスフリー！)

```verilog
module wb_interface #(
    parameter TEST_CSR0 = 32'h3000_0000,
    parameter TEST_CSR1 = 32'h3000_0004,
    parameter TEST_CSR2 = 32'h3000_0008
)(
    // wishbone signals
    input   wire        wb_clk_i,
    input   wire        wb_rst_i,
    input   wire        wbs_stb_i,
    input   wire        wbs_cyc_i,
    input   wire        wbs_we_i,
    input   wire [3:0]  wbs_sel_i,
    input   wire [31:0] wbs_dat_i,
    input   wire [31:0] wbs_adr_i,
    output  reg         wbs_ack_o,
    output  reg  [31:0] wbs_dat_o,
    // CSRs
    output  reg  [31:0] test_csr0,
    output  reg  [31:0] test_csr1,
    output  reg  [31:0] test_csr2
);

    localparam WB_IDLE  = 2'b00;
    localparam WB_READ  = 2'b01;
    localparam WB_WRITE = 2'b10;

    reg [1:0] wb_state;

    always @(posedge wb_clk_i) begin
        if(wb_rst_i) begin
            wb_state    <= WB_IDLE;
            wbs_ack_o   <= 1'b0;
        end else begin
            case(wb_state)
                WB_IDLE : begin
                    wbs_ack_o   <= 1'b0;
                    if(wbs_stb_i && wbs_cyc_i) begin
                        if(wbs_we_i) begin
                            wb_state <= WB_WRITE;
                        end else begin
                            wb_state <= WB_READ;
                        end
                    end
                end
                WB_READ : begin
                    wb_state    <= WB_IDLE;
                    wbs_ack_o   <= 1'b1;
                    case(wbs_adr_i)
                        TEST_CSR0 : begin
                            wbs_dat_o[7:0]   <= wbs_sel_i[0] ? test_csr0[7:0] : 8'h00;
                            wbs_dat_o[15:8]  <= wbs_sel_i[1] ? test_csr0[15:0] : 8'h00;
                            wbs_dat_o[23:16] <= wbs_sel_i[2] ? test_csr0[23:16] : 8'h00;
                            wbs_dat_o[31:24] <= wbs_sel_i[3] ? test_csr0[31:24] : 8'h00;
                        end
                        TEST_CSR1 : begin
                            wbs_dat_o[7:0]   <= wbs_sel_i[0] ? test_csr1[7:0] : 8'h00;
                            wbs_dat_o[15:8]  <= wbs_sel_i[1] ? test_csr1[15:0] : 8'h00;
                            wbs_dat_o[23:16] <= wbs_sel_i[2] ? test_csr1[23:16] : 8'h00;
                            wbs_dat_o[31:24] <= wbs_sel_i[3] ? test_csr1[31:24] : 8'h00;
                        end
                        TEST_CSR2 : begin
                            wbs_dat_o[7:0]   <= wbs_sel_i[0] ? test_csr2[7:0] : 8'h00;
                            wbs_dat_o[15:8]  <= wbs_sel_i[1] ? test_csr2[15:0] : 8'h00;
                            wbs_dat_o[23:16] <= wbs_sel_i[2] ? test_csr2[23:16] : 8'h00;
                            wbs_dat_o[31:24] <= wbs_sel_i[3] ? test_csr2[31:24] : 8'h00;
                        end
                        default   : begin
                            wbs_dat_o <= 32'h0000_0000;
                        end
                    endcase
                end
                WB_WRITE : begin
                    wb_state    <= WB_IDLE;
                    wbs_ack_o   <= 1'b1;
                    case(wbs_adr_i)
                        TEST_CSR0 : begin
                            test_csr0[7:0]   <= wbs_sel_i[0] ? wbs_dat_i[7:0] : 8'h00;
                            test_csr0[15:8]  <= wbs_sel_i[1] ? wbs_dat_i[15:8] : 8'h00;
                            test_csr0[23:16] <= wbs_sel_i[2] ? wbs_dat_i[23:16] : 8'h00;
                            test_csr0[31:24] <= wbs_sel_i[3] ? wbs_dat_i[31:24] : 8'h00;
                        end
                        TEST_CSR1 : begin
                            test_csr1[7:0]   <= wbs_sel_i[0] ? wbs_dat_i[7:0] : 8'h00;
                            test_csr1[15:8]  <= wbs_sel_i[1] ? wbs_dat_i[15:8] : 8'h00;
                            test_csr1[23:16] <= wbs_sel_i[2] ? wbs_dat_i[23:16] : 8'h00;
                            test_csr1[31:24] <= wbs_sel_i[3] ? wbs_dat_i[31:24] : 8'h00;
                        end
                        TEST_CSR2 : begin
                            test_csr2[7:0]   <= wbs_sel_i[0] ? wbs_dat_i[7:0] : 8'h00;
                            test_csr2[15:8]  <= wbs_sel_i[1] ? wbs_dat_i[15:8] : 8'h00;
                            test_csr2[23:16] <= wbs_sel_i[2] ? wbs_dat_i[23:16] : 8'h00;
                            test_csr2[31:24] <= wbs_sel_i[3] ? wbs_dat_i[31:24] : 8'h00;
                        end
                    endcase
                end
            endcase
        end
    end

endmodule

```

多分動きます。

#### Logic Analyzerを扱う
Logic Analyzerで128bitのデータを受送信することが出来る。
Logic Analyzerで用いる信号線は以下の三種類。


| 信号名 | 方向 | 働き |
| ------ | ---- | ---- |
| `la_data_in` | MGMT Core -> user_design | データ入力 |
| `la_data_out` | MGMT Core <- user_diesgn | データ出力 |
| `la_oenb` | MGMT Core -> user_deisgn | データ出力イネーブルバー |

`la_data_in`はデータ入力であり、Logic Analyzerからのデータはここを通じて来る。`la_data_out`はデータ出力であり、Logic Analyzerへのデータはここに送る。注意すべきなのは、`la_oenb`である。これは`la_data_in`と`la_data_out`のどちらが有効になっているかを示す信号であり、MGMT Coreから入力される。つまり、128bitあるLogic Analyzerのどの線が入力用で、どの線が出力用なのかは、MGMT CoreがMMIOを通して決定する。

`la_data_oenb`はデータ出力イネーブル**バー**であるので、0の場合に出力(`la_data_out`)で1の場合に入力(`la_data_in`)が有効になる。**0**utputと**1**nputとして覚えると楽。

以上より、設計者は`la_oenb`の値を考慮してLogic Analyzerの信号線を扱う必要がある。

以下にLogic Analyzerの使用例を載せておくので実装の参考にしてほしい。
```verilog
assign la_data_out[31:0] = test_signals;
assign w_super_data = (&la_oenb[63:32]) ? la_data_in[63:32] : 32'h0000_0000;
```

割と簡単に扱える。ちなみに、2行目の`&`はリダクション演算子である。

#### mprj_ioを扱う
mprj_ioを用いる事で、自分のデザインと外部の間で38bitの信号を直接やり取りする事が出来る。GPIOと同じものだと思ってくれて構わない。
mprj_ioで扱う信号線は以下の3種類。

| 信号名 | 方向 | 働き |
| ------ | ---- | ---- |
| `io_in` | MGMT Core -> user_design | データ入力 |
| `io_out` | MGMT Core <- user_diesgn | データ出力 |
| `io_oenb` | MGMT Core <- user_deisgn | データ出力イネーブルバー |

Logic Analyzerと非常に似通っているが、注意してほしいのが`io_oenb`である。これは、Logic Analyzerと信号の方向が異なっており、自分のデザインからの出力となっている。よってどの線を出力に用いるか、どの線を入力に用いるかは自分のデザイン側で決定できる。

`io_oenb`はデータ出力イネーブル**バー**であるので、0の場合に出力(`io_out`)で1の場合に入力(`io_in`)が有効になる。**0**utputと**1**nputとして覚えると楽。

あとはファームウェア側で色々設定する必要はあるが、自分のデザインでは`io_oenb`の値を操作するだけでよい。

以下にmprj_ioの使用例を載せておくので実装の参考になれば幸いである。
```verilog
assign io_enb[31:0]  = 32'hffff_ffff;
assign io_enb[32]    = 1'b1;
assign io_enb[34:33] = 2'b00;
assign data_in = io_in[31:0];
assign hard_reset = io_in[32];
assign io_out[34:33] = led[1:0];
```
とか。`io_out`への入力をassignかalwaysのどちらでやるかは多分適当でよい。

#### 割り込みの扱い方
Caravelでは、自分のデザインからMGMT Coreに割り込みを掛けることが可能である。
割り込みには`user_irq[2:0]`の3bit幅の信号線を用いる。この3bitの内、どれか一つでも立った時に割り込みが発生する。割り込み発生時、MGMT Coreのプログラムカウンタには`0x00000000`がセットされる。

よって、割り込みハンドラはメモリの`0x00000000`に置くべきである。

割り込みを有効化するためにファームウェアでMGMT CoreのCSRを設定する必要があるが、自分のデザインでは割り込みを行いたい時に`user_irq`を立てるだけでよい。

この動画が非常に参考になる。Interrupt!

https://www.youtube.com/watch?v=pPgnVBguNW8

### 3. 自分のデザインをGDSIIにする
自分のデザインをCaravelに対応できたら、一旦GDSIIに変換する。この操作をHardeningと呼ぶらしい。また、Hardeningされたものをマクロと呼ぶ。

#### 設定ファイルを書く

まずは自分のデザインのVerilogファイルを`verilog/rtl`以下にあることを確認する。無ければ持ってくる。

次にOpenLANE用の設定ファイルを格納するためのディレクトリを`openlane`以下に作成する。このディレクトリの名前はトップレベルモジュールの名前と合わせる。
```bash
mkdir openlane/<user design top module>
```
ここで作成したディレクトリに設定ファイルを置くわけだが、ゼロから書くのは非常にしんどい。そこで、例として用意されているuser_proj_exampleの設定ファイルをコピーして編集する。

設定ファイルのコピー
```bash
cd openlane/<user design top module>
cp ../user_proj_example/config.tcl .
```
ここからはコピーしてきた`config.tcl`の編集すべき部分を説明する。

まずtclそのものについてだが、以下の構文を覚えればOpenLANEでは十分である。
```bash
set ::env(VAR_NAME) value
```
これは`VAR_NAME`という名前の変数に`value`という値をセットしている。以降、変数はこの`VAR_NAME`の事を指す。

`config.tcl`で最初に変更すべき変数は`DESIGN_NAME`と`VERILOG_FILES`の２つである。
`DESIGN_NAME`は文字通りデザインの名前であり、トップレベルモジュールと同じ名前にする。`VERILOG_FILES`は合成するVerilogファイルのパスである。

書き方の例を以下に載せる。`VERILOG_FILES`には複数のVerilogファイルを指定可能である。
```bash
set ::env(DESIGN_NAME) user_design_top_module

set ::env(VERILOG_FILES) "\
    $::env(CARAVEL_ROOT)/verilog/rtl/defines.v \
    $script_dir/../../verilog/rtl/user_design_top_module.v \
    $script_dir/../../verilog/rtl/user_design_sub_module.v"
```
`$script_dir`という謎の変数があるが、これは現在読んでいる`config.tcl`がある場所である。`$script_dir`は今後消滅して`$DESIGN_DIR`に変更される可能性が高い。

次に変更すべき変数は`CLOCK_PORT`、`CLOCK_NET`、`CLOCK_PERIOD`の３つであり、これらはクロックに関係している。`CLOCK_PORT`にはデザインのクロック入力に用いるポート指定する、`CLOCK_NET`は正直よくわからない。`CLOCK_PORT`はSTAのための変数で`CLOCK_NET`はCTSのための変数であるとかドキュメントに書いてあるけど同じ値をセットして**多分**問題無いと思う。

`CLOCK_PERIOD`にはクロック周期をナノ秒単位で設定する。ナノ秒単位なので"10"を設定した場合は100MHzとなる。

以下に例を載せる。`CLOCK_NET`に関しては誰か教えてほしい。
```bash
set ::env(CLOCK_PORT) "wb_clk_i"
set ::env(CLOCK_NET) $::env(CLOCK_PORT)
set ::env(CLOCK_PERIOD) "10"
```

次に変更すべきなのは`DIE_AREA`である。これにはマクロのサイズをマイクロメートル単位で設定する。設定は`x0 y0 x1 y1`の順であり、仮に`"0 0 200 100"`と設定すると200umx100umの長方形となる。座標は第一象限と同じである。

最初は適当でよく、後述する`PL_TARGET_DENSITY`と合わせて最適な組み合わせを探すと良い。

以下に例を載せる。
```bash
set ::env(DIE_AREA) "0 0 900 600"
```

次に変更するのは`FP_PIN_ORDER_CFG`であり、これはマクロにおいて、どのピンをどの方角から出すかを設定する。だが、設定しなくても勝手にやってくれるので、コメントアウトするなり行ごと削除するなりして消してほしい。

コメントアウトした例を以下に載せる。
```bash
#set ::env(FP_PIN_ORDER_CFG) $script_dir/pin_order.cfg
```
必要になったら各自で設定してほしい。

最後に変更するのは`PL_TARGET_DENSITY`である。これは`DIE_AREA`設定した面積で、どの程度の密度で配置するかを決定する。値は0から1の実数を取る。0か1かの極端な値を設定すると、OpenLANEが最適な値を教えてくれるのでそれを参考にしてもいいが、適当に0.4とか入れても問題ない。

以下に例を載せる。
```bash
set ::env(PL_TARGET_DENSITY) 0.2
```

変更すべき変数をまとめた表が以下になる。

| 変数名              | 働き                         | 備考 |
| ------------------- | ---------------------------- | ---- |
| `DESIGN_NAME`       | トップレベルモジュールを指定 |      |
| `VERILOG_FILES`     | Verilogファイルを指定        |      |
| `CLOCK_PORT`        | クロックポートを指定         |      |
| `CLOCK_NET`         | 謎                           | 謎 |
| `CLOCK_PERIOD`      | クロック周期を指定           | ナノ秒単位 |
| `DIE_AREA`          | マクロのサイズを指定         | マイクロメートル単位<br>`x0 y0 x1 y1` |
| `FP_PIN_ORDER_CFG`  | ピンの方角を指定             | 削除 |
| `PL_TARGET_DENSITY` | 配置密度を指定               | 0~1 |


#### GDSIIを生成
設定ファイルが完成したら、次にGDSIIを生成する。`caravel_user_project`以下で`make <design_name>`でビルドが開始される。
```bash
make <user design top module>
```

生成されたGDSIIは、`openlane/<user design top module>/runs/RUN_<date>/results/final/gds`以下に存在しており、klayoutを用いてレイアウトを見られる。
```bash
klayout <user design top module>.gds
```
かわいいね。

### 4. user_project_wrapperに自分のデザインを加える
`verilog/rtl/user_project_wrapper.v`に`user_proj_example`の代わりに自分のモジュールを入れるだけである。なお、`user_project_wrapper`でANDやNOT等のロジックを組むのは予期せぬエラーの原因になるのでオススメしない。

#### 追記: `user_defines.v`を編集
`verilog/rtl/user_defines.v`を編集し、IOの状態を設定する。全ての`GPIO_MODE_INVALID`をデザインに応じて`GPIO_MODE_USER_STD_OUTPUT`や`GPIO_MODE_USER_STD_INPUT_PULLDOWN`に変更する。

### 5. シミュレーションで動作を確認する
Caravelに自分のデザインを接続出来たら次はシミュレーションで動作確認をする。

シミュレーションを行うには、少々の設定と、MGMT Coreで動かすファームウェア及び`io_in`等にどんな信号を入力するかのテストベンチを書く必要があるので、順番に説明する。

#### シミュレーション環境のインストール

このコマンドでシミュレーション用の環境をインストールする。
```bash
make simenv
```

#### 使うファイルを指定する
OpenLANEで自分のデザインのVerilogファイルのパスを`config.tcl`に設定したのと同様に、シミュレーションにおいても、利用するファイルをシミュレーション環境に伝える必要がある。

編集する必要のある設定ファイルは以下の３つである。

* `verilog/includes/includes.gl+sdf.caravel_user_project`
* `verilog/includes/includes.gl.caravel_user_project`
* `verilog/includes/includes.rtl.caravel_user_project`

まず`includes.rtl.caravel_user_project`に関してだが、ここには`user_project_wrapper`を含めた、自分のデザインに必要なVerilogファイルのパスを追加する。

以下に例を載せる。
```bash
-v $(USER_PROJECT_VERILOG)/rtl/user_project_wrapper.v	     
-v $(USER_PROJECT_VERILOG)/rtl/user_design_top_module.v
-v $(USER_PROJECT_VERILOG)/rtl/user_design_sub_module.v
```

次に`includes.gl.caravel_user_project`に関してだが、ここには`verilog/gl/`以下に生成されているVerilogファイルのパスを追加する。`verilog/gl/`以下のファイルは自動生成され、複数のVerilogをまとめたものになっている。

以下に例を載せる。
```bash
-v $(USER_PROJECT_VERILOG)/gl/user_project_wrapper.v
-v $(USER_PROJECT_VERILOG)/gl/user_design_top_module.v
```

最後に`includes.gl+sdf.caravel_user_project`に関してだが、これも`includes.gl.caravel_user_project`と同様に`verilog/gl/`以下のファイルのパスを追加する。

以下に例を載せる。
```bash
$USER_PROJECT_VERILOG/gl/user_project_wrapper.v	     
$USER_PROJECT_VERILOG/gl/user_design_top_module.v
```

以上で設定は完了となる。

#### ファームウェアを書く
ファームウェアはC言語で書ける。CaravelはRISCV toolchainの入ったDockerコンテナを引っ張ってきて`make`コマンドでコンパイルなりなんなりしてくれる。


まずはシミュレーション用のディレクトリを`verilog/dv`以下に作成する。
```bash
mkdir verilog/dv/<sim_name>
```

このディレクトリにファームウェアとテストベンチを格納する。

ファームウェアの名前は`<sim_name>.c`にする。以降、ファームウェアの書き方を説明する。

最初に２つのファイルをインクルードする。
```c
#include <defs.h>
#include <stub.c>
```
この`defs.h`には様々なグローバル変数と定数が定義されている。

[https://github.com/efabless/caravel/blob/main/verilog/dv/caravel/defs.h
](https://github.com/efabless/caravel/blob/main/verilog/dv/caravel/defs.h
)

後は`main`関数内にプログラムを書いていくが、普通のC言語と同様に関数を定義したりファイル分割してもいい。

CSRのコンフィギュレーション部分は以下のように記述する。
```c
reg_mprj_io_0   = GPIO_MODE_USER_STD_INPUT_PULLDOWN;
reg_mprj_io_1   = GPIO_MODE_USER_STD_INPUT_PULLDOWN;
reg_mprj_io_2   = GPIO_MODE_USER_STD_INPUT_PULLDOWN;
reg_mprj_io_3   = GPIO_MODE_USER_STD_INPUT_PULLDOWN;
reg_mprj_io_4   = GPIO_MODE_USER_STD_INPUT_PULLDOWN;
reg_mprj_io_5   = GPIO_MODE_USER_STD_INPUT_PULLDOWN;
reg_mprj_io_6   = GPIO_MODE_USER_STD_INPUT_PULLDOWN;
reg_mprj_io_7   = GPIO_MODE_USER_STD_INPUT_PULLDOWN;
reg_mprj_io_33  = GPIO_MODE_USER_STD_OUTPUT;
reg_mprj_io_34  = GPIO_MODE_USER_STD_OUTPUT;

reg_mprj_xfer = 1;
while (reg_mprj_xfer == 1);
```

この例では、`mprj_io[7:0]`を入力に設定し、`mprj_io[34:33]`を出力に設定している。
その後の`reg_mprj_xfer, while`の部分は設定の適用を行っている部分であり、設定が適用されるとwhile文から抜ける。ここの仕組みは不明。

`GPIO_MODE_USER_STD_INPUT_PULLDOWN`は`mprj_io`を自分のデザインへの入力に設定する定数であり、`GPIO_MODE_USER_STD_OUTPUT`は自分のデザインからの出力に設定する定数である。

その他に`GPIO_MODE_MGMT_STD_OUTPUT`や`GPIO_MODE_MGMT_STD_INPUT_PULLDOWN`などの定数を設定する事ができ、この場合はMGMT CoreとGPIOで信号を受送信を行えるようになる。

基本的に、`while(reg_mprj_xfer == 1)`以前に設定、以降に処理を書けば良い。

#### テストベンチを書く

テストベンチをゼロから書くのは非常に面倒であるため、既存のテストベンチをコピペしてきて使う。今回は例として、`io_ports/io_ports_tb.v`をたたき台に用いる。

たたき台のコピー
```bash
cp ./dv/io_ports/io_ports_tb.v ./dv/<sim_name>/<sim_name>_tb.v
```

以下では必要な設定の変更を行う。

まずテストベンチ名を変える。
```verilog
module <sim_name>_tb;
```

その下に`ifdef ENABLE_SDF`で囲まれている不気味な部分があるが、ここを編集する必要は無い。

次に波形ファイルの対象と名前を変更する。
```verilog
initial begin
    $dumpfile("<sim_name>.vcd");
    $dumpvars(0, <sim_name>_tb);
    ...
```

次にファイルの一番下にある`spiflash`に読み込ませるファイル名を変更する。
```verilog
spiflash #(
    .FILENAME("<sim_name>.hex")
) spiflash (
...
```

設定は以上、これが終わったら162行目から始まるinitial文の中に、自由に信号を書き込んでいく。
[https://github.com/efabless/caravel_user_project/blob/main/verilog/dv/io_ports/io_ports_tb.v#L162](https://github.com/efabless/caravel_user_project/blob/main/verilog/dv/io_ports/io_ports_tb.v#L162)

以下に例を載せる。元のファイルにある`ifdef GL`は気にしなくて良い。`$finish;`だけは忘れないようにする。
```verilog
initial begin
    mprj_io[7:0] <= 8'h55;
    # 100
    mprj_io[7:0] <= 8'haa;
    # 100
    wait(mprj_io[34:33] == 2'b11);
    $finish;
end
```

#### Makefileをコピーする

Makefileをコピーする、内容を編集する必要は無い。
```bash
cp ./dv/io_ports/Makefile ./dv/<sim_name>/
```


#### シミュレーションを実行する
ここまでやればシミュレーションの実行準備は万全である。`caravel_user_project/`で以下のコマンドを実行すればシミュレーションが始まる。
```bash
make verify-<sim_name>-rtl
```

デバッグ頑張ってね。

### 6. OpenLANEの設定をする

作成したデザインのデバッグが完了したら、ラッパーをビルドする準備に移る。
まずは`user_project_wrapper`の`config.tcl`を編集する。

`config.tcl`で変更すべき変数は以下の通り。

| 変数名                   | 働き                     | 備考 |
| ------------------------ | ------------------------ | ---- |
| `CLOCK_NET` | 謎  | `CLOCK_PORT`と同じでよい |
| `CLOCK_PERIOD`      | クロック周期を指定           | ナノ秒単位 |
| `FP_PDN_MACRO_HOOKS`     | マクロへの電源接続の明示 |      |
| `VERILOG_FILES_BLACKBOX` | マクロのVerilogファイルの指定 |      |
| `EXTRA_LEFS`             | 利用するLEFファイルの指定 |      |
| `EXTRA_GDS_FILES`        | 利用するGDSファイルの指定 |  |

#### `CLOCK_NET`の変更
デフォルトでは`mprj.clk`になっているので、`CLOCK_PORT`と同じものにするかその他の適切なものに変更する。

```bash
set ::env(CLOCK_NET) $::env(CLOCK_PORT)
```

#### `CLOCK_PERIOD`の変更
これにはお好みの動作クロック周期を設定する。

#### `FP_PDN_MACRO_HOCKS`の変更
デフォルトでは`mprj`というマクロへの電源の接続になっているので、自分のマクロ名に変更する。
```bash
set ::env(FP_PDN_MACRO_HOOKS) "\
	<user design top module> vccd1 vssd1 vccd1 vssd1"
```

#### `VERILOG_FILES_BLACKBOX`の変更

自分のデザインのトップモジュールのパスに変更する。`define.v`のパスには変更を加えてはいけない。
```bash
set ::env(VERILOG_FILES_BLACKBOX) "\
	$::env(CARAVEL_ROOT)/verilog/rtl/defines.v \
	$script_dir/../../verilog/rtl/<user design top module>.v"
```

#### `EXTRA_LEFS`の変更
`EXTRA_LEFS`には`caravel_user_project/lef/`に自動生成されている、自分のデザインのLEFファイルのパスを設定する。
```bash
set ::env(EXTRA_LEFS) "\
	$script_dir/../../lef/<user design top module>.lef"
```

#### `EXTRA_GDS_FILES`の変更
`EXTRA_GDS_FILES`には`caravel_user_project/gds/`に自動生成されている、自分のデザインのGDSIIファイルのパスを設定する。
```bash
set ::env(EXTRA_GDS_FILES) "\
	$script_dir/../../gds/<user design top module>.gds"
```

#### `macro.cfg`の編集

`config.tcl`の設定が完了したら、最後に`macro.cfg`を編集する。

デフォルトでは`mprj`という名前のインスタンスの位置を指定しているが、これを自分のマクロのインスタンスの位置の指定に変更する。フォーマットは`instance_name X_pos Y_pos Orientation`になっている。

以下に例を載せる。
```bash
<user design instance> 100 100 N
```

この`macro.cfg`には複数のインスタンスの位置を指定することが出来る。


以上の設定が完了したら、最後にラッパーのビルドを行う。

### 7. 最終レイアウトを生成する
`caravel_user_project/`で以下のコマンドを実行することで、ラッパーのビルドが開始される。

```bash
make user_project_wrapper
```

これが成功した時、`gds/`には`user_project_wrapper.gds`が生成されている事だろう。

### 8. プリチェックを実行する

プリチェック用のDockerイメージをインストール
```bash
make precheck
```

プリチェックを実行
```bash
make run-precheck
```

プリチェックで問題が無いことを確認しよう。

**おめでとう！** これであなたのデザインは完成しました！あとはEfablessの指示に従ってリポジトリを登録するだけです。抽選に当たる事を願いましょう！

自分のリポジトリをここに提出しましょう。
[https://efabless.com/open_shuttle_program](https://efabless.com/open_shuttle_program)

お疲れ様でした。


## Tips

### PDKのアップデート
1. caravel_user_projectで`$ make uninstall`
2. dependenciesのpdksとopenlane_srcを削除
3. caravel_user_projectで`$ make install`
4. caravel_user_projectで`$ make setup`

### その他

`[0:0]`なポートを作るとLVS Error

FP_PDN_MACRO_HOOKSはモジュール毎にカンマで区切る

ポートのビット幅はちゃんと揃えた方がいい

OpenROAD reports unconnected nodes as a warning.
OpenLane typically treats unconnected node warnings as a critical issue, and simply quits.

We'll be leaving it up to the designer's discretion to enable/disable this: if LVS passes you're probably fine with this option being turned off.
```bash
set ::env(FP_PDN_CHECK_NODES) 0
```

### podmanで試行錯誤した時のメモ

OpenLANEイメージをpullする際に
```
Error: short-name "efabless/openlane:2022.07.02_01.38.08" did not resolve to an alias and no unqualified-search registries are defined in "/etc/containers/registries.conf"
```
とか言われる場合があるので、`/etc/containers/registeries.conf`に次の行を追加する。

```bash
sudo vim /etc/containers/registries.conf
```
以下の行を追加
```
 unqualified-search-registries = ["docker.io"]
```

実行時のエラーはこれで消せる
```bash
sudo touch /etc/subuid /etc/subgid
sudo usermod --add-subuids 10000-75535 $(whoami)
sudo usermod --add-subgids 10000-75535 $(whoami)
podman system migrate
```
---
