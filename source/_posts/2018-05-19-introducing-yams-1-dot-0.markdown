---
layout: post
title: "Introducing Yams 1.0"
date: 2018-05-19 11:40
comments: true
categories: development swift yaml opensource projects github
---

![Yams: A Sweet & Swifty YAML Parser][yams-logo]

[Norio Nomura][norio] and I (ok, honestly mostly Norio 😅) have been building a
Swift library for encoding & decoding [YAML][YAML] for the last 18 months and
it's now stable enough to make a 1.0 release and share with the world.

It's called Yams, you can find it on GitHub at [jpsim/Yams][yams-github] and API
docs are located at [jpsim.com/Yams][yams-api-docs].

You could say it's a Swift binding for [LibYAML][LibYAML] but I'd argue it's
much more than that.

I'm actually very happy with how this library ended up. It plays nicely with
Swift 4's [Codable protocol][codable], meaning that you get fast & type-safe
encoding & decoding that feels right at home in modern Swift.

Here's a simple example of encoding & decoding `Codable` types:

```swift
import Yams

struct S: Codable {
  var p: String
}

let s = S(p: "test")
let encoder = YAMLEncoder()
let encodedYAML = try encoder.encode(s)
encodedYAML == """
  p: test

  """
let decoder = YAMLDecoder()
let decoded = try decoder.decode(S.self, from: encodedYAML)
s.p == decoded.p
```

Alternatively, you can use it to work with Swift scalar & collection types,
which is probably the easiest way to start parsing arbitrary YAML for your
projects. Finally, there's a third mode, which is a Yams-native API that best
translates to how LibYAML works.

This library's been powering a number of popular projects that use YAML for
configuration, like [SwiftLint][SwiftLint], [SwiftGen][SwiftGen],
[XcodeGen][XcodeGen] & used in [SourceKitten][SourceKitten] to parse Swift
Package Manager build manifests. So if you've wanted to add YAML configuration
files to your Swift CLI, or want to interoperate with other tools that process
YAML, I encourage you to give Yams a try.

[yams-logo]: https://raw.githubusercontent.com/jpsim/Yams/master/yams.jpg
[norio]: https://twitter.com/norio_nomura
[yams-github]: https://github.com/jpsim/Yams
[yams-api-docs]: https://jpsim.com/Yams
[YAML]: http://yaml.org
[LibYAML]: https://github.com/yaml/libyaml
[codable]: https://developer.apple.com/documentation/foundation/archives_and_serialization/encoding_and_decoding_custom_types
[SwiftLint]: https://github.com/realm/SwiftLint
[SwiftGen]: https://github.com/SwiftGen/SwiftGen
[XcodeGen]: https://github.com/yonaskolb/XcodeGen
[SourceKitten]: https://github.com/jpsim/SourceKitten
