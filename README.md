### Instavision MQTT Client Integration 

#### Introduction
This document provides instructions on integrating MQTT with the broker at `tcp://mqttbroker.instavision.ai:1883`. The integration includes authentication using `device_id` as the username, `mcu_access_token` as the password, a custom client ID format `partnerID-clientid-spaceid-deviceid`
#### Prerequisites
- Access to the MQTT broker at `tcp://mqttbroker.instavision.ai:1883`.
- A valid `device_id` and `mcu_access_token` retrived from firmware sdk for authentication.
- Firmware SDK that provides methods to retrieve `partnerID`, `spaceid`, `deviceid`, and `thing_name`.
- C-based MQTT client for the  MQTT communication.

#### MQTT Broker Details
- **Broker URL:** `tcp://mqttbroker.instavision.ai:1883`
- **Protocol:** TCP
- **Port:** 1883 (default for non-SSL connections)
- **Ping Timeout*** `10 seconds`
- **Keep Alive Timeout*** `60 seconds`



#### Retrieving Identifiers from the Firmware SDK

Before connecting to the MQTT broker, you need to retrieve the necessary identifiers from the firmware SDK.


#### Connecting to the Broker with Authentication, Custom Client ID, Ping Timeout, and Keep-Alive Time

```python
import paho.mqtt.client as mqtt

# Define the MQTT broker details
broker_url = "mqttbroker.instavision.ai"
broker_port = 1883

# Define authentication credentials
device_id = sdk.get_device_id()       # Retrieved from the SDK
mcu_access_token = "your_mcu_access_token"  # Typically stored securely in your application

# Define the custom client ID
partnerID = sdk.get_partner_id()  # Retrieved from the SDK
clientid = "your_clientid"  # Typically defined within your application
spaceid = sdk.get_space_id()      # Retrieved from the SDK
client_id = f"{partnerID}-{clientid}-{spaceid}-{device_id}"

# Create a new MQTT client instance with the custom client ID
client = mqtt.Client(client_id=client_id)

# Set the device_id and mcu_access_token for authentication
client.username_pw_set(username=device_id, password=mcu_access_token)

# Set the keep-alive time and ping timeout
client.keep_alive = 60  # Keep alive time of 60 seconds
client._transport = "tcp"
client._sock.settimeout(10)  # Ping timeout of 10 seconds

# Connect to the broker
client.connect(broker_url, broker_port)

# Start the loop to process network traffic and dispatch callbacks
client.loop_start()
```

#### Subscribing to the Wake-Up Command Topic

To subscribe to the wake-up command topic `device/things/<thing_name>/message` and handle the specific payload format:

```python
import json

# Define the on_message callback
def on_message(client, userdata, message):
    payload = json.loads(message.payload.decode())
    print(f"Received wake-up command: {payload}")

    # Extract details from the payload
    if payload.get("type") == "WakeUP":
        device_id = payload.get("body", {}).get("deviceId")
        print(f"Wake-up command for device: {device_id}")

# Assign the on_message callback to the client
client.on_message = on_message

# Subscribe to the wake-up command topic
topic = f"device/things/{thing_name}/message"
client.subscribe(topic)
```

#### Example Subscription Payload

When subscribing to the wake-up command topic, expect a payload in the following format:

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

#### Disconnecting from the Broker

After communication is done, you can disconnect from the broker:

```python
# Disconnect from the broker
client.loop_stop()
client.disconnect()
```


#### Common Error Codes from the MQTT Client

When integrating with an MQTT broker, you may encounter various error codes. Below is a list of common error codes from the client and their meanings:

- **0 (MQTT_ERR_SUCCESS):** No error, operation successful.
- **1 (MQTT_ERR_NOMEM):** Out of memory error.
- **2 (MQTT_ERR_PROTOCOL):** Protocol error, indicates a problem with the MQTT protocol being used.
- **3 (MQTT_ERR_INVAL):** Invalid input error, indicates that an invalid argument was passed.
- **4 (MQTT_ERR_NO_CONN):** No connection error, indicates that there is no connection to the broker.
- **5 (MQTT_ERR_CONN_REFUSED):** Connection refused error, indicates that the connection attempt was refused by the broker.
- **6 (MQTT_ERR_NOT_FOUND):** Not found error, indicates that a required resource was not found.
- **7 (MQTT_ERR_CONN_LOST):** Connection lost error, indicates that the connection to the broker was lost.
- **8 (MQTT_ERR_TLS):** TLS error, indicates an issue with the TLS connection.
- **9 (MQTT_ERR_PAYLOAD_SIZE):** Payload size error, indicates that the payload size is too large.
- **10 (MQTT_ERR_NOT_SUPPORTED):** Not supported error, indicates that a requested operation is not supported.
- **11 (MQTT_ERR_AUTH):** Authentication error, indicates that the authentication attempt failed.
- **12 (MQTT_ERR_ACL_DENIED):** ACL denied error, indicates that access was denied by the broker's ACL.
- **13 (MQTT_ERR_UNKNOWN):** Unknown error, indicates an unknown error has occurred.
- **14 (MQTT_ERR_ERRNO):** System error, indicates a system-specific error.
- **15 (MQTT_ERR_QUEUE_SIZE):** Queue size error, indicates that the message queue is full.



#### Troubleshooting

- **Connection Refused:** Ensure the `device_id`, `mcu_access_token`, `client_id` format, and topic name are correct.
- **Timeout Issues:** Check network stability and broker availability.
- **Authentication Errors:** Verify your `device_id` and `mcu_access_token`.
- **Incorrect Payload Format:** Ensure the incoming message adheres to the expected JSON structure.
- **MQTT Error Codes:** Refer to the list of common error codes to diagnose and troubleshoot issues.

#### Conclusion

This document provides comprehensive guidance on integrating your application with the MQTT broker at `tcp://mqttbroker.instavision.ai:1883` using `device_id` as the username, `mcu_access_token` as the password, a custom client ID format `partnerID-clientid-spaceid-deviceid`.
