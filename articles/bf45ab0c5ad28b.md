---
title: "[C#]パターンマッチングはswitch式でもswitch文でも使える"
emoji: "📏"
type: "tech"
topics:
  - "dotnet"
  - "csharp"
published: true
published_at: "2021-05-25 01:51"
---

すっかり勘違いしていたのですが、C#のパターンマッチングはswitch式だけではなくswitch文でも使えます。

ということで、以下のコードはC# 9.0 の有効なコードです。

```csharp
object Switch式(IList<int> collection, int num) =>
    (num, collection) switch
    {
        (_, int[] array) when array.Length > 0 => array[0],
        ( < 0, { Count: >= 1 } or { IsReadOnly: true }) => (0, 1),
    };

void Switch文(IList<int> collection, int num)
{
    switch ((num, collection))
    {
        case (_, int[] array) when array.Length > 0:
            Console.WriteLine(array[0]);
            break;
        case ( < 0, { Count: >= 1 } or { IsReadOnly: true }):
            Console.WriteLine((0, 1));
            break;
    }
}
```