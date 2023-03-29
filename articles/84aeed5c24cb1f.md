---
title: "C#で高速なジェネリックを書く"
emoji: "💭"
type: "tech"
topics:
  - "csharp"
  - "競技プログラミング"
published: true
published_at: "2021-03-14 23:51"
---

# C#で高速なジェネリック

下記のなんの変哲もないバブルソートのコードは`int`と`string`でそれぞれどれが速いかわかりますか？

```cs
static class BubbleSorts
{
    public static void BubbleSortInt(int[] array, IComparer<int> comparer)
    {
        for (int i = 0; i + 1 < array.Length; i++)
            for (int j = array.Length - 1; j > i; j--)
                if (comparer.Compare(array[j], array[j - 1]) < 0)
                    (array[j], array[j - 1]) = (array[j - 1], array[j]);
    }
    public static void BubbleSortString(string[] array, IComparer<string> comparer)
    {
        for (int i = 0; i + 1 < array.Length; i++)
            for (int j = array.Length - 1; j > i; j--)
                if (comparer.Compare(array[j], array[j - 1]) < 0)
                    (array[j], array[j - 1]) = (array[j - 1], array[j]);
    }
    public static void BubbleSortIComparer<T>(T[] array, IComparer<T> comparer)
    {
        for (int i = 0; i + 1 < array.Length; i++)
            for (int j = array.Length - 1; j > i; j--)
                if (comparer.Compare(array[j], array[j - 1]) < 0)
                    (array[j], array[j - 1]) = (array[j - 1], array[j]);
    }
    public static void BubbleSortGeneric<T, TComparer>(T[] array, TComparer comparer)
        where TComparer : IComparer<T>
    {
        for (int i = 0; i + 1 < array.Length; i++)
            for (int j = array.Length - 1; j > i; j--)
                if (comparer.Compare(array[j], array[j - 1]) < 0)
                    (array[j], array[j - 1]) = (array[j - 1], array[j]);
    }
}
```

答えは「`where TComparer : IComparer<T>` の型制約をつけた一番下のものに`IComparer<T>`を実装した構造体を渡したとき」です。

``` ini

BenchmarkDotNet=v0.12.1, OS=Windows 10.0.19042
Intel Core i7-4790 CPU 3.60GHz (Haswell), 1 CPU, 8 logical and 4 physical cores
.NET Core SDK=5.0.103
  [Host]   : .NET Core 5.0.3 (CoreCLR 5.0.321.7212, CoreFX 5.0.321.7212), X64 RyuJIT
  ShortRun : .NET Core 3.1.12 (CoreCLR 4.700.21.6504, CoreFX 4.700.21.6905), X64 RyuJIT

Job=ShortRun  Toolchain=.NET Core 3.1  IterationCount=3  
LaunchCount=1  WarmupCount=3  

```
|              Method |      Mean |     Error |   StdDev |
|-------------------- |----------:|----------:|---------:|
|       NonGenericInt | 199.86 ms | 10.091 ms | 0.553 ms |
|        IComparerInt | 178.57 ms | 99.504 ms | 5.454 ms |
|     GenericClassInt | 180.05 ms | 24.263 ms | 1.330 ms |
|    GenericStructInt |  54.70 ms |  9.207 ms | 0.505 ms |
|                     |           |           |          |
|    NonGenericString | 190.38 ms | 18.408 ms | 1.009 ms |
|     IComparerString | 230.24 ms | 19.189 ms | 1.052 ms |
|  GenericClassString | 230.90 ms | 24.012 ms | 1.316 ms |
| GenericStructString | 149.49 ms | 10.361 ms | 0.568 ms |

:::details ベンチマークのコード

```cs

[Config(typeof(BenchmarkConfig))]
[GroupBenchmarksBy(BenchmarkLogicalGroupRule.ByCategory)]
public class Benchmark
{
    private Random rnd = new Random(42);
    public const int N = 10_000;
    int[] nums;
    string[] texts;
    public Benchmark()
    {
        texts = new string[N];
        nums = new int[N];
        for (int i = 0; i < nums.Length; i++)
        {
            nums[i] = rnd.Next();
            var chrs = new char[rnd.Next(10, 20)];
            for (int j = 0; j < chrs.Length; j++)
                chrs[j] = (char)(rnd.Next(0, 26) + 'a');
        }
    }

    [Benchmark]
    [BenchmarkCategory("Int")]
    public Array NonGenericInt()
    {
        BubbleSorts.BubbleSortInt(nums, new IntComparerClass());
        return nums;
    }

    [Benchmark]
    [BenchmarkCategory("Int")]
    public Array IComparerClassInt()
    {
        BubbleSorts.BubbleSortIComparer(nums, new IntComparerClass());
        return nums;
    }

    [Benchmark]
    [BenchmarkCategory("Int")]
    public Array IComparerStructInt()
    {
        BubbleSorts.BubbleSortIComparer(nums, new IntComparerStruct());
        return nums;
    }

    [Benchmark]
    [BenchmarkCategory("Int")]
    public Array GenericClassInt()
    {
        BubbleSorts.BubbleSortGeneric(nums, new IntComparerClass());
        return nums;
    }

    [Benchmark]
    [BenchmarkCategory("Int")]
    public Array GenericStructInt()
    {
        BubbleSorts.BubbleSortGeneric(nums, new IntComparerStruct());
        return nums;
    }



    [Benchmark]
    [BenchmarkCategory("String")]
    public Array NonGenericString()
    {
        BubbleSorts.BubbleSortString(texts, StringComparer.Ordinal);
        return texts;
    }

    [Benchmark]
    [BenchmarkCategory("String")]
    public Array IComparerClassString()
    {
        BubbleSorts.BubbleSortIComparer(texts, StringComparer.Ordinal);
        return texts;
    }

    [Benchmark]
    [BenchmarkCategory("String")]
    public Array IComparerStructString()
    {
        BubbleSorts.BubbleSortIComparer(texts, new StringComparerStruct());
        return texts;
    }

    [Benchmark]
    [BenchmarkCategory("String")]
    public Array GenericClassString()
    {
        BubbleSorts.BubbleSortGeneric(texts, StringComparer.Ordinal);
        return texts;
    }

    [Benchmark]
    [BenchmarkCategory("String")]
    public Array GenericStructString()
    {
        BubbleSorts.BubbleSortGeneric(texts, new StringComparerStruct());
        return texts;
    }
}
class IntComparerClass : IComparer<int>
{
    public int Compare(int x, int y) => x.CompareTo(y);
}
struct IntComparerStruct : IComparer<int>
{
    public int Compare(int x, int y) => x.CompareTo(y);
}
struct StringComparerStruct : IComparer<string>
{
    public int Compare(string x, string y) => StringComparer.Ordinal.Compare(x, y);
}
```
:::

ジェネリックに構造体を渡すと型が展開されるため実行時にインライン化がなされて非常に高速に動作します。

この仕様を利用して、`int`でも`BigInteger`でも自作の行列型でも利用できる高速なジェネリックな累乗メソッドを作れます。

```cs
public interface IMultiplicationOperator<T>
{
    T MultiplyIdentity { get; }
    T Multiply(T x, T y);
}

public static class MathLibGeneric
{
    /// <summary>
    /// <paramref name="x"/> の <paramref name="y"/> 乗
    /// </summary>
    /// <remarks>
    /// <para>計算量: O(log <paramref name="y"/>)</para>
    /// </remarks>
    public static T Pow<T, TOp>(T x, long y)
        where TOp : struct, IMultiplicationOperator<T>
    {
        var op = default(TOp);
        T res = op.MultiplyIdentity;
        for (; y > 0; y >>= 1)
        {
            if ((y & 1) == 1)
                res = op.Multiply(res, x);
            x = op.Multiply(x, x);
        }
        return res;
    }
}
```

参考
- https://ufcpp.net/blog/2018/5/metricspace/
- https://github.com/dotnet/csharplang/issues/110
  - 言語仕様に取り入れる動きはあるもののいつになるかは不明

2021/07/23 追記
C# 10でPreviewとして `Static abstract members in interfaces` が入りそうです。
本格採用は C# 11 になる？
https://github.com/dotnet/csharplang/issues/4436