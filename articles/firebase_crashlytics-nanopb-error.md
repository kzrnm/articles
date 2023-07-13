---
title: "[Flutter]firebase_crashlytics が使えない問題への対処"
emoji: "🐛"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: 
  - Flutter
  - Firebase
published: true
---

# Flutter の firebase_crashlytics でエラーがおきる

Flutter アプリで `firebase_crashlytics` をビルドしようとすると

```
Swift Compiler Error (Xcode): Include of non-modular header inside framework module
'FirebaseSessions.FIRSESNanoPBHelpers': './nanopb/pb.h'
/Desktop/nanopb_bug_app/ios/Pods/FirebaseSessions/FirebaseSessions/SourcesObjC/NanoPB/FIRSESNanoP
BHelpers.h:28:8
```

のようなエラーが起きたりしました。

しかもビルドする環境ごとにエラーが起きたり起きなかったりしています。

[firebase-ios-sdk の Crashlytics](https://github.com/firebase/firebase-ios-sdk) が依存している `nanopb` が framework module として認識されていないようです。

下記のようにしばらく前から出ているようです。

https://github.com/firebase/flutterfire/issues/10621

## 原因はわからず

3日かけて調べましたが全くわかりませんでした。

## 対処方法

[issue にもコメントしました](https://github.com/firebase/flutterfire/issues/10621#issuecomment-1634522430) がここにも記載しておきます。


1. まず `ios/nanopb` フォルダを作成
2. [nanopb@0.3.9.9](https://github.com/nanopb/nanopb/releases/tag/0.3.9.9) と [nanopb-podspec](https://github.com/google/nanopb-podspec) の中身を `ios/nanopb` にコピー
3. Podfile の `use_modular_headers!` の下に `pod 'nanopb', :path => './nanopb'` と追記

これでローカルに保存した `nanopb` を使ってビルドするようになります。そうすると framework module として認識されるのかビルドできるようになりました。

なんで直るのか全くわかってませんが、公式対応されるまではこれでしのげるかと思います。