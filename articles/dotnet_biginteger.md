---
title: BigInteger(任意精度整数) の乗算・除算・文字列処理
emoji: "🚄"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: 
 - "csharp"
 - "dotnet"
 - 多倍長整数
 - "Burnikel-Ziegler 法"
 - 競技プログラミング
published: false
---

# BigInteger について

いわゆる「任意精度整数」「多倍長整数」は .NET では `System.Numerics.BigInteger` 型として実装されています。

https://learn.microsoft.com/ja-jp/dotnet/api/system.numerics.biginteger

ただ、実装に改善の余地が大きく存在したのでいくつか Pull Request を出しました。

BigInteger の計算について説明しつつ、それらの Pull Request を説明していきます。

以下の計算量の説明では桁数を $n$ とすることにします。また 基数を $\beta$ とします。$2$ 進数ならば $\beta=2$, $10$ 進数ならば $\beta=10$ です。

また、[競技プログラミングでも使えるよう改造したもの](https://github.com/kzrnm/BigInteger/)も公開しています。

## BigInteger の乗算

BigInteger の乗算は [カラツバ法](https://ja.wikipedia.org/wiki/%E3%82%AB%E3%83%A9%E3%83%84%E3%83%90%E6%B3%95) で実装されています。

ただしく実装されていれば $O(n^{\log_2 3})$ で計算することができます。

ちなみに、BigInteger の乗算については [Toom–Cook 乗算](https://en.wikipedia.org/wiki/Toom%E2%80%93Cook_multiplication) $O(n^{\log_3 5})$ や FFT $O(n \log_n)$ での畳み込みでより最適化する余地もありますが、ひとまずここでは実装していません。

### カラツバ法の概要

$n$ 桁の整数 $A, B$ の乗算を考えます。

整数 $A$ の下位 $n/2$ 桁を $A_0$、上位 $n/2$ 桁を $A_1$ とします。$B$ についても同様に $B_0, B_1$ とします。

この積を $Z = AB$ とすると $n/2$ 桁ごとの乗算で $Z$ を計算することができます。

$$
\begin{aligned}
Z &= AB \\
&= (A_0 + A_1 \beta^{n/2}) (B_0 + B_1 \beta^{n/2}) \\
&= (A_1 + B_1) \beta^n + (A_1 B_0 + A_0 B_1) \beta^{n/2} + (A_0 + B_0)
\end{aligned}
$$

```text
|                 |                 |       A_1       |       A_0       |
|                 |                 |       B_1       |       B_0       |

Z
|                 |                 |             A_0 * B_0             |
|                 |       A_1 * B_0 + A_0 * B_1       |                 |
|             A_1 * B_1             |                 |                 |
```

愚直な実装だと、加減算は $O(n)$、乗算は $O(n^2)$ となるので $(n/2)^2 \times 4 = n^2$ となり、このままでは特に改善にはなりません。

しかし、$A_1 B_0 + A_0 B_1$ を下記のように式変形すると事情が変わってきます。

$$
\begin{aligned}
& A_1 B_0 + A_0 B_1 \\
&= (A_1 B_0 + A_0 B_1 + A_0 B_0 + A_1 B_1) - A_0 B_0 - A_1 B_1 \\
&= (A_0 + A_1) (B_0+B_1) - A_0 B_0 - A_1 B_1 \\
\end{aligned}
$$

$A_0 B_0, A_1 B_1, (A_0 + A_1) (B_0+B_1)$ の 3 回の乗算と加減算で積を求めることが出来ます。再帰的に適用することで、乗算の計算量を $O(n^{\log_2 3})$ とすることができます。

### カラツバ法の実例

$10$ 進数 $8$ 桁の乗算をしてみます。

適当に $A=47948571, B=13636632, Z=653857017652872$ とします。

$$
\beta^n = 100000000 \\
\beta^{n/2} = 10000 \\
A_0=8571 \\
A_1=4794 \\
B_0=6632 \\
B_1=1363
$$

より

$$
\begin{aligned}
A_0 B_0 &=& 56842872\\
A_1 B_1 &=& 6534222\\
(A_0 + A_1 = 13365) (B_0+B_1=7995) &=& 106853175
\end{aligned}
$$

$$
\begin{aligned}
Z &= (A_1 + B_1) \beta^n + (A_1 B_0 + A_0 B_1) \beta^{n/2} + (A_0 + B_0) \\
&= 6534222 \times 100000000 + (106853175-6534222-56842872) \times 10000 + 56842872 \\
&= 653422200000000 + 434760810000 + 56842872 \\
&= 653857017652872 \\
\end{aligned}
$$

### カラツバ法での乗算のバグ

- https://github.com/dotnet/runtime/pull/92208
- https://github.com/dotnet/runtime/pull/98587

.NET 8 までの実装にはバグがありました。

$n \ge m$ として $n$ 桁と $m$ 桁の乗算をするときは、$n/2$ 桁で分割しなければならないのですが、$m/2$ 桁で分割されてしまっていました。

そのため、桁数の差があるときに非常に遅くなってしまう実装になっていました。

変更前は $10^6$ 桁同士の乗算が 1.2 秒程度だったのに対して、$10^6$ 桁と $10^5$ 桁の乗算は 5 秒ほどかかってしまう遅さだったのですが、$10^6$ 桁と $10^5$ 桁の乗算が 450 ミリ秒 程度で終わるよう大幅に改善されました。

## BigInteger の除算

除算は乗算よりかなりややこしいです。愚直な実装での計算量は $O(n^2)$ ですが、乗算よりも重めです。

この除算の計算を乗算に落とし込んで $O(K(n) + n \log n)$ ($K(n)$ は乗算の計算量) で計算するアルゴリズムが考案されていたりします。

競技プログラミングでの実装はニュートン法で実装している人が多い印象です。

参考: [競プロでの提出例](https://judge.yosupo.jp/submissions/?problem=division_of_big_integers&status=AC&order=-id&page=0&pagesize=100)

しかし、実用されているアルゴリズムだと **Burnikel-Ziegler 法** が主流のようです。ざっと調べただけで 3 つヒットしました。GitHub で公開されてないものもあったりしそうなので広く使われていそうです。

- [Golang](https://github.com/golang/go/issues/21960)
- [OpenJDK](https://github.com/openjdk/jdk/blob/be900f1253fe130347385f0daec772c20a79ed57/src/java.base/share/classes/java/math/MutableBigInteger.java#L1246)
- [JavaScript-V8](https://github.com/v8/v8/blob/7fdc2728f2114b083ebbd24710a32fe4574c0c57/src/bigint/div-burnikel.cc)

よって、Burnikel-Ziegler 法での実装での PR を出しました。

- https://github.com/dotnet/runtime/pull/96895

### Burnikel-Ziegler 法

https://zenn.dev/naminodarie/articles/burnikel_ziegler を参照

## string ⇄ BigInteger

BigInteger は `UInt32` 配列で実装されているため、$2^{32}$ 進数とみなせます。そのため、$2$ 進数や $16$ 進数の文字列ならば容易に変換可能です。

しかし、$10$ 進数の文字列との変換はそれなりに難しいです。どちらの計算についても愚直計算では $O(n^2)$ かかります。

[Divide-and-Conquer アルゴリズム](http://www.numberworld.org/y-cruncher/internals/radix-conversion.html) で計算すると、乗算・除算の計算量を $M(n)$ として $O(K(n)\log(n))$ で計算できます。

### Divide-and-Conquer アルゴリズム

あらかじめ、$10^{k 2^n}$ を計算しておきます。

string → BigInteger の変換では $k 2^n$ 桁で文字列を分割してパース、上位に $10^{k 2^n}$ をかける。

BigInteger → string の変換では、$10^{k 2^n}$ で割った商を上位の $k 2^n$ 桁、あまりを下位の $k 2^n$ 桁として再帰的に変換します。

`BigInteger.Parse` (string → BigInteger の変換) は[実装済み](https://github.com/dotnet/runtime/pull/55121)です。

ただし、愚直計算の閾値が大きすぎたので閾値の計算によって $20000$ 桁の文字列のパースでは2倍ほど速くなりました。

- https://github.com/dotnet/runtime/pull/96538

`BigInteger.ToString` (BigInteger → string の変換) は除算の計算量の改善が必要なので除算の改善後に PR 

