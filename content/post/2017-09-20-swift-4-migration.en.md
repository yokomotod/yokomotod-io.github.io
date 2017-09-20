+++
title = "Swift 4 Migration"
date = 2017-09-20T13:56:05+09:00
tags = ["ios", "swift", "xcode"]
+++


## Before Migration

versions:
```
Xcode 8.3.2
Swift 3.2
CocoaPods: 1.3.1
```

Pods:
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

## Upgrade to Xcode9 (with swift3.2)

Don't touch these, yet
```
Swift Conversion
  Convertion to Swift 4 is available
```

```
Validate Project Settings
  Update to recommended settings
```


### Some Pod build will fail (CryptoSwift)

In case of CryptoSwift, it already release swift 4 support version.
Just `bundle update`

### Himotoki

```
'Decodable' is ambiguous for type lookup in this context`
```

https://github.com/ikesyo/Himotoki

> ⚠️ Please note that you should need to add the module name Himotoki to Decodable (Himotoki.Decodable) to avoid type name collision with Foundation.Decodable in Xcode 9 or later. ⚠️

As above notice, replace `Decodable` to `Himotoki.Decodable`.

I faced following error as well, and fixed with `KeyPath` → `Himotoki.KeyPath`.
```
Reference to generic type 'KeyPath' requires arguments in <...>
```


## Upgrade to Swift 4

### Auto Conversion

Execute auto conversion for only own project. (includes Tests, UITests, and Extension but NOT Pods)


According to the document,

https://help.apple.com/xcode/mac/current/#/deve838b19a1?sub=devded6c2001

We can choose from

- Minimize Inference
- Match Swift 3 Behavior

To activate Swift 4's merit, `Minimize Inference`.

It will change
- Some Enums
- Add `@objc` for method which are refered via `#selector()`

Then new warning

```
Dependency Analysis Warning
  The use of Swift 3 @objc inference in Swift 4 mode is deprecated. Please address deprecated @objc inference warnings, test
  your code with “Use of deprecated Swift 3 @objc inference” logging enabled, and then disable inference by changing the
  "Swift 3 @objc Inference" build setting to "Default" for the "Hopping" target.
```

will come.

And some deprecated warning too.

### Update to recommended settings

Execute "Update to recommended settings" on own project, then it will enable some compiler warnings.

Usually, `Pods/` will be excluded from version controll, so we shouldn't execute "Update to recommended settings" for Pod project.

There is an issue on CocoaPods,

https://github.com/CocoaPods/CocoaPods/issues/6863

So we should wait v1.4.0


### `swift_version = '4.0'` in Podfile

Some Pod will build fail at here as well.

In my case, SwiftyRSA failed. They already have `swift4` branch so,
```
  pod 'SwiftyRSA', :git => 'https://github.com/TakeScoop/SwiftyRSA', :branch => 'swift4'
```

## Resolve warnings

### `The use of Swift 3 @objc inference in Swift 4 mode is deprecated.`

In project settings, each targets have `On`, so change it to `Default`.

 `Swift 3 @objc inference`:  `Default`

(refer https://help.apple.com/xcode/mac/current/#/deve838b19a1?sub=devded6c2001 )

### resolve deprecated warnings

In my case, i had only this kind of warning.

```
warning: 'substring(to:)' is deprecated: Please use String slicing subscript with a 'partial range upto' operator.
```

So, change

```
text.substring(to: 3)
text.substring(from: 3)
```

to

```
String(text.prefix(3))
String(text[text.index(text.startIndex, offsetBy: 3)...])
```

## Done !

I feel this time is quite easier, compare to swift 3 migration.
But I will need to adapt iPhone X as well.
