实时生成场景WebRTC渲染的可行性调研

## 1. WebRTC 概述
WebRTC（Web Real-Time Communications）是一项实时通讯技术，它允许网络应用或者站点，在不借助中间媒介的情况下，建立浏览器之间点对点（Peer-to-Peer）的连接，实现视频流和（或）音频流或者其他任意数据的传输。



---

## 2. WebRTC 发展史
* **2011年**：Google 先后收购 GIPS 和 On2，组成 GIPS 音视频引擎 + VPx 系列视频编解码器，并将其代码开源，WebRTC 项目应运而生。
* **2012年**：Google 将 WebRTC 集成到 Chrome 浏览器中
* **现在**：除了 IE 之外的浏览器都已支持 WebRTC

### 兼容性
[https://caniuse.com/?search=WebRTC](https://caniuse.com/?search=WebRTC)

![](https://rte.weiyun.baidu.com/wiki/attach/image/api/imageDownloadAddress?attachId=11e7c4ba4b52491cb9c78916e0ab9af5&docGuid=nXT_pIUOzE3auq "")




---

## 3. WebRTC 组成部分
WebRTC 主要由三部分组成：**浏览器 API**、**音视频引擎**和**网络 IO**。

### 3.1 浏览器 API
用于**采集摄像头和麦克风**生成媒体流，并处理音视频通信相关的**编码、解码、传输**过程：

* **getUserMedia**: 获取麦克风和摄像头的许可，使得 WebRTC 可以拿到本地媒体流
* **RTCPeerConnection**: 建立点对点连接的关键，提供了创建，保持，监控，关闭连接的方法的实现
* **RTCDataChannel**: 支持点对点数据传输，可用于传输文件、文本消息等

### 3.2 音视频引擎
WebRTC **内置了强大的音视频引擎**，可以对媒体流进行编解码、回声消除、降噪、防止视频抖动等处理：

* **OPUS**: 一个开源的低延迟音频编解码器，WebRTC 默认使用
* **G711**: 国际电信联盟 ITU-T 定制出来的一套语音压缩标准
* **VP8/VP9**: Google 开源的视频编解码器，现在主要用于 WebRTC 视频编码
* **H264**: 视频编码领域的通用标准，提供了高效的视频压缩编码
* **AEC(Acoustic Echo Chancellor)**: 回声消除
* **ANS(Automatic Noise Suppression)**: 背景噪音抑制
* **Jitter buffer**: 防止视频抖动

### 3.3 网络 I/O
WebRTC 传输层用的是 **UDP** 协议，因为音视频传输对**及时性**要求更高：

* **RTP/SRTP（**Secure Real-time Transport Protocol**）**: 传输音视频数据流，使用 SRTP 协议进行加密，保证媒体数据的安全
* **RTCP（**Real-time Transport Control Protocol**）**: 通过 RTCP 可以知道各端的网络质量，进行流控处理
* **P2P(ICE + STUN + TURN)**: 利用 ICE、STUN、TURN 等技术，实现了浏览器之间的直接点对点连接，解决了 NAT 穿透问题

---

## 4. WebRTC 应用场景
* **点对点通讯**: 浏览器之间进行音视频通话，例如语音通话、视频通话等
* **电话会议**: 支持多人音视频会议，例如腾讯会议、钉钉会议等
* **屏幕共享**: 不仅可以传输音视频流，还可以用于实时共享屏幕
* **直播**: 构建实时直播，用户可以通过浏览器观看直播内容
* **监控**: 低延迟的音视频通信能力，可以用于物联网设备之间的实时视频监控

---

## 5. WebRTC 的优缺点
### ✅ 优点
|优点|描述|
|-|-|
|**实时通信**|支持浏览器之间的实时音频、视频和数据通信，无需任何插件或第三方软件|
|**高质量的音视频通信**|使用最先进的音频和视频编解码器，如 Opus 和 VP8/VP9/H.264|
|**端到端加密**|所有通信都是端到端加密的，保护了用户的隐私和数据安全|
|**P2P 连接**|支持直接的 P2P 连接，可以减少延迟和带宽消耗|
|**跨平台和跨浏览器**|开源的标准，被大多数现代浏览器和平台支持|

### ❌ 缺点
|缺点|描述|解决方案|
|-|-|-|
|**复杂的信令过程**|WebRTC 本身并不包含信令协议，需要自己实现|使用 WebSocket 实现信令服务器|
|**防火墙和 NAT 问题**|建立连接可能会受到防火墙和 NAT 的限制|使用 STUN/TURN 服务器进行 NAT 穿透|
|**隐私问题**|可能会泄露用户的 IP 地址|使用 SRTP 加密，保护媒体流传输安全|
|**资源消耗**|需要消耗大量的 CPU 和带宽资源|在后端进行视频解码和编码，减少前端负担|

---

## 6. Web 端基础常用相关 API
### 6.1 getUserMedia
用于访问用户音视频设备的接口，包括摄像头和麦克风。

```
// 基本用法
const constraints = { video: true, audio: true }
const localStream = navigator.mediaDevices.getUserMedia(constraints)

// 高级配置
const constraints = {
    audio: { deviceId: { exact: selectedAudioDeviceId } },
    video: {
        deviceId: { exact: selectedVideoDeviceId }, // 指定设备
        width: { min: 320, ideal: 1280, max: 1920 }, // 分辨率和帧率
        height: { min: 240, ideal: 720, max: 1080 },
        facingMode: {
            { ideal: 10, max: 15 }, // 帧率范围
            { exact: "environment" } //前置或后置摄像头 "user" 或 "environment"
        }
    }
}
```
### 6.2 getDisplayMedia
用于在浏览器中实现屏幕分享功能。允许用户选择并分享整个屏幕或特定应用窗口，适用于远程会议和在线演示等场景。

```
async function getShareMedia() {
  // 配置屏幕分享参数
  const constraints = {
      video: { width: 1920, height: 1080 },
      audio: false
  };

  // 注意：在获取新的媒体流之前，建议停止之前的媒体流，以避免冲突。
  if (window.stream) {
    window.stream.getTracks().forEach(track => track.stop());
  }

  try {
    return await navigator.mediaDevices.getDisplayMedia(constraints);
  } catch (error) {
    console.error('失败:', error);
  }
}
```
### 6.3 RTCPeerConnection
用于管理音视频连接的核心 API，负责与各端建立连接，接收、发送音视频数据。

```
const configuration = {
    iceServers: [{ urls: 'stun:stun.l.google.com:19302' }]
};
const peerConnection = new RTCPeerConnection(configuration);
```
#### 6.3.1. 创建连接请求
* `createOffer()`: 发起连接请求
* `createAnswer()`: 响应连接请求

```
peerConnection.createOffer()
    .then(offer => peerConnection.setLocalDescription(offer))
    .then(() => {
        // 发送 offer 给对端
    });
```
#### 6.3.2. 设置描述信息
* `setLocalDescription(description)`: 设置本地的连接信息
* `setRemoteDescription(description)`: 设置对端的连接信息

```
peerConnection.setRemoteDescription(new RTCSessionDescription(remoteOffer))
    .then(() => peerConnection.createAnswer())
    .then(answer => peerConnection.setLocalDescription(answer));
```
#### 6.3.3. 处理媒体流
* `addTrack(track, stream)`: 添加音视频轨道到连接中
* `replaceTrack(track)`: 切换音视频轨道
* `addIceCandidate(candidate)`: 添加网络候选地址

```
navigator.mediaDevices.getUserMedia({ video: true, audio: true })
    .then(stream => {
        stream.getTracks().forEach(track => peerConnection.addTrack(track, stream));
    });

const tracks = stream.getTracks();
peerConnection.replaceTrack(tracks);
peerConnection.addIceCandidate(new RTCIceCandidate(candidate));
```
#### 6.3.4. 事件处理
* `onicecandidate`: 当新的网络候选地址出现时触发
* `ontrack`: 当接收到对端的媒体流时触发
* `oniceconnectionstatechange`: 当连接状态变化时触发

```
peerConnection.onicecandidate = (event) => {
    if (event.candidate) {
        // 发送候选地址到信令服务器
    }
};

peerConnection.ontrack = (event) => {
    const remoteStream = event.streams[0];
    // 显示远程媒体流
    videoElement.srcObject = remoteStream;
};
```
### 6.4 RTCDataChannel
用于在对等端之间传输任意数据的 API，例如文件、文本消息的传输。

```
// 创建数据通道
const dataChannel = peerConnection.createDataChannel("myDataChannel");

// 发送数据
dataChannel.send("Hello, WebRTC!");

// 接收数据
dataChannel.onmessage = (event) => {
    console.log("Received message:", event.data);
};

// 事件处理
dataChannel.onopen = () => {
    console.log("Data channel is open");
};

dataChannel.onclose = () => {
    console.log("Data channel is closed");
};
```
---

## 7. WebRTC 连接
### 7.1 核心概念
|术语|描述|
|-|-|
|**信令（Signaling）**|建立连接前交换控制信息的过程，包括SDP的创建和交换|
|**SDP（Session Description Protocol）**|会话描述协议，用于交换多媒体会话的参数|
|**ICE（Interactive Connectivity Establishment）**|网络协议，用于在网络NAT（Network Address Translation，网络地址转换）或防火墙之后的用户之间建立直接的点对点（P2P）连接。ICE的主要目标是简化在复杂网络环境中的连接建立过程。|
|**Candidate**|网络候选描述，提供可能的网络路径，用于尝试在两个PeerConnection之间建立连接，包括IP地址和端口号等参数|
|**Offer 和 Answer**|SDP 交换过程中，发起方创建一个"offer"描述其媒体能力，接收方根据offer创建一个"answer"|
|**STUN 服务器**|（Session Traversal Utilities for NAT）当两个设备不在同一局域网内时，需要使用 **STUN 服务器**来帮助设备发现自己的公共 IP 地址，从而建立连接|
|**TURN 服务器**|（Traversal Using Relays around NAT），当 P2P 连接失败时，**TURN 服务器**可以作为中间服务器来转发数据，保证通信的连续性。|

### 7.2 WebRTC 的连接流程
```
sequenceDiagram
    autonumber

    participant A as Peer A（呼叫端）
    participant S as 信令服务器
    participant B as Peer B（被呼叫端）
    participant ST as STUN/TURN 服务器

    %% ===============================
    %% 1. 连接信令服务器
    %% ===============================
    rect rgb(230,255,230)
        Note over A: 第一步 连接到信令服务器 用于信令交换
        A->>S: 建立信令连接
        B->>S: 建立信令连接
        S-->>A: 连接成功
        S-->>B: 连接成功
    end

    %% ===============================
    %% 2. A 创建 PeerConnection
    %% ===============================
    rect rgb(240,240,255)
        Note over A: 第二步 A 创建 RTCPeerConnection 并加入媒体流
        A->>A: 创建 RTCPeerConnection
        A->>A: 添加本地媒体流
    end

    %% ===============================
    %% 3. A 创建并发送 SDP Offer
    %% ===============================
    rect rgb(255,245,230)
        Note over A: 第三步 A 创建 SDP Offer 并发送给 B
        A->>A: createOffer
        A->>A: setLocalDescription Offer
        A->>ST: 获取公共 IP 地址
        A->>S: 发送 SDP Offer
        S->>B: 转发 SDP Offer
    end

    %% ===============================
    %% 4. B 处理 Offer 并回答 Answer
    %% ===============================
    rect rgb(255,240,240)
        Note over A: 第四步 B 接收 Offer 并创建 Answer
        B->>B: 创建 RTCPeerConnection
        B->>B: setRemoteDescription Offer
        B->>B: createAnswer
        B->>B: setLocalDescription Answer
        B->>ST: 公共 IP 地址
        B->>S: 发送 SDP Answer
        S->>A: 转发 SDP Answer
        A->>A: setRemoteDescription Answer
    end

    %% ===============================
    %% 5. 交换 ICE Candidate
    %% ===============================
    rect rgb(240,255,255)
        Note over A: 第五步 交换 ICE 候选信息
        A->>A: 收集 ICE 候选
        B->>B: 收集 ICE 候选

        A->>S: 发送 ICE 候选
        S->>B: 转发 ICE 候选
        B->>B: addIceCandidate

        B->>S: 发送 ICE 候选
        S->>A: 转发 ICE 候选
        A->>A: addIceCandidate
    end

    %% ===============================
    %% 6. 建立 P2P 连接
    %% ===============================
    rect rgb(230,245,255)
        Note over A: 第六步 建立点对点 P2P 连接
        A-->>B: 成功建立 P2P 通道
        B->>B: onAddStream 收到远端媒体流
    end
```
1. **连接信令服务器**: Peer A 和 Peer B 分别与信令服务器建立连接
2. **创建 RTCPeerConnection 对象**: Peer A 创建 RTCPeerConnection 对象
3. **创建并发送 SDP Offer**: Peer A 生成 SDP Offer 并发送给 Peer B
4. **Peer B 处理 SDP Offer**: Peer B 接收 Offer 并创建 Answer
5. **Peer A 处理 SDP Answer**: Peer A 接收 Answer 并设置远程描述
6. **交换 ICE 候选信息**: 双方收集并交换 ICE 候选信息
7. **建立 P2P 连接**: 完成连接建立，开始媒体流传输

---

# 方案一：原生WebRTC方案
# 技术框架
## 整体架构
采用 **WebRTC + WebSocket** 的双通道架构：

* **WebSocket 通道**：用于信令交换（SDP Offer/Answer、ICE Candidate）
* **WebRTC DataChannel**：用于控制消息传输
* **WebRTC MediaStream**：用于视频流传输（SRTP 加密的 VP8/H264 编码）

```
sequenceDiagram
    participant Frontend as 🖥️ 前端(Peer A)
    participant WS as 📡 WebSocket
    participant Backend as 🖥️ 后端(Peer B)
    participant DC as 📨 DataChannel
    participant MediaStream as 🎬 MediaStream

    rect rgb(230,255,230)
    Note over Frontend,Backend: 1. WebSocket 连接建立
    Frontend->>WS: 连接 ws://xxxx:xxxx
    WS->>Backend: 建立连接
    end

    rect rgb(230,255,255)
    Note over Frontend,Backend: 2. WebRTC 信令交换（通过 WebSocket）
    Frontend->>Frontend: 创建 RTCPeerConnection
    Frontend->>WS: 发送 SDP Offer
    WS->>Backend: 转发 Offer
    Backend->>Backend: 处理 Offer，创建 Answer
    Backend->>WS: 发送 SDP Answer
    WS->>Frontend: 转发 Answer
    end

    rect rgb(255,255,230)
    Frontend->>WS: 发送 ICE Candidates
    WS->>Backend: 转发 ICE Candidates
    Backend->>WS: 发送 ICE Candidates
    WS->>Frontend: 转发 ICE Candidates
    end

    rect rgb(230,215,245)
    Note over Frontend,Backend: 3. WebRTC 连接建立完成
    Note over Frontend,Backend: 4. DataChannel 建立（通过 WebRTC）
    Frontend->>DC: 创建 DataChannel
    DC->>Backend: DataChannel 打开
    Backend->>DC: 发送 消息
    DC->>Frontend: 接收 消息
    end

    rect rgb(210,215,230)
    Note over Frontend,Backend: 5. 媒体流传输开始（通过 WebRTC）
    Backend->>MediaStream: 推送视频帧
    MediaStream->>Frontend: 传输视频流 (SRTP)
    Frontend->>Frontend: 解码并播放视频
    end

    rect rgb(200,205,200)
    Note over Frontend,Backend: 6. 控制消息传输（通过 DataChannel）
    Frontend->>DC: 发送 消息（prompt）
    DC->>Backend: 接收控制命令
    Backend->>Backend: 处理命令，切换视频
    Backend->>MediaStream: 推送新视频流
    MediaStream->>Frontend: 传输新视频流
    end
```


## 实现步骤
### WebSocket 连接建立
**前端 (**`webrtcClient.ts`**)**:

```
// 1. 创建 WebSocket 连接
this.ws = new WebSocket('ws://localhost:8080');

// 2. 创建 RTCPeerConnection
this.pc = new RTCPeerConnection({
    iceServers: [{urls: 'stun:stun.l.google.com:19302'}]
});

// 3. 添加仅接收的视频收发器
this.pc.addTransceiver('video', { direction: 'recvonly' });

// 4. 创建 DataChannel
this.dataChannel = this.pc.createDataChannel('control', {ordered: true});
```
**后端 (**`index.ts`**)**:

```
// 1. 创建 WebSocket 服务器
const wss = new WebSocketServer({port: 8080});

// 2. 创建会话
wss.on('connection', async (ws) => {
    // 初始化 StreamManager 和 PeerSession
    const streamManager = new StreamManager();
    const peerSession = new PeerSession(streamManager);
    // ...
});
```
###  信令交换（SDP Offer/Answer）
```
// 前端创建 Offer
const offer = await this.pc.createOffer();
await this.pc.setLocalDescription(offer);
this.ws.send(JSON.stringify({ type: 'offer', offer }));

// 后端处理 Offer 并创建 Answer
const videoSource = await this.streamManager.getCurrentSource();
this.currentSender = this.pc.addTrack(videoSource.getTrack());
await this.pc.setRemoteDescription(offer);
const answer = await this.pc.createAnswer();
await this.pc.setLocalDescription(answer);
```
### ICE Candidate 交换
```
// 前端发送 ICE Candidate
this.pc.onicecandidate = (event) => {
    if (event.candidate) {
        this.ws.send(JSON.stringify({
            type: 'ice-candidate',
            candidate: event.candidate
        }));
    }
};

// 后端处理 ICE Candidate
await this.pc.addIceCandidate(candidate);
```
### DataChannel 建立
```
// 前端创建 DataChannel
this.dataChannel = this.pc.createDataChannel('control', {ordered: true});
this.dataChannel.onopen = () => console.log('✅ DataChannel opened');

// 后端监听 DataChannel
this.pc.ondatachannel = (event) => {
    this.dataChannel = event.channel;
    this.dataChannel.onopen = () => {
        ...
    };
};
```
### 消息传输
```
// 前端发送消息
async sendMsg(): Promise<void> {
    if (this.dataChannel?.readyState === 'open') {
        this.dataChannel.send('msg:xxx');
    }
}

// 后端处理消息
this.dataChannel.onmessage = async (event: any) => {
    if (message === 'msg:xxx') {
        await this.streamManager.waitForCurrentVideoComplete();
        await this.streamManager.next();
        this.sendCurrentIndex();
        await this.switchVideoTrack();
    }
}
```
### 视频流传输
**推送视频帧** → `RTCVideoSource.onFrame()`****

```
import wrtc from '@koush/wrtc';
const {nonstandard} = wrtc;
async initialize(): Promise<void> {
    // 使用非标准 API 创建视频源
    const videoSource = new nonstandard.RTCVideoSource();
    this.videoTrack = videoSource.createTrack();

    // 处理视频数据
    ...

    // 启动帧推送
    this.startFramePushing(videoSource);
}

private startFramePushing(videoSource: any): void {
    this.isPlaying = true;
    this.currentFrameIndex = 0;
    const interval = 1000 / this.fps; // 例如 30fps = 33.33ms/帧

    this.frameInterval = setInterval(() => {
        // 检查是否播放完成
        if (this.currentFrameIndex >= this.frameBuffer.length) {
            this.stop();
            return;
        }

        // 获取当前帧数据
        const frameData = this.frameBuffer[this.currentFrameIndex];
        this.currentFrameIndex++;

        // 推送到 WebRTC
        videoSource.onFrame({
            width: this.width,
            height: this.height,
            data: frameData
        });
    }, interval);
}
```
### 视频流切换
使用`replaceTrack()`

```
// 添加视频轨道
async handleOffer(offer: RTCSessionDescriptionType) {
    const videoSource = await this.streamManager.getCurrentSource();
    const videoTrack = videoSource.getTrack();

    // 添加到 PeerConnection
    this.currentSender = this.pc.addTrack(videoTrack);

    // 然后设置远程描述
    await this.pc.setRemoteDescription(offer);

    // 创建 Answer（此时 SDP 会包含视频轨道信息）
    const answer = await this.pc.createAnswer();
    await this.pc.setLocalDescription(answer);
}

// 切换视频
private async switchVideoTrack(): Promise<void> {
    // 获取新的视频源
    const videoSource = await this.streamManager.getCurrentSource();
    const newTrack = videoSource.getTrack();

    // 使用 replaceTrack 无缝切换（不会中断连接）
    await this.currentSender.replaceTrack(newTrack);
}
```
### 前端接收渲染
1. **前端接收** → `RTCPeerConnection.ontrack` 事件
2. **前端解码播放** → `<video>` 标签自动解码

```
export class WebRTCClient {
      constructor(wsUrl: string = 'ws:xxx') {
        this.wsUrl = wsUrl;
        this.pc = new RTCPeerConnection({
            iceServers: [{urls: 'stun:xxx'}]
        });

        this.pc.ontrack = event => {
            let stream: MediaStream | null = null;
            if (event.streams && event.streams.length > 0) {
                stream = event.streams[0];
            } else {
                stream = new MediaStream([event.track]);
            }

            if (stream) {
                for (const listener of this.listeners) {
                    listener(stream);
                }
            }
        };
    }

    onRemoteStream(callback: StreamListener): () => void {
        this.listeners.add(callback);
    }
    ...
}


const connection = new WebRTCClient();
connectionRef.current = connection;
connection.onRemoteStream(stream => {
if (videoRef.current) {
     videoRef.current.srcObject = stream;
     videoRef.current.play()
    }
});

```
# 效果演示
[附件]


# 移动端兼容情况


|设备|百度端内|浏览器|是否兼容|
|-|-|-|-|
|iphone 16|![](https://rte.weiyun.baidu.com/wiki/attach/image/api/imageDownloadAddress?attachId=6a1716ad77624f99b55baef0c45cb369&docGuid=nXT_pIUOzE3auq "")|![](https://rte.weiyun.baidu.com/wiki/attach/image/api/imageDownloadAddress?attachId=98dc5c8020484565a39949ad9b6fbf10&docGuid=nXT_pIUOzE3auq "")|兼容|
|vivoX27|![](https://rte.weiyun.baidu.com/wiki/attach/image/api/imageDownloadAddress?attachId=945d420b45c8467790f490c609d35ed1&docGuid=nXT_pIUOzE3auq "")|![](https://rte.weiyun.baidu.com/wiki/attach/image/api/imageDownloadAddress?attachId=01d089ee2e7d4fa5b7e73953f76cd201&docGuid=nXT_pIUOzE3auq "")|兼容|
|nova 10z(HarmonyOs)|![](https://rte.weiyun.baidu.com/wiki/attach/image/api/imageDownloadAddress?attachId=f33930f0fdc84e6497004eacd59dcac5&docGuid=nXT_pIUOzE3auq "")|![](https://rte.weiyun.baidu.com/wiki/attach/image/api/imageDownloadAddress?attachId=8837f504d26447328bc66d1bf0751bc3&docGuid=nXT_pIUOzE3auq "")|兼容|
|iPhone 12|![](https://rte.weiyun.baidu.com/wiki/attach/image/api/imageDownloadAddress?attachId=5f1390234c854983bdc98ee119543746&docGuid=nXT_pIUOzE3auq "")|![](https://rte.weiyun.baidu.com/wiki/attach/image/api/imageDownloadAddress?attachId=6748dfc7f82842ff8b487f72d06eb402&docGuid=nXT_pIUOzE3auq "")|兼容|



## 实现代码
icode demo:[文件页：baidu/personal-code/webrtc_  *master](https://console.cloud.baidu-int.com/devops/icode/repos/baidu/personal-code/webrtc_/tree/master?t=mention&mt=doc&dt=sdk)



# 方案二：[百度云RTC web SDK 渲染 + java SDK推流方案](https://cloud.baidu.com/doc/RTC/s/Uk3o54lnm)
> 百度云rtc没有提供推真实数据的demo,真实数据需要接入java SDK自己推视频
# 整体架构
## 架构概览
```
graph LR
  subgraph 客户端
    A[RTC Web SDK]
    B[web前端]
  end
  subgraph 百度云中台服务
    C[业务服务 Backend]
    D[鉴权服务<br>Token生成]
  end
  subgraph 百度云 RTC
    E[RTC 信令/媒体服务]
    F[记录/转码/媒体存储]
  end
  subgraph 推流
    G[Java SDK 实时推流]
    H[音视频源<br>编码器]
  end
  B --> A
  A -- 加入房间 / 发布媒体 --> E
  A -- 拉流 --> E
  C -- REST API --> D
  D -- Token/签名 --> A
  C -- 业务控制 --> G
  G -- 推流 --> E
  E -- 录制/转码 --> F
```
## 时序图
```
sequenceDiagram
  participant User as Web SDK
  participant Backend as 鉴权/业务服务
  participant JavaApp as Java SDK 推流
  participant RTC as 百度云 RTC
  participant Storage as 录制/转码

  User->>Backend: 发起登录/入会请求
  Backend->>User: 返回房间配置 & Token
  User->>RTC: 初始化 Web SDK，加入房间
  RTC-->>User: 返回入会结果、远端流列表

  JavaApp->>Backend: 请求推流配置/Token
  Backend->>JavaApp: 返回房间号、推流 Token
  JavaApp->>RTC: 作为推流端加入房间
  JavaApp->>RTC: 发布外部音视频流
  RTC-->>User: 下发推流媒体


  RTC-->>JavaApp: 分发需订阅的媒体

  Note over RTC,Storage: 若开启云端录制/转码
  RTC->>Storage: 触发录制/转码
  Storage-->>Backend: 回调录制状态/文件信息
```


# 接入流程
[RTC实时音视频接入流程](https://cloud.baidu.com/doc/RTC/s/Qjxbh7jpu)

![](https://rte.weiyun.baidu.com/wiki/attach/image/api/imageDownloadAddress?attachId=1994bc9315674939ad63cd01920cda67&docGuid=nXT_pIUOzE3auq "")
## 服务开通与配置
### 1.1 开通服务
参考文档：[https://cloud.baidu.com/doc/RTC/s/Nlgt0pkcl](https://cloud.baidu.com/doc/RTC/s/Nlgt0pkcl)

**步骤：**

1. 登录百度智能云控制台
2. 完成实名认证
3. 开通RTC服务
4. 创建应用，获取 `AppID` 和 `AppKey`

### 1.2 获取Token
Token用于鉴权，需要通过后端服务生成。

**Token生成接口（示例）：**

```
/**
 * Token生成请求参数
 */
interface TokenRequest {
    /** 应用ID */
    appId: string;
    /** 用户ID */
    userId: string;
    /** 房间ID */
    roomName: string;
    /** 过期时间（秒），默认3600 */
    expireTime?: number;
}

/**
 * Token生成响应
 */
interface TokenResponse {
    /** 生成的Token字符串 */
    token: string;
    /** Token过期时间戳（秒） */
    expireAt: number;
}

/**
 * 生成Token（需要在后端实现）
 * @param params Token生成参数
 * @returns Promise<TokenResponse>
 */
async function generateToken(params: TokenRequest): Promise<TokenResponse> {
    // 实现Token生成逻辑
    // 参考：https://cloud.baidu.com/doc/RTC/s/Qjxbh7jpu
    throw new Error('需要在后端实现Token生成逻辑');
}
```
---

## Web端接入（渲染客户端）
### 2.1 引入SDK
```
<!-- 方式1: 通过CDN引入 -->
<script src="https://brtc-sdk.cdn.bcebos.com/npm/baidurtc@1.2.20/baidu.rtc.sdk.js"></script>

<!-- 方式2: 通过npm安装 -->
npm install baidurtc
```
### 2.2 类型定义
```
/**
 * 百度RTC全局对象类型定义
 */
declare global {
    interface Window {
        /** 初始化并启动RTC客户端 */
        BRTC_Start: (config: BRTCStartConfig) => void;

        /** 停止RTC客户端并释放资源 */
        BRTC_Stop: () => void;

        /** 开始发布本地音视频流 */
        BRTC_StartPublish: () => Promise<void>;

        /** 停止发布本地音视频流 */
        BRTC_StopPublish: () => Promise<void>;

        /** 订阅远程视频流，viewId为视频视图容器ID，feedId为远程流ID（用户ID） */
        BRTC_SubscribeStreaming: (viewId: string, feedId: number) => void;

        /** 取消订阅远程视频流，feedId为远程流ID（用户ID） */
        BRTC_StopSubscribeStreaming: (feedId: number) => void;

        /** 替换当前推流的媒体流，用于动态切换视频源（摄像头/屏幕分享/Canvas流） */
        BRTC_ReplaceStream: (stream: MediaStream) => void;

        /** 替换视频输入设备（摄像头），deviceId可通过BRTC_GetVideoDevices获取 */
        BRTC_ReplaceVideo: (deviceId: string) => void;

        /** 替换音频输入设备（麦克风），deviceId可通过BRTC_GetAudioInputDevices获取 */
        BRTC_ReplaceAudio: (deviceId: string) => void;

        /** 静音/取消静音麦克风，muted为true表示静音，false表示取消静音 */
        BRTC_MuteMicphone: (muted: boolean) => void;

        /** 关闭/开启摄像头，muted为true表示关闭摄像头，false表示开启摄像头 */
        BRTC_MuteCamera: (muted: boolean) => void;

        /** 通过数据通道发送数据消息，data为要发送的数据字符串 */
        BRTC_SendData: (data: string) => void;

        /** 获取可用的视频输入设备列表（摄像头），通过callback回调返回设备列表 */
        BRTC_GetVideoDevices: (callback: DeviceCallback) => void;

        /** 获取可用的音频输入设备列表（麦克风），通过callback回调返回设备列表 */
        BRTC_GetAudioInputDevices: (callback: DeviceCallback) => void;

        /** 获取可用的音频输出设备列表（扬声器/耳机），通过callback回调返回设备列表 */
        BRTC_GetAudioOutputDevices: (callback: DeviceCallback) => void;

        /** 设置音频输出设备（扬声器/耳机），deviceId可通过BRTC_GetAudioOutputDevices获取，传入'default'表示使用默认设备 */
        BRTC_SetAudioOutputDevice: (deviceId: string) => void;

        /** 设置RTC参数配置，用于动态更新设备ID等参数，无需重新初始化 */
        BRTC_SetParamSettings: (settings: ParamSettings) => void;

        /** 获取远程音频音量级别，返回包含feedId和volume的数组 */
        BRTC_GetRemoteAudioLevels: () => Array<{feedId: number; volume: number}>;

        /** 设置用户属性，用于在房间中传递用户自定义信息（如昵称、头像等），attribute为用户属性JSON字符串 */
        BRTC_SetUserAttribute: (attribute: string) => void;

        /** 切换到屏幕分享，会触发浏览器屏幕分享权限请求，需要先调用BRTC_Start初始化且aspublisher为true */
        BRTC_SwitchScreen: () => void;
    }
}

/**
 * BRTC_Start 配置参数
 */
interface BRTCStartConfig {
    /** 调试级别：true/false/'all'/['debug','log','error'] */
    debuglevel?: boolean | string | string[];
    /** BRTC服务器地址，如：wss://rtc.exp.bcelive.com/janus */
    server?: string;
    /** 应用ID */
    appid: string;
    /** 鉴权Token */
    token: string;
    /** 房间名称 */
    roomname: string;
    /** 视频编码：'h264' | 'vp8' | 'vp9' | 'h263' | 'h265' | 'h266' | 'av1' */
    videocodec?: string;
    /** 音频编码：'opus' | 'pcmu' | 'pcma' | 'g722' | 'isac16' | 'isac32' */
    audiocodec?: string;
    /** 用户ID */
    userid: string;
    /** 用户显示名称 */
    displayname?: string;
    /** 远程视频视图ID（用于显示远程视频） */
    remotevideoviewid?: string;
    /** 本地视频视图ID（用于显示本地视频） */
    localvideoviewid?: string;
    /** 远程视频视图ID 2-5（多路视频） */
    remotevideoviewid2?: string;
    remotevideoviewid3?: string;
    remotevideoviewid4?: string;
    remotevideoviewid5?: string;
    /** 是否镜像本地视频 */
    mirrorlocalvideo?: boolean;
    /** 是否作为发布者 */
    aspublisher?: boolean;
    /** 是否使用数据通道 */
    usingdatachannel?: boolean;
    /** 是否使用视频 */
    usingvideo?: boolean;
    /** 是否使用音频 */
    usingaudio?: boolean;
    /** 是否接收视频 */
    receivingvideo?: boolean;
    /** 是否接收音频 */
    receivingaudio?: boolean;
    /** 视频码率（kbps） */
    bitrate?: number;
    /** 屏幕分享码率（kbps） */
    screenbitrate?: number;
    /** 音频码率（kbps） */
    audiobitrate?: number;
    /** 是否立体声 */
    audiostereo?: boolean;
    /** 是否显示视频码率 */
    showvideobps?: boolean;
    /** 是否屏幕分享 */
    sharescreen?: boolean;
    /** 音频设备ID */
    audiodeviceid?: string | null;
    /** 视频设备ID */
    videodeviceid?: string | null;
    /** 音频输出设备ID */
    audiooutputdeviceid?: string | null;
    /** RTMP服务器地址 */
    rtmpserver?: string;
    /** RTMP混流模板 */
    rtmpmixtemplate?: string;
    /** 是否RTMP混流 */
    rtmpmix?: boolean;
    /** RTMP混流布局索引 */
    rtmpmixlayoutindex?: string;
    /** 是否录制 */
    recording?: boolean;
    /** 是否自动订阅 */
    autosubscribe?: boolean;
    /** 是否静音自动播放 */
    autoplaymuted?: boolean;
    /** 是否自动发布 */
    autopublish?: boolean;
    /** 是否自动重新订阅 */
    autoresubscribing?: boolean;
    /** 链路断开阈值（百分比） */
    linkdownupthreshold?: number;
    /** 等待权限超时时间（毫秒） */
    waitpermissiontimeoutms?: number;
    /** 是否上报监控数据 */
    reportmonitordata?: boolean;
    /** 上报环境：'online' | 'test' */
    reportenv?: string;
    /** 候选IP地址（用于NAT穿透） */
    candidateip?: string | null;
    /** 候选端口（用于NAT穿透） */
    candidateport?: number | null;
    /** 媒体服务器IP */
    mediaserverip?: string | null;
    /** 视频配置：'lowres' | 'stdres' | 'hires' | 'fhdres' | '4kres' | '480x480@15' | MediaTrackConstraints */
    videoprofile?: string | MediaTrackConstraints;
    /** 屏幕分享视频配置 */
    screensharevideoprofile?: string;
    /** 是否多流SDK */
    multistream?: boolean;
    /** 是否使用V4协议 */
    v4?: boolean;
    /** 链路断开事件回调 */
    linkdownevent?: () => void;
    /** 链路恢复事件回调 */
    linkupevent?: () => void;
    /** 是否启用用户事件 */
    userevent?: boolean;
    /** 用户加入房间事件回调 */
    userevent_joinedroom?: (id: number, display: string, attribute: string) => void;
    /** 用户离开房间事件回调 */
    userevent_leavingroom?: (id: number, display: string) => void;
    /** 是否启用会话事件 */
    sessionevent?: boolean;
    /** 媒体状态回调 */
    mediastate?: (medium: string, on: boolean) => void;
    /** 登录超时时间（毫秒） */
    logintimeout?: number;
    /** 登录超时事件回调 */
    logintimeoutevent?: () => void;
    /** 是否显示加载动画 */
    showspinner?: boolean;
    /** 是否显示无视频提示 */
    shownovideo?: boolean;
    /** 远程视频加载中回调 */
    remotevideoloading?: (idx: number) => void;
    /** 远程视频开启回调 */
    remotevideoon?: (idx: number) => void;
    /** 远程视频关闭回调 */
    remotevideooff?: (idx: number) => void;
    /** 远程视频到来回调 */
    remotevideocoming?: (id: number, display: string, attribute: string) => void;
    /** 远程视频连接状态回调 */
    remotevideoconnected_state?: (id: number, on: boolean) => void;
    /** 是否使用远程媒体状态 */
    usingremotemediastate?: boolean;
    /** 远程媒体状态回调 */
    remotemediastating?: (id: number, medium: string, on: boolean) => void;
    /** 远程视频离开回调 */
    remotevideoleaving?: (id: number) => void;
    /** 远程视频取消发布回调 */
    remotevideounpublished?: (id: number) => void;
    /** 远程数据接收回调 */
    remotedata?: (data: string, label: string) => void;
    /** 成功回调 */
    success?: () => void;
    /** 本地视频发布中回调 */
    localvideopublishing?: () => void;
    /** 本地视频连接状态回调 */
    localvideoconnected_state?: (on: boolean) => void;
    /** 本地视频发布成功回调 */
    localvideopublished_ok?: () => void;
    /** 错误回调 */
    error?: (error: string) => void;
    /** 销毁回调 */
    destroyed?: (error?: string) => void;
    /** 本地流回调 */
    onlocalstream?: (stream: MediaStream, name: string) => void;
    /** 本地流结束回调 */
    onlocalstream_end?: (name: string) => void;
    /** 远程视频关闭回调（服务器主动关闭） */
    remotevideo_closed?: (feedid: number) => void;
    /** 消息接收回调 */
    onmessage?: (msg: {id: number; data: string}) => void;
    /** 属性变更回调 */
    onattribute?: (a: {id: number; attribute: string}) => void;
    /** RTMP推流回调 */
    onrtmpstreaming?: (mode: string, url: string, status: string) => void;
    /** 录制回调 */
    onrecording?: (e: {status: string}) => void;
    /** 消息延迟信息回调 */
    onmsgdelayinfo?: (e: {id: number; data: string; delayms: number}) => void;
    /** 获取屏幕分享流回调（返回Promise<MediaStream>） */
    ongetsharescreenstream?: () => Promise<MediaStream>;
}

/**
 * 设备回调接口
 */
interface DeviceCallback {
    /** 成功回调 */
    success: (devices: MediaDeviceInfo[]) => void;
    /** 失败回调（可选） */
    error?: (error: Error) => void;
}

/**
 * 参数设置接口
 */
interface ParamSettings {
    /** 视频设备ID */
    videodeviceid?: string;
    /** 音频设备ID */
    audiodeviceid?: string;
    /** 音频输出设备ID */
    audiooutputdeviceid?: string;
}
```
### 2.3 web端使用
```
/**
 * 百度RTC Web端封装
 * 用于渲染远程视频流（作为接收端）
 */
export class BaiduRTCWebClient {
    private appId: string;
    private userId: string;
    private roomName: string;
    private token: string;
    private server?: string;
    private isInitialized: boolean = false;
    private remoteStreams: Map<number, MediaStream> = new Map();

    /**
     * 构造函数
     * @param config 初始化配置
     */
    constructor(config: {
        appId: string;
        userId: string;
        roomName: string;
        token: string;
        server?: string;
    }) {
        this.appId = config.appId;
        this.userId = config.userId;
        this.roomName = config.roomName;
        this.token = config.token;
        this.server = config.server;
    }

    /**
     * 初始化并加入房间
     * @param config 初始化配置
     * @returns Promise<void>
     */
    async initialize(config: {
        /** 本地视频视图容器ID（可选，如果不推流可不传） */
        localVideoViewId?: string;
        /** 远程视频视图容器ID */
        remoteVideoViewId: string;
        /** 是否自动订阅远程流 */
        autoSubscribe?: boolean;
        /** 是否自动发布本地流（作为接收端通常为false） */
        autoPublish?: boolean;
    }): Promise<void> {
        return new Promise((resolve, reject) => {
            if (this.isInitialized) {
                reject(new Error('RTC客户端已初始化'));
                return;
            }

            const startConfig: BRTCStartConfig = {
                // 基础配置
                appid: this.appId,
                token: this.token,
                roomname: this.roomName,
                userid: this.userId,
                server: this.server,

                // 视频视图配置
                localvideoviewid: config.localVideoViewId,
                remotevideoviewid: config.remoteVideoViewId,

                // 作为接收端配置
                aspublisher: config.autoPublish ?? false,
                autosubscribe: config.autoSubscribe ?? true,
                autopublish: config.autoPublish ?? false,

                // 媒体配置
                usingvideo: false, // 接收端不推视频
                usingaudio: false, // 接收端不推音频
                receivingvideo: true, // 接收视频
                receivingaudio: true, // 接收音频

                // 视频编码配置
                videocodec: 'h264',
                audiocodec: 'opus',

                // 事件回调
                success: () => {
                    this.isInitialized = true;
                    console.log('RTC客户端初始化成功');
                    resolve();
                },
                error: (error: string) => {
                    console.error('RTC客户端初始化失败:', error);
                    reject(new Error(error));
                },
                remotevideocoming: (id: number, display: string, attribute: string) => {
                    console.log(`远程视频流到来: feedId=${id}, display=${display}`);
                    // 自动订阅（如果启用）
                    if (config.autoSubscribe) {
                        this.subscribeStream(id, config.remoteVideoViewId);
                    }
                },
                remotevideoleaving: (id: number) => {
                    console.log(`远程视频流离开: feedId=${id}`);
                    this.remoteStreams.delete(id);
                },
                remotevideoon: (idx: number) => {
                    console.log(`远程视频${idx}已开启`);
                },
                remotevideooff: (idx: number) => {
                    console.log(`远程视频${idx}已关闭`);
                },
            };

            // 调用SDK初始化
            if (typeof window !== 'undefined' && window.BRTC_Start) {
                window.BRTC_Start(startConfig);
            } else {
                reject(new Error('百度RTC SDK未加载，请先引入SDK'));
            }
        });
    }

    /**
     * 订阅远程视频流
     * @param feedId 远程流ID
     * @param viewId 视频视图容器ID
     * @returns void
     */
    subscribeStream(feedId: number, viewId: string): void {
        if (!this.isInitialized) {
            throw new Error('RTC客户端未初始化');
        }

        if (window.BRTC_SubscribeStreaming) {
            window.BRTC_SubscribeStreaming(viewId, feedId);
            console.log(`已订阅远程流: feedId=${feedId}, viewId=${viewId}`);
        } else {
            throw new Error('BRTC_SubscribeStreaming方法不存在');
        }
    }

    /**
     * 取消订阅远程视频流
     * @param feedId 远程流ID
     * @returns void
     */
    unsubscribeStream(feedId: number): void {
        if (!this.isInitialized) {
            throw new Error('RTC客户端未初始化');
        }

        if (window.BRTC_StopSubscribeStreaming) {
            window.BRTC_StopSubscribeStreaming(feedId);
            console.log(`已取消订阅远程流: feedId=${feedId}`);
            this.remoteStreams.delete(feedId);
        } else {
            throw new Error('BRTC_StopSubscribeStreaming方法不存在');
        }
    }

    /**
     * 获取所有远程流ID列表
     * @returns number[] 远程流ID数组
     */
    getRemoteStreamIds(): number[] {
        return Array.from(this.remoteStreams.keys());
    }

    /**
     * 停止并清理资源
     * @returns void
     */
    stop(): void {
        if (this.isInitialized && window.BRTC_Stop) {
            window.BRTC_Stop();
            this.isInitialized = false;
            this.remoteStreams.clear();
            console.log('RTC客户端已停止');
        }
    }
}

/**
 * Web端使用示例
 */
async function webClientExample() {
    // 1. 初始化客户端
    const client = new BaiduRTCWebClient({
        appId: 'your-app-id',
        userId: 'web-user-001',
        roomName: 'room-001',
        token: 'your-token-here',
        server: 'wss://xxx',
    });

    try {
        // 2. 初始化并加入房间
        await client.initialize({
            remoteVideoViewId: 'remote-video-container',
            autoSubscribe: true, // 自动订阅远程流
            autoPublish: false, // 不推流
        });

        console.log('已加入房间，等待接收视频流...');

        // 3. 手动订阅特定流（如果需要）
        // client.subscribeStream(12345, 'remote-video-container');

        // 4. 监听远程流变化
        // 可以通过事件回调处理

    } catch (error) {
        console.error('初始化失败:', error);
    }

    // 5. 页面卸载时清理
    window.addEventListener('beforeunload', () => {
        client.stop();
    });
}
```


## javaSDK
详见：[https://cloud.baidu.com/doc/RTC/s/rlkj7gdxx](https://cloud.baidu.com/doc/RTC/s/rlkj7gdxx)

# 成本计算
参考[实时音视频RTC计费价格说明](https://cloud.baidu.com/product-price/rtc.html)

![](https://rte.weiyun.baidu.com/wiki/attach/image/api/imageDownloadAddress?attachId=af1a055a942f4142a633de3f62244a94&docGuid=nXT_pIUOzE3auq "")


# 方案三：百度云RTC+大模型实时互动+曦灵数字人 方案
> 大模型实时互动+数字人demo: [https://brtc-sdk.bj.bcebos.com/ai/agent/h5/brtc_agent.html?a=apppibr915pu75m&tts=DHV2&it=DigitalHuman&fid=209403](https://brtc-sdk.bj.bcebos.com/ai/agent/h5/brtc_agent.html?a=apppibr915pu75m&tts=DHV2&it=DigitalHuman&fid=209403)
# 整体架构
![](https://rte.weiyun.baidu.com/wiki/attach/image/api/imageDownloadAddress?attachId=0c4f6d3f5087450f8bddaf503d0df8da&docGuid=nXT_pIUOzE3auq "")
# 接入流程
## 服务开通与配置
### 1.1 [RTC实时音视频开通-同上](https://ku.baidu-int.com/knowledge/HFVrC7hq1Q/pKzJfZczuc/a1C_Cp3G5K/nXT_pIUOzE3auq#anchor-e306ee61-beac-11f0-8532-d5cb1bdd38a9)
### 1.2 大模型实时互动服务开通
进入官网控制台，在大模型实时互动服务中，创建互动应用，配置互动服务（[帮助文档](https://cloud.baidu.com/doc/RTC/s/Hmcu5yhvh)）。开通服务后，可采用工单、邮件、联系客户经理3种方式申请免费试用授权（在线申请及购买即将开通），申请中需携带：用户账号、APPID、业务概述、服务区域（国内、欧美、东南亚等）、服务内容、联系方式内容。邮件地址：liuzhen29@baidu.com,youli01@baidu.com,keyugang@baidu.com 。 服务内容配置项主要包括：

* 互动类型：音频互动、视频互动、数字人互动（[链接](https://cloud.baidu.com/doc/RTC/s/Em9qgne0f)）、第三方模型互动([链接](https://cloud.baidu.com/doc/RTC/s/0m9qgqj7h))
* LLM参数：FunctionCalling、话题、人设、场景等
* TTS参数：发音人、倍速等
* 可选功能：**有声读物（音乐、故事、相声等）**、**地图导航**、同声翻译、**电话呼叫**、**声纹及情绪识别**等。注意：标粗为单独收费项

### 1.3 构建数字人互动实例
* 在曦灵管理平台创建数字人应用， 购买云渲染交互组件, 并绑定云渲染交互组件。[https://xiling.cloud.baidu.com/open/widgetStore/list](https://xiling.cloud.baidu.com/open/widgetStore/list)
* 获得曦灵数字人的AppID 和AppKey后使用如下的 Token 生成工具生成Token（根据业务需要选择token有效期，建议为900000小时）：[https://open.xiling.baidu.com/token-gen-tool.html](https://open.xiling.baidu.com/token-gen-tool.html)

## Web端接入
### 2.1 [引入SDK-同上](https://ku.baidu-int.com/knowledge/HFVrC7hq1Q/pKzJfZczuc/a1C_Cp3G5K/nXT_pIUOzE3auq?block_id=docyg-e306ee76-beac-11f0-8532-d5cb1bdd38a9)
### 2.2 [类型定义-同上](https://ku.baidu-int.com/knowledge/HFVrC7hq1Q/pKzJfZczuc/a1C_Cp3G5K/nXT_pIUOzE3auq?block_id=docyg-e306ee76-beac-11f0-8532-d5cb1bdd38a9)
```
/**
 * BRTC_Start 配置参数
 */
interface BRTCStartConfig {
    ...
    /** Agent配置对象（包含LLM、TTS、ASR等配置） */
    cfg: AgentConfig;
}

/**
 * Agent配置对象（cfg）
 *
 * @description 传递给后端API的配置对象，包含LLM、TTS、ASR等完整配置
 */
export interface AgentConfig {
    /** 标签标识 */
    tag: 'H5.SDK';
    /** 禁用语音自动中断 */
    disable_voice_auto_int?: boolean;
    /** TTS结束延迟（毫秒） */
    tts_end_delay_ms?: number;
    /** 模型类别 */
    llm?: LLMType;
    /** 模型地址URL */
    llm_url?: string;
    /** 模型配置 */
    llm_cfg?: string;
    /** LLM鉴权Token */
    llm_token?: string;
    /** LLM服务的http扩展头 */
    llm_extra_headers?: Record<string, string>;
    /** LLM服务的json扩展字段 */
    llm_extra_body?: Record<string, any>;
    /** 语言设置 */
    lang?: 'zh' | 'en';
    /** 招呼语 */
    tts_sayhi?: string;
    /** TTS类型 */
    tts?: TTSType;
    /** TTS的URL信息 */
    tts_url?: string;
    /** ASR类型 */
    asr?: ASRType;
    /** ASR的URL信息 */
    asr_url?: string;
    /** 智能体的角色设定 */
    role?: string;
    /** 位置设定 */
    location?: string;
    /** 曦灵数字人形象id */
    fid?: string;
    /** 曦灵数字人鉴权 Token */
    xiling_auth?: string;
    /** 配置曦灵数字人背景图URL */
    xiling_bgimg?: string;
    /** 数字人宽度 */
    dh_w?: number;
    /** 数字人高度 */
    dh_h?: number;
    /** ASR VAD配置 */
    asr_vad?: boolean;
    /** GIS配置 */
    gis?: string;
}

```
### 2.3 工具函数
```

/**
 * 解析URL参数
 *
 * @param url URL字符串（可选，默认使用window.location.search）
 * @returns URLParamsConfig URL参数配置对象
 *
 * @description 从URL查询参数中解析配置信息
 *
 * @example
 * const config = parseURLParams();
 * console.log(config.appid); // 输出AppID
 */
export function parseURLParams(url?: string): URLParamsConfig {
    const searchParams = new URLSearchParams(url || window.location.search);
    const config: URLParamsConfig = {};

    // 解析所有参数
    searchParams.forEach((value, key) => {
        switch (key) {
            case 'xxx':
            config.xxx = xxx
        }
    });

    return config;
}

/**
 * 构建Agent配置对象
 *
 * @param urlConfig URL参数配置
 * @returns AgentConfig Agent配置对象
 *
 * @description 从URL参数配置构建Agent配置对象
 *
 * @example
 * const urlConfig = parseURLParams();
 * const agentConfig = buildAgentConfig(urlConfig);
 */
export function buildAgentConfig(urlConfig: URLParamsConfig): AgentConfig {
    const config: AgentConfig = {
        tag: 'H5.SDK',
        disable_voice_auto_int: urlConfig.disable_voice_auto_int,
        tts_end_delay_ms: 20
    };

    if (urlConfig.llm_extra_headers) {
        try {
            config.llm_extra_headers = JSON.parse(urlConfig.llm_extra_headers);
        } catch (e) {
            console.warn('Failed to parse llm_extra_headers:', e);
        }
    }
    if (urlConfig.llm_extra_body) {
        try {
            config.llm_extra_body = JSON.parse(urlConfig.llm_extra_body);
        } catch (e) {
            console.warn('Failed to parse llm_extra_body:', e);
        }
    }

    return {...config, ...urlConfig};
}

/**
 * 处理环境配置
 *
 * @param urlConfig URL参数配置
 * @returns 处理后的配置（包含apihost和brtcserver）
 *
 * @description 根据env参数设置API主机地址和BRTC服务器地址
 */
export function handleEnvironmentConfig(urlConfig: URLParamsConfig): {
    apihost?: string;
    brtcserver?: string;
} {
    const result: {apihost?: string; brtcserver?: string} = {};

    if (urlConfig.env) {
        result.apihost = `https://ai.agent.kaywang.cn/${urlConfig.env}`;

        if (urlConfig.env === 'hk') {
            result.brtcserver = 'wss://rtc-out.exp.bcelive.com/janus';
        }
    }

    if (urlConfig.brtcserver) {
        result.brtcserver = urlConfig.brtcserver;
    }

    return result;
}

/**
 * RTC Agent客户端
 *
 * @description 百度智能云RTC AI Agent的客户端类，提供启动、停止、消息发送等功能
 */
export class BaiduRtcAgentClient {
    private appID: string = '';
    private agentID: string = '';
    private ak?: string;
    private sk?: string;
    private agentStarted: boolean = false;
    private agentApiHost: string = 'https://ai.agent.kaywang.cn/api';

    /**
     * 启动Agent
     *
     * @param param 启动参数
     * @description
     * 1. 调用后端API创建AI Agent实例
     * 2. 获取连接Token和AgentID
     * 3. 通过BRTC SDK连接到实时通信服务器
     *
     * @returns void
     */
    public Start(param: AgentStartParams): void {
        this.appID = param.appid;
        console.log('Agent.Start appid: ' + this.appID);

        if (param.apihost) {
            this.agentApiHost = param.apihost;
        }

        this.ak = param.ak;
        this.sk = param.sk;

        // 构建API请求
        const requestData: GenerateAIAgentCallRequest = {
            app_id: this.appID,
            quick_start: true,
            instance_type: param.instance_type,
            ak: param.ak,
            sk: param.sk,
            config: JSON.stringify(param.cfg)
        };

        // 发送HTTP请求
        const xmlHttp = new XMLHttpRequest();
        xmlHttp.open('POST', this.agentApiHost + '/v1/aiagent/generateAIAgentCall', true);
        xmlHttp.setRequestHeader('Content-Type', 'application/json');

        xmlHttp.onreadystatechange = () => {
            if (xmlHttp.readyState === 4) {
                if (xmlHttp.status === 200) {
                    console.log(xmlHttp.responseText);
                    const response: GenerateAIAgentCallResponse = JSON.parse(xmlHttp.responseText);

                    if (response.ai_agent_instance_id) {
                        this.agentStarted = true;
                        this.agentID = response.ai_agent_instance_id;
                        let userID = this.agentID;
                        const token = response.context.token;

                        if (response.context && response.context.uid) {
                            userID = response.context.uid;
                        }

                        if (response.context && response.context.appid) {
                            this.appID = response.context.appid;
                            param.appid = this.appID;
                        }

                        // 登录BRTC
                        this.loginBRTC({
                            ...param,
                            appid: param.appid,
                            roomname: this.agentID,
                            userid: userID,
                            token: token
                        } as BRTCStartParams & AgentStartParams);
                    }
                } else {
                    // 处理错误
                    if (param.error) {
                        param.error(`HTTP ${xmlHttp.status}: ${xmlHttp.responseText}`);
                    }
                }
            }
        };

        xmlHttp.send(JSON.stringify(requestData));
    }

    /**
     * 停止Agent
     *
     * @description
     * 1. 调用后端API停止AI Agent实例
     * 2. 断开BRTC连接
     * 3. 清理资源
     *
     * @returns Promise<StopAgentResult> 停止操作的结果
     */
    public Stop(): Promise<StopAgentResult> {
        return new Promise((resolve, reject) => {
            if (!this.agentStarted) {
                resolve({success: true});
                return;
            }

            this.agentStarted = false;
            console.log('Agent.Stop begin.');

            const requestData: StopAIAgentInstanceRequest = {
                app_id: this.appID,
                ai_agent_instance_id: this.agentID,
                ak: this.ak,
                sk: this.sk
            };

            const xmlHttp = new XMLHttpRequest();
            xmlHttp.open('POST', this.agentApiHost + '/v1/aiagent/stopAIAgentInstance', true);
            xmlHttp.setRequestHeader('Content-Type', 'application/json');

            xmlHttp.onreadystatechange = () => {
                if (xmlHttp.readyState === 4) {
                    if (xmlHttp.status === 200) {
                        console.log('Agent.Stop OK:' + xmlHttp.responseText);
                        // 停止BRTC连接
                        if (typeof window.BRTC_Stop === 'function') {
                            window.BRTC_Stop();
                        }
                        resolve({success: true});
                    } else {
                        console.log('Agent.Stop failed:' + xmlHttp.responseText);
                        reject({success: false, error: xmlHttp.responseText});
                    }
                }
            };

            xmlHttp.send(JSON.stringify(requestData));
        });
    }

    /**
     * 获取Agent ID
     *
     * @returns string Agent实例ID
     */
    public GetAgentID(): string {
        return this.agentID;
    }

    /**
     * 登录BRTC
     *
     * @param param BRTC连接参数
     * @description 通过BRTC SDK连接到实时通信服务器
     * @returns void
     */
    private loginBRTC(param: BRTCStartParams & AgentStartParams): void {
        // 注意：BRTC_Start是百度RTC SDK提供的全局函数
        // 这里仅做类型定义，实际调用需要引入SDK
        if (typeof window.BRTC_Start === 'function') {
            window.BRTC_Start({
                ...param,
                ...
            } as any);
        } else {
            console.error('BRTC_Start function not found. Please include baidu.rtc.sdk.js');
        }
    }

    /**
     * 发送消息给用户
     *
     * @param msg 消息内容
     * @param uid 目标用户ID（可选）
     * @description 通过BRTC数据通道发送消息
     * @returns void
     */
    public sendMessageToUser(msg: string, uid?: string): void {
        // 注意：BRTC_SendMessageToUser是百度RTC SDK提供的全局函数
        if (typeof window.BRTC_SendMessageToUser === 'function') {
            window.BRTC_SendMessageToUser(msg, uid);
        }
    }

    /**
     * 获取版本号
     *
     * @returns string 版本号
     */
    public Version(): string {
        return 'V1.0.6';
    }
}
```
### 2.4 web使用示例
```
/**
 * 使用示例
 *
 * @description 展示如何从初始化到使用的完整流程
 */
export function Example(): void {
    // 1. 解析参数
    const urlConfig = parseURLParams();

    // 2. 构建Agent配置
    const agentConfig = buildAgentConfig(urlConfig);

    // 3. 处理环境配置
    const envConfig = handleEnvironmentConfig(urlConfig);

    // 4. 创建Agent客户端
    const agent = new BaiduRtcAgentClient();

    // 5. 启动Agent
    agent.Start({
        apihost: envConfig.apihost || 'https://ai.agent.kaywang.cn/api',
        debuglevel: urlConfig.log || false,
        server: envConfig.brtcserver,
        instance_type: urlConfig.instance_type,
        ak: urlConfig.ak,
        sk: urlConfig.sk,
        cfg: agentConfig,
        appid: urlConfig.appid || '',
        displayname: urlConfig.displayname || 'web chat',
        remotevideoviewid: 'remotevideo',
        localvideoviewid: 'localvideo',
        userevent_joinedroom: (id: string, display: string, attribute: string) => {},
        userevent_leavingroom: (id: string, display: string) => {},
        success: () => {},
        error: (error: string) => {},
        onmessage: (msg: MessageData) => {
            console.log('Received message:', msg);
        },
        audiocodec: urlConfig.audiocodec,
        usingremotemediastate: true,
        remotemediastating: (id: string, medium: MediaType, on: boolean) => {}
    });

    // 6. 发送消息
    const sendMessage = (msg: string): void => {
        agent.sendMessageToUser(msg);
    };
    sendMessage('你好');

    // 7. 发送中断消息
    sendMessage('[B]');

    // 8. 发送设备信息
    sendMessage(`[SET]:[DEVICE_INFO]:${JSON.stringify({
        os: 'H5',
        soc: 'H5',
        model: 'H5',
        user_id: urlConfig.userid || 'default_user'
    })}`);

    // 9. 停止Agent
    agent.Stop().then(result => {
        ...
    }).catch(error => {
        ...
    });
}
```


# 成本计算
1. 参考[实时音视频RTC计费价格说明](https://cloud.baidu.com/product-price/rtc.html)

![](https://rte.weiyun.baidu.com/wiki/attach/image/api/imageDownloadAddress?attachId=af1a055a942f4142a633de3f62244a94&docGuid=nXT_pIUOzE3auq "")
2. 参考[大模型实时互动计费](https://cloud.baidu.com/doc/RTC/s/Ym8y3zcmt)

![](https://rte.weiyun.baidu.com/wiki/attach/image/api/imageDownloadAddress?attachId=44b9a84267b84bc9a2d32ce8e9cbce4d&docGuid=nXT_pIUOzE3auq "")
3. 参考[数字员工-价格指南](https://cloud.baidu.com/doc/AI_DH/s/Vlysgbz88)

# 方案一/二/三 对比
|对比维度|方案一：原生 WebRTC|方案二：百度云RTC Web SDK渲染 + Java SDK推流|方案三：百度云RTC + 大模型实时互动 + 曦灵数字人|
|-|-|-|-|
|定位|完全自主可控|稳定实时音视频基础能力，后端自行推真实数据|在方案二基础上，集成实时互动大模型与数字人渲染|
|接入难度|高：需自建信令、STUN/TURN、编解码与媒体管线|中：前端/后端按 SDK 接入，需对接推流侧|中：除RTC外，还需对接互动API、鉴权、数字人|
|开发周期|长|中|中|
|可定制性|高，所有链路均可控|中：围绕 SDK 能力范围内|中：AI 与数字人部分在平台可配置，深度定制需配合|
|功能完备性|低：云端录制、转码、统计需自建|高：录制、转码、弱网优化、统计齐备|高：在高功能 RTC 上叠加 LLM、TTS、ASR、数字人|
|运维成本|高：信令/穿透/媒体服务/监控自运维|低：云端托管|低：RTC云托管，互动/数字人由平台与服务端协同|
|弱网与质量|需自行优化|优：自研抗弱网、码率自适应、H.264/H.265|优：同方案二|
|端到端时延|需自行优化|优：常见场景 300ms 量级|优：同方案二（视 TTS/渲染策略）|
|成本结构|人力与时间成本高|云用量成本 + 适度开发|云用量成本 + 互动/数字人服务费用|
|合规与安全|需自建|平台侧具备完备能力|同方案二，另含互动数据安全考量|
|适用场景|不限|常规实时音视频|对话式交互、虚拟主播/导览/客服等智能体场景|

* 优缺点概览
    * 方案一（原生 WebRTC）
        * 优点：灵活度最高、无厂商绑定、可自由定制与实验
        * 缺点：研发/运维成本高，质量与弱网体验需要自己兼容

    * 方案二（百度云RTC SDK + 自主推流）
        * 优点：音视频基建齐备、弱网与质量稳定、集成成本适中、可控性较强
        * 缺点：深度定制受 SDK 边界限制，特殊需求需配合提需

    * 方案三（RTC + 实时互动大模型 + 曦灵数字人）
        * 优点：提供端到端智能互动与数字人渲染，最快达成“可用体验”的整套方案
        * 缺点：体系更复杂，对授权、鉴权、计费与服务编排有更多依赖，自由度较弱，依赖百度云数字人平台功能限制。


* 选型建议
    * 优先方案二，若产品形态需要智能体与数字人互动，上升为方案三




# 参考文档
* [WebRTC 官方文档](https://webrtc.org/)
* [MDN - WebRTC](https://developer.mozilla.org/zh-CN/docs/Web/API/WebRTC_API)
* [百度云 RTC Web SDK](https://cloud.baidu.com/doc/RTC/s/Uk3o54lnm)
* [百度云 RTC Java SDK](https://cloud.baidu.com/doc/RTC/s/rlkj7gdxx)
* [大模型实时互动H5 SDK](https://cloud.baidu.com/doc/RTC/s/cm8y513mz)
* [快速集成大模型实时互动](https://cloud.baidu.com/doc/RTC/s/9m8y4gdux)