[NOTE] Bomeans IR SDK for Linux is still in alpha phase.

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

*

##### include

* `Web.h`

##### input

*



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

*

##### include

* `Web.h`

##### input

*

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

*

##### include

* `Web.h`

##### input

*

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

*

##### include

* `Web.h`

##### input

*

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

*

##### include

* `IRKit.h`

##### input

*

##### remark

* You must manually delete the created Remote instance when not in use to avoid memory leak.


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
| TV-like | TV-Like Universal | AC | AC Universal |
| :---: | :---: | :---: | :---: |
| V | V | V | V |

### transmitIR
```cpp
int transmitIR(
	const std::string &keyID, 
	const std::string &option)
```

### beginTransmitIR
```cpp
int beginTransmitIR(
	const std::string keyId)
```

### endTransmitIR
```cpp
void endTransmitIR()
```

### getModuleName
```cpp
std::string getModuleName()
```

### getBrandName
```cpp
std::string getBrandName()
```

### setRepeatCount
```cpp
void setRepeatCount(int count)
```

### getRepeatCount
```cpp
int getRepeatCount()
```

### getActiveKeys
```cpp
std::vector<std::string> getActiveKeys()
```

### getKeyOption
```cpp
std::pair<KeyOption, bool>  getKeyOption(
	const std::string keyID)
```

### getGuiFeature
```cpp
std::pair<GUIFeature, bool> getGuiFeature()
```

### getTimerKeys
```cpp
std::vector<std::string>  getTimerKeys()
```

### setOffTime
```cpp
void setOffTime(
	int hour, 
	int minute, 
	int sec)
```

### setOnTime
```cpp
void setOnTime(
	int hour, 
	int minute, 
	int sec)
```

### getACStoreDatas
```cpp
std::vector<StoreData> getACStoreDatas()
```

### restoreACStoreDatas
```cpp
bool restoreACStoreDatas(
	const std::vector<StoreData> &storeDatas)
```


## IR Learning

### IRKit::createIRReader
```cpp
IRReader* createIRReader(
	bool newData)
```

##### description

*

##### include

* `IRKit.h`

##### input

*

##### remark

* You must manually delete the created IRReader instance when not in use to avoid memory leak.


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
| Auto	| The best matched format is decided by the parsing core. |
| AC | The best matched format is the AC format if any. If no AC format is matched, TV format will be selected. |
| TV | The best matched format is the TV format if any. If no TV format is matched, AC format will be selected. |

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
* `preferRemoteType`: AC, TV, or Auto. This affects the decision of the "best match" returned.
* `callback`: ReaderRemoteMatchCallback callback to receive the matching/searching results.

##### output

* n/a

##### remark

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

###
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

### onFormatMatchFailed
```cpp
void onFormatMatchFailed(
	FormatParsingErrorCode::V errorCode)
```

### onLearningDataReceived
```cpp
void onLearningDataReceived( 
	const std::vector<uint8_t> &learningData)
```

### onLearningDataFailed
```cpp
void onLearningDataFailed(
	LearningErrorCode::V errorCode)
```



## ReaderRemoteMatchCallback (Class)

##### include
* IRReader.h

### onRemoteMatchSucceeded
```cpp
void onRemoteMatchSucceeded(
	const ArrayList<Web::RemoteUID> &remoteMatchResultList)
```


### onRemoteMatchFailed
```cpp
void onRemoteMatchFailed(
	CloudMatchErrorCode::V errorCode)
```

### onFormatMatchSucceeded
```cpp
void onFormatMatchSucceeded(
	const ArrayList<ReaderMatchResult>& formatMatchResultList)
```

### onFormatMatchFailed
```cpp
void onFormatMatchFailed(
	FormatParsingErrorCode::V errorCode)
```















					 
					 


