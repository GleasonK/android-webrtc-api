# PubNub Android WebRTC Signaling API

[![Android
Arsenal](https://img.shields.io/badge/Android%20Arsenal-android--webrtc--api-green.svg?style=flat)](https://android-arsenal.com/details/1/2259)

![cover_img](http://kevingleason.me/android-webrtc-api/assets/PnWebRTC.png)

![version](https://img.shields.io/badge/Current%20Version-1.0.6-green.svg?style=flat)

PnWebRTC is an Android module that makes WebRTC signaling easy!

__[View the official PnWebRTC JavaDoc here.][JavaDoc]__

___NOTE:___ This API uses PubNub for signaling to transfer the metadata and establish the peer-to-peer connection. Once the connection is established, the video and voice runs on public Google STUN/TURN servers.

Keep in mind, PubNub can provide the signaling for WebRTC, and requires you to combine it with a hosted WebRTC solution. For more detail on what PubNub does, and what PubNub doesn’t do with WebRTC, check out this article: https://support.pubnub.com/support/solutions/articles/14000043715-does-pubnub-provide-webrtc-and-video-chat-

## Usage instructions

You have two options, the first involves compiling your own binaries, and the second uses the hosted library from Pristine. I strongly recommend you take the second path since it is much quicker and cleaner.

## Compiling your own WebRTC binaries

The PubNub Android WebRTC Signaling API was compiled using [Pristine's](https://pristine.io/) hosted WebRTC library. If you wish to compile your own WebRTC binaries, follow [this guide][NativeAndroid]. Then, clone this repository and import it as a module. You will have to modify the module's `build.gradle` and use your own version codes.

## Using Pristine's WebRTC binaries


When getting started I recommend this method, it is quick and Pristine keeps up to date with their WebRTC libraries. 

### Permissions and Dependencies

In your application's `build.gradle`, you will first need to include a few dependencies. First, the WebRTC library from Pristine. Second, include the PubNub Signaling Library. Optionally, you may include the PubNub Android SDK. I recommend including it as it is useful in messaging, presence, and signaling features.

```gradle
dependencies {
    ...
    compile 'io.pristine:libjingle:9694@aar'
    compile 'me.kevingleason:pnwebrtc:1.0.6@aar'
    compile 'com.pubnub:pubnub-android:3.7.4'    //optional
}
```

Then, in your application's `AndroidManifest.xml` you will need to grant the following permissions for both PubNub and WebRTC to function properly.

```xml
<!-- WebRTC Dependencies -->
<uses-feature android:name="android.hardware.camera" />
<uses-feature android:name="android.hardware.camera.autofocus" />
<uses-feature android:glEsVersion="0x00020000" android:required="true" />
<uses-permission android:name="android.permission.CAMERA" />
<uses-permission android:name="android.permission.RECORD_AUDIO" />
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.MODIFY_AUDIO_SETTINGS" />

<!-- PubNub Dependencies -->
<!--<uses-permission android:name="android.permission.INTERNET" />-->
<uses-permission android:name="android.permission.WAKE_LOCK" />
<uses-permission android:name="com.google.android.c2dm.permission.RECEIVE" />
<permission android:name="your.package.name.permission.C2D_MESSAGE" android:protectionLevel="signature" />
<uses-permission android:name="your.package.name.permission.C2D_MESSAGE" />
```

### Using PnWebRTC in an Activity

__PnWebRTC__ is a signaling service, meaning you will have to gather video and audio resources separately using the WebRTC Android SDK, but this is easy. In your video chatting activity initialize the PeerConnectionFactory with your application context and some options. With these configurations set, we can create an instance of a `PeerConnectionFactory` to create audio and video tracks.

```java
PeerConnectionFactory.initializeAndroidGlobals(
        this,  // Context
        true,  // Audio Enabled
        true,  // Video Enabled
        true,  // Hardware Acceleration Enabled
        null); // Render EGL Context

PeerConnectionFactory pcFactory = new PeerConnectionFactory();
```

These globals effect the `PnPeerConnectionClient` as well, so set them before instantiating your `PnWebRTCClient`. 

#### PnWebRTCClient Constraints and Settings

__PnWebRTCClient__ contains everything you will need to develop video chat applications. This class has all the functions for signaling with WebRTC protocols, including [SDP Offer Options][OfferOptions] known as `MediaConstraints`. You are allowed to define custom `MediaConstraints` for the Client, and default values are used if you do not. The default values look as follows:

| PeerConnection Constraint | Value |
|---------------------------|-------|
| DtlsSrtpKeyAgreement      | true  |
| OfferToReceiveAudio       | false | 
| OfferToReceiveVideo       | true  |

| Video Constraint     | Value |
|----------------------|-------|
| maxWidth             | 1280  |
| maxHeight            | 720   |
| minWidth             | 640   |
| minHeight            | 480   |

| Audio Constraint     | Value |
|----------------------|-------|
| None                 |       |

__Optional:__ To create your own constraints, use `PnRTCClient.setSignalParams(PnSignalingParams params)`

```java
MediaConstraints videoConstraints = new MediaConstraints();
videoConstraints.mandatory.add(
    new MediaConstraints.KeyValuePair("maxWidth", "1280"));
...
PnSignalingParams params = new PnSignalingParams(pcConstraints, videoConstraints, audioConstraints);
pnRTCClient.setSignalParams(params);
```

__PnSignalingParams__ holds all the constraints for a [PeerConnection], Video, and Audio, as well as the list of [ICE Servers]. To set up custom ICE servers, you may instantiate your `PnSignalingParams` using a `List<IceServer>`. 

_Note: All arguments to PnSignalingParams may be null. A null value will simply use the default constraint/ice server._

#### PnRTCListener Callbacks

__PnRTCListener__ is an abstract class that should be extended to implement all desired WebRTC callbacks. This is what connects and powers your application. The callbacks that are defined in the PubNub Signaling API are:

| Listener Callback   | Description |
|---------------------|--------|
| onCallReady(String callId) | Called when you are ready to receive a WebRTC connection. |
| onConnected(String userId) | Called when you have successfully subscribed to a PubNub channel and are ready to receive a WebRTC connection. |
| onPeerStatusChanged(PnPeer peer) | Called whenever a PnPeer's connection status is changed. Can be `CONNECTING`, `CONNECTED`, or `DISCONNECTED`. |
| onPeerConnectionClosed(PnPeer peer) | Called when a PnPeer closes the WebRTC connection. |
| onLocalStream(MediaStream localStream) | Called when local MediaStream is ready and attached using `PnRTCClient#setLocalStream`. |
| onAddRemoteStream(MediaStream remoteStream, PnPeer peer)| Called when a remote stream is added, after being connected to a Peer. |
| onRemoveRemoteStream(MediaStream remoteStream, PnPeer peer)| Called when a peer removes their MediaStream. |
| onMessage(PnPeer peer, Object message)| Called when a user message is transmitted using `PnRTCClient#transmitUser` or `PnRTCClient#transmitAll`. For chat and utility. |
| onDebug(PnRTCMessage message) | Called throughout signaling procedures, helpful for finding WebRTC related errors. |

The best way of extending this is using a private class nested in your video activity that extends `PnRTCListener` and implements the callbacks as you app requires. 

```java
private class DemoRTCListener extends PnRTCListener {
    @Override
    public void onLocalStream(final MediaStream localStream) {
        VideoChatActivity.this.runOnUiThread(new Runnable() {
            @Override
            public void run() {
                localStream.videoTracks.get(0).addRenderer(new VideoRenderer(localRender));
            }
        });
    }

    @Override
    public void onAddRemoteStream(final MediaStream remoteStream, final PnPeer peer) {
        // Handle remote stream added
    }

    @Override
    public void onMessage(PnPeer peer, Object message) {
        /// Handle Message
    }

    @Override
    public void onPeerConnectionClosed(PnPeer peer) {
        // Quit back to MainActivity
        Intent intent = new Intent(VideoChatActivity.this, MainActivity.class);
        startActivity(intent);
        finish();
    }
}
```

Then add the listener to your PnRTCClient using

```java
pnRTCClient.attachRTCListener(new DemoRTCListener());
```

It is important that you attach the callbacks to your `PnRTCClient` before creating connections or attaching local media stream since default callbacks _(none)_ are used prior to calling `attachRTCListener`.

#### Using Video and Audio Tracks

With WebRTC, you share create a [MediaStream] and attach it to your `PeerConnection`s to begin chatting. You can access camera/mic resources using the default or custom constraints you defined in the `PnSignalingParams` of your `PnRTCClient`. Once you have gathered the desired video and audio tracks, you can create a MediaStream and attach it to the `PnRTCClient`.

```java
// Ex: Create a Video Source, then we can make a Video Track to stream
localVideoSource = pcFactory.createVideoSource(capturer, this.pnRTCClient.videoConstraints());
VideoTrack localVideoTrack = pcFactory.createVideoTrack(VIDEO_TRACK_ID, localVideoSource);

// Note that LOCAL_MEDIA_STREAM_ID can be any string
MediaStream mediaStream = pcFactory.createLocalMediaStream(LOCAL_MEDIA_STREAM_ID);
mediaStream.addTrack(localAudioTrack);
pnRTCClient.attachLocalMediaStream(mediaStream);
```

This will trigger you `PnRTCListener`'s `onLocalStream` callback, where you should display it on a 	`GLSurfaceView`.

#### Listening for a connection

All your resources, callbacks, and configurations are now set up. You are ready to listen for a WebRTC connection. This is simply a [PubNub Subscribe][PN Android] which will configure a WebRTC PeerConnection and handle all SDP signaling.

```java
pnRTCClient.setMaxConnections(1);
pnRTCClient.listenOn("Username");
```

This will only allow one incoming `PeerConnection` at a time, meaning this code could be used for 1-to-1 connections. If you don't call `setMaxConnections`, your app will allow all connections. Then, `listenOn` will subscribe you to your call channel so that you may begin receiving calls. It is important to note that the Username you listen on should be unique, much like a phone number. Now we are ready to place a call.


#### Creating Peer Connections

To create a connection with a Peer, they must first be listening on that channel, e.g. they have called `PnRTCClient#listenOn(String user)`. You can then create a connection with that username by calling `PnRTCClient#connect(String user)`.

```java
pnRTCClient.connect("Username");
```
	
In practice, this will only be used to create the RTC PeerConnection. All other signaling to get to that point from other activities or fragments should probably be coordinated by a separate `Pubnub` instance, possibly using push notifications. My practice has been reserving the `-stdby` suffix from usernames, and using it as the standby pub/sub channel for each user. A user will be subscribed to their `-stdby` channel in all other activities until it is time to create a `PeerConnection` to another user.

#### Ending Peer Connections

To close a connection you have two options. One being the classing phone hangup where a user is done talking and thus all connections to and from that user are closed. Use `PnRCTClient#closeAllConnection()` to accomplish the classic hangup. The other is to close a connection with a single Peer. This feature would use `pnRTCClient#closeConnection("Username")`.

```java
// Close All Connections
pnRTCClient.closeAllConnections();

// Close a Single Connection
pnRTCClient.closeConnection("Username");
```
	
Both will disconnect the [PeerConnection] and send a hangup signal via a [PubNub Publish][PN Android].

#### Sending User Messages

There are methods of `PnRTCClient` that allow you to publish a message to all/some users you are currently connected with. If you wish every user in your chat to receive the message, use `PnRTCClient#transmitAll(JSONObject message)`. If you intend the message to go to a single user, call `PnRTCClient#transmit(String user, JSONObject message)` instead.

```java
JSONObject message = new JSONObject();
message.put("name",    "Kevin Gleason");
message.put("message", "Hello WebRTC");
pnRTCClient.transmitAll(message);
// pnRTCClient.transmit("Username",message)
```

#### On Pause/Resume

Common practice for WebRTC applications involves stopping your `MediaStream` when you background the app. To do this, override `onPause` and pause the `GLSurfaceView` from rendering video, then stop your `VideoSource`.

```java
@Override
protected void onPause() {
    super.onPause();
    this.videoView.onPause();     // GLSurfaceView
    this.localVideoSource.stop(); // VideoSource
}
```  
    
To start both these functionalities back up, also override `onResume` to resume the `GLSurfaceView` and `VideoSource`.

```java
@Override
protected void onResume() {
    super.onResume();
    this.videoView.onResume();        // GLSurfaceView
    this.localVideoSource.restart(); // VideoSource
}
```
    
#### On Destroy

Finally, you should override `onDestroy` to properly prepare all objects for garbage collection.

```java
@Override
protected void onDestroy() {
    super.onDestroy();
    if (this.localVideoSource != null) {
        this.localVideoSource.stop();
    }
    if (this.pnRTCClient != null) {
        this.pnRTCClient.onDestroy();
    }
}
```
    
We stop our local video source so that it will not continue streaming after we have left the video activity. We then call `PnRTCClient#onDestroy()` which cleans up the client and closes all open connections.

#### Static Methods, Hangup and User Message

If you need to communicate with `PnRTCClient` instances in situations where no client exists in your activity, perhaps an accept/reject call activity, there are static methods that handle this for you with any `Pubnub` instance. If you need to generate a hangup message to reject a call, you can do that as follows with a simple `Pubnub` instance.

```java
JSONObject hangupMsg = PnPeerConnectionClient.generateHangupPacket("myUsername");
this.mPubNub.publish("userCalling",hangupMsg, new Callback() {
    @Override
    public void successCallback(String channel, Object message) {
        Intent intent = new Intent(IncomingCallActivity.this, MainActivity.class);
        startActivity(intent);
    }
});
```

Simply pass __your__ username to `PnPeerConnectionClient.generateHangupPacket(String user)` and publish the returned `JSONObject` to the user who is calling.

The same goes for User Messages. With any `Pubnub` object, you can send a message through the UserMessage protocol using `PnPeerConnectionClient.generateUserMessage(String user, JSONObject message)`. Again, pass __your__ username and a JSON message. This will return a `JSONObject` of the proper format that you can publish to any user the same way as a hangup.

## Want some more?

- See Android RTC app, [AndroidRTC](https://github.com/GleasonK/AndroidRTC), for a concise demo of how to use `PnRTCClient` to quickly create a native Android WebRTC application.
- The PubNub Android WebRTC Signaling API is fully compatible with the [PubNub JavaScript WebRTC SDK](https://github.com/stephenlb/webrtc-sdk). This means that browser to mobile communication is simple!
- JS Blog: [Building a WebRTC Video and Voice Chat Application](http://www.pubnub.com/blog/building-a-webrtc-video-and-voice-chat-application/)

[JavaDoc]:http://kevingleason.me/android-webrtc-api/
[NativeAndroid]:http://www.webrtc.org/native-code/android
[PeerConnection]:http://w3c.github.io/webrtc-pc/#rtcpeerconnection-interface
[MediaStream]:http://w3c.github.io/webrtc-pc/#dfn-mediastream
[OfferOptions]:http://w3c.github.io/webrtc-pc/#idl-def-RTCOfferOptions
[ICE Servers]:http://w3c.github.io/webrtc-pc/#rtcicecredentialtype-enum
[PN Android]:https://www.pubnub.com/docs/android-java/pubnub-java-sdk
