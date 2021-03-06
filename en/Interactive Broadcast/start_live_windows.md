
---
title: Start Live Interactive Video Streaming
description: 
platform: Windows
updatedAt: Fri Jul 10 2020 06:45:44 GMT+0800 (CST)
---
# Start Live Interactive Video Streaming
Use this guide to quickly start the live interactive video streaming with the Agora Video SDK for Windows. 

## Sample projects
We provide open-source [OpenLive-Windows](https://github.com/AgoraIO/Basic-Video-Broadcasting/tree/master/OpenLive-Windows) and [OpenLive-Windows-MFC](https://github.com/AgoraIO/Basic-Video-Broadcasting/tree/master/OpenLive-Windows-MFC) sample projects that implements the basic live interactive video streaming on GitHub. You can download it and view the source code.

## Prerequisites
- Microsoft Visual Studio 2017 or later
- A Windows device running Windows 7 or later
- A valid Agora account. ([Sign up](https://dashboard.agora.io) for free)

<div class="alert note">Open the specified ports in <a href="https://docs.agora.io/cn/Agora%20Platform/firewall?platform=All%20Platforms">Firewall Requirements</a> if your network has a firewall.</div>

## Set up the development environment
In this section, we will create a Windows project and integrate the SDK into the project.

### Create a Windows project
Now, let's build a Windows project from scratch. Skip to [Integrate the SDK](#inte) if a Windows project already exists.

<details>
	<summary><font color="#3ab7f8">Create a Windows project</font></summary>

1. Open <b>Microsoft Visual Studio</b> and click <b>Create new project</b>.
2. On the <b>New Project</b> panel, choose <b>MFC Application</b> as the project type, input the project name, choose the project location, and click <b>OK</b>.
3. On the <b>MFC Application</b> panel, choose <b>Application type > Dialog based</b>, and click <b>Finish</b>.
	

</details>

<a name="inte"></a>
### Integrate the SDK
Follow these steps to integrate the Agora Video SDK into your project.

**1. Configure the project files**

- Go to [SDK Downloads](https://docs.agora.io/en/Agora%20Platform/downloads), download the latest version of the Agora SDK for Windows, and unzip the downloaded SDK package.

- Copy the **x86** or **x86_64** folder of the downloaded SDK package to your project files according to your development environment.

**2. Configure the project properties**

Right-click the project name in the **Solution Explorer** window, click **Properties** to configure the following project properties, and click **OK**.

- Go to the **C/C++ > General > Additional Include Directories** menu, click **Edit**, and input **$(SolutionDir)include** in the pop-up window.

- Go to the **Linker > General > Additional Library Directories** menu, click **Edit**, and input **$(SolutionDir)** in the pop-up window.

- Go to the **Linker > Input > Additional Dependencies** menu, click **Edit**, and input **agora_rtc_sdk.lib** in the pop-up window.

## Implement the basic live interactive streaming
This section introduces how to use the Agora SDK to start the live interactive video streaming. The following figure shows the API call sequence of the live interactive video streaming.

![](https://web-cdn.agora.io/docs-files/1568257918372)

### 1. Create the UI
Create the user interface (UI) for the interactive video streaming in your project. Skip to [Initialize IRtcEngine](#ini) if you already have a UI in your project.

If you are implementing the interactive video streaming, we recommend adding the following elements into the UI:
- The view of the host
- The exit button

When you use the UI setting of the sample project, you can see the following interface:

![](https://web-cdn.agora.io/docs-files/1568792708592)

<a name="ini"></a>
### 2. Initialize IRtcEngine
Create and initialize the `IRtcEngine` object before calling any other Agora APIs.

In this step, you need to use the App ID of your project. Follow these steps to [create an Agora project](https://docs.agora.io/en/Agora%20Platform/manage_projects?platform=All%20Platforms) in Console and get an [App ID](https://docs.agora.io/en/Agora%20Platform/terms?platform=All%20Platforms#a-nameappidaapp-id ).

1. Go to [Console](https://dashboard.agora.io/) and click the **[Project Management](https://dashboard.agora.io/projects)** icon ![](https://web-cdn.agora.io/docs-files/1551254998344) on the left navigation panel. 
2. Click **Create** and follow the on-screen instructions to set the project name, choose an authentication mechanism, and Click **Submit**. 
3. On the **Project Management** page, find the **App ID** of your project. 

Call the `createAgoraRtcEngine` method and the `initialize` method, and pass in the App ID to initialize the `IRtcEngine` object.

You can also listen for callback events, such as when the local user joins the channel, and when the first video frame of the host is decoded.

```C++
// Create the instance.
m_lpRtcEngine = createAgoraRtcEngine();
RtcEngineContext ctx;
         
// Add the register events and callbacks.
ctx.eventHandler = &m_engineEventHandler;
 
// Input your App ID.
ctx.appId = "YourAppID";
 
// Initialize the IRtcEngine object.
m_lpRtcEngine->initialize(ctx);
```

```C++
// Inherit the events and callbacks of IRtcEngineEventHandler.
class CAGEngineEventHandler :
    public IRtcEngineEventHandler
{
public:
    CAGEngineEventHandler();
    ~CAGEngineEventHandler();
    void setMainWnd(HWND wnd);
    HWND GetMsgReceiver() {return m_hMainWnd;};
 
    // Listen for the onJoinChannelSuccess callback.
    // This callback occurs when the local user successfully joins the channel.
    virtual void onJoinChannelSuccess(const char* channel, uid_t uid, int elapsed);
 
    // Listen for the onLeaveChannel callback.
    // This callback occurs when the local user successfully leaves the channel.
    virtual void onLeaveChannel(const RtcStats& stat);
 
    // Listen for the onFirstRemoteVideoDecoded callback.
    // This callback occurs when the first video frame of the host is received and decoded after the host successfully joins the channel.
    // You can call the setupRemoteVideo method in this callback to set up the remote video view.
    virtual void onFirstRemoteVideoDecoded(uid_t uid, int width, int height, int elapsed);
 
    // Listen for the onUserOffline callback.
    // This callback occurs when the remote user leaves the channel or drops offline.
    virtual void onUserOffline(uid_t uid, USER_OFFLINE_REASON_TYPE reason);
private:
    HWND m_hMainWnd;
};
```

### 3. Set the channel profile
After initializing the `IRtcEngine` object, call the `setChannelProfile` method to set the channel profile as `LIVE_BROADCASTING`. 

One `IRtcEngine` object uses one profile only. If you want to switch to another profile, release the current `IRtcEngine` object with the `release` method and create a new one before calling the `setChannelProfile` method.

```C++
m_lpRtcEngine->setChannelProfile(CHANNEL_PROFILE_LIVE_BROADCASTING);
```

### 4. Set the client role
A `LIVE_BROADCASTING` channel has two client roles: `BROADCASTER` and `AUDIENCE`, and the default role is `AUDIENCE`. After setting the channel profile to `LIVE_BROADCASTING`, your app may use the following steps to set the client role:
- 1. Allow the user to set the role as `BROADCASTER` or `AUDIENCE`. 
- 2. Call the `setClientRole` method and pass in the client role set by the user.

Note that in the live interactive streaming, only the host can be heard and seen. If you want to switch the client role after joining the channel, call the `setClientRole` method.

```C++
// Set the client role.
BOOL CAgoraObject::SetClientRole(CLIENT_ROLE_TYPE role, LPCSTR lpPermissionKey)
{
    int nRet = m_lpAgoraEngine->setClientRole(role);
    m_nRoleType = role;
    return nRet == 0 ? TRUE : FALSE;
}
```

```C++
// Show a dialog box to choose a user role.
void CEnterChannelDlg::OnCbnSelchangeCmbRole()
{
    int nSel = m_ctrRole.GetCurSel();
    if (nSel == 0)
        CAgoraObject::GetAgoraObject()->SetClientRole(CLIENT_ROLE_BROADCASTER);
    else
        CAgoraObject::GetAgoraObject()->SetClientRole(CLIENT_ROLE_AUDIENCE);
}
```

### 5. Set the local video view
After setting the channel profile and client role, set the local video view before joining the channel so that the host can see the local video in the interactive streaming. Follow these steps to configure the local video view:
- Call the `enableVideo` method to enable the video module.
- Call the `setupLocalVideo` method to configure the local video display settings. 

```C++
// Enable the video module.
m_lpRtcEngine->enableVideo();
 
// Set the local video view.
VideoCanvas vc;
vc.uid = 0;
vc.view = hVideoWnd;
vc.renderMode = RENDER_MODE_TYPE::RENDER_MODE_HIDDEN;
m_lpRtcEngine->setupLocalVideo(vc);
```

<a name="join"></a>
### 6. Join a channel
After setting the client role and the local video view, you can call the `joinChannel` method to join a channel. In this method, set the following parameters:

- `channelName`: Specify the channel name that you want to join.

* `token`: Pass a token that identifies the role and privilege of the user.  You can set it as one of the following values:
  * `NULL`.
  * A temporary token generated in Console. A temporary token is valid for 24 hours. For details, see [Get a Temporary Token](https://docs.agora.io/en/Agora%20Platform/token#get-a-temporary-token).
  * A token generated at the server. This applies to scenarios with high-security requirements. For details, see [Generate a token from Your Server](../../en/Interactive%20Broadcast/token_server_cpp.md).
  
 <div class="alert note">If your project has enabled the app certificate, ensure that you provide a token.</div>

- `uid`: ID of the local user that is an integer and should be unique. If you set `uid` as 0,  the SDK assigns a user ID for the local user and returns it in the `onJoinChannelSuccess` callback.

For more details on the parameter settings, see [joinChannel](https://docs.agora.io/en/Interactive%20Broadcast/API%20Reference/cpp/classagora_1_1rtc_1_1_i_rtc_engine.html#adc937172e59bd2695ea171553a88188c).

```C++
// For SDKs earlier than v3.0.0, call this method to enable interoperability between the Native SDK and the Web SDK if the Web SDK is in the channel. As of v3.0.0, the Native SDK enables the interoperability with the Web SDK by default.
m_lpRtcEngine->enableWebSdkInteroperability(true);
 
// Join a channel with a token.
BOOL CAgoraObject::JoinChannel(LPCTSTR lpChannelName, UINT nUID, LPCSTR lpDynamicKey)
{
    int nRet = 0;
    LPCSTR lpStreamInfo = "{\"owner\":true,\"width\":640,\"height\":480,\"bitrate\":500}";
#ifdef UNICODE
    CHAR szChannelName[128];
    ::WideCharToMultiByte(CP_ACP, 0, lpChannelName, -1, szChannelName, 128, NULL, NULL);
    nRet = m_lpAgoraEngine->joinChannel(lpDynamicKey, szChannelName, lpStreamInfo, nUID);
#else
    nRet = m_lpAgoraEngine->joinChannel(lpDynamicKey, lpChannelName, lpStreamInfo, nUID);
#endif
    if (nRet == 0)
        m_strChannelName = lpChannelName;
    return nRet == 0 ? TRUE : FALSE;
}
```

### 7. Set the remote video view
In the interactive video streaming, you should also be able to see all the hosts, regardless of your role. This is achieved by calling the `setupRemoteVideo` method after joining the channel.

Shortly after a remote host joins the channel, the SDK gets the host's user ID in the `onFirstRemoteVideoDecoded` callback. Call the `setupRemoteVideo` method in the callback and pass in the `uid` to set the video view of the host.

```C++
// Listen for the onFirstRemoteVideoDecoded callback.
// This callback occurs when the first video frame of the host is received and decoded after the host successfully joins the channel.
void CAGEngineEventHandler::onFirstRemoteVideoDecoded(uid_t uid, int width, int height, int elapsed)
{
    LPAGE_FIRST_REMOTE_VIDEO_DECODED lpData = new AGE_FIRST_REMOTE_VIDEO_DECODED;
    lpData->uid = uid;
    lpData->width = width;
    lpData->height = height;
    lpData->elapsed = elapsed;
    if(m_hMainWnd != NULL)
        ::PostMessage(m_hMainWnd, WM_MSGID(EID_FIRST_REMOTE_VIDEO_DECODED), (WPARAM)lpData, 0);
}
```

```C++
VideoCanvas canvas;
canvas.renderMode = RENDER_MODE_FIT;
POSITION pos = m_listWndInfo.GetHeadPosition();
......
AGVIDEO_WNDINFO &agvWndInfo = m_listWndInfo.GetNext(pos);
canvas.uid = agvWndInfo.nUID;
canvas.view = m_wndVideo[nIndex].GetSafeHwnd();
agvWndInfo.nIndex = nIndex;
  
// Set the remote video view.
CAgoraObject::GetEngine()->setupRemoteVideo(canvas);
```

### 8. Leave the channel
Call the `leaveChannel` method to leave the current channel according to your scenario, for example, when the interactive streaming ends, when you need to close the app, or when your app runs in the background.

```C++
BOOL CAgoraObject::LeaveCahnnel()
{
    m_lpAgoraEngine->stopPreview();
    // Leave the current channel.
    int nRet = m_lpAgoraEngine->leaveChannel();
    m_nSelfUID = 0;
    return nRet == 0 ? TRUE : FALSE;
}
 
 
void CAgoraObject::CloseAgoraObject()
{
    if(m_lpAgoraEngine != NULL)
        // Release the IRtcEngine object.
        m_lpAgoraEngine->release();
    if(m_lpAgoraObject != NULL)
        delete m_lpAgoraObject;
    m_lpAgoraEngine = NULL;
    m_lpAgoraObject = NULL;
}
```

## Run the project
Run the project on your Windows device. When you set the role as the host and successfully start the interactive video streaming, you can see the video view of yourself in the app. When you set the role as the audience and successfully join the interactive video streaming, you can see the video view of the host in the app.

## Reference

- [How can I listen for an audience joining or leaving a live interactive streaming channel?](https://docs.agora.io/en/faq/audience_event)
- [How can I set the log file?](https://docs.agora.io/en/faq/logfile)
- [How can I solve black screen issues?](https://docs.agora.io/en/faq/video_blank)
