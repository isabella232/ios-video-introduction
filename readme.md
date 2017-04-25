# Building an iOS Video Calling App
<hr/>
**Update for Beta 2**

The new Sinch iOS SDK now supports push and voice calling and video calling in the same SDK. 

This tutorial assumes you've already signed up for a Sinch account and have an application key and secret ready to use. If you don't, you can easily go ahead and follow the steps at the [Sinch website](https://www.sinch.com/).
<hr/>


[WebRTC is on its way to being one of the most influential technologies in the mobile space] (http://www.programmableweb.com/news/webrtc-2015-and-why-apple-will-never-join-party/analysis/2015/08/13). Facebook Messenger, Slack and WhatsApp have already integrated WebRTC into their mobile apps to make this technology mainstream. [Research shows] (http://www.smartinsights.com/mobile-marketing/mobile-marketing-analytics/mobile-marketing-statistics/) that native mobile apps consume 89% of all mobile data. The rise of [WebRTC communications](https://www.sinch.com/products/webrtc/) has enabled another level of communication, with video being the centrepiece allowing users to seamelessly video-chat in any mobile app.


At Sinch we are on a mission to help developers like you to develop the next 'Uber of X'. To help on our mission we are bringing Video Calling to all our users as a Public Beta, now available on iOS, Android and JS (web). To help you get started with WebRTC we have made this simple getting-started guide specifically for integrating Video technology on iOS. Don't be scared, video calling with Sinch is just as easy as making a voice call. If you're interested you can check out how to make voice calls on iOS [here](https://www.sinch.com/tutorials/ios-simple-voice-app-tutorial/).

![](images/screenshot.png)

Firstly we are going to check out what the differences are between making a video call and a voice call with Sinch. A good starting point would be to download the [SDK](https://www.sinch.com/downloads/#videosdk) and check out the example code.

Unfortunately the beta is currently **not** available via cocoa pods. If you are wanting to add the beta to an existing project, you need to do it old-school way and add the required frameworks. At present the Sinch service is not yet supported. Check out the [documentation](https://www.sinch.com/docs/video/ios/) to see how and what you need to add to get started! **(Don't worry, it isn't hard!)**

## Setup
Open up the SinchVideo app which is located in the sample directory, which can be found packaged with the SDK. Have a bit of a look around the project, you will notice it's a similar setup as our voice calling app. We do all the same things to initalize the client. Go ahead and open up **appDelegate.m** and fill the appropriate placeholders with your key and secret:

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
In this sample we also use the active connections in the background, if you would like to use push there is also a Sample project with that to.

If you're new to iOS development, appDelegate.m is the home of app setup and initialisation. The appDelegate file is also where you can make changes to your app based on your app's status. The above code simply sets up the Sinch client, you can find your Sinch application key and secret in the Sinch portal. If you don't currently have an account, [sign up here](www.sinch.com), it's free!

## A new method for calling with Video
If you got the chance to try the first beta SDK you will remember that video calling was a seperate feature. We have gone ahead and consolidated voice and video calling into one SDK. It is now as simple as calling:

```[callClient callUserVideoWithId:self.destination.text];```

**MainViewController.m**


```objectivec
- (IBAction)call:(id)sender {
  if ([self.destination.text length] > 0 && [self.client isStarted]) {
    id<SINCall> call = [self.client.callClient callUserVideoWithId:self.destination.text];
    [self performSegueWithIdentifier:@"callView" sender:call];
  }
}
```

The next difference you will see is in **CallViewController.m**, the first thing you will notice is:

```
- (id<SINVideoController>)videoController {
  return [[(AppDelegate *)[[UIApplication sharedApplication] delegate] client] videoController];
}
```
This method takes care of all the necessary video functionality for you, like disabling the idle timer so the screen doesn't go blank, or providing you with a view of yourself and the remote stream. It also has some fancy gesture recognizers built-in to help you switch cameras and go in and out of fullscreen mode. All these built-in features help you build a polished mobile app, without all the hard work.


## Two views
Within the app we have gone ahead and built in two IBOutlets to display video streams.


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

This is where we add the remote video stream to the main view, as a subview, once the video chat has been established.

## Running the app on your device
If you don't already have an apple developer account, you can go and sign up for one at the [apple developer portal](https://developer.apple.com/). Once you have an account you can login to your account in Xcode. If you are yet to deploy an iOS app from Xcode to your iOS device, you will need to press 'play' in Xcode to run your iOS app and then navigate to 

Settings > General > Device Management > Your Developer Account

on your iOS device. Once there you will be able to allow apps to be deployed and run on your iOS device. 

That's really all there is to it. Download the SDK and give it a spin! 

**Got questions? Just ask!**





