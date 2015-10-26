# Building an iOS Video Calling App

[Mobile is where WebRTC will make the most impact] (http://www.programmableweb.com/news/webrtc-2015-and-why-apple-will-never-join-party/analysis/2015/08/13). Apps like Facebook Messenger, Slack and WhatsApp have already integrated WebRTC into mobile, and accordingly, [research shows] (http://www.smartinsights.com/mobile-marketing/mobile-marketing-analytics/mobile-marketing-statistics/) that app usage dominates mobile data consumption completely (with 89%). The rise of [WebRTC communications](https://www.sinch.com/products/webrtc/) will allow for richer communication - and that is where video comes in.


On the mission of helping you guys become the Uber of X, we're now introducing Video Calling as Public Beta for iOS/Android and JS. As usual, we have focused on how to make it as easy as possible to make a video call. In fact, it's just like making a voice call with the Sinch client. If you want to learn how to set it up from scratch you can look [here](https://www.sinch.com/tutorials/ios-simple-voice-app-tutorial/).

![](images/screenshot.png)

We're going to look at what makes a video call different from regular voice calling. Let's dive in to the code from the sample app we ship with the [SDK](https://www.sinch.com/downloads/#videosdk).

It's worth noting that the beta is **not** available via cocoa pods, so if you want to add the beta to an existing project, you need to do it old school by hand and add the required frameworks. Also, the Sinch service we provide is not supported yet. Check out the [documentation](https://www.sinch.com/docs/video/ios/) to see the exact things to add!

## Setup
Open up the SinchVideo located in the samples directory. When you look around, it's the same setup as with our Voice calling. Open up the **appDelegate.m** and add your key/secret in:

```
- (void)initSinchClientWithUserId:(NSString *)userId {
  if (!_client) {
    _client = [Sinch clientWithApplicationKey:@"<APPLICATION KEY>"
                            applicationSecret:@"<APPLICATION SECRET>"
                              environmentHost:@"sandbox.sinch.com"
                                       userId:userId];
    _client.delegate = self;
    [_client setSupportCalling:YES];
    [_client setSupportActiveConnectionInBackground:YES];
    [_client start];
    [_client startListeningOnActiveConnection];
  }
}
```

Until you look in **CallViewController.m**, the first thing you will notice is:

```
- (id<SINVideoController>)videoController {
  return [[(AppDelegate *)[[UIApplication sharedApplication] delegate] client] videoController];
}
```
This takes care of all the Video stuff for you, like disabling the idle timer so the screen doesn't go blank, or providing you with a view of yourself and the remote stream. 

## Two views
The next difference is that you have two views as IBOutlets where we will show the video. 


```
- (void)viewDidLoad {
  [super viewDidLoad];
  if ([self.call direction] == SINCallDirectionIncoming) {
    [self setCallStatusText:@""];
    [self showButtons:kButtonsAnswerDecline];
    [[self audioController] startPlayingSoundFile:[self pathForSound:@"incoming.wav"] loop:YES];
  } else {
    [self setCallStatusText:@"calling..."];
    [self showButtons:kButtonsHangup];
  }

  [self.localVideoView addSubview:[[self videoController] localView]];

  [self.localVideoFullscreenGestureRecognizer requireGestureRecognizerToFail:self.switchCameraGestureRecognizer];
  [[[self videoController] localView] addGestureRecognizer:self.localVideoFullscreenGestureRecognizer];
  [[[self videoController] remoteView] addGestureRecognizer:self.remoteVideoFullscreenGestureRecognizer];
}

- (IBAction)onSwitchCameraTapped:(id)sender {
  AVCaptureDevicePosition current = self.videoController.captureDevicePosition;
  self.videoController.captureDevicePosition = SINToggleCaptureDevicePosition(current);
}
- (IBAction)onFullScreenTapped:(id)sender {
  UIView *view = [sender view];
  if ([view sin_isFullscreen]) {
    view.contentMode = UIViewContentModeScaleAspectFit;
    [view sin_disableFullscreen:YES];
  } else {
    view.contentMode = UIViewContentModeScaleAspectFill;
    [view sin_enableFullscreen:YES];
  }
}

```

As soon as you load this view, the localStream is added to it. This is important because you want to make sure you look your best when the other party picks up.

There is also a gestureRecognizer to enable you to switch to full screen, and one to toggle front or back facing camera.

## Remote video
The next key part in the call is this:

```
- (void)callDidAddVideoTrack:(id<SINCall>)call {
  [self.remoteVideoView addSubview:[[self videoController] remoteView]];
}
```

This is where we add remote video to the view.

That's really all there is to it. Download the SDK and give it a spin! Fingers crossed, you'll get rich tryin.






