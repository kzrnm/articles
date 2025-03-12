---
title: "PowerShell 用の　Git 補完モジュールをリリースしました"
emoji: "🛷"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: 
  - Git
  - PowerShell
published: true
---

# PowerShell 用の　Git 補完モジュール

PowerShell 用の Git 補完モジュール [git-completion-pwsh](https://github.com/kzrnm/git-completion-pwsh/) をリリースしました。[posh-git](https://github.com/dahlbyk/posh-git) の補完がかなり限られているので、bash の補完の完全再現と PowerShell 用の最適化を目指して作りました。Windows PowerShell でも最新のマルチプラットフォーム PowerShell でも動作します。

![Usage of git-completion](/images/git-completion-pwsh-usage.png)

## インストール

お使いの PowerShell で

```powershell
Install-Module git-completion
```

と実行するだけです。

## 機能の特徴

### Bash の補完と同等の機能

Bash では `ls-files` などのあらゆるサブコマンドに対して補完が効きます。posh-git では一般的なコマンドは網羅されていますが、あらゆるコマンドに対処できるとは言い難いです。また、posh-git ではオプションをスクリプトに埋め込んでいるので git の更新に追従できないという問題もあります。Bash での 補完は git コマンドの持つ `--git-completion-helper` という補完用のオプションを活用して、git の更新にも対処しやすい構成となっています。これを PowerShell でも再現して変更に強いモジュールにしました。

### オプションの補完

posh-git ではショートオプションも補完してくれますが、Bash では不便なことにショートオプションは補完してくれません。オプションを覚えなくてはならないのは面倒なのでショートオプションもロングオプションも補完してくれるようにしています。

さらに、Bash の補完と同様にオプションの値に基づいて補完する内容を動的に変更するなどの機能も備えています。

### ファイルパスの補完

Bash の補完でも posh-git の補完でも状態に基づいてファイルを補完するので、git-completion-pwsh でも補完してくれるように実装しました。例えば、`git add` では新規ファイルや修正されたファイルのみを補完対象とするなど、必要なファイルのみを賢く選択します。

### Tooltip の活用

デフォルトのタブ補完だと表示されないのですが、下記のように MenuComplete モードにしたうえでの補完では Tooltip が表示されます。上記のスクリーンショットは MenuComplete モードでの動作です。

```powershell
Set-PSReadLineKeyHandler -Chord Tab -Function MenuComplete
```

![commit completion](/images/git-completion-pwsh-usage-log.png)

ブランチを選択したらコミットの内容が表示されたり、オプションの説明が表示されるので直感的な選択ができます。

## 余談

この記事の内容を元に ChatGPT に英語翻訳してもらって [Medium にも投稿](https://kzrnm.medium.com/a-more-powerful-git-completion-module-for-powershell-5905d103ce0f) しています。いい感じの文章を作ってくれるので、LLM は Google 翻訳や DeepL とは別格の力強さを感じました。
