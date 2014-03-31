---
layout: post
title: "JPSThumbnailAnnotation"
date: 2013-04-22 06:43
comments: true
categories: ios development opensource github personal projects
---
I just released a new iOS component called [JPSThumbnailAnnotation](https://github.com/jpsim/JPSThumbnailAnnotation) which is a great way to display thumbnails on a map.

![JPSThumbnailAnnotation in action](https://github.com/jpsim/JPSThumbnailAnnotation/raw/master/screenshots.jpg)

This component was actually originally built at [MBS](http://mgn.tc) for the ScalaOne iPhone app (now called TypesafeCon). You can find the original source [here](https://github.com/magneticbear/scalaone_iphone).

After seeing [Sam Vermette](http://samvermette.com)'s talk on Saturday at [NSNorth](http://nsnorth.ca) on Open Source and his philosophy towards it, I decided to go back and extract this little component. It was built pretty quickly at the time, so it was very tightly coupled to the app itself. So I abstracted it a bit, and chose to release it as an individual component. This should make it easier for other iOS developers to integrate this component into their own map apps.

Here's to hoping we find a few things at [Dashbook](http://dashbook.co) that we can release in the open in the near future!
