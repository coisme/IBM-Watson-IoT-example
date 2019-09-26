# Mbed OSデバイスからIBM Watson IoT Platformに接続する手順

このページでは、Mbed OSデバイスからIBM Watson IoT Platformに接続する手順を説明します。

![](https://os.mbed.com/media/uploads/coisme/ibm-001.png)

アプリケーションとして、Mosquittoを使用します。

## 事前準備

* IBM Cloudアカウントの作成（アカウントがない場合は、[ここ](https://cloud.ibm.com/registration?locale=ja)から作成）
* Mbed OS 対応ボード
* ネットワーク接続環境
* Mosquitto
* Mbed アカウント
* Mbed CLI（オプション）

## テスト環境
* macOS Mojave 10.14.6
* Mbed OS 対応ボード - FRDM-K64F, Seeed Wio 3G, Renesas GR-PEACH, GR-LYCHEE
* Ethernet, Wi-Fi, 3Gセルラー
* mosquitto version 1.6.4

## Watson IoT Platformサービスの設定

[この資料](https://www.ibm.com/developerworks/cloud/library/cl-mqtt-bluemix-iot-node-red-app/index.html)のstep 1を参照してWatson IoT Platformをセットアップします。以下のパラメータをメモしてください。

デバイス側

* 組織 ID
* デバイス・タイプ
* デバイス ID
* 認証トークン

アプリケーション側

* APIキー
* 認証トークン

## デバイス側のファームウェアをビルドする

ビルドするには、オンライコンパイラかCLIが使用可能です。以下は、オンラインコンパイラの例です。

### プロジェクトのインポート

オンラインコンパイラにログインしてインポートをクリックします。インポートウィザードの画面になります。以下の`Click here`をクリックします。

![](https://os.mbed.com/media/uploads/coisme/ms-003.png)

`Source URL`にリポジトリのURL（https://github.com/coisme/Mbed-to-IBM-Watson-IoT）を入力します。

![](https://os.mbed.com/media/uploads/coisme/ibm-002.png)

自分のワークスペースにプロジェクトがインポートされます。

### パラメータを変更する

プロジェクトのルートにある`MQTT_server_setting.h` を編集します。以下の4つのパラメータを変更してください。

* `ORG_ID` - デバイス側の`組織 ID`です
* `DEVICE_TYPE` - デバイス側の`デバイスタイプ`です。
* `DEVICE_ID` - デバイス側の`デバイス ID`です
* `TOKEN` - デバイス側の`認証トークン`です

### ビルド

メニュー上部のCompileボタンを押してビルドします。

![](https://os.mbed.com/media/uploads/coisme/ms-005.png)

ビルドが成功すると、<プロジェクト名>.bin ファイルがダウンロードされます。このファイルをターゲットボードに書き込んでください（USBマスストレージにコピーする）。

### ボードに接続する

ボードからはUSBシリアル経由で実行ログが表示されます。TeraTermやCoolTerm等のシリアルモニタで、接続します。ボーレート、データ、パリティ、ストップビットは、115200,8,N,1です。

ボード上のリセットボタンを押すとプログラムが起動します。以下のようなメッセージが表示されます。

```
Mbed to Watson IoT : version is 0.10

Opening network interface...
IP address: 10.128.4.77
Network interface opened successfully.

Time is now Thu Sep 26 05:34:42 2019
Connecting to host xxxxx.messaging.internetofthings.ibmcloud.com:8883 ...
Connection established.

MQTT client is connecting to the service ...
Client connected.

Client is trying to subscribe a topic "iot-2/cmd/myevt/fmt/text".
Client has subscribed a topic "iot-2/cmd/myevt/fmt/text".

To send a packet, push the button 1 on your board.
```

## デバイスからアプリケーションにメッセージを送る

デバイスからアプリケーションにメッセージを送ってみます。 デバイスからアプリケーションのメッセージはeventと呼ばれます。Mosquittoを使ってデバイスからのイベントをIBM Watson IoT Platform経由で受信します。

![](https://os.mbed.com/media/uploads/coisme/ibm-004.png)

ボード上のユーザーボタンを押すと、eventがWatson IoT Platformに転送されます。Watson IoT Platformはそれをアプリケーションに中継します。

### アプリケーションの起動

 サーバールートCA証明書 IoTFoundation.pem を[ここからダウンロード](https://github.com/ibm-watson-iot/iot-cpp/blob/master/IoTFoundation.pem)します。

Mosquittoのアプリケーションを起動するには、ターミナルからスクリプトを実行します。以下のスクリプトの`orgId`, `cafile`, `password`, `username`, `deviceType`, 及び`deviceId`を変更してください。

```shell
#!/bin/tcsh
 
set orgId="<< YOUR ORG ID >>"
set cafile="<< path to >>/IoTFoundation.pem"
set password="<< YOUR AUTHENTICATION TOKEN for APPLICATION >>"
set username="<< YOUR API KEY >>"
set deviceType="<< YOUR DEVICE TYPE >>"
set deviceId="<< YOUR DEVICE ID >>"
 
set cid="a:${orgId}:apitest"
set host="${orgId}.messaging.internetofthings.ibmcloud.com"
set port=8883
set eventId="start"
set topic="iot-2/type/${deviceType}/id/${deviceId}/evt/${eventId}/fmt/text"
 
mosquitto_sub -h ${host} -p ${port} -q 1\
    -t "${topic}" \
    -i ${cid} \
    -d \
    -u ${username}\
    -P ${password}\
    --cafile ${cafile}\
    -V mqttv311
```
アプリケーションスクリプトを実行させると、メッセージを受信することが出来ます。

## デバイスからメッセージを送る

ボード上のユーザーボタンを押すと、デバイスはメッセージトピック`iot-2/evt/myevt/fmt/text`をパブリッシュします。
Seeed Wio 3Gを使う場合は、D20コネクタにプッシュスイッチ（Grove Button）を取り付けてください。


```
To send a packet, push the button 1 on your board.

Publishing message to the topic iot-2/evt/start/fmt/text:
Message #0 from 112233.
Message published.

Publishing message to the topic iot-2/evt/start/fmt/text:
Message #1 from 112233.
Message published.
```

アプリケーションは、上記のスクリプトで指定されたトピックを介してWatson IoT Platformからメッセージを受け取ります。

```
Client a:xxxxx:apitest sending CONNECT
Client a:xxxxx:apitest received CONNACK (0)
Client a:xxxxx:apitest sending SUBSCRIBE (Mid: 1, Topic: iot-2/type/mbed-os-dev-1/id/112233/evt/start/fmt/text, QoS: 1, Options: 0x00)
Client a:xxxxx:apitest received SUBACK
Subscribed (mid: 1): 1
Client a:xxxxx:apitest sending PINGREQ
Client a:xxxxx:apitest received PINGRESP
Client a:xxxxx:apitest received PUBLISH (d0, q0, r0, m0, 'iot-2/type/mbed-os-dev-1/id/112233/evt/start/fmt/text', ... (23 bytes))
Message #0 from 112233.
Client a:xxxxx:apitest received PUBLISH (d0, q0, r0, m0, 'iot-2/type/mbed-os-dev-1/id/112233/evt/start/fmt/text', ... (23 bytes))
Message #1 from 112233.
```

## アプリケーションからデバイスにメッセージを送る

![](https://os.mbed.com/media/uploads/coisme/ibm-005.png)

## アプリケーションからメッセージを送る

Mosquittoを使用して、メッセージをアプリケーションとして送信します。以下のスクリプトの`orgId`, `cafile`, `password`, `username`, `deviceType`, 及び`deviceId`を変更してください。

```shell
#!/bin/tcsh
 
set orgId="<< YOUR ORG ID >>"
set username="<< YOUR APPLICATION KEY >>"
set password="<< YOUR AUTHENTICATION TOKEN for APPLICATION >>"
set deviceType="<< YOUR DEVICE TYPE >>"
set deviceId="<< YOUR DEVICE ID >>"
set cafile="<<path to >>/IoTFoundation.pem"
 
set host="${orgId}.messaging.internetofthings.ibmcloud.com"
set cid="a:${orgId}:apitest"
set cmdId="myevt"
set topic="iot-2/type/${deviceType}/id/${deviceId}/cmd/${cmdId}/fmt/text"
 
mosquitto_pub -h "${host}" -p 8883\
        -u "${username}"\
        -P "${password}"\
        -i ${cid}\
        -t "${topic}"\
        -m "`date`"\
        --cafile "${cafile}"
```
スクリプトを実行すると、`date`コマンドの出力をメッセージとして送信します。デバイスは受信したコマンドをログとして表示します。

```
Mbed to Watson IoT : version is 0.10

Opening network interface...
IP address: 10.128.4.27
Netmask: 255.255.255.0
Gateway: 10.128.4.1
Network interface opened successfully.

Time is now Thu Sep 26 05:54:08 2019
Connecting to host xxxxx.messaging.internetofthings.ibmcloud.com:8883 ...
Connection established.

MQTT client is connecting to the service ...
Client connected.

Client is trying to subscribe a topic "iot-2/cmd/myevt/fmt/text".
Client has subscribed a topic "iot-2/cmd/myevt/fmt/text".

To send a packet, push the button 1 on your board.

Message arrived:
Thu Sep 26 15:01:49 JST 2019

```
