---
layout: post
title: "Building an NFC Music Box"
date: 2023-03-31 11:00
comments: true
categories: development music nfc raspberry-pi python swiftui ios projects
---

My son and I built an NFC-based music player so he can play the music he
wants, develop his own musical taste but most of all so that he and I
could have something fun to build together.

Kind of like a modern day record player, but infinitely more extensible,
and with each album costing 1/100th the price of a vinyl record.

Here it is in action (*music starts 9 seconds into the video, set your
volume to low*):

<iframe title="vimeo-player" src="https://player.vimeo.com/video/813573763?h=6b3ebec0bc" width="640" height="360" frameborder="0" allowfullscreen></iframe>

<br />

And here's how we built it:

## Materials

|<br />|<br />|<br />|<br />|<br />|
|-|-|-|-|-|
|![](/images/posts/musicbox/echo.jpg)|![](/images/posts/musicbox/case.jpg)|![](/images/posts/musicbox/pi.jpg)|![](/images/posts/musicbox/nfc_module.jpg)|![](/images/posts/musicbox/nfc_cards.jpg)|

* [Amazon Echo Dot, $20][dot]:
  For the price, this is a surprisingly hackable, portable speaker. I would have
  prefered something without any microphones at all, but I _mostly_ trust the
  hardware mute switch.
* [Case, $20][case]: Beautiful
  retro-style case for the Raspberry Pi. The top compartment is made to
  house a fan, but this is where I put the NFC module instead. The folks
  who make this at [C4 Labs][c4labs] were [super nice][c4labs_tweet].
  We broke a piece when we started building the box and they sent a
  replacement part right away.
* [Raspberry Pi, $35][pi]:
  I chose to go with a Raspberry Pi because they're cheap, have good
  compatible NFC modules, lots of case options on Etsy and easy to
  develop on.
* [NFC Module, $5][nfc_module]: I can't
  believe this thing is just $5. Works great. Lots of good tutorials on
  connecting this to a Raspberry Pi.
* [NFC Cards, $29][nfc_cards]:
  I got 100 cards at $0.29 each.

In total, this adds up to $109, but in my case I already had a Raspberry
Pi and an Echo Dot lying around that I could repurpose for this, so it
cost me closer to $55.

## Services

I'm already an Apple Music subscriber, so using that as the music source
was the cheapest solution, although this setup would work with any music
service that works with Amazon Echo: Spotify, Amazon Music, TuneIn,
CloudPlayer, Deezer, iHeartRadio.

In the exploration phase for this project, I wanted to buy DRM-free
music, wire a speaker directly to the Raspberry Pi and play it locally.
However, that would have meant that adding music later would be a much
more tedious task. Plus it's hard to find DRM-free music these days, and
when you do find what you want it ends up being pricey. Especially
compared to the convenience and cost of today's streaming options.

I also explored using HomePods as the speakers (see addendum below).

The biggest downside to the current streaming approach is that there's a
5-10 second delay after tapping a card and music starting.

## Assembly

|<br />|<br />|<br />|
|-|-|-|
|![Assembly #1](/images/posts/musicbox/assembly1.jpg)|![Assembly #2](/images/posts/musicbox/assembly2.jpg)|![Assembly #3](/images/posts/musicbox/assembly3.jpg)|
|![Assembly #4](/images/posts/musicbox/assembly4.jpg)|![Assembly #5](/images/posts/musicbox/assembly5.jpg)|![Assembly #6](/images/posts/musicbox/assembly6.jpg)|

<br />

I assembled this with my son over a few days in May 2020, and then
finished the setup in September 2021. The actual hardware assembly
probably only took a combined total of 3 hours though.

## Software

_Disclaimer: This code is rough, it's suitable for a toy project. I'm
sharing it in case it's useful for others getting started with a similar,
project. I'm not claiming this is beautiful quality code._

### Echo Dot Remote Control

I use the [alexa-remote-control][alexa-remote-control] shell script to
control the Echo Dot. When it's up and running, it couldn't be easier:

```shell
$ alexa_remote_control.sh -e "playmusic:APPLE_MUSIC:The Lion King"
$ alexa_remote_control.sh -e pause
$ alexa_remote_control.sh -e play
$ alexa_remote_control.sh -e vol:15
```

### NFC Reader

I use [MFRC522-python](https://github.com/pelwell/MFRC522-python) to
interface with the NFC module.

The Python script that reads cards and plays music looks like this:

```python
import RPi.GPIO as GPIO
import MFRC522
import os
import signal

continue_reading = True
last_seen_card = None
echo_name = "MusicBox Echo"
current_dir = os.path.dirname(os.path.abspath(__file__))
alexa_control_script = os.path.join(current_dir, "alexa_remote_control.sh")

def alexa(command):
    os.system(
        '{alexa_control_script} -d "{echo_name}" -e "{command}"'.format(
            alexa_control_script=alexa_control_script,
            echo_name=echo_name,
            command=command,
        )
    )

def pause():
    alexa("pause")

def play_music(query):
    print(query)
    pause()
    alexa("playmusic:APPLE_MUSIC:{query}".format(query=query))

def set_volume(volume):
    alexa("vol:{volume}".format(volume=volume))

def end_read(signal, frame):
    global continue_reading
    continue_reading = False
    GPIO.cleanup()

signal.signal(signal.SIGINT, end_read)

MIFAREReader = MFRC522.MFRC522()

while continue_reading:
    # Scan for cards
    (status, TagType) = MIFAREReader.MFRC522_Request(MIFAREReader.PICC_REQIDL)

    # If we have a card, continue
    if status == MIFAREReader.MI_OK:
        # Get the UID of the card
        (status, uid) = MIFAREReader.MFRC522_Anticoll()

        # If we have the UID, continue
        if status == MIFAREReader.MI_OK and last_seen_card != uid:
            last_seen_card = uid

            # Print UID
            print("Card UID: %s" % uid)
            if uid == [136, 4, 76, 240, 48]:
                print("PAUSE")
                pause()
            elif uid == [136, 4, 236, 236, 140]:
                set_volume(10)
            elif uid == [136, 4, 247, 42, 81]:
                set_volume(40)
            elif uid == [136, 4, 137, 170, 175]:
                play_music("Star Wars A New Hope Soundtrack")
            elif uid == [136, 4, 148, 191, 167]:
                play_music("Paco de Lucia")
            elif uid == [136, 4, 160, 224, 204]:
                play_music("Dexter Gordon")
            elif uid == [136, 4, 197, 238, 167]:
                play_music("Rainbow Connection")
            elif uid == [136, 4, 2, 220, 82]:
                play_music("Genesis")
            elif uid == [136, 4, 232, 57, 93]:
                # and so on...
```

You'll notice that there are special cards for pausing, setting a high
volume and setting a low volume. The rest of the cards map to music.

The main thing I'd like to improve at some point is to avoid hardcoding
a mapping of the card UIDs to music and instead program it using the
companion iPhone app by writing a payload to the card. The iOS side I
know how to do pretty quickly, but I'd have to spend more time to figure
out how to do it on the MFRC522 reader side of things.

### API Server

There's an API server that can be used to control the music box using
the companion iPhone app.

```python
from flask import Flask, request, jsonify
import os

api = Flask(__name__)
echo_name = "MusicBox Echo"
current_dir = os.path.dirname(os.path.abspath(__file__))
alexa_control_script = os.path.join(current_dir, "alexa_remote_control.sh")

def alexa(command):
    os.system(
        '{alexa_control_script} -d "{echo_name}" -e "{command}"'.format(
            alexa_control_script=alexa_control_script,
            echo_name=echo_name,
            command=command,
        )
    )

@api.route("/pause", methods=["POST"])
def pause():
    alexa("pause")
    return "{}"

@api.route("/play", methods=["POST"])
def play():
    if request.json:
        play_music(request.json["query"])
    else:
        alexa("play")
    return "{}"

@api.route("/play/<query>", methods=["POST"])
def play_music(query):
    print(query)
    pause
    alexa("playmusic:APPLE_MUSIC:{query}".format(query=query))
    return "{}"

@api.route("/volume/<int:volume>", methods=["POST"])
def set_volume(volume):
    alexa("vol:{volume}".format(volume=volume))
    return "{}"

if __name__ == "__main__":
    api.run(host="0.0.0.0")
```

### iPhone App

Of course I made an iPhone app using [SwiftUI][swiftui] &
[Composable Architecture][tca].

This helps me quickly adjust the volume, play/pause and play specific
music. If my kid falls alseep to music, I can stop it without having to
walk into his room.

Siri intents work too, so I can play/pause music by speaking to Siri
without having to launch the app.

![](/images/posts/musicbox/app.jpg)

## Closed Source

Hopefully this post is helpful to someone interested in building
something similar. I've posted enough code and details to get you
started, but I won't be open sourcing the whole project because I'm just
not willing to field support questions or feature requests. I'm happy to
answer questions about my experience building this, but I'm sorry I
can't help you figure out why something's not working if you go build
something similar.

---

## Addendum: HomePod Attempt

At one point I wanted to use a stereo pair of HomePods for this project,
but all my attempts to reverse engineer a way to play music on them from
a Raspberry Pi were fruitless. I tried sniffing the network traffic via
Charles Proxy while playing music from the iOS Music app or even the
Shortcuts app and wasn't able to crack it. I did end up getting it
working using [forked-daap][owntone] but this set the HomePods in a
weird state. I don't remember all the details because I gave up on that
approach for two reasons: the first is that I couldn't get it to work
well and the second is that I wanted to keep the HomePods in our living
room while the music box was meant to be in my son's bedroom.

---

[dot]: https://www.amazon.com/dp/B08YT3BWMP
[case]: https://www.etsy.com/listing/739034156
[pi]: https://www.raspberrypi.org/products/raspberry-pi-4-model-b/
[nfc_module]: https://www.amazon.com/dp/B01CSTW0IA
[nfc_cards]: https://store.gototags.com/nfc-pvc-card-ntag213/
[c4labs]: https://twitter.com/theC4Labs
[c4labs_tweet]: https://twitter.com/simjp/status/1259980961665576960
[swiftui]: https://developer.apple.com/xcode/swiftui/
[tca]: https://github.com/pointfreeco/swift-composable-architecture
[owntone]: https://github.com/owntone/owntone-server
[alexa-remote-control]: https://github.com/thorsten-gehrig/alexa-remote-control
