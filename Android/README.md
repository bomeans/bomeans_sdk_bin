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
| webGetTypeList | Get type list | TypeItem
| webGetBrandList	| Get brand list | BrandItem
| webGetTopBrandList | Get popular brand list | BrandItem (Note1)
| webGetModelList	| Get remote ID list |	ModelItem (Note2)
| webVSearch	| Get remote ID list by keyword |	VoiceSearchResultItem (Note3)

Notes

<li>1: Popular brand list is the ranked list according to the popularity.</li>
<li>2: Remote ID is the internal unique id for each remote controller in the database. This ID is not necessary the model name of the remote controller itself or the model name of the appliance.</li>
<li>3: The keywords for searching the remote controllers are usually the one containing the type, brand, and/or the button name. For example, "Turn on the Panasonic TV" is a valid search string that conatins the type (TV), brand name (Panasonic), and the optional button name (Turn on as power button.)</li>



