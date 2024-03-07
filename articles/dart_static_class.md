---
title: "Dart で static しか扱わないクラスを作る"
emoji: "🗃️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: 
 - "dart"
published: true
---

# Dart で static しか扱わないクラスを作る

表題の通りです。

`abstract final class` でインスタンス化不可になります。

`abstract` は継承しないとインスタンス不可、`final` は継承不可を表すので理にかなっていますね。

```dart
abstract final class Converter {
  static double hourToSecond(double hour) => hour * 3600;
}

main() {
  print(Converter.hourToSecond(3));
  Converter(); // エラー: Abstract classes can't be instantiated. Try creating an instance of a concrete subtype.
}
```

調べたところ、https://github.com/dart-lang/language/issues/2270#issuecomment-1485824233 で開発メンバーからの言及もありました。

たとえば、[Flutter の Colors](https://github.com/flutter/flutter/blob/f677027655ede052a268a900c559f5e1556652b2/packages/flutter/lib/src/material/colors.dart#L270) でもこの手法が使われています。
