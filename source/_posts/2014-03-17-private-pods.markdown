---
layout: post
title: "Private Pods"
date: 2014-03-17 19:12
comments: true
categories: cocoapods ios development
---
Keeping a project modular is a lofty software development goal that we all strive for even though it can never be fully attained.

One way to encourage a modular architecture is to use a dependency manager like [Cocoapods](http://cocoapods.org). I've blogged about using Cocoapods [before](/cocoapods-tips-tricks), but I didn't touch on splitting up portions of your code that wouldn't benefit from being open-sourced. Either because they're too application-specific, or they're proprietary.

For example, say you were building an application backed by a private web API, that needed to persist data to disk. A naïve implementation (*read what **most** of us would do, including myself*) would combine UI, API and persistence all in one project/workspace. In a fully modular app, UI/API/persistence would all have some degree of separation.

Private pods go a long way to help split up your codebase into logical modules. This becomes especially valuable in medium-large projects with 10k+ lines of code.

There are two main ways to make a private pod: remote and local.

### Remote

You can either create a Specs-like private repo as indicated in [this official Cocoapods guide](http://guides.cocoapods.org/making/private-cocoapods.html) or you can host your `.podspec` online and reference it directly in your `Podfile`.

Here's how you'd reference it directly in a `Podfile`:

```ruby
pod 'PrivatePod', :podspec => 'https://github.com/jpsim/JPSDisplayLink/raw/master/JPSDisplayLink.podspec'
```

### Local

Though I find it unlikely, it might make sense to keep your private pods in the same repo as your project. In which case, you'd commit all its code in a special folder (i.e. `local_pods/POD_NAME`) and you'd specify the podspec path in your `Podfile`.

```ruby
pod 'PrivatePod', :path => 'local_pods/PrivatePod'
```

In this case, the `.podspec` would be located at `local_pods/PrivatePod/PrivatePod.podspec`.

*Note: I've had inconsistent results writing local podspecs. Specs that worked perfectly in a remote repo didn't work at all locally for no apparent reason. Your mileage may vary, I might have made a stupid mistake.*

## Word of caution

Splitting up your codebase is not something you should do for its own sake. The additional overhead probably won't be worthwhile in smaller projects. But the larger the project, the bigger the gain.

On that note, I hope this article helps you decide how you want to split your monolithic 300k line app. But hey, you could always build static libraries instead (*cue laughter*).
