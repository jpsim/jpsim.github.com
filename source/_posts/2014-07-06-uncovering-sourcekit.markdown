---
layout: post
title: "Uncovering SourceKit"
date: 2014-07-06 16:07
comments: true
categories: sourcekit swift jazzy xcode ios development
---
To support a [fancy new language][swift], nifty [realtime IDE][playgrounds] features and impressive [cross-language interoperability][interoperability], Apple had to develop several new underlying tools. Here, we'll focus on *SourceKit*, Xcode's under-appreciated sidekick.

![sidekick](/images/posts/sidekick.jpg)
<center><sub>SourceKitSidekick temporarily wearing a cape</sub></center><br/>

## What is SourceKit?

SourceKit is the set of tools that enables most of Swift's source code manipulation features: source code parsing, syntax highlighting, typesetting, autocomplete, cross-language header generation, and lots more.

## Architecture

Xcode traditionally runs its compiler ([Clang][clang]) *in-process*, which means that any time the compiler would crash, so would the IDE.

![house of cards](/images/posts/house_of_cards.jpg)
<center><sub>Xcode architecture diagram</sub></center><br/>

Exacerbating the problem, Xcode can easily invoke the compiler thousands of times to parse, highlight and typeset source code, all before a user ever hits *⌘+B*. That's because unlike most editors (Vim/Sublime/etc), Xcode doesn't use regular expressions to parse source code, but rather Clang's powerful (though much more complex) parser/tokenizer.

Thankfully, Swift in Xcode 6 moves away from this architecture<sup>1</sup>, combining all these source code manipulation features into a separate process that communicates with Xcode through [XPC][xpc]: `sourcekitd`. This XPC daemon is launched whenever Xcode 6 loads any Swift code.

![sourcekit terminated](/images/posts/sourcekit_terminated.jpg)
<center><sub>Life would be miserable if Xcode crashed every time this appeared</sub></center><br/>

## How Xcode uses SourceKit

Since SourceKit is a private and undocumented tool, we need to get a little creative to learn how to use it. By setting the `SOURCEKIT_LOGGING`<sup>2</sup> environment variable, Xcode will log its SourceKit communications to `stdout`, allowing us to monitor its communications in realtime. This is how many of the commands covered in this article were discovered.

## Unified Symbol Resolution

SourceKit uses a Clang feature called the USR (Unified Symbol Resolution) as a unique identifier for a source code token (i.e. class, property, method, etc.). This is what allows you to *⌘+click* any token in Xcode and navigate to its definition. The USR is even more powerful now that it can unify a representation across languages (Swift/ObjC).

![usr](/images/posts/usr.jpg)
<center><sub>The USR at work</sub></center><br/>

To print the USR's from a Swift file (and their locations), you can run the following command:

```
$ xcrun swift-ide-test -print-usrs -source-filename=Musician.swift
10:7 s:C14swift_ide_test8Musician
14:9 s:vC14swift_ide_test8Musician4nameSS
19:9 s:vC14swift_ide_test8Musician9birthyearSu
33:5 s:FC14swift_ide_test8MusiciancFMS0_FT4nameSS9birthyearSu_S0_
33:10 s:vFC14swift_ide_test8MusiciancFMS0_FT4nameSS9birthyearSu_S0_L_4nameSS
33:24 s:vFC14swift_ide_test8MusiciancFMS0_FT4nameSS9birthyearSu_S0_L_9birthyearSu
34:9 s:vFC14swift_ide_test8MusiciancFMS0_FT4nameSS9birthyearSu_S0_L_4selfS0_
34:21 s:vFC14swift_ide_test8MusiciancFMS0_FT4nameSS9birthyearSu_S0_L_4nameSS
35:9 s:vFC14swift_ide_test8MusiciancFMS0_FT4nameSS9birthyearSu_S0_L_4selfS0_
35:26 s:vFC14swift_ide_test8MusiciancFMS0_FT4nameSS9birthyearSu_S0_L_9birthyearSu
```

## Swift*ish* header generation

*⌘+clicking* on a token defined in Objective-C from Swift will cause Xcode to trigger a Swift-like header to be generated. I say Swift-like because this generated file is not valid Swift<sup>3</sup>, but at least displays the Swift syntax equivalent to the Objective-C tokens.

[![Generated Swift Header](/images/posts/generated_swift_header.jpg)](/images/posts/generated_swift_header.jpg)
<center><sub>Left: original Objective-C header. Right: SourceKit-generated Swift-ish version.</sub></center><br/>

## Using SourceKit from the command line

![SourceKit Playground](/images/posts/sourcekit_playground.jpg)

There are 3 main command line tools that allow to interact with SourceKit: `sourcekitd-test`, `swift-ide-test` and `swift`.

I compiled a shell script with documentation that runs through many useful commands like syntax highlighting, interface generation, AST parsing, demangling, and more.

The script is available on GitHub as a [gist][sourcekit-playground].

## 3rd Party Tools Using SourceKit

Because SourceKit lives outside of Xcode, it’s possible to leverage it to build anything from a Swift IDE to a documentation generator.

### jazzy<sup>♪♫</sup>

![jazzy](/images/posts/jazzy.jpg)

[jazzy][jazzy] is a command-line utility that generates documentation for your Swift and Objective-C projects. It uses SourceKit to derive Swift syntax from Objective-C defined tokens (i.e. class, property, method, etc.).

### SwiftEdit

[SwiftEdit][SwiftEdit] is a proof-of-concept editor that supports syntax highlighting for Swift files.

![SwiftEdit](/images/posts/SwiftEdit.png)

## SourceKit & You

We’re just scratching the surface of what’s possible to build with SourceKit. Tools could be made to measure cross-language code coverage, or provide an editor where Objective-C and Swift can be edited simultaneously. Hopefully this article inspires you to build something with SourceKit and improve our tools in the process.

---

*1: Objective-C in Xcode 6 (Beta 2) doesn't use SourceKit at all, keeping Xcode's traditional clang-in-process architecture. I expect this to change before Xcode 6 GM.*

*2: For SourceKit logging, launch Xcode with <sub>`export SOURCEKIT_LOGGING=3 && /Applications/Xcode6-Beta2.app/Contents/MacOS/Xcode`</sub>*

*3: Speculation: I expect private Swift modules to expose public interfaces using a similar syntax once the language has [access control mechanisms][access-control].*

[swift]: http://developer.apple.com/swift
[playgrounds]: https://developer.apple.com/library/prerelease/ios/recipes/xcode_help-source_editor/ExploringandEvaluatingSwiftCodeinaPlayground/ExploringandEvaluatingSwiftCodeinaPlayground.html
[interoperability]: https://developer.apple.com/library/prerelease/ios/documentation/Swift/Conceptual/BuildingCocoaApps/InteractingWithObjective-CAPIs.html
[clang]: http://clang.llvm.org
[xpc]: https://developer.apple.com/library/mac/documentation/macosx/conceptual/bpsystemstartup/chapters/CreatingXPCServices.html
[generate_swift_header]: https://github.com/realm/jazzy/blob/master/bin/generate_swift_header.sh
[sourcekit-playground]: https://gist.github.com/jpsim/13971c81445219db1c63#file-sourcekit_playground-sh
[jazzy]: https://github.com/realm/jazzy
[SwiftEdit]: https://github.com/jpsim/SwiftEdit
[access-control]: https://github.com/ksm/SwiftInFlux#access-control
