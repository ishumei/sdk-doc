## 1 Proxy Access

The main task of the proxy server is to receive smsdk request data, determine whether it is necessary to decrypt the device data according to business requirements, transmit the data to the Shumei device fingerprint server, and then transmit the response data from the device fingerprint service back to smsdk.

### 1.1 Construct Request Parameters

1. Request Method: POST

2. Request Domain: The proxy server needs to request the corresponding Shumei device fingerprint server based on the business data center region.

   | Device Fingerprint Request | Android/iOS                                              |
   | -------------------------- | -------------------------------------------------------- |
   | Domestic                   | http://fp-proxy.fengkongcloud.com/deviceprofile/v4       |
   | Southeast Asia             | http://fp-sa-proxy.fengkongcloud.com/deviceprofile/v4    |
   | Europe and America         | http://fp-na-proxy.fengkongcloud.com/deviceprofile/v4    |
   | Frankfurt                  | http://api-device-eur.fengkongcloud.com/deviceprofile/v4 |

   | Cloud Configuration Request | Android/iOS                                          |
   | --------------------------- | ---------------------------------------------------- |
   | Domestic                    | http://fp-proxy.fengkongcloud.com/v3/cloudconf       |
   | Southeast Asia              | http://fp-sa-proxy.fengkongcloud.com/v3/cloudconf    |
   | Europe and America          | http://fp-na-proxy.fengkongcloud.com/v3/cloudconf    |
   | Frankfurt                   | http://api-device-eur.fengkongcloud.com/v3/cloudconf |

3. Request Header: Place the real IP, X-Forwarded-For, and User-Agent uploaded by smsdk in the HTTP Header.

   | Parameter        | Source                                               |
   | ---------------- | ---------------------------------------------------- |
   | X-Real-IP        | Client's real IP, avoid using the IP of the relay server |
   | X-Forwarded-For  | Client's X-Forwarded-For, place the proxy service IP at the end |
   | User-Agent       | Client's real User-Agent, avoid using the UA of the relay server |

   Note: The Content-Type in the header information of the request sent by the SDK is application/octet-stream.

4. Request Body: The Body data reported by smsdk.

### 1.2 Processor Response

1. Success Data: code is 1100

   ```json
   // Example 1
   {
     "code": 1100,
     "detail": {
       "deviceId": "H7AGEniCL9dZficNw7ppO4Emp7S4xFKhz+WXukRNiKI3s0Oto9Hq5Zzev4oUy2NQtmlT9Xkdj4tOrO5brIMMbQ=="
     },
     "requestId": "d8ec50b3835d84e4483f49be8cec7e5b"
   }
   ```

   ```json
   // Example 2
   {
     "code":1100,
     "detail":{
       "c":50,
       "deviceId":"d+l72wI2iir+FcnCyCQf8WgSOQcHiCbYcUaGF4QxY/x7nYmJzhMRRqdrTisaphGMTv9tfzrHxFn11YTAapc/og==",
       "t":1440
     },
     "requestId":"77bba96c0dead4bb53c2a1ee4f3122e0"
   } 
   ```

   Directly transmit this type of response to smsdk.

2. Failure Data: code is not 1100, such as

   ```json
   {
     "code":1902,
     "requestId":"6dc6a15f940b9a3e50adb9c29f38bbdc"
   } 
   ```

   Directly transmit this type of response to smsdk, return code and processing instructions.

   | code | Description                                                  |
   | ---- | ------------------------------------------------------------ |
   | 1100 | Success, no processing required                              |
   | 1901 | QPS limit exceeded, notify Shumei customer service to adjust the QPS limit |
   | 1902 | Invalid smsdk parameters, check whether the smsdk configuration is correct on the client side, or provide the corresponding requestId to contact Shumei technical support for troubleshooting |
   | 1903 | Service failure, contact Shumei technical support for troubleshooting, provide the corresponding requestId |
   | 9101 | Unauthorized operation, contact Shumei customer service to confirm whether the corresponding service is enabled |

3. No Response or Response Timeout: When the proxy service request to the Shumei server fails, such as handshake failure, timeout, etc., the following content can be transmitted to the client.

   ```json
   {
     "code":1903,
     "message":"Error reason"
   }
   ```

   The client SDK will retry when encountering the 1903 error code.

### 1.3 Decrypt Device Data

The proxy server receives the data reported by smsdk, determines the platform based on the `os` field, and then uses the device fingerprint decryption tool + the corresponding platform key to decrypt it. Example data:

```json
{
  "organization": "ORGANIZATION",
  "os": "android",
  "appId": "default",
  "encode": 2,
  "compress": 3,
  "data": "ENCRYPT-DATA",
  "tn": "TN",
  "ep": "EP"
}
```

The device fingerprint decryption tool is provided by default in Java and Go versions. If other language SDKs are needed, please contact Shumei staff. For specific usage, refer to the device fingerprint decryption tool API documentation. Below is the usage method of the Java version tool `SmFpCrypto.decrypt`.

```java
// postData is the data reported by smsdk, the format is as shown in the "Example Data" above
JSONObject postDataJson = new JSONObject(postData);
// 1 Get the platform
String os = postDataJson.optString("os", "");

// 2 Get the corresponding RSA private key based on the platform
String privateKey;
if (os.equals("android")) {
  privateKey = new String(Files.readAllBytes(Paths.get("YOUR-Android-PRIVATE-KEY-PATH")));
} else if (os.equals("ios")) {
  privateKey = new String(Files.readAllBytes(Paths.get("YOUR-iOS-PRIVATE-KEY-PATH")));
} else if (os.equals("harmony")) {
  privateKey = new String(Files.readAllBytes(Paths.get("YOUR-Harmony-PRIVATE-KEY-PATH")));
} else if (!os.isEmpty()) {
  System.out.println("Other platforms are not supported yet");
  return null;
} else {
  System.out.println("postData format error");
  return null;
}

// 3 Call the decryption tool to decrypt, decData is the decrypted data, the format is a json string, refer to the "Device Fingerprint Collection List" document for the data content
String decData = SmFpCrypto.decrypt(postData, privateKey, "");
```

## 2 Device Fingerprint Identification Decryption

boxId and boxData need to be decrypted to be used as device identifiers.

The boxId decryption tool is provided by default in Java and Go versions. It is prohibited to place the decryption tool on the terminal (such as Android or iOS) for decryption. If other language SDKs are needed, please contact Shumei staff. For specific usage, refer to the boxId decryption tool API documentation. Below is the usage method of the Java version tool `BoxIdTranslator.outputSmid`.

```java
final String boxId = "Bxxxxxx=="; // boxId
String smid = null;
try {
  // YOUR-KEY is the key for the boxId decryption tool
  smid = BoxIdTranslator.outputSmid(boxId, YOUR-KEY);
} catch (Exception e) {
  // boxId is illegal, modify according to the log prompt
  e.printStackTrace();
}
```

boxData decryption requires the use of the Tianxiang service, and you need to contact Shumei staff to enable the relevant service. After the service is enabled, request the device risk label according to the Tianxiang user manual, and the returned data will contain the plaintext device fingerprint identifier.

If there is no device fingerprint label in the Tianxiang data, you need to provide the boxData and requestId to Shumei staff for troubleshooting.