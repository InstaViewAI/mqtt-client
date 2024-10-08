### Instavision MQTT 客户端集成

#### 介绍
本文件提供了将 MQTT 集成到位于 `tcp://mqttbroker.us.instavision.ai:1883` 的代理服务器的说明。集成包括使用 `device_id` 作为用户名，`mcu_access_token` 作为密码，采用自定义的客户端 ID 格式 `partnerID-clientid-spaceid-deviceid` 进行身份验证。

#### 先决条件
- 访问 MQTT 代理服务器 `tcp://mqttbroker.us.instavision.ai:1883` 的权限。
- 从固件 SDK 中获取有效的 `device_id` 和 `mcu_access_token` 以进行身份验证。
- 提供方法来检索 `partnerID`、`spaceid`、`deviceid` 和 `thing_name` 的固件 SDK。
- 用于 MQTT 通信的基于 C 的 MQTT 客户端。

#### MQTT 代理服务器详情
- **代理 URL:** `tcp://mqttbroker.us.instavision.ai:1883`
- **协议:** TCP
- **端口:** 1883（非 SSL 连接的默认端口）
- **Ping 超时:** `10 秒`
- **保持活动超时:** `60 秒`

#### 从固件 SDK 中检索标识符

在连接到 MQTT 代理之前，需要从固件 SDK 中检索必要的标识符。

#### 使用身份验证、自定义客户端 ID、Ping 超时和保持活动时间连接到代理

```python
import paho.mqtt.client as mqtt

# 定义 MQTT 代理服务器详情
broker_url = "mqttbroker.us.instavision.ai"
broker_port = 1883

# 定义身份验证凭据
device_id = sdk.get_device_id()       # 从 SDK 中获取
mcu_access_token = "your_mcu_access_token"  # 通常安全地存储在应用程序中

# 定义自定义客户端 ID
partnerID = sdk.get_partner_id()  # 从 SDK 中获取
clientid = "your_clientid"  # 通常在应用程序中定义
spaceid = sdk.get_space_id()      # 从 SDK 中获取
client_id = f"{partnerID}-{clientid}-{spaceid}-{device_id}"

# 使用自定义客户端 ID 创建新的 MQTT 客户端实例
client = mqtt.Client(client_id=client_id)

# 设置 device_id 和 mcu_access_token 进行身份验证
client.username_pw_set(username=device_id, password=mcu_access_token)

# 设置保持活动时间和 Ping 超时
client.keep_alive = 60  # 60 秒的保持活动时间
client._transport = "tcp"
client._sock.settimeout(10)  # 10 秒的 Ping 超时

# 连接到代理
client.connect(broker_url, broker_port)

# 启动循环以处理网络流量和分发回调
client.loop_start()
```

#### 订阅唤醒命令主题

要订阅唤醒命令主题 `device/things/<thing_name>/message` 并处理特定的有效负载格式：

```python
import json

# 定义 on_message 回调
def on_message(client, userdata, message):
    payload = json.loads(message.payload.decode())
    print(f"收到唤醒命令: {payload}")

    # 从有效负载中提取详细信息
    if payload.get("type") == "WakeUP":
        device_id = payload.get("body", {}).get("deviceId")
        print(f"设备的唤醒命令: {device_id}")

# 将 on_message 回调分配给客户端
client.on_message = on_message

# 订阅唤醒命令主题
topic = f"device/things/{thing_name}/message"
client.subscribe(topic)
```

#### 示例订阅有效负载

在订阅唤醒命令主题时，预期的有效负载格式如下：

```json
{
  "message_id": "uuid",
  "type": "WakeUP",
  "timestamp": 1678886400,
  "body": {
    "deviceId": "<>"
  }
}
```

#### 从代理断开连接

通信完成后，可以从代理断开连接：

```python
# 从代理断开连接
client.loop_stop()
client.disconnect()
```

#### 常见的 MQTT 客户端错误代码

在与 MQTT 代理集成时，您可能会遇到各种错误代码。以下是客户端常见错误代码及其含义的列表：

- **0 (MQTT_ERR_SUCCESS):** 无错误，操作成功。
- **1 (MQTT_ERR_NOMEM):** 内存不足错误。
- **2 (MQTT_ERR_PROTOCOL):** 协议错误，表示 MQTT 协议存在问题。
- **3 (MQTT_ERR_INVAL):** 无效输入错误，表示传递了无效的参数。
- **4 (MQTT_ERR_NO_CONN):** 无连接错误，表示没有与代理服务器的连接。
- **5 (MQTT_ERR_CONN_REFUSED):** 连接被拒绝错误，表示代理服务器拒绝了连接尝试。
- **6 (MQTT_ERR_NOT_FOUND):** 未找到错误，表示未找到所需资源。
- **7 (MQTT_ERR_CONN_LOST):** 连接丢失错误，表示与代理服务器的连接丢失。
- **8 (MQTT_ERR_TLS):** TLS 错误，表示 TLS 连接存在问题。
- **9 (MQTT_ERR_PAYLOAD_SIZE):** 有效负载大小错误，表示有效负载大小过大。
- **10 (MQTT_ERR_NOT_SUPPORTED):** 不支持错误，表示请求的操作不受支持。
- **11 (MQTT_ERR_AUTH):** 身份验证错误，表示身份验证尝试失败。
- **12 (MQTT_ERR_ACL_DENIED):** ACL 拒绝错误，表示访问被代理服务器的 ACL 拒绝。
- **13 (MQTT_ERR_UNKNOWN):** 未知错误，表示发生了未知错误。
- **14 (MQTT_ERR_ERRNO):** 系统错误，表示系统特定的错误。
- **15 (MQTT_ERR_QUEUE_SIZE):** 队列大小错误，表示消息队列已满。

#### 故障排除

- **连接被拒绝:** 确保 `device_id`、`mcu_access_token`、`client_id` 格式和主题名称正确。
- **超时问题:** 检查网络稳定性和代理服务器可用性。
- **身份验证错误:** 验证您的 `device_id` 和 `mcu_access_token`。
- **无效的有效负载格式:** 确保传入的消息符合预期的 JSON 结构。
- **MQTT 错误代码:** 参考常见错误代码列表以诊断和解决问题。

#### 集成测试

请参考 `cmd` 文件夹内订阅和发布者文件夹中的 readme 文档。

#### 结论

本文件提供了将应用程序与位于 `tcp://mqttbroker.us.instavision.ai:1883` 的 MQTT 代理服务器集成的全面指南，使用 `device_id` 作为用户名，`mcu_access_token` 作为密码，自定义客户端 ID 格式 `partnerID-clientid-spaceid-deviceid`。