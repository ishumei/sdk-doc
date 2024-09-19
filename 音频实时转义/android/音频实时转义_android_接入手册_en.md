# android Audio Real-time Transcription SDK Integration Steps

1. **Import SDK Package**

   1. Copy `smrtasr-${version}.aar` to the libs directory of the app Module, as shown in the figure

   ![image-20211110150536784](./res/res_001.png)

   The name `smrtasr-x.x.x.aar` where `x.x.x` represents the version number, such as `smrtasr-1.0.0.aar`

   2. Add the following configuration to the `app/build.gradle` file

   ```groovy
   dependencies {
     // Add the SDK to libs and replace smrtasr-x.x.x.aar with the corresponding version SDK
     implementation fileTree(dir: "libs", include: ["smrtasr-x.x.x.aar"])
   }
   ```

   3. Add permissions in the `AndroidManifest.xml` file

   ```xml
   <uses-permission android:name="android.permission.INTERNET" />
   ```

   4. The SDK supports a minimum of Android API LEVEL 14, ensure that the project's minSdkVersion is not less than 14

   ```groovy
   // build.gradle
   android {
     ...
     defaultConfig {
       minSdkVersion 14
       ...
     }
     ...
   }
   ```

   The above import process is carried out in Android Studio. If using the SDK in other IDEs, you need to adapt it yourself.

2. **SDK Code Integration**

   1. Start the SDK

      ```java
      SmRtAsrClient.AsrOption option = new SmRtAsrClient.AsrOption();
      // Required parameter, Organization provided by SmRtAsr
      option.setOrganization(YOUR_ORGANIZATION);
      // Required parameter, AccessKey provided by SmRtAsr
      option.setAccessKey(YOUR_ACCESS_KEY);
      // Optional parameter, default value is "default", format: a string of letters less than 64 characters
      option.setAppId(YOUR_APP_ID);
      // Optional parameter, set the websocket data transmission data center, contact SmRtAsr customer service for the data center location
      option.setWsUrl(WS_URL);
      // Optional parameter, set the http data transmission data center, contact SmRtAsr customer service for the data center location, will temporarily switch to http communication when websocket communication fails
      option.setHttpUrl(SESSION_START_URL);
      
      // Start, context is a non-null required parameter
      SmRtAsrClient client = SmRtAsrClient.create(context, option);
      ```

      If required parameters are empty or have format exceptions, such as parameters containing whitespace characters, the `create` method will return a `null` object. Please modify the relevant parameters according to the `logcat` prompt, the log TAG is `Smlog`.

   2. Start a session

      ```java
      SessionConfig config = new SessionConfig();
      // Set the session language, default is "zh", format: http://www.lingoes.cn/zh/translator/langcode.htm
      config.setLanguage(String);
      // Set whether matching function is needed, default is false, if set to true, matching content must be set
      config.setEnableMatch(boolean);
      // Matching mode, keyword contains matching mode, supports parameters
      //  - text: text contains matching (default value)
      //  - number: number equals matching
      config.setMatchMode(String);
      // Set matching content, text contains matching sets keywords, number equals matching sets key number in Arabic form (e.g., "123")
      // Up to 5 elements can be passed in the matching content, each less than or equal to 30 characters
      List<String> keywords = new ArrayList<>();
      keywords.add(String);
      config.setKeywords(keywords);
      // Set whether to return transcribed text, default is true
      config.setReturnText(boolean);
      // Set whether number extraction function is needed, default is false
      config.setReturnNumbers(boolean);
      // Set audio type, default is pcm
      config.setVoiceType(String);
      // Set audio sampling rate, default is 16000
      config.setVoiceSample(int);
      // Set audio encoding format, default is s16le
      config.setVoiceEncode(String);
      // Extended fields
      config.setExtra(JSONObject);
      // Callback listener, note: the callback method is in the main thread, do not perform time-consuming operations directly in the callback method.
      config.setListener(new SmAsrSessionListener());
      // Start session
      client.startSession(config);
      ```

      After starting the session, you can send audio data. If no audio data is sent for a period of time after starting the session, the session will expire and need to be restarted. The SDK currently does not support multi-session interaction, multiple calls to `startSession` will only make the latest session valid.

      The structure of the SmAsrSessionListener class is as follows

      ```java
      public interface SmAsrSessionListener {
          //
          // Receive data returned from the server, this method will be triggered multiple times during postAudio
          //
          // @param sessionId Session ID, used to uniquely identify a session
          // @param requestId Request ID, used to uniquely identify a request
          // @param response {@link SmAsrResponse}
          //
          void onReceived(String sessionId, String requestId, SmAsrResponse response);
      
          //
          // Exception, see the error code table for specific error codes and handling methods
          //
          // @param sessionId Session ID, used to uniquely identify a session
          // @param requestId Request ID, used to uniquely identify a request
          // @param errCode Exception status code, a status code corresponds to a type of error
          // @param message Information corresponding to the exception code, a status code may have multiple error messages
          //
          void onError(String sessionId, String requestId, int errCode, String message);
      }
      ```

      The main methods of SmAsrResponse are as follows

      ```java
      // Recognition result, original text
      public String getText();
      // Sentence break identifier
      //  0 represents an intermediate sentence
      //  1 represents a complete sentence
      public int getType();
      // Determine whether the recognition result hits the keyword
      public boolean isHit();
      // Matching result
      public JSONArray getMatchResults();
      // Extracted numbers
      public JSONArray getNumbers();
      ```

      The data structure returned by the `getMatchResults()` method is as follows:

      ```json
      [{
        // Matched content, String type
        "hitItemContent": "你好", // For number equals matching, this field value is a number string, such as "123"	
        // The start (inclusive) and end (exclusive) positions of the matched content in the original text, if the matched content appears multiple times, multiple positions will appear
        "positions": [[0, 2], [2, 4]]
      }, {
        "hitItemContent": "世界", 
        "positions": [[5, 7]]
      }]
      ```

      The data structure returned by the `getNumbers()` method is as follows:

      ```json
      [{
        // Extracted number, integer type
        "number": 123,	
        // The start and end positions of the extracted number in the original text, if the matched content appears multiple times, multiple positions will appear
        "positions": [[0, 3], [5, 8]]
      }, {
       "number": 456,	
        "positions": [[10, 13]]
      }]
      ```

   3. Send audio data
   
      ```java
      // Send audio data
      //
      // This method will copy the audio data and then internally process the copied data for reporting. To ensure recognition effect, do not modify the audio data during the copying process
      //
      // @param audio Audio data, default type: pcm, encoding: s16le, sampling: 16000
      // @return Whether the audio data was sent successfully
      //  - false: No session started, or session failure caused the current audio not to be added to the forwarding queue
      //  - true: Sent successfully, the result will be displayed in the {@link SmAsrSessionListener#onReceived(SmAsrResponse)} callback
      client.postAudio(byte[] audio);
      ```
   
      Each audio slice needs to be controlled between 300 to 400 milliseconds.
   
      After sending audio data, you can listen to the callback result in the `void onReceived(String, String, SmAsrResponse)` callback, such as calling `response.getText()` to get the current audio-to-text information, and using `response.isHit()` to determine whether the current audio hits the keyword.
   
   4. Close the session
   
      ```java
      // End session
      client.stopSession();
      ```
   
      Call the `stopSession()` method to actively close the session.
   
   5. Destroy the SDK
   
      ```java
      // Destroy the SDK to release resources
      client.destroy()
      ```
   
      Call the `destroy()` method, the SDK will release resources such as network threads, cached data, etc.
   
3. **Error Codes**

   | Error Code | Message       | Cause                                                         | Recommended Handling Method                                   |
   | ---------- | ------------- | ------------------------------------------------------------- | ------------------------------------------------------------- |
   | 1901       | qps limit     | Request reached the server's preset qps limit                 | If triggered frequently, contact SmRtAsr personnel to increase the qps limit |
   | 1902       | Invalid parameter | Parameter error                                             | 1. Call stopSession to end this session<br />2. Check if Organization and AccessKey are filled in incorrectly |
   | 2101       | No network    | Device has no network                                         | Ignore                                                        |
   | 2102       | Voice packet loss | Network exception caused the voice segment to fail to upload to the server | Ignore                                                        |
   | 2103       | Voice abandoned | Sending voice packets too frequently (e.g., more than 100 segments sent in 4 seconds) exceeded the cache limit, causing newly added voice segments to be discarded | Ignore                                                        |
   | 2301       | Status error  | SDK API call error, such as calling startSession without starting | Ensure the SDK is started first                               |
   | 2401       | Server error  | Server did not send data according to the protocol            | Ignore                                                        |
   | 2402       | Server error  | Server did not send data according to the protocol            | Ignore                                                        |
   | 2403       | Server error  | Server did not send data according to the protocol            | Ignore                                                        |
   | 9101       | Unauthorized operation | Parameter error or related service not activated           | Call stopSession to end this session<br />Check if Organization and AccessKey are filled in incorrectly, contact SmRtAsr personnel to check if the related service is activated |