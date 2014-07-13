<!--
.. title: AVAudioPlayer does not Play Sound File
.. slug: avaudioplayer-does-not-play-sound-file
.. date: 2014-07-08 03:30:25 UTC+05:00
.. tags: iOS, Programming
.. link:
.. description:
.. type: text
-->

Just wasted three hours on this, due to a bug in ARC, AVAudioPlayer won't play any sounds unless a strong reference to it is stored somewhere. So, this doesn't work:

```objectivec
NSString *azaanPath = [[NSBundle mainBundle] pathForResource:@"azaan-alaqsa_short" ofType:@"caf"];
NSURL *azaanURL = [NSURL fileURLWithPath:azaanPath];
NSError *error = nil;
// note that AVAudioPlayer is a local variable in the statement below
AVAudioPlayer *audioPlayer = [[AVAudioPlayer alloc] initWithContentsOfURL:azaanURL error:&error];
// check and handle error
[audioPlayer play];
```

but this works:

```objectivec
NSString *azaanPath = [[NSBundle mainBundle] pathForResource:@"azaan-alaqsa_short" ofType:@"caf"];
NSURL *azaanURL = [NSURL fileURLWithPath:azaanPath];
NSError *error = nil;
// note that AVAudioPlayer is an ivar in the statement below
self.audioPlayer = [[AVAudioPlayer alloc] initWithContentsOfURL:azaanURL error:&error];
// check and handle error
[self.audioPlayer play];
```

this has been a known issue since at least 2012, Apple has been busy developing Swift.
