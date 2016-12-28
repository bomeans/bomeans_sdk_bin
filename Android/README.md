#Introduction

This SDK is for building Android universal remote contoller Apps.

This SDK requires Android 4.0 (API Level 19) or later version, the supported CPU architectures are arm, x86, or mips.

Two major packages in this SDK are<br>
<li>com.bomeans.IRKit: The main package providing data downloading and IR data interpretion.</li>
<li>com.bomeans.wifi2ir: Optional package which handling wifi conneciton to Bomeans WiFi IR Blaster using specific Bomeans protocol. (Users who handles the data communication on his own can safely ignore this package.)

#How to Use
##Adding SDK Files
If you are using Eclipse + ADT:<br>
<li>Copy the sub-folders containing the .so files to the \libs folder in your project.</li>
<li>Copy the .jar files to the \libs folder in your project.</li>
<p>

If you are using Android Studio:<br>
<li>Copy the sub-folders containing the .so files to the \src\main\jniLibs folder in your project.</li>
<li>Copy the .jar files to the \libs folder in your project.</li>
Note: You also need to have the following settings in your build.gradle(Module:your_app):<br>
```
dependencies {
    ...
    compile fileTree(dir: 'libs', include: ['*.jar'])
    ...
}
```
##Adding App Permissions
The following permissions need to be added to your AndroidMenifest.xml:
```
android.permission.INTERNET
android.permission.ACCESS_WIFI_STATE
```
#Initial Setup
##Getting the API Key
You need to apply a valid API Key for the SDK to run normally.<br>
Apply the key here: http://www.bomeans.com/Mainpage/Apply/apikey

##Setting your SDK
```
IRKit.setup(apiKey, applicationContext)
```
where apiKey is the API Key issued by Bomeans Design. applicationContext is the application context of your app.

##Switch your target server if your target users are in China
If your target users locate in China, you can optionally switch the connected cloud server the one located in China for better performance.<br>
To switch to China server,
```
IRKit.setUseChineseServer(true)
```
Note: If the above function is not call, a default server located outside China will be used.

#Getting the Basic Information/Lists
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

#Creating the Remote Controller Instance

Two types of remote controllers can be created:

<li>Universal Remote Controller - Univeral remote Controller is a pseudo remote controller which contains several most used remote controllers. When a button of the universal remote is pressed, the IR signals corresponding the underlying remote controllers will be send in a batch.</li>
<li>Single Remote Controller - Single remote controller in the database.</li>

| API | Description | Callback | Remark
|------------------- | ---------------------------------------------------- | ----------- | ------------------------------
| IRKit.createRemote | Create single remote (with specific remote id), or<br>create simple universal remote (with null as remote id) | BIRRemote | Both TV-type and AC-type remotes are supported.<br>The 'simple' universal remote indicates only a few most used buttons are conatined in the returned universal remote controller.
| IRKit.createBigCombineRemote | Create complete universal remote | BIRRemote | Only TV-type remotes are supported.<br>The 'complete' universal remote indicates the the remote contains full keys of the underlying remote controllers.

#Remote Controller Manipulation

Once the Remote Controller instance (BIRRemote) is created, you can manipulate the remote by using the following APIs:

| API | Description | TV-type<br>Remote | TV-type<br>Universal Remote | AC-type<br>Remote | AC-type<br>Universal Remote
| ------------- | ---------------------------------------------------------------| ----- | ----- | ----- | -----
| getAllKeys | Get all key(button) of the remote | V | V | V | V
| getModuleName	| Get the appliance model names for this remote | V	| | V |
| getBrandName | Get the brand name of this remote | V | V | V | V
| transmitIR | Send the IR data (Note1) | V | V | V | V
| beginTransmitIR | Start a IR transmission<br>(Call endTrasmitIR to stop transmission) | V			
| endTransmitIR | Stop IR transmission | V			
| getGUIFeature | Get the display option for AC remote (Note2) | | | V |	
| getActiveKeys | Get the currently active key(button) of the AC remote | | | V | V
| getKeyOption | Get the currently active options of the specific key(button) of a AC remote  | | | V | V	
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

#Smart Picker
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

#IR Learning





