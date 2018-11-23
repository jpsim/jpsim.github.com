---
layout: post
title: "Evaluating SwiftSyntax for use in SwiftLint"
date: 2018-11-22 14:50
comments: true
categories: development swift swiftlint sourcekit sourcekitten apple opensource projects github
---

**tl;dr; Implementing SwiftLint using SwiftSyntax instead of SourceKitten would make it run over 20x slower 😭**

I have for some time been looking forward to reimplementing some of [SwiftLint](https://github.com/realm/SwiftLint)'s simpler syntax-only rules with [SwiftSyntax](https://github.com/apple/swift-syntax). If you're not familiar with it, the recent [NSHipster article](https://nshipster.com/swiftsyntax/) gives a great overview. My motivation for integrating it into SwiftLint was that it would be nice to use an officially maintained library directly to obtain the syntax tree rather than the open source but community-maintained [SourceKitten](https://github.com/jpsim/SourceKitten) library. I was also under the false impression that SwiftSyntax would be significantly faster than SourceKit/SourceKitten.

SourceKitten gets its syntax tree by dynamically loading [SourceKit](https://github.com/apple/swift/tree/master/tools/SourceKit) and making cross-process XPC calls to a SourceKit daemon. In a typical uncached lint run, SwiftLint spends a significant amount of time waiting on this syntax tree for each file being linted. Because SwiftSyntax is [code-generated](https://github.com/apple/swift-syntax#building-swiftsyntax-from-master) from the same syntax definition files as the Swift compiler, I had (incorrectly) assumed that calculating a Swift file's syntax tree using SwiftSyntax was done entirely in-process by the library, which would have lead to significant performance gains by avoiding the cross-process XPC call made by SourceKitten for equivalent functionality.

In reality, SwiftSyntax delegates all parsing & lexing to the `swiftc` binary, [launching the process](https://github.com/apple/swift-syntax/blob/0.40200.0/Sources/SwiftSyntax/SwiftSyntax.swift#L100-L101), [reading its output from stdout](https://github.com/apple/swift-syntax/blob/0.40200.0/Sources/SwiftSyntax/SwiftSyntax.swift#L102) and [deserializing the JSON response](https://github.com/apple/swift-syntax/blob/0.40200.0/Sources/SwiftSyntax/SwiftSyntax.swift#L103-L104) into its `SourceFileSyntax` Swift type. This is repeated for each file being parsed 😱.

**Launching a new instance of the Swift compiler for each file parsed is orders of magnitude slower than SourceKitten's XPC call to a long-lived SourceKit daemon.**

I discovered this after [reimplementing](https://github.com/realm/SwiftLint/pull/2476) a very simple SwiftLint rule with a SwiftSyntax-based implementation: [Fallthrough](https://github.com/realm/SwiftLint/blob/master/Rules.md#fallthrough). This opt-in rule is a perfect proof-of-concept for integrating SwiftSyntax into SwiftLint because it literally just finds all occurrences of the `fallthrough` keyword and reports a violation at that location. I measured the time it took to lint a folder of ~100 Swift files from Lyft's iOS codebase with only the `fallthrough` rule whitelisted.

```yaml
# .swiftlint.yml configuration file
included:
  - path/to/lint/dir # contains ~100 Swift files
whitelist_rules:
  - fallthrough
```

I compiled both SwiftLint from `master` and again with this `fallthrough-swift-syntax` branch with `swift build -c release` and named the binaries `swiftlint-master` and `swiftlint-swift-syntax`. I then benchmarked both binaries using the excellent [hyperfine](https://github.com/sharkdp/hyperfine) utility.

```shell
$ hyperfine './swiftlint-master lint --quiet --no-cache' './swiftlint-swift-syntax lint --quiet --no-cache'
Benchmark #1: ./swiftlint-master lint --quiet --no-cache
  Time (mean ± σ):     231.3 ms ±   5.1 ms    [User: 130.5 ms, System: 29.2 ms]
  Range (min … max):   224.3 ms … 238.3 ms
 
Benchmark #2: ./swiftlint-swift-syntax lint --quiet --no-cache
  Time (mean ± σ):      5.254 s ±  0.149 s    [User: 20.309 s, System: 23.110 s]
  Range (min … max):    4.839 s …  5.354 s
 
Summary
  './swiftlint-master lint --quiet --no-cache' ran
   22.71 ± 0.82 times faster than './swiftlint-swift-syntax lint --quiet --no-cache'
```

**The SwiftSyntax version was 22x slower than the existing SourceKitten version**

_Note that I ran SwiftLint with its caching mechanism and logging disabled to accurately measure the time it took just to perform the lint, rather than the overhead from logging or skipping the lint entirely by just returning cached results. Although logging only added 3ms to 10ms in my tests._

---

Ultimately, this means SwiftLint will be keeping its SourceKitten-based implementation for the foreseeable future, unless SwiftSyntax removes its reliance on costly compiler invocations and drastically improves its performance. I really hope the Swift team can somehow find a way to move parsing and lexing into SwiftSyntax itself, making the library much more appealing to use.
