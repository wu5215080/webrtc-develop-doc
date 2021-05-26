
# 配置文档

```json
{
    "log": {
        "file": "./logs/pms.log",
        "workerLevel": "debug",
        "fileLevel": "debug",
        "consolLevel": "debug",
        "workerTags": ["info", "ice", "dtls", "rtp", "srtp", "rtcp", "rtx", "bwe", "score", "simulcast", "svc", "sctp", "message"]
    },

    "websocket": {
        "port": 8889,
        "ssl": true,
        "keyFile": "./certs/privkey.pem",
        "certFile": "./certs/full_chain.pem",
        "passPhrase": "",
        "location": "/"
    },

    "master": {
        "numOfWorkerProcess": 0,
        "execPath": "./",
        "workerName": "mediasoup-worker",
        "unixSocketPath": "./logs/pms"
    },

    "webrtc": {
        "listenIp": "172.17.0.17",
        "announcedIp": "122.51.177.240",
        "minPort": 20000,
        "maxPort": 30000,
        "dtlsCertificateFile": "./certs/full_chain.pem",
        "dtlsPrivateKeyFile": "./certs/privkey.pem"
    },

    "rtsp": {
        "port": 8554,
        "listenIp": "0.0.0.0"
    },

    "record": {
        "targetHost": "127.0.0.1",
        "targetPort": 8554,
        "recordPath": "./record/",
        "execRecordDone": "",
        "cmdPort": 8888
    },

    "pull":
        [
            {"ip": "xxx", "port": 8554},
            {"ip": "xxx", "port": 8554}
        ]
}

```

主要配置项：

- los
    - file：日志文件路径
    - workerLevel：子进程的日志等级，如"debug"/"warn"/"error"
    - fileLevel：输出到文件的日志等级，如"debug"/"info"/"warn"/"error"
    - consolLevel：输出到控制台的日志等级，如"debug"/"info"/"warn"/"error"
    - workerTags：子进程的日志标记，如果["info", "ice", "dtls", "rtp", "srtp", "rtcp", "rtx", "bwe", "score", "simulcast", "svc", "sctp", "message"]

- websocket
    - port：wss端口
    - ssl：是否使用ssl
    - keyFile：证书私钥文件
    - cerFile：证书文件
    - passPhrase：证书认证密码
    - location：wss的uri路径

- master
    - numOfWorkerProcess：子进程个数，如果为0则与CPU核数量一致
    - execPath：指定运行目录
    - workerName：子进程文件名
    - unixSocketPath：`目前已经弃用该配置`

- webrtc
    - listenIp：监听网卡的ip
    - announcedIp：对外服务的公网ip
    - minPort：udp端口范围的最小值
    - maxPort：udp端口范围的最大值
    - dtlsCertificateFile: 证书文件
    - dtlsPrivateKeyFile：证书私钥

- rtsp
    - port：rtsp服务器端口
    - listenIp：监听IP

- record
    - targetHost: 需要被录制的服务器ip，如果想对本服务器录制 则设置为：127.0.0.1
    - targetPort: 需要被录制的服务器rtsp端口
    - recordPath: 录制文件存储目录
    - execRecordDone: 录制文件生成后如果还需要做后续的处理则在此处填写shell命令
    - cmdPort: 录制控制端口

- pull
    - ip: 需要互联的sfu ip
    - port: 需要互联的sfu port
