---
title: "AWS IoT Device Shadowと通信"
emoji: "👏"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AWS","IoT","shadow","MQTT"]
published: true
---

# はじめに

遠隔にあるIoT機器に対して設定値を更新したいと思った際に、AWS IoT Device Shadowを使うことがあります。

デバイスの状態を把握する用途で使うことが多いですが、今回のようなこともユースケースとしてはあるようです。
<https://docs.aws.amazon.com/ja_jp/iot/latest/developerguide/using-device-shadows.html#using-device-shadows-app-update>

# 問題

MQTTやAWS SDKから接続しDevice Shadowを取得、更新する方法が分からない。

# 解決方法

あまりメジャーではないのか、情報が少なかったので手探りで実装してみました。
なお、前提としてIoT Coreでモノや証明書、ポリシー、Shadowなどは設定済みであることとします。

| 項目                           | 値                  |
| ------------------------------ | ------------------- |
| モノ名                         | Test-Device         |
| シャドウ名                     | test1               |
| 証明書のファイル名             | certificate.pem.crt |
| 秘密鍵のファイル名             | private.pem.key     |
| ルート CA の証明書のファイル名 | AmazonRootCA1.pem   |

## MQTTのコネクションを確立する関数

まず、機器とAWSをMQTTで接続します。

```python
def mqtt_connect() -> mqtt.Connection:
    event_loop_group = io.EventLoopGroup(1)
    host_resolver = io.DefaultHostResolver(event_loop_group)
    client_bootstrap = io.ClientBootstrap(event_loop_group, host_resolver)

    mqtt_connection = mqtt_connection_builder.mtls_from_path(
        endpoint="IoT Coreのエンドポイント(それぞれのアカウントで異なる。https://ap-northeast-1.console.aws.amazon.com/iot/home?region=ap-northeast-1#/settings)",
        cert_filepath="./certificate.pem.crt",
        pri_key_filepath="./private.pem.key",
        ca_filepath="./AmazonRootCA1.pem",
        client_bootstrap=client_bootstrap,
        client_id="client_id",
        clean_session=False,
        keep_alive_secs=6,
    )
    connected_future = mqtt_connection.connect()
    connected_future.result()

    return mqtt_connection
```

## Subscribeするファイル

Shadowが更新、取得のリクエストをした際に、結果をSubscribeすることができます。

```python:subscribe.py
from awscrt import io, mqtt
from awsiot import mqtt_connection_builder, iotshadow
import threading


def on_update_shadow_accepted(response: iotshadow.UpdateShadowResponse):
    print("Shadow Update Accepted:")
    print(response.state)


def on_shadow_update_rejected(error):
    print("Shadow Update Rejected:")
    print(error)


def on_get_shadow_accepted(response: iotshadow.GetShadowResponse):
    print("Shadow Get Accepted:")
    print(response.state)


def on_shadow_get_rejected(error):
    print("Shadow Get Rejected:")
    print(error)


def main():
    mqtt_connection = mqtt_connect()
    shadow = iotshadow.IotShadowClient(mqtt_connection)

    # Updateの成功をサブスクライブする
    update_accepted_future, _ = shadow.subscribe_to_update_named_shadow_accepted(
        request=iotshadow.UpdateNamedShadowSubscriptionRequest(
            thing_name="Test-Device",
            shadow_name="test1",
        ),
        qos=mqtt.QoS.AT_LEAST_ONCE,
        callback=on_update_shadow_accepted,
    )
    update_accepted_future.result()

    # Updateの失敗をサブスクライブする
    update_rejected_future, _ = shadow.subscribe_to_update_named_shadow_rejected(
        request=iotshadow.UpdateNamedShadowSubscriptionRequest(
            thing_name="Test-Device",
            shadow_name="test1",
        ),
        qos=mqtt.QoS.AT_LEAST_ONCE,
        callback=on_shadow_update_rejected,
    )
    update_rejected_future.result()

    # Getの成功をサブスクライブする
    get_accepted_future, _ = shadow.subscribe_to_get_named_shadow_accepted(
        request=iotshadow.GetNamedShadowSubscriptionRequest(
            thing_name="Test-Device",
            shadow_name="test1",
        ),
        qos=mqtt.QoS.AT_LEAST_ONCE,
        callback=on_get_shadow_accepted,
    )
    get_accepted_future.result()

    # Getの失敗をサブスクライブする
    get_rejected_future, _ = shadow.subscribe_to_get_named_shadow_rejected(
        request=iotshadow.GetNamedShadowSubscriptionRequest(
            thing_name="Test-Device",
            shadow_name="test1",
        ),
        qos=mqtt.QoS.AT_LEAST_ONCE,
        callback=on_shadow_get_rejected,
    )
    get_rejected_future.result()

    print("Waiting for shadow publish...")
    try:
        threading.Event().wait()
    finally:
        disconnect_future = mqtt_connection.disconnect()
        disconnect_future.result()

if __name__ == "__main__":
    main()
```

## Publishするファイル

Shadowが更新、取得のリクエストをPublishすることができます。

```python:publish.py
from awscrt import io, mqtt
from awsiot import mqtt_connection_builder, iotshadow


def main():
    mqtt_connection = mqtt_connect()
    shadow = iotshadow.IotShadowClient(mqtt_connection)

    # Updateを送信する
    update_shadow_future = shadow.publish_update_named_shadow(
        request=iotshadow.UpdateNamedShadowRequest(
            thing_name="Test-Device",
            shadow_name="test1",
            state=iotshadow.ShadowState(
                # デバイスの現在の状態
                reported=None,
                reported_is_nullable=True,
                # デバイスが達成すべき目標の状態
                desired={
                    "settings": {
                        "mode": 1,
                        "temp": 20,
                    },
                    "firmware_url": "https://s3.com/firmware.bin",
                },
                desired_is_nullable=True,
                # reportedとdesiredの差分
                # delta=None,
                # delta_is_nullable=True,
            ),
        ),
        qos=mqtt.QoS.AT_LEAST_ONCE,
    )
    update_shadow_future.result()

    # Getを送信する
    get_shadow_future = shadow.publish_get_named_shadow(
        request=iotshadow.GetNamedShadowRequest(
            thing_name="Test-Device",
            shadow_name="test1",
        ),
        qos=mqtt.QoS.AT_LEAST_ONCE,
    )
    get_shadow_future.result()

    print("Publish Complete!")

    disconnect_future = mqtt_connection.disconnect()
    disconnect_future.result()

if __name__ == "__main__":
    main()
```

## AWS SDK(boto3)からShadowを取得するファイル

(おまけです)
もし、アプリを作っている時に、アプリ側からShadowの情報を操作したい時はAWS SDKを使います。
取得する場合のコードになります。

```python:get_shadow.py
import json
import boto3

def main():
    client = boto3.client("iot-data")
    response = client.get_thing_shadow(thingName="Test-Device", shadowName="test1")
    # StreamingBodyからデータを読み込む
    stream = response["payload"]
    shadow_payload = stream.read()

    # バイトデータをJSONにデコード
    shadow_data = json.loads(shadow_payload)

    # デコードされたデータを表示
    print(shadow_data)

if __name__ == "__main__":
    main()
```

# 使い方

それぞれ別タブで起動します。

```sh
python subscribe.py
python publish.py
```

subscribe側のタブでは、PublishされたことをSubscribeしているので、以下のような情報を取得できると思います。

```sh
Waiting for shadow publish...
Shadow Update Accepted:
awsiot.iotshadow.ShadowState(desired={'settings': {'mode': 1, 'temp': 20}, 'firmware_url': 'https://s3.com/firmware.bin'}, desired_is_nullable=False, reported=None, reported_is_nullable=True)

Shadow Get Accepted:
awsiot.iotshadow.ShadowStateWithDelta(delta={'settings': {'mode': 1, 'temp': 20}, 'firmware_url': 'https://s3.com/firmware.bin'}, desired={'settings': {'mode': 1, 'temp': 20}, 'firmware_url': 'https://s3.com/firmware.bin'}, reported=None)
```

# おわりに

MQTTやPublish,Subscribeがややこしいですが、
分かってくると普通のJSONをやりとりしているような気分で使えそうです。

# 参考

<https://github.com/aws/aws-iot-device-sdk-python-v2/blob/main/samples/shadow.py>

<https://docs.aws.amazon.com/ja_jp/iot/latest/developerguide/iot-device-shadows.html>
<https://docs.aws.amazon.com/ja_jp/iot/latest/developerguide/using-device-shadows.html#using-device-shadows-app-update>

<https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/iot-data/client/get_thing_shadow.html>
