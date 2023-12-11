---
title: .NET Native AOT で Hardware Intrinsics を使用する
emoji: "🚄"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["C#", "dotnet", "競技プログラミング"]
published: true
---

# Hardware Intrinsics を使用する

AtCoder で Native AOT の方に提出したらきちんと動作しなかったので調査したところ、Native AOT で Hardware Intrinsics を使用するには設定が必要なようでした。

- https://atcoder.jp/contests/APG4b/submissions/48434173
- https://atcoder.jp/contests/APG4b/submissions/48434089


## .NET 8 以降

csproj に

```xml
<PropertyGroup>
  <IlcInstructionSet>native</IlcInstructionSet>
</PropertyGroup>
```

と設定する必要があります。

## .NET 7 以前

`native` は使えないので、使用する機能を csproj に列挙します。

たとえば x86 では下記のようにします。

```xml
<PropertyGroup>
  <IlcInstructionSet>avx2,bmi2,fma,pclmul,popcnt,aes</IlcInstructionSet>
</PropertyGroup>
```

## 参考文献

[公式ドキュメント](https://github.com/dotnet/runtime/blob/6de7549ae1872009baade9091be76071f955c47d/src/coreclr/nativeaot/docs/optimizing.md)
