# WebRTC SDK说明

## Api函数接口列表

- 加入`会议频道`（JoinChannel）

    > 输入参数：appId、secretKey、channelId、endpoint
    >
    > 操作结果：异步回调通知(回调参数包含：SDK上下文对象、errorcode)
    >
    > 流程：调用MeetingServer的 `channel.join` 接口

- 离开`会议频道`（LeaveChannel）

    > 输入参数：空
    >
    > 操作结果：通过函数返回值同步返回
    >
    > 流程：调用MeetingServer的 `channel.leave` 接口

- 推流（Publish）

    > 输入参数：type（camera/desktop）、hasVideo、hasAudio、resolution（如 1920*1080，如果为空字符串则使用系统默认分辨率）、maxBitrate
    >
    > 操作结果：异步回调通知（回调参数包含：SDK上下文对象、errorcode）
    >
    > 流程：调用 [WebRtcServer](../webrtc-server/webrtc-server-api.md) 的 `stream.publish` 接口 和 [MeetingServer](../meeting-server/meeting-server-api.md)的 `stream.add` 接口

- 切换摄像头（SwitchCamera）

    > 输入参数：frontCamera/rearCamera（如果当前在推桌面流则函数调用始终失败）
    >
    > 操作结果：通过函数返回值同步返回（如果不会长时间阻塞线程的话）
    >
    > 流程：`本地切换，无需告知服务器`

- 暂停/打开音视频（PauseVideo/ResumeVideo）

    > 输入参数：audio(true/false) video(true/false)
    >
    > 操作结果：可通过函数返回值同步返回
    >
    > 流程：调用[WebRtcServer](../webrtc-server/webrtc-server-api.md)的`stream.mute`接口

- 停止推流（Unpublish）

    > 输入参数：空
    >
    > 操作结果：可通过函数返回值同步返回
    >
    > 流程：调用[WebRtcServer](../webrtc-server/webrtc-server-api.md)的`stream.close`接口 和 [MeetingServer](../meeting-server/meeting-server-api.md)的 `stream.delete` 接口

- 订阅用户的流（Subscribe）

    > 输入参数：endpointId
    >
    > 操作结果：异步回调通知（回调参数包含：SDK上下文对象、被订阅的endpointId、errorcode）
    >
    > 流程：调用 [WebRtcServer](../webrtc-server/webrtc-server-api.md) 的 `stream.play` 接口

- 停止订阅流（Unsubscribe）

    > 输入参数：endpointId
    >
    > 操作结果：通过函数返回值同步返回
    > 流程：调用 [WebRtcServer](../webrtc-server/webrtc-server-api.md) 的 "stream.close" 接口

- 更新`会议频道`内的可订阅的用户列表（UpdateSubscribeList）

    > 输入参数：空
    >
    > 操作结果：通过函数返回值同步返回
    >
    > 流程：调用 [MeetingServer](../meeting-server/meeting-server-api.md) 的 `channel.dump` 接口

- 获取本地的订阅列表（GetSubscribeList）

    > 输入参数：空
    >
    > 操作结果：通过函数返回值同步返回
    >
    > 流程：`本地操作，无需服务器交互`

## 事件回调接口

- 流状态变化通知（stream_stat)
    > 频道内有客户端推流成功时或者推流结束，sdk直接将事件回调给上层app
    > 携带参数：

- 会议频道内有终端上下线通知（endpoint_stat）
    > 频道内有终端上下线，sdk直接将事件回调给上层app

- 终端接收到错误通知（error）
    > 有针对性终端的错误信息产生，sdk直接将事件回调给上层app

- broadcast
    > 终端收到广播事件，sdk直接将事件回调给上层app
