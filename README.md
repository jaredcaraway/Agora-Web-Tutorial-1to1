
# Agora Tutorial for Web on PC - 1to1

This tutorial enables you to quickly get started with creating an Agora account, downloading the SDK, and using the Agora sample app to give you an overview on how to develop requests to the API, such as:

- Join / Leave calls
- Publish / Unpublish streams
- Enable / Disable audio
- Enable / Disable video

## Prerequisites
- Agora.io Developer Account
- Web server that supports SSL (https)

## Quick Start
This section shows you how to prepare, build, and run the sample application.

### Create an Account and Obtain an App ID
In order to build and run the sample application you must obtain an App ID: 

1. Create a developer account at [agora.io](https://dashboard.agora.io/signin/). Once you finish the signup process, you will be redirected to the Dashboard.
2. Navigate in the Dashboard tree on the left to **Projects** > **Project List**.
3. Copy the App ID that you obtained from the Dashboard into a text file. You will need this later, when you launch the app.

### Integrate the Agora Video SDK into the sample project
The SDK must be integrated into the sample project before it can run.

1. Download the Agora Video SDK from [Agora.io SDK](https://www.agora.io/en/download/). Under the **Video + Interactive Broadcasting SDK** heading, choose the **Web** download.
2. Unzip the downloaded SDK package.

	![download.jpg](images/download.jpg)

3. Copy the `AgoraRTCSDK` js file into the root of your GitHub project folder. The file will have a sample name similar to `AgoraRTCSDK-2.2.0.js`. 

	**Note:** `2.2.0` is a placeholder for the version number of the SDK js file you downloaded.


### Update and Run the Sample Application 

1. Open the `index.html` file in a code editor.
2. At the top of the file, in the `<head>` section, make sure the JavaScript file source is changed to `AgoraRTCSDK-2.2.0.js` (if it isn't already). Remember that to make sure the `2.2.0` placeholder is the version number of the SDK js file you downloaded.

	Before

	``` JavaScript
	<script src="build/AgoraRTC-development.js"></script>
	```

	After

	``` JavaScript
	<script src="build/AgoraRTC-2.2.0.js"></script>
	```
	
3. Deploy the project on a web server. Make sure you access the page through an SSL (https) connection. The Agora SDK needs a secure connection, to use the audio and video devices connected to the browser.
4. Use your browser to navigate to the index.html file. Once you've loaded the sample app, your browser will look like this:

	![sdksource_after.jpg](images/demoLoad.jpg)

5. In your browser window, paste the AppID into the `Key` UI text field.

	![download.jpg](images/demoKey.jpg)

6. Press the `Join` UI button to join the call. As soon as someone else joins the call, the call will be started and you will see each other in the browser window.

	**Note:** If your sample app needs to run on mobile browsers, be sure that `createClient` is called with the proper `mode` (See section *Create the Client*). For further details, reference the [Agora API documentation](https://docs.agora.io/en/).

## Steps to Create the Sample

- [Check AgoraRTC and System Requirements](#agorartc-and-system-requirements)
- [Setup App Initialization and Logging](#app-initialization-and-logging)
- [Create the Client and Join a Channel Session](#create-the-client-and-join-a-channel-session)
- [Create and Manage a Stream](#create-and-manage-a-stream)
- [Add Client Event Listeners](#add-client-event-listeners)
- [Leave a Channel](#leave-a-channel)
- [Publish/Unpublish a Stream](#publish/unpublish-a-stream)
- [Enable/Disable Audio and Video](#enable/disable-audio-and-video)

### Check AgoraRTC and System Requirements

`AgoraRTC` is the base JavaScript class of Agora Web SDK, which enables the use of Agora Web SDK's communication functionality. 

Agora requires WebRTC to run. Alert users if their browser does not support WebRTC, but calling the (`checkSystemRequirements()`) API method.

``` JavaScript
if(!AgoraRTC.checkSystemRequirements()) {
  alert("browser is no support webRTC");
}
```

### Setup App Initialization and Logging

- [Initialize Variables](#initialize-variables)
- [Initialize Audio and Video Devices](#initialize-audio-and-video-devices)
- [Agora Logging](#agora-logging)

#### Initialize Variables

For convenience, the sample app declares the `client`, `localStream`, `camera`, and `microphone` variables after the page loads, so they are easily referenced in the rest of the codebase.

``` JavaScript
var client, localStream, camera, microphone;
```

Similarly, the sample app initializes the audio source `audioSelect` and video source `videoSelect` dropdown menus, which is used for population, when the user's available audio and video devices is retrieved.

``` JavaScript
var audioSelect = document.querySelector('select#audioSource');
var videoSelect = document.querySelector('select#videoSource');

```

#### Initialize Audio and Video Devices

At the bottom of the `index.html` file you'll see the (`getDevices()`) method. This method loads all the available browser-enabled audio and video connections, into the `audio source` and `video source` dropdown menus.

``` JavaScript
// function to find and load browser-enabled audio and video devices

function getDevices() {
  AgoraRTC.getDevices(function (devices) {
    for (var i = 0; i !== devices.length; ++i) {
      var device = devices[i];
      var option = document.createElement('option');
      option.value = device.deviceId;
      if (device.kind === 'audioinput') {
        option.text = device.label || 'microphone ' + (audioSelect.length + 1);
        audioSelect.appendChild(option);
      } else if (device.kind === 'videoinput') {
        option.text = device.label || 'camera ' + (videoSelect.length + 1);
        videoSelect.appendChild(option);
      } else {
        console.log('Some other kind of source/device: ', device);
      }
    }
  });
}

// initialize audio and video devices on page load
getDevices();	

```

#### Agora Logging

To use logging, call it with `AgoraRTC.Logger`. The sample app shows examples of `error`, `warning`, `info`, and `debug` logging.

``` JavaScript
/* simulated data to proof setLogLevel() */
AgoraRTC.Logger.error('this is error');
AgoraRTC.Logger.warning('this is warning');
AgoraRTC.Logger.info('this is info');
AgoraRTC.Logger.debug('this is debug');

```

### Create the Client and Join a Channel Session

- [Create the Client](#create-the-client)
- [Initialize the Client](#initialize-the-client)
- [Join the Session](#join-the-session)

The sample app uses the (`join()`) JavaScript method to join the user to a specific channel.

``` JavaScript
function join() {
	...
}
```

#### Create the Client

The (`join()`) JavaScript method starts with creating the broadcast client object. The `interop` mode means that the app has permission to also work with Agora Native SDKs. (This is required if you want the app to be usable on mobile devices.)

Specifically:

- In communication scenarios, interoperability with Agora Native SDKs is enabled automatically
- In live broadcast scenarios, interoperability with Agora Native SDKs is enabled when the user calls enableWebSdkInteroperability()

**Note:** The broadcast client object can only be created once per call session.


``` JavaScript
  client = AgoraRTC.createClient({mode: 'interop'});
```

#### Initialize the Client

Once the client has been created, it must be initialized with the app ID. In the sample app, this is the UI text field labeled `Key`. 

``` JavaScript
  client.init(key.value, function () {
    console.log("AgoraRTC client initialized");
    
    ...
    
  }, function (err) {
    console.log("AgoraRTC client init failed", err);
  });
```

#### Join the Session

Once the client initialization (`client.init()`) is complete, the SDK's (`client.join()`) method is added to the  `onSuccess` callback. Pass in the `channel key`, `channel name`, and `user ID`.

- The first parameter `channel key`, is a string used for broadcast security. For low security requirements, pass null as the parameter value.
- The second parameter is the channel name. This is a string that provides a unique channel name for the Agora session. This should be a numerical value for the Web SDK. The sample app uses `channel.value` (the value from the `Channel` UI text field)
- The third parameter is the user ID. The user ID is a 32-bit unsigned integer ranging from 1 to (2^32-1). If you set the user ID to null, the Agora server allocates one and return it in the `onSuccess` callback. If you do decide to enter a specific user ID, make sure that the integer is unique or you will run into errors.

**Note:** Users in the same channel can talk to each other, however users using different App IDs cannot call each other (even if they join the same channel). Once this method is called successfully, the SDK triggers the callbacks.


``` JavaScript
    var channel_key = null;
    
    client.join(channel_key, channel.value, null, function(uid) {
      console.log("User " + uid + " join channel successfully");
      
      ...
      
    }, function(err) {
      console.log("Join channel failed", err);
    });
```

### Create and Manage a Stream

- [Host a Stream](#host-a-stream)
- [Create a Stream](#create-a-stream)
- [Set a Stream's Video Profile](#set-a-stream's-video-profile)
- [Set a Stream's Event Listeners for Camera/Mic Access](#set-a-stream's-event-listeners-for-camera/mic-access)

#### Host a Stream

Once joining, if the user will act as the *host* for the stream, the app must create a stream. For the sample app, it checks if the `Host` UI checkbox is checked. This check is applied within the API's (`join()`) method, `onSuccess` callback.

``` JavaScript
      if (document.getElementById("video").checked) {
	      ...
      }
```

#### Create a Stream

If the user is a host, start the stream using the (`createStream()`) API method. The sample app passes in an object with the following properties:

- **streamID**: The stream ID. Normally the stream ID is set as the uid, which can be retrieved from the (`client.join()`) callback.
- **audio**: Indicates if this stream contains an audio track.
- **video**: Indicates if this stream contains a video track.
- **screen**: Indicates if this stream contains a screen sharing track. Currently screen sharing is only supported by the Google Chrome Explorer.
- **cameraId**: (*Optional*) The camera device ID retrieved from the getDevices method. 
- **microphoneId**: (*Optional*) The microphone device ID retrieved from the getDevices method.

The `createStream` object is set up for additional optional attributes. Refer to the [Agora API documentation](https://docs.agora.io/en/) for more details.

The sample app passes in the callback's user ID `uid` as the streamID, enables audio, uses the selected audio and video devices from the dropdowns, enables video for the host only, and disables screen sharing.

``` JavaScript
        camera = videoSource.value;
        microphone = audioSource.value;
        
        localStream = AgoraRTC.createStream({
          streamID: uid, 
          audio: true, 
          cameraId: camera, 
          microphoneId: microphone, 
          video: document.getElementById("video").checked, 
          screen: false});
        
```
#### Set a Stream's Video Profile

If the user is a host, the video profile must be set. The sample app sets the video profile to `720p_3`, which means a Resolution of 1280x720, Frame Rate (fps) of 30, and a Bitrate (kbps) of 1710. Refer to the [Agora API documentation](https://docs.agora.io/en/) for additional video profile options.

``` JavaScript
        if (document.getElementById("video").checked) {
          localStream.setVideoProfile('720p_3');
        }
```

#### Set a Stream's Event Listeners for Camera/Mic Access

Once the stream has been set up and configured, the sample app adds event listeners using the API method (`localStream.on()`), to check for the user's microphone and camera permissions. These event listeners are used for debugging and/or to send alerts to request permissions. The sample app uses console logs to check if access to the camera and microphone was allowed or denied by the user.


``` JavaScript
        // The user has granted access to the camera and mic.
        localStream.on("accessAllowed", function() {
          console.log("accessAllowed");
        });

        // The user has denied access to the camera and mic.
        localStream.on("accessDenied", function() {
          console.log("accessDenied");
        });
```

Next, the sample app initializes the stream by calling the API's `localStream.init` method. Once initialized, the stream's host publishes the stream using the API's (`client.publish()`) method.

``` JavaScript
        localStream.init(function() {
          console.log("getUserMedia successfully");
          localStream.play('agora_local');

          client.publish(localStream, function (err) {
            console.log("Publish local stream error: " + err);
          });

          client.on('stream-published', function (evt) {
            console.log("Publish local stream successfully");
          });
        }, function (err) {
          console.log("getUserMedia failed", err);
        });
```

### Add Client Event Listeners

- [Client Error Handling](#client-error-handling)
- [Add a Stream to the Client](#add-a-stream-to-the-client)
- [Subscribe a Stream to the Client and Add to the DOM](#subscribe-a-stream-to-the-client-and-add-to-the-dom)
- [Remove a Stream from the Client](#remove-a-stream-from-the-client)
- [Remove a Peer from the Client](#remove-a-peer-from-the-client)

Adding listeners to the broadcast client is helpful for debugging, and to help manage streams. The sample app has event listeners for error messages, stream add/subscribe/remove, and peer leaves.

#### Client Error Handling

Passing `error` into the (`client.on()`) API method will return the error type `err.reason`. The sample app uses this error type for debugging and re-invoke methods that failed.

Since the Channel Key has an expiration, the sample app checks for the error `DYNAMIC_KET_TIMEOUT` in the `onFailure` callback. It then renews the channel key using the (`client.renewChannelKey()`) method. 

**Note:** If the channel key is not renewed, the communication to the SDK will disconnect.

``` JavaScript
  channelKey = "";
  
  client.on('error', function(err) {
    console.log("Got error msg:", err.reason);
    if (err.reason === 'DYNAMIC_KEY_TIMEOUT') {
      client.renewChannelKey(channelKey, function(){
        console.log("Renew channel key successfully");
      }, function(err){
        console.log("Renew channel key failed: ", err);
      });
    }
  });
  
```

#### Add a Stream to the Client

The `stream-added` event listener detects when a new stream is added to the client. The sample app subscribes the newly added stream to the client after a new stream is added to the client.

``` JavaScript
  client.on('stream-added', function (evt) {
  
    var stream = evt.stream;
    
    console.log("New stream added: " + stream.getId());
    console.log("Subscribe ", stream);
    
    client.subscribe(stream, function (err) {
      console.log("Subscribe stream failed", err);
    });
  });
  
```

#### Subscribe a Stream to the Client and Add to the DOM

The sample app uses the `stream-subscribed` event listener to detect when a new stream has been subscribed to the client, and to retrieve its stream ID using the (`stream.getId()`) API method. The stream is appended to the UI display using a `<div>`. 

Once the stream has been appended to the DOM, the sample app plays the stream by calling the (`stream.play()`) API method, passing in the string `agora_remote` followed by the stream ID.

``` JavaScript
  client.on('stream-subscribed', function (evt) {
    var stream = evt.stream;
    console.log("Subscribe remote stream successfully: " + stream.getId());
    if ($('div#video #agora_remote'+stream.getId()).length === 0) {
      $('div#video').append('<div id="agora_remote'+stream.getId()+'" style="float:left; width:810px;height:607px;display:inline-block;"></div>');
    }
    stream.play('agora_remote' + stream.getId());
  });
  
```

#### Remove a Stream from the Client

If the stream is removed from the client (`stream-removed`), the sample app stops the stream from playing by calling the (`stream.stop()`) API method. It then removes it from the DOM using JQuery's (`remove()`) method.

``` JavaScript
  client.on('stream-removed', function (evt) {
    var stream = evt.stream;
    stream.stop();
    $('#agora_remote' + stream.getId()).remove();
    console.log("Remote stream is removed " + stream.getId());
  });
  
```

#### Remove a Peer from the Client

When the sample app detects that a peer leaves the client using the (`peer-leave`) event listener, it stops the stream from playing, and removes the stream from the DOM.

``` JavaScript
  client.on('peer-leave', function (evt) {
    var stream = evt.stream;
    if (stream) {
      stream.stop();
      $('#agora_remote' + stream.getId()).remove();
      console.log(evt.uid + " leaved from this channel");
    }
  });
}

```

### Leave a Channel

The sample app applies the JavaScript (`leave()`) method to the `Leave` UI button.

The (`client.leave()`) API method enables the user to leave the current video call (channel). The sample app checks if the action succeeds or fails using the `onSuccess` and `onFailure` callbacks. 

``` JavaScript
function leave() {
  document.getElementById("leave").disabled = true;
  client.leave(function () {
    console.log("Leavel channel successfully");
  }, function (err) {
    console.log("Leave channel failed");
  });
}

```

### Publish/Unpublish a Stream

- [Publish a Stream](#publish-a-stream)
- [Unpublish a Stream](#unpublish-a-stream)

#### Publish a Stream

The sample app applies the JavaScript (`publish()`) method to the `Publish` UI button.

The (`client publish()`) API method enables publishing the local stream to the server. The sample app passes in the stream object and uses the `onFailure` callback to check for errors during the publishing process. 

``` JavaScript
function publish() {
  document.getElementById("publish").disabled = true;
  document.getElementById("unpublish").disabled = false;
  client.publish(localStream, function (err) {
    console.log("Publish local stream error: " + err);
  });
}

```

#### Unpublish a Stream

The sample app applies the JavaScript function (`unpublish()`) to the `Unpublish` UI button.

Similar to (`client.publish()`), the (`client.unpublish()`) API method enables unpublishing the local stream from the server. Again, the sample app passes in the stream object and using the `onFailure` callback to check for errors during the publishing process. 

``` JavaScript
function unpublish() {
  document.getElementById("publish").disabled = false;
  document.getElementById("unpublish").disabled = true;
  client.unpublish(localStream, function (err) {
    console.log("Unpublish local stream failed" + err);
  });
}
```


### Enable/Disable Audio and Video

- [Enable / Disable Audio](#enable-/-disable-audio)
- [Enable / Disable Video](#enable-/-disable-video)

For this section, run the `cdn.html` sample file. Make sure to apply the same instructions from the **File updates** section to the `cdn.html` file.

#### Enable / Disable Audio

For each connected stream, the Agora API can enable or disable audio.

To enable audio, call the (`enableAudio()`) API on the stream. The sample app applies the (`enableAudio()`) JavaScript method on the `Enable Audio` UI button, to call enable audio on the open `localStream`.

``` JavaScript
function enableAudio() {
  localStream.enableAudio();
}
```

Disabling audio is exactly the same, but using the (`disableAudio()`) API method. The sample app, uses the (`disableAudio()`) JavaScript method applied to the `Disable Audio` UI button.

``` JavaScript
function disableAudio() {
  localStream.disableAudio();
}
```

#### Enable / Disable Video

For each connected stream, the Agora API can enable or disable video.

Enabling video is the same as enabling audio, but uses the (`enableVideo()`) API method on the stream. The sample app applies the (`enableVideo()`) JavaScript method to the `Enable Video` UI button.

``` JavaScript
function enableVideo() {
  localStream.enableVideo();
}
```

To disable video, use the (`disableVideo()`) API method. The sample app applies the (`disableVideo()`) JavaScript method applied to the `Disable Video` UI button.

``` JavaScript
function disableVideo() {
  localStream.disableVideo();
}
```

## Resources
* You can find full API documentation at the [Document Center](https://docs.agora.io/en/).
* You can file bugs about this sample [here](https://github.com/AgoraIO/Agora-Web-Tutorial-1to1/issues).


## License
This software is under the MIT License (MIT). [View the license](LICENSE.md).

