
---
title: Peer-to-peer or Channel Messaging
description: 
platform: Android
updatedAt: Thu Aug 27 2020 07:35:09 GMT+0800 (CST)
---
# Peer-to-peer or Channel Messaging

You can use this guide to quickly start messaging with the [Agora RTM Android SDK](https://docs.agora.io/en/Real-time-Messaging/downloads). 


## Try the demo

We provide an open-source demo project on GitHub, [Agora-RTM-Tutorial-Android](https://github.com/AgoraIO/RTM/tree/master/Agora-RTM-Tutorial-Android), which implements an elementary messaging system. You can try this demo out and view our source code:

- [LoginActivity.java](https://github.com/AgoraIO/RTM/blob/master/Agora-RTM-Tutorial-Android/app/src/main/java/io/agora/activity/LoginActivity.java) 
- [MessageActivity.java](https://github.com/AgoraIO/RTM/blob/master/Agora-RTM-Tutorial-Android/app/src/main/java/io/agora/activity/MessageActivity.java) 
- [SelectionActivity.java](https://github.com/AgoraIO/RTM/blob/master/Agora-RTM-Tutorial-Android/app/src/main/java/io/agora/activity/SelectionActivity.java)

## Prerequisites



- Android SDK API Level 16 or higher
- To run your app on Android 9, see [Android Privacy Changes](https://developer.android.com/about/versions/pie/android-9.0-changes-28#privacy-changes-p) for more information.
- Android Studio 3.0 or later

- A valid Agora account. ([Sign up](https://sso.agora.io/en/signup) for free)

<div class="alert note">Open the specified ports in <a href="https://docs.agora.io/en/Agora%20Platform/firewall?platform=All%20Platforms">Firewall Requirements</a > if your intranet has a firewall.</div> 



## <a name="setup"></a>Set up the development environment

We will walk you through the following steps in this section:

- [Get an App ID](#appid)
- [Integrate the SDK into your project](#sdk)
- [Add Android device permissions](#permission)
- [ Prevent Obfuscation of the Agora Classes](#obfuscated)

### <a name="appid"></a>Get an App ID

You can skip to [Integrate the SDK into your project](#sdk) if you already have an App ID. 

<details>
	<summary><font color="#3ab7f8">Get an App ID</font></summary>
	
1. Sign up for a developer account at [Agora Console](https://dashboard.agora.io/). See [Sign in and Sign up](../../en/Real-time-Messaging/sign_in_and_sign_up.md).

2. Click ![](https://web-cdn.agora.io/docs-files/1551254998344) in the left navigation menu to enter the [**Project Management**](https://dashboard.agora.io/projects) page.

3. Click **Create**. 

![](https://web-cdn.agora.io/docs-files/1574924327108)

4.  Enter your project name and select your authentication mechanism ("App ID") in the dialog box.

![](https://web-cdn.agora.io/docs-files/1574924446798)
	
5. Click **Submit** and you can find the **App ID** of your newly created project.

![](https://web-cdn.agora.io/docs-files/1574924570426)
</details>

### <a name="sdk"></a> Integrate the SDK into your project


Choose either of the following methods to integrate the Agora RTM Android SDK into your project.

**Method 1: Automatically integrate the SDK using JCenter**

Add the following line in the **/app/build.gradle** file of your project (1.2.2 is the version number):

```java
...
dependencies {
    ...
    implementation 'io.agora.rtm:rtm-sdk:1.2.2'
}
```

**Method 2: Manually copy the SDK files**

<ol>

<li>Download the latest version of the <a href="https://docs.agora.io/en/Real-time-Messaging/downloads">Agora RTM Android SDK</a> and unzip.</li>
<li>Save the <b>.jar</b> package and <b>.so</b> files under the <b>libs</b> folder of the unzipped SDK package to the corresponding folders of your project. </li>

| File                                     | Project Folder                          |
| ---------------------------------------- | --------------------------------------- |
| **agora-rtm_sdk.jar**                    | **~/app/libs/**                         |
| **/arm64-v8a/libagora-rtm-sdk-jni.so**   | **~/app/src/main/jniLibs/arm64-v8a/**   |
| **/armeabi-v7a/libagora-rtm-sdk-jni.so** | **~/app/src/main/jniLibs/armeabi-v7a/** |
| **/x86/libagora-rtm-jni.so**             | **~/app/src/main/jniLibs/x86/**         |
| **/x86_64/libagora-rtm-sdk-jni.so**      | **~/app/src/main/jniLibs/x86_64/**      |



### <a name="permission"></a>Add project permissions

Add the following permissions in the **/app/src/main/AndroidManifest.xml** file for device access according to your needs:


```
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="io.agora.rtmtutorial">

    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />

...
</manifest>
```




### <a name="obfuscated"></a>Prevent code obfuscation

Add the following line in the **app/proguard-rules.pro** file to prevent code obfuscation:

```java
-keep class io.agora.**{*;}
```

## Implement peer-to-peer and channel messaging


This section provides API call sequence diagrams and sample codes related to peer-to-peer messaging and channel messaging. 


### Create and Initialize an Agora RTM Client


1. Put in the `App ID` you get from **Console** when creating and initializing an Agora RTM client. 
2. Implement an event callback `RtmClientListener`, the SDK uses its callbacks to notify the app of ongoing events, including:
   - The connection between the SDK and the Agora RTM system changes. 
   - A peer-to-peer message arrives. 
   - Miscellaneous events. 

```java
import io.agora.rtm.ErrorInfo;
import io.agora.rtm.ResultCallback;
import io.agora.rtm.RtmChannel;
import io.agora.rtm.RtmChannelListener;
import io.agora.rtm.RtmChannelMember;
import io.agora.rtm.RtmClient;
import io.agora.rtm.RtmClientListener;
import io.agora.rtm.RtmMessage;
import io.agora.rtm.SendMessageOptions;

public void init() {
		try {
				mRtmClient = RtmClient.createInstance(mContext, APPID,
												new RtmClientListener() {
						@Override
						public void onConnectionStateChanged(int state, int reason) {
								Log.d(TAG, "Connection state changes to "
										+ state + " reason: " + reason);
						}

						@Override
						public void onMessageReceived(RtmMessage rtmMessage, String peerId) {
								String msg = rtmMessage.getText();
								Log.d(TAG, "Message received " + " from " + peerId + msg 
														);
						}
				});
		} catch (Exception e) {
				Log.d(TAG, "RTM SDK initialization fatal error!");
				throw new RuntimeException("You need to check the RTM initialization process.");
		}
}
```

### Log in to and log out of the Agora RTM system

Only when you successfully log in the Agora RTM system, can you use most of the core features provided by the Agora RTM SDK. When logging in the Agora RTM system, you need to:

- Pass an RTM Token, which identifies a user and its privilege. If your development environment does not require high security, set `token` as null. You need to generate an RTM Token from your business server. For more information, see [Token Security](../../en/Real-time-Messaging/rtm_token.md).
- An `userId` is the unique identifier of each user. You must not set it as empty, null, or "null". 
- Pass a `ResultCallback` instance, which returns the `onSuccess` or `onFailure` callback. 

```java
mRtmClient.login(null, userId, new ResultCallback<Void>() {
        @Override
        public void onSuccess(Void responseInfo) {
                loginStatus = true;
                Log.d(TAG, "login success!");
        }
        @Override
        public void onFailure(ErrorInfo errorInfo) {
                loginStatus = false;
                Log.d(TAG, "login failure!");
        }
});
```

To log out of the system:

```java
mRtmClient.logout(null);
```

### Peer-to-peer messaging

#### Send an offline peer-to-peer message

You need to call the following method to send an peer-to-peer message to a specified user. 

```java
sendMessageToPeer(@NonNull String, @NonNull RtmMessage, @NonNull SendMessageOptions, @Nullable ResultCallback<void>)
```

You need to:

- Pass a user ID of the specified message receiver.
- Call the `createMessage` method to create an `RtmMessage`instance and pass it to

```java
public void sendPeerMessage(String dst, String content) {
        
        final RtmMessage message = mRtmClient.createMessage();
        message.setText(content);

        SendMessageOptions option = new SendMessageOptions();
        option.enableOfflineMessaging = true;

        mRtmClient.sendMessageToPeer(dst, message, option, new ResultCallback<Void>() {

            @Override
            public void onSuccess(Void aVoid) {

            }

            @Override
            public void onFailure(ErrorInfo errorInfo) {

            }
        });
    }
```

#### Receive a peer-to-peer message

You can use the `RtmClientListener` instance that you passed in when creating an `RtmClient` instance to listen for events that a peer-to-peer message is received. The `onMessageReceived(RtmMessage message, String peerId)` callback of the listener takes an `RtmMessage` instance with it.

- You can use the `message.getText()` method to get the text content of the message.
- `peerId` refers to the user ID of the message sender. 

> The received `RtmMessage` object cannot be reused.

### Channel messaging

Ensure that you have logged in the Agora RTM system before being able to use the channel messaging function. 

#### Create and join a channel

When creating an RtmChannel, you need to pass a channel ID, a string that must not be empty, null, "null", or exceed 64 Bytes in length. You also need to specify a channel listener: 

```java
private RtmChannelListener mRtmChannelListener = new RtmChannelListener() {
    @Override
    public void onMessageReceived(RtmMessage message, RtmChannelMember fromMember) {
        String text = message.getText();
        String fromUser = fromMember.getUserId();
    }
 
    @Override
    public void onMemberJoined(RtmChannelMember member) {
 
    }
 
    @Override
    public void onMemberLeft(RtmChannelMember member) {
 
    }
};
```

```java
try {
		mRtmChannel = mRtmClient.createChannel("demoChannelId", mRtmChannelListener);
} catch (RuntimeException e) {
		Log.e(TAG, "Fails to create channel. Maybe the channel ID is invalid," +
						" or already in use. See the API Reference for more information.");
}

		mRtmChannel.join(new ResultCallback<Void>() {
				@Override
				public void onSuccess(Void responseInfo) {
						Log.d(TAG, "Successfully joins the channel!");
				}

				@Override
				public void onFailure(ErrorInfo errorInfo) {
						Log.d(TAG, "join channel failure! errorCode = "
																+ errorInfo.getErrorCode());
				}
		});
```

#### Channel messaging

After successfully joining a channel, you can send messages to the channel. As we mentioned in the previous section, you need to set a channel listener for events such as a channel message is received. 

When calling a `sendChannelMessage()`message method to send channel messages, you need to: 

- Pass an `RtmMessage` object. You need to use the `createMessage()` method of the `RtmClient` class to create an `RtmMessage` object and use the `setText()` method to set the message content. 
- When sending a channel message, pass a `ResultCallback` instance.

```java
public void sendChannelMessage(String msg) {
		RtmMessage message = mRtmClient.createMessage();
		message.setText(msg);

		mRtmChannel.sendMessage(message, new ResultCallback<Void>() {
				@Override
				public void onSuccess(Void aVoid) {
				}

				@Override
				public void onFailure(ErrorInfo errorInfo) {
				}
		});
}
```

#### Leave a channel

You can call the `leave()` method to leave a channel. 


## Considerations


- The Agora RTM SDK supports creating multiple [RtmClient](https://docs.agora.io/en/Real-time-Messaging/API%20Reference/RTM_java/classio_1_1agora_1_1rtm_1_1_rtm_client.html) instances that are independent of each other. 

-  To send and receive peer-to-peer or channel messages, ensure that you have successfully logged in the Agora RTM system (i.e., ensure that you have received [onSuccess](https://docs.agora.io/en/Real-time-Messaging/API%20Reference/RTM_java/interfaceio_1_1agora_1_1rtm_1_1_result_callback.html#a7206b30500655c4a73d146acf50cb6f5)). 

- To use any of the channel features, you must first call the [createChannel](https://docs.agora.io/en/Real-time-Messaging/API%20Reference/RTM_java/classio_1_1agora_1_1rtm_1_1_rtm_client.html#a95ebbd1a1d902572b444fef7853f335a) method to create a channel instance. 
- You can create multiple channel instances for each [RtmClient](https://docs.agora.io/en/Real-time-Messaging/API%20Reference/RTM_java/classio_1_1agora_1_1rtm_1_1_rtm_client.html) instance, but you can only join a maximum of 20 channels at the same time. The `channelId` parameter needs to be channel-specific.

- When you leave a channel and do not want to join it again, you can call the [release](https://docs.agora.io/en/Real-time-Messaging/API%20Reference/RTM_java/classio_1_1agora_1_1rtm_1_1_rtm_channel.html#a725d008cb19f44496948ee8f1826deaf) method to release all resources used by the channel instance.

- You cannot reuse a received [RtmMessage](https://docs.agora.io/en/Real-time-Messaging/API%20Reference/RTM_java/classio_1_1agora_1_1rtm_1_1_rtm_message.html) instance.



