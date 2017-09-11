[NOTE] 
* Linux IR SDK is still in alpha phase for tesing.
* SmartPicker is not included in the Linux SDK.

# Linux SDK Documentation

##### namespace : BOMEANS_LIB_NAME

## Initialization

### IRKit::setup
```cpp
void setup(
	const std::string &apiKey, 
	bool useChinaServer, 
	BIRIRBlaster *myIrBlaster)
```
	
##### description

* Initialize the SDK.

##### include

* `IRKit.h`

##### input

* `apiKey`:  api key applied from Bomeans Design
* `useChinaServer`: switch to China Server
* `myIrBlaster`: user-defined class instance which inherits from BIRIRBlaster class

##### output
* n/a

##### remark
* Must be called first for the SDK to work properly


### IRKit::setUseChineseServer
```cpp
void setUseChineseServer(
	bool cn)
```

##### descripton

* Switch between China/International Server.

##### include

* `IRKit.h`

##### input

* `cn`: true to switch to China Server, false to International Server (default) 

##### output
* n/a

##### remark
* If not called, the default server is the international one.


### IRKit::setIRHW 
```cpp
void setIRHW(
	BIRIRBlaster *irBlaster)
```

##### description

* Assign a user-defined class instance which processing all the in/out traffic between the SDK and the underlying IR blaster hardware.

##### include

* `IRKit.h`

##### input

* `irBlaster`: user-defined class instance which inherits from BIRIRBlaster class

##### output
* n/a

##### remark
* If not called, the default IR data will be routed to the WiFi interface in the form of Bomeans-defined package and protocol. All users with customized hardware should provide this instance to the SDK.


## Basic Information

### Web::getTypeList
```cpp
bool getTypeList(
	const std::string &language, 
	bool getNew, 
	std::vector<TypeItem>& items, 
	IAPIProgress *userIf)
```

##### description

* get the supported appliance category (type) list.

##### include

* `Web.h`

##### input

* ```language```: language(location) code such as "cn", "tw", "en", etc.
* ```getNew```: true to pull the data from cloud, false to read the previous downloaded data (cached data) first if any.
* ```userIf```: callback thats will be invoked when download is completed. The ```TypeItem[]``` will be returned if succeeded.

##### output

* true if succeeded, or false if failed.

##### remark
* `TypeItem` has the following public members:

| member | type | description
| --- | --- | ---
| `Id` | `std::string` | unique id for identifying this type
| `LocalizedName` | `std::string` | localized name of this type
| `EnglisgName` | `std::string` | English name of this type

##### example


### Web::getBrandList
```cpp
bool getBrandList(
	const std::string &typeId, 
	int start, 
	int number, 
	const std::string &language, 
	const std::string &brandName, 
	bool getNew, 
	std::vector<BrandItem>& items, 
	IAPIProgress *userIf)
```

##### description

* get the supported brand list of the specified type.

##### include

* `Web.h`

##### input

* ```typeId```: type id which can be retrieved from ```webGetTypeList```
* ```start```: start index of the brand list
* ```number```: number of brand entries to be returned
* ```language```: language code such as "cn", "tw", etc.
* ```brandName```: brand name keyword to filter the returned list. Can be null to get the whole list. 
* ```getNew```: true to pull the data from cloud, false to read the previous downloaded data (cached data) first if any.
* ```userIf```: callback thats will be invoked when download is completed. The ```BrandItem[]``` will be returned if succeeded.

##### output
* true if succeeded, or false if failed.

##### remark
* The returned Boolean can be used to cancel the dowloading.
* `BrandItem` has the following public members:

| member | type | description
| --- | --- | ---
| `Id` | `std::string` | unique id for identifying this brand
| `LocalizednName` | `std::string` | localized name of this brand
| `EnglishName` | `std::string` | English name of this brand

##### example


### Web::getTopBrandList
```cpp
bool getTopBrandList(
	const std::string &typeId, 
	int start, 
	int number, 
	const std::string &language, 
	bool getNew, 
	std::vector<BrandItem>& items, 
	IAPIProgress *userIf)
```

##### description

* get the brand list of the specified type, sorted by the popularity ranking.

##### include

* `Web.h`

##### input

* `typeId`: type id which can be retrieved from `webGetTypeList`
* `start`: start index of the brand list
* `number`: number of brand entries to be returned
* `language`: language code such as "cn", "tw", etc.
* `getNew`: true to pull the data from cloud, false to read the previous downloaded data (cached data) first if any.
* `userIf`: callback thats will be invoked when download is completed. The `BrandItem[]` will be returned if succeeded.

##### output

* true if succeeded, or false if failed.

##### remark
* See `getBrandList` for the `BrandItem` information.

##### example


### getRemoteModelList
```cpp
bool getRemoteModelList(
	const std::string &typeId, 
	const std::string &brandId, 
	bool getNew, 
	std::vector<ModelItem>& items, 
	IAPIProgress *userIf)
```

##### description

* get the remote id list of the specified type and brand

##### include

* `Web.h`

##### input

* `typeId`: type id which can be retrieved from `webGetTypeList`
* `brandId`: brand id which can be retrieved from `webGetBrandList` or `webGetTopBrandList`
* `newData`: true to pull the data from cloud, false to read the previous downloaded data (cached data) first if any.
* `userIf`: callback thats will be invoked when download is completed. The `ModelItem[]` will be returned if succeeded.

##### output

* true if succeeded, or false if failed.

##### remark
* `ModelItem` has the following members:

| member | type | description
| --- | --- | ---
| `Id` | `std::string` | The id for each individual remote entry in the database. This id is not necessary the model name of the remote controller itself nor the target appliamce model name.
| `MachineModel` | `std::string` | target appliance model names, separated by comma (,) <br><br>(Note: the list may be appear to be empty or incompleted. This field is for only reference and not recommended for using as the main function for picking the remote controller.)
| `Country` | `std::string` | country (location) code such as "cn", "tw", "en", etc.
| `ReleaseTime` | `std::string` | date this remote been added to the database.


##### example


## Create Remote

### IRKit::createRemote
```cpp
Remote* createRemote(
	const std::string &type, 
	const std::string &brand, 
	const std::string &model,
	bool getNew, 
	Web::IAPIProgress *userIf)
```

##### description

* create a remote controller.

##### include

* `IRKit.h`

##### input

* `type`: type id which can be retrieved from webGetTypeList
* `brand`: brand id which can be retrieved from `webGetBrandList` or `webGetTopBrandList`
* `model`: model id, or the remote id, which can be retrieved from `webGetModelList`. If null is passing, a universal remote controller will be created. (see note)
* `getNew`: true to pull the data from cloud, false to read the previous downloaded data (cached data) first if any.
* `userIf`: callback thats will be invoked when download is completed. The `BIRRemote` will be returned if succeeded.

##### note

* A universal remote controller is a pseudo remote which is the combination of several popular remotes of a specific brand. To compare with a general single remote controller, several IR signals which corresponding to the underlying remotes are sent back to back when a button of the universal remote is activated.

##### output

* true if succeeded, or false if failed.

##### remark

* You must manually delete the created Remote instance when not in use to avoid memory leak.

##### availability
| <sub>TV-like</sub> | <sub>TV-Like Universal</sub> | <sub>AC</sub> | <sub>AC Universal</sub>
|:---:|:---:|:---:|:---:|
| V | V | V | V (Simplified-keys*) |

<sub>*Simplified-keys means the created universal remote contains only a few most common keys. For AC remote controller, these keys are Power / Mode / Temperature.</sub>

##### example


## Remote (Class)

##### include
* `Remote.h`

### getAllKeys
```cpp
std::vector<std::string> getAllKeys()
```
##### description
* get all supported keys in this remote.

##### input
* n/a

##### output
* string array contains all the key ids.

##### avialablity
| <sub>TV-like</sub> | <sub>TV-Like Universal</sub> | <sub>AC</sub> | <sub>AC Universal</sub>
| :---: | :---: | :---: | :---: |
| V | V | V | V |

### transmitIR
```cpp
int transmitIR(
	const std::string &keyID, 
	const std::string &option)
```

##### description
* send the IR data of the specified key

##### input
* `keyId`: key id of the target key(button) of the remote
* `option`: key option id of the target key state(option). Set to nil for non-AC type of remote.

##### output
* 0(`BIRError::BIRNoError` / `BIRError::BIROK`) if succeeded, or error code if failed. The actual error code retruned depends on the user implementation of `BIRIRBlaster`.

##### remark
* for AC remote, option id set to empty(`""`) will set the option to the next available option of the key. For temperature key (`IR_ACKEY_TEMP`), option can be set to "UP" or "DOWN" to automatically move to the next temperature up or down respectively.

##### availability
| <sub>TV-like</sub> | <sub>TV-Like Universal</sub> | <sub>AC</sub> | <sub>AC Universal</sub>
|:---:|:---:|:---:|:---:
|V|V|V|V

### beginTransmitIR
```cpp
int beginTransmitIR(
	const std::string keyId)
```

##### description
* start sending IR data continuously (must call `endTransmitIR` to stop the data sending)

##### input
* `keyId`: key id of the target key(button) of the remote
 
##### output
* 0(`BIRError::BIRNoError` / `BIRError::BIROK`) if succeeded, or error code if failed

##### availability
| <sub>TV-like</sub> | <sub>TV-Like Universal</sub> | <sub>AC</sub> | <sub>AC Universal</sub>
|:---:|:---:|:---:|:---:
|V| | |


### endTransmitIR
```cpp
void endTransmitIR()
```

##### description
* stop sending IR data.

##### input
* n/a
 
##### output
* n/a

##### availability
| <sub>TV-like</sub> | <sub>TV-Like Universal</sub> | <sub>AC</sub> | <sub>AC Universal</sub>
|:---:|:---:|:---:|:---:
|V| | |


### getModuleName
```cpp
std::string getModuleName()
```
##### description
get the remote id. (though a bit confusing, the module name here is actually the remote id which is used for uniquely identifying the remote entry in the database.)

##### input
* n/a
 
##### output
* remote id

##### availability
| <sub>TV-like</sub> | <sub>TV-Like Universal</sub> | <sub>AC</sub> | <sub>AC Universal</sub>
|:---:|:---:|:---:|:---:
|V| |V|


### getBrandName
```cpp
std::string getBrandName()
```

##### description
* get the brand id for this remote

##### input
* n/a
 
##### output
* brand id

##### availability
| <sub>TV-like</sub> | <sub>TV-Like Universal</sub> | <sub>AC</sub> | <sub>AC Universal</sub>
|:---:|:---:|:---:|:---:
|V|V|V|V


### setRepeatCount
```cpp
void setRepeatCount(int count)
```

##### description
* set the repeat count for the IR data. The repeat count is the number of repeat frames to be sent for one `transmitIR` call. 

##### input
* `count`: number of repeat frames
 
##### output
* n/a

##### availability
| <sub>TV-like</sub> | <sub>TV-Like Universal</sub> | <sub>AC</sub> | <sub>AC Universal</sub>
|:---:|:---:|:---:|:---:
|V| | |


### getRepeatCount
```cpp
int getRepeatCount()
```

##### description
* get the repeat count for the IR data for one `transmitIR` call. 

##### input
* n/a 
 
##### output
* number of repeat frames

##### availability
| <sub>TV-like</sub> | <sub>TV-Like Universal</sub> | <sub>AC</sub> | <sub>AC Universal</sub>
|:---:|:---:|:---:|:---:
|V| | |


### getActiveKeys
```cpp
std::vector<std::string> getActiveKeys()
```

##### description
* get currently activated keys of the AC type remote. Activated keys by definition are the keys that are enabled for user to press. Activated keys are not necessary the same number of the returned from `getAllKeys`.

##### input
* n/a 
 
##### output
* `std::vector<std::string>` containing currently activated keys

##### availability
| <sub>TV-like</sub> | <sub>TV-Like Universal</sub> | <sub>AC</sub> | <sub>AC Universal</sub>
|:---:|:---:|:---:|:---:
| | |V|V


### getKeyOption
```cpp
std::pair<KeyOption, bool>  getKeyOption(
	const std::string keyID)
```

##### description
* get all possible options of the specified key

##### input
* `keyId`: the target key id
 
##### output
* 'std::pair<KeyOption, bool>'. The returned `bool` value indicating if the returned `KeyOption` is valid. If the returned `bool` valus is false, most likely you are giving the wrong keyID, or you are asking the key options for TV-like remote controllers which do not have key options.

##### remark
* `KeyOption` has the following public members

| member | type | description
|---|---|---
| `currentOption` | int | current selected option
| `options` | `std::vector<std::string>` | option id list
| `enable` | `bool` | `true` if this is key currently enabled(activated), `false` otherwise

##### availability
| <sub>TV-like</sub> | <sub>TV-Like Universal</sub> | <sub>AC</sub> | <sub>AC Universal</sub>
|:---:|:---:|:---:|:---:
| | |V|V


### getGuiFeature
```cpp
std::pair<GUIFeature, bool> getGuiFeature()
```
##### description
* get the GUI features of this remote. These features are for helping the GUI implementation for AC type remote.

##### input
* n/a
 
##### output
* 'std::pair<GUIFeature, bool>' describing the GUI features. The returned `bool` value indicating if the returned `GUIFeature` value is valid.

##### remark
* `GUIFeature` has the following public members

| member | type | description
|---|---|---
| `DisplayType ` | `int(enum)` | possible value:<br><ul><li>`GUIFeature::TYPE_NO` - no display panel</li><li>`GUIFeature::TYPE_YES` - has a display panel which is on while powered on</li><li>`GUIFeature::TYPE_ALWAYS` - has a always on display panel regardless of the power on or off</li></ul>
| `RTC ` | `bool` | <ul><li>`true` - has RTC support</li><li>`false` - no RTC support</li></ul>
| `timerMode` | `int` | <ul><li>1 - Only OFF timer is supported</li><li>2 - Support ON and/or OFF timer, can be set only when power is on.</li><li>3 - Support ON and/or OFF timer, can be set regardless of power state.</li><li>4 - Either ON or OFF timer, can be set only when power is on.</li><li>5 - Can set ON timer while powered off; set OFF timer while powered on.</li></ul>
| `timerCountDown` | `bool` | <ul><li>`true` - timer is count down type</li><li>`false` - timer is not count down type</li></ul>
| `timerClock` | `bool` | <ul><li>`true` - timer is clock type</li><li>`false` - timer is not clock type</li></ul>


##### availability
| <sub>TV-like</sub> | <sub>TV-Like Universal</sub> | <sub>AC</sub> | <sub>AC Universal</sub>
|:---:|:---:|:---:|:---:
| | |V|


### getTimerKeys
```cpp
std::vector<std::string>  getTimerKeys()
```

##### description
* get timer key(s) of this remote

##### input
* n/a 
 
##### output
* timer key id list

##### availability
| <sub>TV-like</sub> | <sub>TV-Like Universal</sub> | <sub>AC</sub> | <sub>AC Universal</sub>
|:---:|:---:|:---:|:---:
| | |V|


### setOffTime
```cpp
void setOffTime(
	int hour, 
	int minute, 
	int sec)
```

##### description
* set the off timer

##### input
* `hour`: hour of the off timer
* `minute`: minute of the off timer
* `sec`: second of the off timer
 
##### output
* n/a

##### availability
| <sub>TV-like</sub> | <sub>TV-Like Universal</sub> | <sub>AC</sub> | <sub>AC Universal</sub>
|:---:|:---:|:---:|:---:
| | |V|


### setOnTime
```cpp
void setOnTime(
	int hour, 
	int minute, 
	int sec)
```

##### description
* set the on timer

##### input
* `hour`: hour of the on timer
* `minute`: minute of the on timer
* `sec`: second of the on timer
 
##### output
* n/a

##### availability
| <sub>TV-like</sub> | <sub>TV-Like Universal</sub> | <sub>AC</sub> | <sub>AC Universal</sub>
|:---:|:---:|:---:|:---:
| | |V|


### getACStoreDatas
```cpp
std::vector<StoreData> getACStoreDatas()
```

##### description
* get the internal states of all the keys of the remote. The returned `StoreData` list can be saved in storage, and call `restoreACStoreDatas` to restore the states of the remote. 

##### input
* n/a
 
##### output
* `std::vector<StoreData>` containing all key states of the remote.

##### remark
* `StoreData` has the following public members
| member | type | description
|---|---|---
| `mode` | `int` | The mode this data relates to
| `state` | `std::string` | the state id of this data
| `value` | `int` | the value of this data
*<sub>You do not need to parse the data content. Just save the returned `std::vector<StoreData>` for later use if you are planning to retore the remote controller's state in later time.</sub>

##### availability
| <sub>TV-like</sub> | <sub>TV-Like Universal</sub> | <sub>AC</sub> | <sub>AC Universal</sub>
|:---:|:---:|:---:|:---:
| | |V|V


### restoreACStoreDatas
```cpp
bool restoreACStoreDatas(
	const std::vector<StoreData> &storeDatas)
```

##### description
* restore the remote states by passing the `ACStoreDataItem[]` got from `getACStoreDatas`. 

##### input
* `storeDatas`: previously saved `std::vector<StoreData>` data.
 
##### output
* true if succeeded, false otherwise.

##### availability
| <sub>TV-like</sub> | <sub>TV-Like Universal</sub> | <sub>AC</sub> | <sub>AC Universal</sub>
|:---:|:---:|:---:|:---:
| | |V|V


## IR Learning

### IRKit::createIRReader
```cpp
IRReader* createIRReader(
	bool newData)
```

##### description

* create an IR reader instance which implements '[BIRReader](BIRReader)' interface.

##### include

* `IRKit.h`

##### input

* 'newData': true to pull the data from cloud, false to read the previous downloaded data (cached data) first if any.

##### output
* n/a

##### remark

* You must manually delete the created IRReader instance when not in use to avoid memory leak.

##### example


## IRReader (Class)

##### include

* `IRReader.h`

### startLearningAndGetData
```cpp
void startLearningAndGetData(
	PREFER_REMOTE_TYPE preferRemoteType, 
	ReaderFormatMatchCallback *callback)
```

##### description

* Start a learning session, and get the result from the callback.

##### input

* `preferRemoteType`: preferred type for the IR matching (see remark below for more information). If you just need to get the IR learning data and store it for later use, you may not be interested in the IR format match results so just simply passing in Auto is good enough.
* `callback`: `ReaderFormatMatchCallback` callback that returns the learning results.

##### output

* n/a

##### remark

* PREFER_REMOTE_TYPE has the following values:

| value | description |
| :--- | :--- |
| `Auto` | The best matched format is decided by the parsing core. |
| `AC` | The best matched format is the AC format if any. If no AC format is matched, TV format will be selected. |
| `TV` | The best matched format is the TV format if any. If no TV format is matched, AC format will be selected. |

##### example

### startLearningAndSearchCloud
```cpp
void startLearningAndSearchCloud(
	bool isNewSearch, 
	PREFER_REMOTE_TYPE preferRemoteType, 
	ReaderRemoteMatchCallback *callback)
```

##### description

* Start the learning process, get the matched format and matched remote controller(s) in the passing callback functions.

##### input

* `isNewSearch`: Set to true if you are starting a new learning. Set to false if you are adding the learning to filter the previous matched result.
* `preferRemoteType`: `AC`, `TV`, or `Auto`. This affects the decision of the "best match" returned.
* `callback`: ReaderRemoteMatchCallback callback to receive the matching/searching results.

##### output

* n/a

##### example

### reset
`void reset()`

##### description

* Reset the internal learning states if you have previous called `startLearningAndSearchCloud`

##### input

* n/a

##### output

* n/a

##### remark
* `startLearningAndSearchCloud` supports the so-called accumulated matching for TV-like remotes.<br> 
when the startLearningAndSearchCloud is continuously invoked, the IR learning data previously sent will be kept in the internal memory for filtering the matching remotes. That is, you can have more accurate match result if several IR learning data belong to the same remote controller is passed in startLearningAndSearchCloud.
Calling reset will reset the internal memory to start a new accumulated matching.


### stopLearning
```cpp
int stopLearning()
```

##### description

* Send the stop learning command to IR Blaster. (Return to normal mode).

##### input

* n/a

##### output

* `BIRError.BIROK` if succeeded, other error code if failed.


### sendLearningData
```cpp
int sendLearningData(
	const std::vector<uint8_t> &learningData)
```

##### description

* Send the learned IR data.

##### input

* `learningData`: The IR data is the raw data bytes returned from the callback of `startLearningAndGetData`.

##### output

* `BIRError.BIROK` if succeeded, other error code if failed.


## ReaderFormatMatchCallback (Class)

##### include
* IRReader.h

### onFormatMatchSucceeded
```cpp
void onFormatMatchSucceeded(
	const ReaderMatchResult &formatMatchResult)
```

##### description
* Invoked if the learned IR data matches one of the known IR format.

##### input
* `formatMatchResult`: matched IR format(s)

##### output
* n/a

##### remark
* This function is mostly for debugging purpose.
* `ReaderMatchResult` has the following public members:

| member | type | description
| --- | --- | ---
| `formatId` | `std::tring` | IR format id
| `customCode` | `long` | custom code
| `keyCode` | `long` | key code


### onFormatMatchFailed
```cpp
void onFormatMatchFailed(
	FormatParsingErrorCode::V errorCode)
```

##### description
* Invoked if there is no match for the learned result.

##### input
* `errorCode`: predefined error code that describes the reason why the match failed.

##### output
* n/a

##### remark
* This function is mostly for debugging purpose.


### onLearningDataReceived
```cpp
void onLearningDataReceived( 
	const std::vector<uint8_t> &learningData)
```

##### description
* Invoked when the learning data is received.

##### input
* `learningData`: the learned IR signal data. This data can be stored and re-transmit by calling `sendLearningData`

##### output
* n/a


### onLearningDataFailed
```cpp
void onLearningDataFailed(
	LearningErrorCode::V errorCode)
```

##### description
* Invoked when the learning data is not received or is incorrect.

##### input
* `errorCode`: predefined error code that describes the reason why the match failed.

##### output
* n/a


## ReaderRemoteMatchCallback (Class)

##### include
* IRReader.h

### onRemoteMatchSucceeded
```cpp
void onRemoteMatchSucceeded(
	const ArrayList<Web::RemoteUID> &remoteMatchResultList)
```

##### description
* The matched remote controller(s) of the loaded learning data.

##### input
* `remoteMatchResultList`: `ArrayList<Web::RemoteUID>` which is the list of matched remotes.

##### output
* n/a

##### remark
* the matched remote controller list is the result of accumulated matching data since the last call of reset().
* `RemoteUID` has the following public members:

| member | type | description
| --- | --- | ---
| `TypeID` | `std::string` | type id of the remote
| `BrandID` | `std::string` | brand id of the remote
| `ModelID` | `std::string` | remote id of the remote



### onRemoteMatchFailed
```cpp
void onRemoteMatchFailed(
	CloudMatchErrorCode::V errorCode)
```

##### description
* If failed to find the matched remote in the cloud database.

##### input
* `errorCode`: predefined error code that describes the reason why the match failed.

##### output
* n/a


### onFormatMatchSucceeded
```cpp
void onFormatMatchSucceeded(
	const ArrayList<ReaderMatchResult>& formatMatchResultList)
```

##### description
* Invoked if the learned IR data matches one of the known IR format.

##### input
* `formatMatchResultList`: list of the matched IR formats.

##### output
* n/a

##### remark
* the matched remote controller list is the result of accumulated matching data since the last call of reset().
* `ReaderMatchResult` has the following public members:

| member | type | description
| --- | --- | ---
| `formatId` | `std::string` | IR format id
| `customCode` | `long` | custom code
| `keyCode` | `long` | key code


### onFormatMatchFailed
```cpp
void onFormatMatchFailed(
	FormatParsingErrorCode::V errorCode)
```

##### description
* Invoked if there is no match for the learned result.

##### input
* `errorCode`: predefined error code that describes the reason why the match failed.

##### output
* n/a

##### remark
* This function is mostly for debugging purpose.
















					 
					 


