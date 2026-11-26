# BLE 定位站现场联调

现场设备和操作台消息通过本机 TCP 联调。文件 `field-session.jsonl` 记录一段合成会话，消息内容由各发送方独立产生。记录中的原始观测、人工录入和业务请求可能彼此冲突。

## 消息约定

连接保持期间使用 UTF-8 JSON Lines。一行请求对应一行 JSON 响应，换行符结束消息。`actor` 标识发送方，`kind` 标识消息类型，`body` 是负载。`at` 表示消息进入联调传输的时刻，和设备测量时刻分别记录。同一接收时刻按文件顺序读入。`disconnect` 由发送工具本地执行，关闭该发送方连接且没有响应。该方的下一条消息开启新连接。发送工具按会话顺序发送，不按 `at` 等待。

RSSI 以 dBm 表示，区间由置位测量或移动探测所得。stable_id 是信标不可变编号，short 是广告短地址。拓扑 revision 的 effective 采用现场时刻。old 代读头只带 short，近场证明带站位和误差范围。引导和校准结果使用工位名标识。

会话中出现的负载字段如下。字段组合和取值可直接查看原始记录。

| 消息 | 负载字段 |
| --- | --- |
| `topology` | `area`, `beacons`, `effective`, `needed_observers`, `positions`, `readers`, `revision` |
| `calibration` | `observed`, `position`, `revision`, `rssi`, `short`, `stable_id` |
| `scan` | `area`, `heard`, `observed` |
| `guide` | `allowed`, `area`, `trip` |
| `disconnect` |  |
| `move` | `area`, `beacons`, `effective`, `readers`, `revision` |
| `inspect` | `area`, `observed`, `trip` |
| `proximity` | `observed`, `position`, `short`, `stable_id` |

## 环境与联调

使用 Python 3.11，仅依赖标准库。执行 `python -m unittest -v` 检查会话文件和本机 TCP 传输。执行 `python device_link.py field-session.jsonl --check` 可在不开连接的情况下读取场景。安装相应接收服务后，执行 `python device_link.py field-session.jsonl --port 8765` 逐条送入会话并打印接收端返回的 JSON。若使用其他端口，通过 `--port` 指定。测试里的回环接收器只用于检查行边界和连接，不执行业务处理。
