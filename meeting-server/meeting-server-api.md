# 房间服务器文档

- role：客户端角色，normal 代表 android、ios、web客户端；recorder 代表 录像客户端；relayer 代表 转发客户端；android、ios客户端sdk无需对role做任何处理，sdk的调用者可参考role参数（例如app通过房间里是否存在recorder角色的客户端来判断录像任务是否在执行）。
- device：sdk在某台设备上安装后，device固定不变并且全局唯唯一。
- endpoint：endpoint是用来区分SDK实例的标识（类似userId），SDK的调用者确保endpoint全局唯一，如果调用者传入的endpoint为空则SDK自动生成全局唯一的endpoint。之所以没有使用userId命名，是因为每个SDK实例只允许推送一条流，调用者为实现同时推送多条流的功能会创建多个sdk实例，这时候再使用userId来命名不同的sdk实例，可能会引起误解。
- stream_id：SDK自动生成，由SDK确保stream_id全局唯一，当前的设计模型中一个endpoint只会推送一条流，所以endpoint和stream_id可以相同。
- MeetingServer的网络链接类型：房间服务器与客户端之间通过websocket链接。
- MeetingServer的消息类型：房间服务器与客户端之间的消息分为两类，一类是客户端主动发起的请求消息，另一类是由服务器主动向客户端下发的通知。

`客户端主动发起的请求消息：`

method | 描述
--- | ---
channel.join | 加入房间，服务器对token鉴权，如果不合法则断开连接
channel.leave | 退出房间（销毁所有与原房间有关的资源）
stream.add | 如果客户端推流成功，则向房间服务器声明这条流
stream.delete | 如果客户端主动结束推流或推流重连失败后发送这条消息。
channel.dump | 获取当前房间里的在线用户列表，列表中包含每个用户推流的流名，获取到该列表就能订阅列表里的任意条流
media_server.dump | 获取媒体服务器地址，客户端可以向该媒体服务器发起推流和拉流请求。（推拉流重连的时候需要通过这个接口更新媒体服务器地址）
broadcast | 客户端广播请求，服务器收到这类请求后将data部分的内容广播发送给全房间里的客户端（房间内其他客户端接收到broadcat事件）。

`服务器端主动向客户端下发的通知消息：`

event | 描述
--- | ---
stream | 房间内有客户端推流成功时或者推流结束时触发（`直接将该事件回调给SDK调用者，SDK内部暂不需要处理`）
endpoint | 房间内有客户端上线或者下线时触发（`直接将该事件回调给SDK调用者，SDK内部暂不需要处理`）
error | 有错误信息产生时触发（`直接将错误事件回调给SDK调用者，SDK内部暂不需要处理`）
broadcast | 广播事件（`直接将错误事件回调给SDK调用者，SDK内部暂不需要处理`）

## 1. 客户端发起的请求

### 1.1 channel.join

`加入指定的房间，注意需要携带session`

```json
{
    "version": 0,
    "cseq": 0,
    "method": "channel.join",
    "session": "", // joinChannel时session为空
    "data": {
        "app": "xxx",
        "channel": "xxx",
        "endpoint": "xxx",
        "role": "normal"
    }
}
```

- 返回

```json
{
    "version": 0,
    "cseq": 0,
    "err": 0,
    "err_msg": "xxx",
    "data": {
        "session": "xxx",
        "media_server": {
            "wss": "wss://xxx:port/xxx", // normal类型的客户端只关心wss地址
            "ws": "ws://xxx:port/xxx",
            "https": "https://xxx:port/xxx",
            "http": "http://xxx:port/xxx",
            "rtsp": "rtsp://xxx:port/xxx"
        }
    }
}
```

### 1.2 channel.leave

```json
{
    "version": 0,
    "cseq": 0,
    "method": "channel.leave",
    "session": "xxx",
    "data": {
    }
}
```

- 返回

```json
{
    "version": 0,
    "cseq": 0,
    "err": 0,
    "err_msg": "xxx",
    "session": "xxx",
    "data": {
    }
}
```

### 1.3 stream.add

`客户端向流媒体服务器推流成功后需要向MeetingServer声明这条流，MeetingServer可以将流成功的消息广播出去，以便其他客户端来订阅`

```json
{
    "version": 0,
    "cseq": 0,
    "method": "stream.add",
    "session": "xxx",
    "data": {
        "stream": {
            "type": "xxx", // camera 或 desktop
            "stream_id": "xxxx",
            "has_video": true,
            "has_audio": true,
            "audio_codec": "opus",
            "video_codec": "vp8"
        }
    }
}
```

- 返回

```json
{
    "version": 0,
    "cseq": 0,
    "err": 0,
    "err_msg": "xxx",
    "session": "xxx",
    "data": {
    }
}
```

### 1.4 stream.delete

```json
{
    "version": 0,
    "cseq": 0,
    "method": "stream.delete",
    "session": "xxx",
    "data": {
        "stream_id": "xxx"
    }
}
```

- 返回

```json
{
    "version": 0,
    "cseq": 0,
    "err": 0,
    "err_msg": "xxx",
    "session": "xxx",
    "data": {
    }
}
```

### 1.5 channel.dump

`获取房间内在线的用户列表,用户角色分为normal、recorder、relayer。`

- normal：普通客户端，如 android、ios、html5等客户端都是普通角色
- recorder：录像客户端，录像客户端加入房间的目的是将房间内所有在线流都录制下来，相对于普通客户端和MeetingServer而言不用关心用户角色，但是客户端sdk需要将`用户角色`暴露给sdk的调用者，由调用者决定是否在ui上显示recorder角色的客户端。
- relayer：转推客户端，将房间内所有的流转推到第三方服务器。参考recorder

```json
{
    "version": 0,
    "cseq": 0,
    "method": "channel.dump",
    "session": "xxx",
    "data": {
    }
}
```

- 返回

```json
{
    "version": 0,
    "cseq": 0,
    "err": 0,
    "err_msg": "xxx",
    "data": {
        "endpoint_list": [
            {
                "endpoint": "xxx",
                "device": "xxx",
                "role": "normal/recorder/relayer",
                "stream": {
                    "stream_id": "xxxx",
                    "type": "xxx", // camera 或者 desktop
                    "has_video": true,
                    "has_audio": true,
                    "audio_codec": "opus",
                    "video_codec": "vp8"
                }
            }
        ]
    }
}
```

### 1.6 获取媒体服务器地址

```json
{
    "version": 0,
    "cseq": 0,
    "method": "media_server.dump",
    "session": "xxx",
    "data": {
        "protocol": "webrtc"
    }
}
```

- 返回

```json
{
    "version": 0,
    "cseq": 0,
    "err": 0,
    "err_msg": "xxx",
    "data": {
        "media_server": {
            "wss": "wss://xxx:port/xxx", // normal类型的客户端只关心wss地址
            "ws": "ws://xxx:port/xxx",
            "https": "https://xxx:port/xxx",
            "http": "http://xxx:port/xxx",
            "rtsp": "rtsp://xxx:port/xxx"
        }
    }
}
```

### 1.7 发送广播消息

```json
{
    "version": 0,
    "cseq": 0,
    "method": "broadcast",
    "session": "xxx",
    "data": {
        ....
        ....
        ....
    }
}
```

- 返回

```json
{
    "version": 0,
    "cseq": 0,
    "err": 0,
    "err_msg": "xxx",
    "data": {
    }
}
```

## 2. 服务器主动发出的通知事件

### 2.1 stream事件

```json
{
    "version": 0,
    "cseq": 0,
    "event": "stream",
    "session": "xxx",
    "data": {
        "device": "xxx",
        "endpoint": "xxx",
        "stream": "xxx",
        "stat": "publish/unpublish"
    }
}
```

> 客户端推流成功时：其他客户端收到 stream publish 事件通知；
>
> 推流断开时：其他客户端收到 stream unpublish 事件通知

### 2.2 endpoint事件

```json
{
    "version": 0,
    "cseq": 0,
    "event": "endpoint",
    "session": "xxx",
    "data": {
        "device": "xxx",
        "endpoint": "xxx",
        "stat": "join/leave"
    }
}
```

> SDK 加入会议频道时：其他客户端收到 endpoint join 事件通知；
>
> SDK 离开会议频道时：其他客户端收到 endpoint leave 事件通知

### 2.3 error事件

`客户端接收到error事件后将错误事件回调给sdk的调用者即可，暂不需要做特殊处理。`

```json
{
    "version": 0,
    "cseq": 0,
    "event": "error",
    "session": "xxx",
    "data": {
        "type": "xxx"
    }
}
```

### 2.4 通用广播事件

`MeetingServer将收到的broadcast请求视为广播事件，MeetingServer会把请求的data部分广播给同房间里的所有端`

```json
{
    "version": 0,
    "cseq": 0,
    "event": "broadcast",
    "session": "xxx",
    "data": {
        "source_endpoint": {
            "device": "xxx",
            "endpoint": "xxx"
        },
        "payload": {
            ....
            ....
            ....
        }
    }
}
```
