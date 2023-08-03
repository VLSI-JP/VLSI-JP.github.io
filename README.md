---
layout: default
title: README.md
---
# README.md

愛と半導体と計算機のゆるふわ個人サイト

## 記事の追加方法

1. [VLSI-JP.github.io](https://github.com/VLSI-JP/VLSI-JP.github.io)にアクセス
2. `Add file`をクリック
3. `ファイル名.md`を作成、内容を書く
4. `ファイル名.md`の先頭に以下を追加
```
---
layout: default
title: 記事のタイトル
---
```
5. `_data/article_list.yml`を開き、編集ボタンを押す
6. 以下を追記
```yml
- title: 記事のタイトル
  author: 筆者名
  date: 書いた日
  link: ファイル名.html
```
7. HAPPY

Optional : `_data/article_list.yml`に`icon: `でアイコンを変更できる。`dir`, `file`, `link`が使用可能。無指定なら`file`になります。

記事内で自己紹介とかはガンガンやってください

不明な点があったり↑の手順がダルかったりIndex of VLSI.JPにリンクだけ置きたい場合は@Cra2yPierr0tマデ、私は遍在します。

## 記事の編集方法

1. [VLSI-JP.github.io](https://github.com/VLSI-JP/VLSI-JP.github.io)にアクセス
2. `ファイル名.md`をクリック
3. 編集ボタンをクリック
4. 記事を編集
5. `_data/article_list.yml`の`date`の日付を編集
6. HAPPY

## リンクの追加方法

Index of VLSI.JPから別サイトに飛ばしたい場合の手順

1. [VLSI-JP.github.io](https://github.com/VLSI-JP/VLSI-JP.github.io)にアクセス
2. `_data/article_list.yml`を開き、編集ボタンを押す
3. 適当な位置に以下を追記
```yml
- title: タイトル
  author: 筆者名
  date: 書いた日
  link: リンク
  icon: out
```
4. 保存
5. HAPPY

## Gitコマンドで色々やる方法

**`git pull`は絶対にやろうな！**

1. `git pull`
2. 色々やる
3. `git add やった内容`
4. `git commit -m "やった内容"`
5. `git push`
