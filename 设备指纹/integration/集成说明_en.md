## 1 Terminology Explanation

| Term         | Explanation                                                  |
| ------------ | ------------------------------------------------------------ |
| Organization | Organization identifier, obtained through service activation or sent via separate email; one of the required parameters to start the SDK |
| AppId        | Application identifier, obtained through service activation or sent via separate email; one of the required parameters to start the SDK |
| PublicKey    | Encryption public key, obtained through service activation or sent via separate email; one of the required parameters to start the SDK |
| boxId        | Device fingerprint encryption identifier, value obtained by the client through the getDeviceId() method, a string starting with the letter 'B', with a length of 89 characters |
| boxData      | Device fingerprint encryption data, value obtained by the client through the getDeviceId() method, a string starting with the letter 'D', with a length of about 8K characters |
| Standard Access | SAAS mode, device data is hosted on the Shumei platform for unified management, divided into domestic, Southeast Asia, and Europe and America data centers |
| Proxy Access | Device data is first sent to the business proxy server for data review or other data management, and the business proxy server forwards the uploaded device data |
| Private Access | Device data is stored in the customer's data center, equivalent to deploying SAAS services in the customer's data center |

## 2 Standard Access

All non-customized access belongs to standard access, which is the simplest and fastest, suitable for domestic users. The overall data flow diagram of standard access is as follows

![fp-std-flow](./res/fp-std-flow.png)

Key steps for access

1. The app integrates the mobile SDK, including platforms: Android/iOS/Web/mini programs
2. (Steps 1 to 4 in the diagram) Start the SDK for data collection and obtain the boxId
3. (Step 5 in the diagram) Carry the boxId or boxData when reporting business events such as registration and login to the business server
4. (Steps 6 to 8 in the diagram) The business server queries device risks or device identifiers based on the boxId or boxData

## 3 Proxy Access

Proxy access requires the customer to set up a proxy server, and the device fingerprint SDK transfers device data through the proxy server. Proxy access mainly has two purposes: first, for overseas user access, data is first processed by the business proxy server for compliance; second, users need to use the collected data for secondary development. The overall data flow diagram of proxy access is as follows

![fp-proxy-flow](./res/fp-proxy-flow.png)

Key steps for access

1. The app integrates the mobile SDK, including platforms: Android/iOS/Web/mini programs
2. Set up a proxy server, where the device data reported in step 2.1 of the diagram can be decrypted, and data review or other secondary development can be performed.
3. (Steps 1 to 4.2 in the diagram) Collect and pass through device data and boxId
4. (Step 5 in the diagram) Carry the boxId or boxData when reporting business events such as registration and login to the business server
5. (Steps 6 to 8 in the diagram) The business server queries device risks or device identifiers based on the boxId or boxData

The proxy server needs to be set up by the customer, and its stability must be ensured to avoid data anomalies caused by the proxy server.

## 4 Private Access

Private access is highly customized and requires Shumei to provide deployment services, including device fingerprint services, device profiling, event services, etc. Once deployed, the entire data flow is the same as standard access, with the difference being that all services are deployed in the customer's data center.