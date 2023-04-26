---
title: "競技プログラミングライブラリ用のテストツール"
emoji: "🌠"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: 
  - "競技プログラミング"
  - "atcoder"
published: true
---

# 競技プログラミングライブラリのテスト

競技プログラミングライブラリ向けのテストツールを作ったので簡単に記事にまとめます。

## 作成したツール

https://github.com/competitive-verifier/competitive-verifier

### 使い方

リポジトリ内の examples のような形式でディレクトリを作成するようにしてください。

https://github.com/competitive-verifier/competitive-verifier/tree/HEAD/examples

C# を使用している場合は csharp-resolver との併用を勧めます。

https://github.com/competitive-verifier/csharp-resolver

ファイルを設置したら、下記のページの内容を埋めて `.github/workflows/verify.yml` を作成してください。

https://competitive-verifier.github.io/competitive-verifier/installer.ja.html


### ドキュメント

https://competitive-verifier.github.io/competitive-verifier/

competitive-verifier のドキュメント自身も competitive-verifier で生成されています。


## 既存のテストツール

競技プログラミングライブラリのテストツールとしては [online-judge-tools/verification-helper](https://github.com/online-judge-tools/verification-helper) a.k.a `oj-verify` が広く使われています。

競技プログラミングサイトの問題を使用してテストを行うことができます。

対応サイト
- [Library Checker](https://judge.yosupo.jp/)
- [Aizu Online Judge](https://onlinejudge.u-aizu.ac.jp/home)
- [AtCoder](https://atcoder.jp/)

また、GitHub Pages にドキュメントを作成してくれる機能まであります。

非常に優れたツールなのですが、いくつかの課題があることや更新が停止していることから新たなライブラリを作成することにしました。

### 既存のテストツールの課題点

- 言語ごとの実装がコマンドに埋め込まれており、外部から機能を注入することが難しい
  - このため C# ライブラリをテストできない
- [Library Checker](https://judge.yosupo.jp/) の問題生成などで実行に非常に時間がかかる
- ユニットテストとの統合が困難
  - 特に .NET や Go などの main() と無関係なユニットテストを持つ言語で非常に厳しい
- GitHub Pages の形式が古い
  - ブランチを作成しない最新の GitHub Pages に未対応


### 課題点の解決方法

- oj-verify から容易に移行できる
  - migration コマンドを用意
- [poetry](https://github.com/python-poetry/poetry) や [Pydantic](https://docs.pydantic.dev/) などの
- 言語ごとの動作を独立に操作できるようにする
  - [C#](https://github.com/competitive-verifier/csharp-resolver) のように独自に実装できる
- 並列実行できるようにする
  - GitHub Actions の job を複数実行する状況を想定
  - oj-verify では20分かかっても終わらないライブラリでも20並列で実行することで高速に完了する。

### oj-verify からの移行方法

oj-verify から移行する場合は

```sh
pip install competitive-verifier

cd "ライブラリのディレクトリ"
competitive-verifier migrate
```

とすれば自動でファイルを書き換えてくれます。