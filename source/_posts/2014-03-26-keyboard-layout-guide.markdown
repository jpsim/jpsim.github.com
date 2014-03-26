---
layout: post
title: "Keyboard Layout Guide"
date: 2014-03-26 14:41
comments: true
categories: ios development opensource github personal projects
---
I really like iOS 7's `topLayoutGuide` and `bottomLayoutGuide`. They're immensely useful. But in my adventures with Auto Layout, I've often wished that Apple had added a third member to this exclusive group: a sort of `keyboardLayoutGuide`.

It's a bit of a drag to have to set up `NSNotification` observers just to be able to keep your textfield on the screen when the keyboard barges into view. So I added a `keyboardLayoutGuide` to `UIViewController` myself. You can check it out on GitHub here: [jpsim/JPSKeyboardLayoutGuide][github].

If you've used Apple's layout guides before, you'll know that they're actually just `id`'s that conform to the [`UILayoutSupport` Protocol][protocol]. So I figured that having the guide just be a zero-sized UIView was probably the best way to go. This way, I can easily add it to the view and bind it to the keyboard frame by modifying a constraint's constant.

In all honesty, I dislike the inheritance approach I took here; it seems like the easy way out. I'm hoping that either myself or a contributor will have a stroke of genius and find a composition-based way to do this. Perhaps a starting point would be a category on `UIViewController` along with an associated object as the `keyboardLayoutGuide` property... But then the `NSNotification`s will be troublesome. If you have an idea, please fork and send a PR or open up an issue on [github][github]!

[github]: https://github.com/jpsim/JPSKeyboardLayoutGuide
[protocol]: https://developer.apple.com/library/ios/documentation/uikit/reference/UILayoutSupport_Protocol/Reference/Reference.html
