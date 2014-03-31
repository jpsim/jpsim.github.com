---
layout: post
title: "JPSVolumeButtonHandler"
date: 2014-02-19 14:38
comments: true
categories: ios development opensource github personal projects
---
The fun times building a [tiny open source class](https://github.com/jpsim/JPSVolumeButtonHandler) to use an iOS device's hardware volume buttons in 3rd party apps.

This wasn't *too* hard, but the I thought the process merited a short blog post.

I recently made a pixel-for-pixel clone of Apple's iOS 7 `UIImagePickerController` (the camera portion, not the library portion). The full code can be found on [github](https://github.com/jpsim/JPSImagePickerController).

My motivation for doing this was that I was working on a client app that included functionality to take a picture and analyze it to determine whether or not it passed certain quality checks (legibility, resolution, completeness, etc). I wanted to display a message based on the result of this quality check during the "review" process of `UIImagePickerController`.

![JPSImagePickerController in action](https://raw.github.com/jpsim/JPSImagePickerController/master/screenshots.png)

The part I want to discuss is the code that uses the phone's hardware volume up button to snap a picture.

There are a few hurdles to jump over to make this seamless:

1. There's no official API or notification to observe the audio volume or hardware volume button presses
2. `UIImagePickerController` doesn't pass-through volume button presses to the system audio
3. There's no official way to stop the system audio from being modified on a volume button press
4. The only way to disable the HUD that appears on volume change is to display an `MPVolumeView` (it won't work if it's hidden or if its alpha is zero)
5. The only way to set the system audio level is to use a deprecated method (i.e. if we want to undo a volume change caused by a volume button press)
6. The system audio volume won't go up if it's already set to 1 and won't go down if it's set to 0. This means that button presses won't be registered if audio is at its maximum or its minimum.

So I wrote a class called [`JPSVolumeButtonHandler`](https://github.com/jpsim/JPSVolumeButtonHandler) that solves the previous problems in the following way:

1. Create an off-screen `MPVolumeView`
2. If the volume is at 1, set it to `0.99999f`, if it's at 0, set it to `0.00001f`, otherwise do nothing
3. KVO observe the `outputVolume` property of an `AVAudioSession`
4. Set the `AVAudioSession`'s category to `AVAudioSessionCategoryAmbient` to not duck any other volume
5. Trigger up/down code blocks when KVO notifies us of a change
6. Reset the system audio to its initial value when KVO notifies us that it has changed
7. Discard KVO notifications when the new volume value is the same as our initial value, since that means we reset the volume

Hopefully either the [image picker clone](https://github.com/jpsim/JPSImagePickerController), or this [volume button class](https://github.com/jpsim/JPSVolumeButtonHandler) ends up being useful to a few people.

Thanks for reading!
