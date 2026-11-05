# 多人视频聊天室

基于 **Qt6 + FFmpeg + OpenCV** 的多人视频聊天室：Qt 客户端负责音视频采集、编解码与渲染，Linux C++ 服务端负责信令转发与 RTP 媒体包中继。

Multi-user video chat room built with Qt6, FFmpeg and OpenCV.

## 项目结构

```
video-chat-room/
├── client/          # Qt6 客户端（.pro 工程，MinGW 64-bit 构建）
│   ├── src/         # 界面与业务逻辑
│   ├── network/     # TCP 信令连接、协议收发
│   ├── media/       # FFmpeg / OpenCV 音视频处理
│   └── common/      # 与服务端共用的协议定义
└── server/          # 服务端：C++17 + epoll Reactor，零第三方依赖
    ├── core/        # EventLoop / Acceptor / TcpConnection
    ├── session/     # 连接与会话管理
    ├── room/        # 房间与成员管理
    ├── protocol/    # 消息分发
    ├── media/       # UDP RTP 中继
    └── common/      # JSON 协议编解码
```

## 架构

| 通道 | 协议 | 端口 | 说明 |
|---|---|---|---|
| 信令 | TCP | 8888 | 一行一条 JSON，`\n` 分帧：注册 / 登录 / 建房 / 进房 / 广播 / 心跳 |
| 媒体 | UDP | 10001 | 服务端只按房间转发 RTP 包，不解码不转码 |

服务端是**单线程 Reactor（epoll 水平触发）**，所有用户、房间、会话状态都在进程内存中，没有持久化。

## 技术栈

| 层 | 技术 |
|---|---|
| 客户端界面 | Qt6 Widgets / Qt6 Multimedia |
| 音视频 | FFmpeg（编解码）、OpenCV（图像处理） |
| 服务端 | C++17、epoll、POSIX socket、GNU Make |
| 传输 | TCP（信令）+ UDP（RTP 中继） |

## 构建与运行

### 服务端（Linux / WSL）

```bash
cd server
make clean && make
./server
```

> 依赖 `<sys/epoll.h>`，只能在 Linux 或 WSL 下编译，Windows 原生工具链（MinGW / MSVC）不支持。

### 客户端（Windows + Qt6）

```bash
cd client
qmake videomeeting.pro
make
```

需要 Qt 6.12 + MinGW 64-bit。客户端连本机服务端时填 `127.0.0.1:8888`。

## 信令协议

请求：`register`、`login`、`create_room`、`join_room`、`register_rtp`、`sdp_offer`、`heartbeat`、`leave_room`、`broadcast`、`quit`、`list_rooms`

响应：`login_resp`、`create_room_resp`、`join_room_resp`、`register_rtp_resp`、`room_users`、`user_joined`、`user_left`、`broadcast_resp`、`room_list_resp`、`heartbeat_resp`、`error`

## 开发状态

开发中。服务端信令与房间管理已完成，媒体转发链路联调中。
