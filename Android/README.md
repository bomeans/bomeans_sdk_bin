# Introduction

This SDK is for building Android universal remote contoller Apps.

This SDK requires Android 4.0 (API Level 19) or later version, the supported CPU architectures are arm, x86, or mips.

Two major packages in this SDK are<br>
<li>com.bomeans.IRKit: The main package providing data downloading and IR data interpretion.</li>
<li>com.bomeans.wifi2ir: Optional package which handling wifi conneciton to Bomeans WiFi IR Blaster using specific Bomeans protocol. (Users who handles the data communication on his own can safely ignore this package.)

# How to Use
## Adding SDK Files
If you are using Eclipse + ADT:<br>
<li>Copy the sub-folders containing the .so files to the \libs folder in your project.</li>
<li>Copy the .jar files to the \libs folder in your project.</li>
<p>

If you are using Android Studio:<br>
<li>Copy the sub-folders containing the .so files to the \src\main\jniLibs folder in your project.</li>
<li>Copy the .jar files to the \libs folder in your project.</li>
Note: You also need to have the following settings in your build.gradle(Module:your_app):<br>
```java
dependencies {
    ...
    compile fileTree(dir: 'libs', include: ['*.jar'])
    ...
}
```
## Adding App Permissions
The following permissions need to be added to your AndroidMenifest.xml:
```java
android.permission.INTERNET
android.permission.ACCESS_WIFI_STATE
```
# Initial Setup
## Getting the API Key
You need to apply a valid API Key for the SDK to run normally.<br>
Apply the key here: http://www.bomeans.com/Mainpage/Apply/apikey

## Setting your SDK
```java
IRKit.setup(apiKey, applicationContext)
```
where apiKey is the API Key issued by Bomeans Design. applicationContext is the application context of your app.

## Switch your target server if your target users are in China
If your target users locate in China, you can optionally switch the connected cloud server the one located in China for better performance.<br>
To switch to China server,
```java
IRKit.setUseChineseServer(true)
```
Note: If the above function is not call, a default server located outside China will be used.

# Getting the Basic Information/Lists
The SDK provides the following APIs for accessing the basic information of the remote controllers.<br>
These APIs are asynchronous, the results will be returned in the callbacks.

| API	| Description | Returned Object in Callback
| ------------- | ------------------ | -------------
| IRKit.webGetTypeList | Get type list | TypeItem
| IRKit.webGetBrandList	| Get brand list | BrandItem
| IRKit.webGetTopBrandList | Get popular brand list | BrandItem (Note1)
| IRKit.webGetModelList	| Get remote ID list |	ModelItem (Note2)
| IRKit.webVSearch	| Get remote ID list by keyword |	VoiceSearchResultItem (Note3)

Notes

<li>1: Popular brand list is the ranked list according to the popularity.</li>
<li>2: Remote ID is the internal unique id for each remote controller in the database. This ID is not necessary the model name of the remote controller itself or the model name of the appliance.</li>
<li>3: The keywords for searching the remote controllers are usually the one containing the type, brand, and/or the button name. For example, "Turn on the Panasonic TV" is a valid search string that conatins the type (TV), brand name (Panasonic), and the optional button name (Turn on as power button.)</li>

# Creating the Remote Controller Instance

Two types of remote controllers can be created:

<li>Universal Remote Controller - Univeral remote Controller is a pseudo remote controller which contains several most used remote controllers. When a button of the universal remote is pressed, the IR signals corresponding the underlying remote controllers will be send in a batch.</li>
<li>Single Remote Controller - Single remote controller in the database.</li>

| API | Description | Callback | Remark
|------------------- | ---------------------------------------------------- | ----------- | ------------------------------
| IRKit.createRemote | Create a single remote controller (with specific remote id) (Note1), or<br> create a simplified-keys universal remote controller (with null as remote_id parameter) (Note2) | BIRRemote | Both TV-type and AC-type remotes are supported.
| IRKit.createBigCombineRemote | Create a full-keys universal remote (Note3) | BIRRemote | Only TV-type remotes are supported.

Note:

<li>1. Single remote controller: A remote controller that have a one-to-one key mapping of a specific real-world remote controller. The other kind of remote controller is the Universal Remote Controller (URC) which is a pesudo remote controller containing several most popular remotes of a specific brand in it. When a key button of a URC is pressed, a sequence of IR signals corresponding to each of the underlying remotes will be sent.</li>
<li>2. Simplified-keys URC: A universal remote which have only a limited key buttons (usually the most common keys) exposed for controlling.</li>
<li>3. Full-keys URC: A universal remote which have all the available key buttons exposed for controlling.</li>

# Remote Controller Manipulation

Once the Remote Controller instance (BIRRemote) is created, you can manipulate the remote by using the following APIs:

| API | Description | TV-type<br>Remote | TV-type<br>Universal Remote | AC-type<br>Remote | AC-type<br>Universal Remote
| ------------- | ---------------------------------------------------------------| ----- | ----- | ----- | -----
| getAllKeys | Get all key(button) of the remote | V | V | V | V
| getModuleName	| Get the id of this remote | V	| | V |
| getBrandName | Get the brand name of this remote | V | V | V | V
| transmitIR | Send the IR data (Note1) | V | V | V | V
| beginTransmitIR | Start a IR transmission<br>(Call endTrasmitIR to stop transmission) | V			
| endTransmitIR | Stop IR transmission | V			
| getGUIFeature | Get the display option for AC remote (Note2) | | | V |	
| getActiveKeys | Get the currently active key(button) of the AC remote | | | V | V
| getKeyOption | Get the currently active options of the specific key(button) of a AC remote  | | | V | V	
| setKeyOption | Set the currently active option of the specific key(button) of a AC remote<br>(Since v.20161228) | | | V | V
| getTimerKeys | Get the timer-related keys(buttons) of the AC remote | | | V |	
| setOnTime | Set the ON timer of the AC remote  | | | V |
| setOffTime | Set the OFF timer of the AC remote | | | V |
| getACStoreDatas | Get the data of current states of the AC remote<br>(For storing the states of the AC remote) | | | V | V	
| restoreACStoreDatas | Restore the current states of the AC remote | | | V | V	

Notes:

<li>1. For the AC remotes, an optional "key option" parameter can be passed to switch the remote controller to the specified key state (such as switch the mode to Cool mode directly.) If the "key option" is obmitted, the key state will switch to next available state cycliclly.</li>
<li>2. The display option of the AC remote could be one of the following values

| Value | Descroption
| -------------------------- | ------------------------------------------
| BIRGuiDisplayType_NO | This AC remote does not have a display panel
| BIRGuiDisplayType_YES | This AC remote has a normal display panel <br>(The panel is off when power on; is off when power off)
| BIRGuiDisplayType_ALWAYS | The panel is always on regardless of the power state (on or off)

Most display panel type of AC remotes is BURGuiDisplayType_Yes.
</li>

# Smart Picker
Most used way to pick up a remote controller from the database containing massive remotes is the so-called smart picker. The user aim the remote controller to the appliance, press a test key to see if the appliance reactives to the key, and repeat this procedure until a proper remote is selected. 

The SDK provides some APIs for helping the Apps to integrate the above steps for TV-type remotes. Note for the AC-type remotes, you need to download the list of all AC remotes of the specified brand and show to the user one by one to test if the correctly remote is selected.

To create a smart picker instance, call ```IRKit.createSmartPicker``` and get the returned instance of BIRTVPicker in the callback function. After that, you can manipulate the picker by the provided functions of the BIRTVPicker interface.

| API | Description | Remark
| ------------- | --------------------------------------- | -----------------------------------
| begin	| Start a new picking process | The key ID for testing is returned
| getNextKey | Get the next key ID for testing | The key ID for testing is returned
| transmitIR | Transmit the IR data of the current testing key |
| keyResult	| Pass the user feedback to the picker | returned <li> BIR_PNext(More test is needed)</li><li>BIR_PFind (Test completed)</li><li>BIR_PFail(No matched remote)</li>
| getPickerResult | Get the matched remotes once the test is completed | Return the matched remote ID(s)

![Smart Picker Flow](../_docs/Smart-picker-flow.png?raw=true "Smart Picker Flowchart")

# IR Learning (Since v.20161107)
## Introduction of IR Learning
Two operation modes are supported:
<li>Learn and Store: Act as a IR signal recorder/player. The IR signal is learned (recorded) and can be re-transmit (replay). The App or the host CPU is responsible for storing the learned IR data.</li>
<li>Learn and Match: The learned IR data is analyzed by the SDK for extracting the characteristics, these characteristics are then sent to the cloud for matching the existing remote controller(s).</li>

The "Learn and Store" is for traditional IR learning application. For the remote controller not registered to the cloud database, learning provides a way to copy the IR signal for controlling the target appliance.

For the "Learn and Match", it provides an alternative way to picking out the correct remote controller from the database. User can press only several keys of the remote controller and the cloud can find the same or similar remote controller for him.

### Learn and Store
The learned IR data (for a specific key on the remote) can be passed back to the App in a compressed form. The App or host application can save the data in its own storage with the specific key name or key ID. To replay the IR signal, simply read the data back from storage and send to the SDK for transmission.

![Fig.1](../_docs/learning_diagrams_1.png)

### Learn and Match
Learn and Match allow the learned IR data to be analyzed and sent to the cloud for matching the existing remote controllers in database. This is sutable for 
<li>Providing a easy way to find the exact the same or similar remote controller(s) by only a few key presses on the remote controller.</li>
<li>Download the similar remote controller to reduce the number of amount of keys to be learned. User need to learn the keys not exist or not matched with their target appliance.</li>

![Fig.2](../_docs/learning_diagrams_2.png)

## Learning APIs
The learning APIs are separated into two parts. The upper APIs are for App to issue commands to switch the IR Blaster into learning mode, and receive the learned IR data from the callbacks. The lower APIs are for passing the data comes from the IR blaster back to the SDK for processing. 

![Fig.3](../_docs/learning_diagrams_3.png)

### Lower APIs
SDK provides a BIRIRBlaster interface for briding the in/out data between the SDK and the IR Blaster hardware. The App should have a instance which extends the BIRIRBlaster interface, and passing this instance to the SDK via IRKit.setIRHW(). Once this is done, all communication data for the IR Blaster hardware will be passed through the instance the App provided. 

Note: How to handle the communication data in/out to/from the IR Blaster is depending on the product specific communication channel therefore is not covered in this document.

Here's an example:

```java
public class MyNetworkIRDevice implements BIRIRBlaster {
    private BIRReceiveDataCallback mReceiveDataCallback = null;

    @Override
    public int sendData(byte[] irData) {
        // send data to IR Blaster
        sendToNetwrok(irData)
        return IRKit.BIROK;
    }

    @Override
    public int isConnection() {
        // check IR Blaster connection status
        if (deviceIsConnected()) {
            return IRKit.BIROK;
        } else {
            return IRKit.BIRNotConnectToNetWork;
        }
    }

    @Override
    public void setReceiveDataCallback(BIRReceiveDataCallback birReceiveDataCallback) {
        // keep this callback.
        // all the traffic come from the IR Blaster should be passed back to the SDK via this callback.
        mReceiveDataCallback = birReceiveDataCallback;
    }

    public void onNetworkDataArrived(byte[] irLearningData) {
        // Once the data from IR Blaster is received, pass it back to SDK (all data should pass back to SDK)
        if (null != mReceiveDataCallback) {
            mReceiveDataCallback.onDataReceived(irLearningData);
        }
    }
}
```
## Upper APIs
The App should first create a BIRReader instance for manipulating the learning functions.

Here's an example for creating a BIRReader instance:

```java
// initialize SDK
IRKit.setup(API_KEY, getApplicationContext());

// create and get the BIRReader instance from the callback
IRKit.createIRReader(forceReload, new BIRReaderCallback() {
    @Override
    public void onReaderCreated(BIRReader birReader) {
        // get the BIRReader instance
        mMyIrReader = birReader;
    }
    @Override
    public void onReaderCreateFailed() {
        // error handling
    }
}
```

Once the BIRReader instance is created, use the following BIRReader APIs to do the learning manipulations.

| API | Description
| ---------------- | ---------------------------------
| startLearningAndGetData() | Start a learning session, and get the result from the callback
| startLearningAndSearchCloud() | Start a learning session, and get the matched remote IDs from the callback
| stopLearning() | Abort the learning (Note: Each learning session will be ended automatically either a valid IR signal is learned or 15 seconds time-out time is reached.)
| sendLearningData() | The learning data got from startLearningAndGetData() can be passed into this function for re-transmission.
| reset() | Reset the startLearningAndSearchCloud() session.(startLearningAndSearchCloud will cache the previously passed-in learning data. If reset() is not called, invoke startLearningAndSearchCloud() twice with different learning data will result in passing two learning data to the cloud to match the remote(s) conaining both IR data.)

Example:

```java
mIrReader.startLearningAndGetData(
    BIRReader.PREFER_REMOTE_TYPE.Auto, 
    new BIRReader.BIRReaderFormatMatchCallback() {
    @Override
    public void onFormatMatchSucceeded(final BIRReader.ReaderMatchResult readerMatchResult) {
        // Format match result
        thisActivity.runOnUiThread(new Runnable() {
            @Override
            public void run() {
                if (readerMatchResult.isAc()) {
                    // if the matched remote is AC-type
                } else {
                    // if the matched remote is TV-type
                }
            }
        });
    }

    @Override
    public void onFormatMatchFailed(BIRReader.FormatParsingErrorCode errorCode) {
        thisActivity.runOnUiThread(new Runnable() {
            @Override
            public void run() {
                // doing GUI error handling
            }
        });
    }

    @Override
    public void onLearningDataReceived(final byte[] learningDataBytes) {

        // Get learned IR data
        mLearnedDataForSending = learningDataBytes;
        thisActivity.runOnUiThread(new Runnable() {
            @Override
            public void run() {
                // GUI handling
            }
        });
    }

    @Override
    public void onLearningDataFailed(BIRReader.LearningErrorCode errorCode) {
        thisActivity.runOnUiThread(new Runnable() {
            @Override
            public void run() {
                // doing GUI error handling
            }
        });
    }
});

// Transmit Learned Data example:

mSendButton.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {

        if (mLearnedDataForSending != null) {
            if (null != mIrReader) {
                mIrReader.sendLearningData(mLearnedDataForSending);
            }
        }
    }
});


// Learn and Recognize example:

mIrReader.startLearningAndSearchCloud(
    false, preferRemoteType, new BIRReader.BIRReaderRemoteMatchCallback() {
    @Override
    public void onRemoteMatchSucceeded(final List<BIRReader.RemoteMatchResult> list) {
        // get the matched remote controller list
        thisActivity.runOnUiThread(new Runnable() {
            @Override
            public void run() {
                showMatchResult(list); // showing match results 
            }
        });
    }

    @Override
    public void onRemoteMatchFailed(BIRReader.CloudMatchErrorCode errorCode) {
        thisActivity.runOnUiThread(new Runnable() {
            @Override
            public void run() {
                // doing GUI error handling
            }
        });
    }

    @Override
    public void onFormatMatchSucceeded(final List<BIRReader.ReaderMatchResult> list) {
        // Format match results
        thisActivity.runOnUiThread(new Runnable() {
            @Override
            public void run() {
                // GUI handling
            }
        });
    }

    @Override
    public void onFormatMatchFailed(final BIRReader.FormatParsingErrorCode errorCode) {
        thisActivity.runOnUiThread(new Runnable() {
            @Override
            public void run() {
                // doing GUI error handling
            }
        });
    }
});
```






