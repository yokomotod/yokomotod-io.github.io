+++
title = "Swift 4 マイグレーション、またはXcode9対応 メモ"
date = 2017-09-20T13:56:05+09:00
tags = ["ios", "swift", "xcode"]
+++

beta時代はサボっていたけど、正式版も出たことだし恒例のSwiftマイグレーションやったのでログ。

## 開始前

各種バージョン
```
Xcode 8.3.2
Swift 3.2
CocoaPods: 1.3.1
```

使っているPod
```
  pod 'APIKit'
  pod 'Crashlytics'
  pod 'CryptoSwift'
  pod 'Fabric'
  pod 'Firebase/Core'
  pod 'Google/Analytics'
  pod 'GoogleIDFASupport'
  pod 'Himotoki'
  pod 'PureLayout'
  pod 'SDWebImage'
  pod 'SVProgressHUD'
  pod 'SwiftyRSA'
  pod 'TTTAttributedLabel'
```

## Xcode9 に上げる (まだSwift3.2)

```
Swift Conversion
  Convertion to Swift 4 is available
```

```
Validate Project Settings
  Update to recommended settings
```

が出てるけどまだ触らない

### この時点で一部のPodでビルドがコケる。 (CryptoSwiftがコケた)

CryptoSwiftの場合、最新版に上げるとswift4対応済みだったので単純に `bundle update`

### Himotoki

```
'Decodable' is ambiguous for type lookup in this context`
```

https://github.com/ikesyo/Himotoki

> ⚠️ Please note that you should need to add the module name Himotoki to Decodable (Himotoki.Decodable) to avoid type name collision with Foundation.Decodable in Xcode 9 or later. ⚠️

に注意書きがあるとおり、 `Decodable` を `Himotoki.Decodable` に書き直す。

```
Reference to generic type 'KeyPath' requires arguments in <...>
```

というエラーもあり、こちらは `KeyPath` → `Himotoki.KeyPath` で直った。

## Swift 4 に移行

### Auto Conversion

自分のプロジェクトに自動変換を当てる。

https://help.apple.com/xcode/mac/current/#/deve838b19a1?sub=devded6c2001


にあるように

- Minimize Inference
- Match Swift 3 Behavior

が選べる。Swift 4のメリットを享受するために `Minimize Inference` 。

自動置換の対象は、自身のターゲットのみ。(Tests, UITestsやExtension用のターゲット含む)

問題なく自動変換され、ビルドも通った。変更されたのは

- 一部Enum系が置き換わる
- `#selector()` で参照しているメソッドに `@objc` が付与される

```
Dependency Analysis Warning
  The use of Swift 3 @objc inference in Swift 4 mode is deprecated. Please address deprecated @objc inference warnings, test
  your code with “Use of deprecated Swift 3 @objc inference” logging enabled, and then disable inference by changing the
  "Swift 3 @objc Inference" build setting to "Default" for the "Hopping" target.
```

と出るようになる。

あとSwift4でdeprecatedになったメソッドの警告も出始める。

### Update to recommended settings

メインプロジェクトで "Update to recommended settings" 実行。
各種コンパイル時の警告系が有効化される。

`Pods` プロジェクトについては、普通バージョン管理から除外しているので、
"Update to recommended settings" を実行するのはあまりよくない。

https://github.com/CocoaPods/CocoaPods/issues/6863

にissueが出ていて、1.4.0で最初から推奨される設定で`Pods.xcodeproj`が生成されるようになりそうなので、それまでは我慢。


### Podfile で `swift_version = '4.0'`

ここでもPodによっては互換性が無くてビルドがコケる。
自分のところだとSwiftyRSAがコケた。`swift4` ブランチが作られているので
```
  pod 'SwiftyRSA', :git => 'https://github.com/TakeScoop/SwiftyRSA', :branch => 'swift4'
```
に変更

## 警告潰し

### `The use of Swift 3 @objc inference in Swift 4 mode is deprecated.`

プロジェクト設定で、各ターゲットが`On`になっているので、`Default` に変更。

 `Swift 3 @objc inference`:  `Default`

https://help.apple.com/xcode/mac/current/#/deve838b19a1?sub=devded6c2001
の5.の指示。

### deprecated潰し

手元ではそんなに多くなく

```
warning: 'substring(to:)' is deprecated: Please use String slicing subscript with a 'partial range upto' operator.
```

のみだった。

```
text.substring(to: 3)
text.substring(from: 3)
```
を
```
String(text.prefix(3))
String(text[text.index(text.startIndex, offsetBy: 3)...])
```
に修正。

## 完了

Swift3のときに比べたら全然すぐ終わった気分。
ただし別途 iPhoneX の対応は必要そう。