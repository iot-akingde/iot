# Establish communication over the CoAP protocol {#concept_gn3_kr5_wdb .concept}

IoT Platform supports connections over CoAP. CoAP is suitable for resource-constrained, low-power devices, such as NB-IoT devices. This topic describes how to connect devices to IoT Platform over CoAP and two supported authentication methods, which are DTLS and symmetric encryption.

## CoAP-based connection procedure {#section_sf1_mr5_wdb .section}

The following figure shows the procedure for connecting NB-IoT devices to IoT Platform.

![](http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/7504/15518531613114_en-US.png)

The procedure is as follows:

1.  Integrate an Alibaba Cloud IoT Platform SDK into the NB-IoT module of device clients. Specifically, in the IoT Platform console, you need to register products and devices, obtain the unique device certificates \(that is, the ProductKey, DeviceName, and DeviceSecret components\), and then install the certificates to the devices.
2.  Establish a connection over your target carriers' cellular networks for NB-IoT devices to connect to IoT Platform. We recommend that you contact your local carrier to make sure that the NB-IoT network is available in the region where your devices are located.
3.  After the devices are connected to IoT Platform, a machine-to-machine \(M2M\) platform manages the data traffic and fees incurred by the NB-IoT devices. The M2M platform is operated by your specified carrier.
4.  Over the CoAP/UDP protocol, devices send data to IoT Platform in real time. IoT Platform is a secure service that can connect and manage data for hundreds of millions of NB-IoT devices. Then, through Rules Engine of IoT Platform, the device data can be forwarded to other Alibaba Cloud product instances, such as big data products, ApsaraDB for RDS, Table Store, and other products.
5.  Use the APIs and message pushing services provided by IoT Platform to forward data to supported service instances and quickly integrate device assets and actual applications.

## Use the symmetric encryption method {#section_j5f_cq3_bfb .section}

1.  Connect to the CoAP server. The endpoint address is `${YourProductKey}.coap.cn-shanghai.link.aliyuncs.com:${port}`.

    Note:

    -   `${YourProductKey}`: Replace it with the ProductKey value of the device.
    -   `${port}`: The port number. Set the value to 5682.
2.  Authenticate the device.

    Request message:

    ```
    POST /auth
    Host: ${YourProductKey}.coap.cn-shanghai.link.aliyuncs.com
    Port: 5682
    Accept: application/json or application/cbor
    Content-Format: application/json or application/cbor
    payload: {"productKey":"a1NUjcVkHZS","deviceName":"ff1a11e7c08d4b3db2b1500d8e0e55","clientId":"a1NUjcVkHZS&ff1a11e7c08d4b3db2b1500d8e0e55","sign":"F9FD53EE0CD010FCA40D14A9FEAB81E0", "seq":"10"} 
    ```

    |Parameter|Description|
    |:--------|:----------|
    |Method|The request method. The supported method is POST.|
    |URL|/auth.|
    |Host|The endpoint address. The format is `$\{YourProductKey\}.coap.cn-shanghai.link.aliyuncs.com`. Replace $\{YourProductKey\} with the ProductKey value of the device.|
    |Port|The port number. Set the value to 5682.|
    |Accept|The encoding format of the data that is to be received by the device. Currently, application/json and application/cbor are supported.|
    |Content-Format|The encoding format of the data that the device sends to IoT Platform. Currently, application/json and application/cbor are supported.|
    |payload|The device information for authentication, in JSON format. For more information, see the following table [payload parameters](#).|

    |Parameter|Required|Description|
    |:--------|:-------|:----------|
    |productKey|Yes|The unique identifier issued by IoT Platform to the product. You can obtain this information on the device details page in the IoT Platform console.|
    |deviceName|Yes|The device name that you specified, or is generated by IoT Platform, when you registered the device. You can obtain this information on the device details page in the IoT Platform console.|
    |ackMode|No|The communication mode. Options:    -   0: After receiving a request from the device client, the server processes data and then returns the result with an acknowledgment \(ACK\).
    -   1: After receiving a request from the client, the server immediately returns an ACK and then starts to process data. After the data processing is complete, the server returns the result.
The default value is 0.

|
    |sign|Yes|Signature.The signature algorithm is `hmacmd5(DeviceSecret,content)`.

The value of content is a string that is built by sorting and concatenating all the parameters \(except version, sign, resources, and signmethod\) that need to be submitted to the server in alphabetical order, without any delimiters.

Signature calculation example:

    ```
sign= hmac_md5(deviceSecret, clientId123deviceNametestproductKey123seq123timestamp1524448722000)
    ```

 |
    |signmethod|No|The algorithm type. The supported types are hmacmd5 and hmacsha1.|
    |clientId|Yes|The device client ID, which can be any string up to 64 characters in length. We recommend that you use the MAC address or the SN code of the device as the clientId.|
    |timestamp|No|The timestamp. Currently, timestamp is not verified.|

    Response example:

    ```
    {"random":"ad2b3a5eb51d64f7","seqOffset":1,"token":"MZ8m37hp01w1SSqoDFzo0010500d00.ad2b"}
    ```

    |Parameter|Description|
    |---------|-----------|
    |random|The encryption key used for data communication.|
    |seqOffset|The authentication sequence offset.|
    |token|The returned token after the device is authenticated.|

3.  The device sends data.

    Request message:

    ```
    POST /topic/${topic}
    Host: ${YourProductKey}.coap.cn-shanghai.link.aliyuncs.com
    Port: 5682
    Accept: application/json or application/cbor
    Content-Format: application/json or application/cbor
    payload: ${your_data}
    CustomOptions: number:2088(token), 2089(seq)
    ```

    |Parameter|Required|Description|
    |---------|--------|-----------|
    |Method|Yes|The request method. The supported request method is POST.|
    |URL|Yes|The format is `/topic/${topic}`. Replace the variable $\{topic\} with the device topic used by the device to publish data.|
    |Host|Yes|The endpoint address. The format is `${YourProductKey}.coap.cn-shanghai.link.aliyuncs.com`. Replace the variable $\{YourProductKey\} with the ProductKey value.|
    |Port|Yes|The port number. Set the value to 5682.|
    |Accept|Yes|The encoding format of the data which is to be received by the device. Currently, application/json and application/cbor are supported.|
    |Content-Format|Yes|The encoding format of the data which is sent by the device. Currently, application/json and application/cbor are supported.|
    |payload|Yes|The encrypted data that is to be sent. Encrypt the data using the Advanced Encryption Standard \(AES\) algorithm.|
    |CustomOptions|Yes|The option value can be 2088 and 2089, which are described as follows:    -   2088: Indicates the token. The value is the token returned after the device is authenticated.

**Note:** Token information is required every time the device sends data. If the token is lost or expires, initiate a device authentication request again to obtain a new token.

    -   2089: Indicates the sequence. The value must be greater than the seqOffset value that is returned after the device is authenticated, and must be a unique random number. Encrypt the value with AES.
Response message for option

    ```
number:2090 (IoT Platform message ID)
    ```

|

    After a message has been sent to IoT Platform, a status code and a message ID are returned.


## Establish DTLS connections {#section_vlc_pdk_b2b .section}

1.  Connect to the CoAP server. The endpoint address is `${YourProductKey}.coap.cn-shanghai.link.aliyuncs.com:${port}`.

    Note:

    -   $\{YourProductKey\}: Replace it with the ProductKey value of the device.
    -   $\{port\}: The port number. Set the port number to 5684 for DTLS connections.
2.  Download the [root certificate](http://aliyun-iot.oss-cn-hangzhou.aliyuncs.com/cert_pub/root.crt?spm=5176.doc30539.2.1.1MRvV5&file=root.crt).
3.  Authenticate the device. Call auth to authenticate the device and obtain the device token. Token information is required when the device sends data to IoT Platform.

    Request message:

    ```
    POST /auth
    Host: ${YourProductKey}.coap.cn-shanghai.link.aliyuncs.com
    Port: 5684
    Accept: application/json or application/cbor
    Content-Format: application/json or application/cbor
    payload: {"productKey":"ZG1EvTEa7NN","deviceName":"NlwaSPXsCpTQuh8FxBGH","clientId":"mylight1000002","sign":"bccb3d2618afe74b3eab12b94042f87b"}
    ```

    For more information about parameters \(except for Port parameter, where the port for this method is 5684\) and payload content, see [Parameter description](#).

    Response example:

    ```
    response: {"token":"f13102810756432e85dfd351eeb41c04"}
    ```

    |Code|Message|Payload|Description|
    |----|-------|-------|-----------|
    |2.05|Content|The token is contained in the payload if the authentication has passed.|The request is successful.|
    |4.00|Bad Request|no payload|The payload in the request is invalid.|
    |4.01|Unauthorized|no payload|The request is unauthorized.|
    |4.03|Forbidden|no payload|The request is forbidden.|
    |4.04|Not Found|no payload|The requested path does not exist.|
    |4.05|Method Not Allowed|no payload|The request method is not allowed.|
    |4.06|Not Acceptable|no payload|The value of Accept parameter is not in a supported format.|
    |4.15|Unsupported Content-Format|no payload|The value of Content-Format parameter is not in a supported format.|
    |5.00|Internal Server Error|no payload|The authentication request is timed out or an error occurred on the authentication server.|

4.  The device sends data.

    The device publishes data to a specified topic.

    In the IoT Platform console, on the Topic Categories tab page of the product, you can create topic categories.

    Currently, only topics with the permission to publish messages can be used for publishing data, for example, `/${YourProductKey}/${YourDeviceName}/pub`. Specifically, if a device name is device, and its product key is a1GFjLP3xxC, the device can send data through the address `a1GFjLP3xxC.coap.cn-shanghai.link.aliyuncs.com:5684/topic/a1GFjLP3xxC/device/pub`.

    Request message:

    ```
    POST /topic/${topic}
    Host: ${YourProductKey}.coap.cn-shanghai.link.aliyuncs.com
    Port: 5684
    Accept: application/json or application/cbor
    Content-Format: application/json or application/cbor
    payload: ${your_data}
    CustomOptions: number:2088(token)
    ```

    |Parameter|Required|Description|
    |:--------|:-------|:----------|
    |Method|Yes|The request method. The supported request method is POST.|
    |URL|Yes|`/topic/${topic}` Replace the variable `${topic}` with the device topic which will be used to publish data.|
    |Host|Yes|The endpoint address. The format is `$\{YourProductKey\}.coap.cn-shanghai.link.aliyuncs.com`. Replace $\{YourProductKey\} with the ProductKey value of the device.|
    |Port|Yes|Set the value to 5684.|
    |Accept|Yes|The encoding format of the data that is to be received by the device. Currently, application/json and application/cbor are supported.|
    |Content-Format|Yes|The encoding format of the data that the device sends to IoT Platform. Currently, application/json and application/cbor are supported.|
    |CustomOptions|Yes|     -   Number: 2088.
    -   The value of token is the token information returned after [auth](#) is called to authenticate the device.
 **Note:** Token information is required every time the device sends data. If the token is lost or expires, initiate a device authentication request again to obtain a new token.

 |


