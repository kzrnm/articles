---
title: "シェルスクリプトでファイルサイズを取得する"
emoji: "📏"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: 
 - "shell"
 - "bash"
published: true
---

# シェルスクリプトでファイルサイズを取得する

## `ls`, `awk` コマンドで取得

```sh
ls -l image.jpg | awk '{print $5}'
```

## `stat` コマンドで取得

単一コマンドで済みますが、GNU版かどうかでオプションが違います。

### Mac

```sh
stat -f "%z" image.jpg
```

### Linux(GNU command)

```sh
stat -c "%s" image.jpg
```
