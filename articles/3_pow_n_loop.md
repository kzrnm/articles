---
title: 部分集合の部分集合を列挙する方法(3のN乗のループ)
emoji: "➰"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["C#", "dotnet", "C", "cpp", "競技プログラミング"]
published: true
---

# 部分集合の部分集合を $O(3^N)$ で列挙する方法

## ビット列の部分集合を列挙する方法

```cpp
int n = 列挙したいビット列;
for (int sub = n; sub > 0; sub = (sub - 1) & n)
    // sub が部分集合になっている
```

というように 1 引いて元のビット列との AND を取ると部分集合を列挙することができます。

C++ の例

```cpp
#include <iostream>
#include <bitset>
using namespace std;

int main(void){
    int n = 0b10101;
    for (int sub = n; sub > 0; sub = (sub - 1) & n)
    {
        bitset<5> bs(sub);
        cout << bs << endl;
    }
}
```

**実行結果**

```
10101
10100
10001
10000
00101
00100
00001
```

:::details AtCoder での出題例

- https://atcoder.jp/contests/abc187/tasks/abc187_f
  - 提出: https://atcoder.jp/contests/abc187/submissions/48418685
- https://atcoder.jp/contests/abc332/tasks/abc332_e
  - 提出: https://atcoder.jp/contests/abc187/submissions/48418685

:::
### 部分集合の部分集合の列挙

以下のコードは C# で記述していますが C 系の言語ならば同様の記述ができるかと思います。

$N$ 個の要素を持つ集合の部分集合は $2^N$ 個あり、そこからさらに部分集合を取る場合の数は $3^N$ 個あります。

上記の列挙の仕方で if を入れることなく $3^N$ 回の処理が可能です。

```cpp
#include <iostream>
#include <bitset>
using namespace std;

int main(void){
  int size = 1<<3;
  for (int i = 0; i < size; i++)
  {
      bitset<3> bi(i);
      int subCount = 0;
      for (int sub = i; sub > 0; sub = (sub - 1) & i)
      {
        bitset<3> bs(sub);
        if(sub == i) cout << bi;
        else         cout << "   ";
        cout << " → "  << bs << endl;
      }
      {
        // sub=0 の場合が必要なら別途入れる
        int sub = 0;
        bitset<3> bs(sub);

        if(sub == i) cout << bi;
        else         cout << "   ";
        cout << " → "  << bs << endl;
      }
  }
}
```

**実行結果**

```
000 → 000
001 → 001
    → 000
010 → 010
    → 000
011 → 011
    → 010
    → 001
    → 000
100 → 100
    → 000
101 → 101
    → 100
    → 001
    → 000
110 → 110
    → 100
    → 010
    → 000
111 → 111
    → 110
    → 101
    → 100
    → 011
    → 010
    → 001
    → 000
```

### 部分集合の組み合わせのループの場合

`sub` は単調減少、`other` は単調増加する。
`sub` と `other` を区別しない場合は `sub > other` としてよい
`sub` と `other` を区別する場合は「部分集合の部分集合の列挙」と同様に列挙する。

```csharp
#include <iostream>
#include <bitset>
using namespace std;

int main(void){
  int size = 1<<3;
  for (int i = 0; i < size; i++)
  {
      bitset<3> bi(i);
      int subCount = 0;
      for (int sub = i, other = 0; sub > other; sub = (sub - 1) & i, other = ~sub & i)
      {
        bitset<3> bs(sub);
        bitset<3> bo(other);
        if(sub == i) cout << bi;
        else         cout << "   ";
        cout << " → "  << bs << "|" << bo << endl;
      }
  }
}
```

**実行結果**

```
001 → 001|000
010 → 010|000
011 → 011|000
    → 010|001
100 → 100|000
101 → 101|000
    → 100|001
110 → 110|000
    → 100|010
111 → 111|000
    → 110|001
    → 101|010
    → 100|011
```
