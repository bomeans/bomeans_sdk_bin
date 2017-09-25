[NOTE] 
* Linux IR SDK is still in alpha phase for tesing.
* SmartPicker is not included in the Linux SDK.
* libaray dependency: curl
* library files build environment: Ubuntu 16.04.03 LTS / gcc 5.4.0

## Contents
### Initialization
* [`IRKit::setup`](#irkitsetup)
* [`IRKit::setUseChineseServer`](#irkitsetusechineseserver)
* [`IRKit::setIRHW`](#irkitsetirhw)
#### BIRIRBlaster (Class)
* [`sendData`](#senddata)
* [`setReceiveDataCallback`](#setreceivedatacallback)
* [`isConnection`](#isconnection)
### Basic Information
* [`Web::getTypeList`](#webgettypelist)
* [`Web::getBrandList`](#webgetbrandlist)
* [`Web::getTopBrandList`](#webgettopbrandlist)
* [`Web::getRemoteModelList`](#webgetremotemodellist)
* [`Web::getKeyName`](#webgetkeyname)
### Create Remote
* [`IRKit::createRemote`](#irkitcreateremote)
#### Remote (Class)
* [`getAllKeys`](#getallkeys)
* [`transmitIR`](#transmitir)
* [`setKeyOption`](#setkeyoption) 
* [`beginTransmitIR`](#begintransmitir)
* [`endTransmitIR`](#endtransmitir)
* [`getModuleName`](#getmodulename)
* [`getBrandName`](#getbrandname)
* [`setRepeatCount`](#setrepeatcount)
* [`getRepeatCount`](#getrepeatcount)
* [`getActiveKeys`](#getactivekeys)
* [`getKeyOption`](#getkeyoption)
* [`getGuiFeature`](#getguifeature)
* [`getTimerKeys`](#gettimerkeys)
* [`setOffTime`](#setofftime)
* [`setOnTime`](#setontime)
* [`getACStoreDatas`](#getacstoredatas)
* [`restoreACStoreDatas`](#restoreacstoredatas)
### IR Learning
* [`startLearningAndGetData`](#irkitcreateirreader)
* [`startLearningAndSearchCloud`](#startlearningandsearchcloud)
* [`reset`](#reset)
* [`stopLearning`](#stoplearning)
* [`sendLearningData`](#sendlearningdata)
#### ReaderFormatMatchCallback (Class)
* [`onFormatMatchSucceeded`](#onformatmatchsucceeded)
* [`onFormatMatchFailed`](#onformatmatchfailed)
* [`onLearningDataReceived`](#onlearningdatareceived)
* [`onLearningDataFailed`](#onlearningdatafailed)
#### ReaderRemoteMatchCallback (Class)
* [`onRemoteMatchSucceeded`](#onremotematchsucceeded)
* [`onRemoteMatchFailed`](#onremotematchfailed)
* [`onFormatMatchSucceeded`](#onformatmatchsucceeded)
* [`onFormatMatchFailed`](#onformatmatchfailed)


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

## BIRIRBlaster (Class)

This interface acts as for bridging the data sending to or coming from the IR Blaster hardware.

For the users that have their own hardware implementations, a class derived from `BIRIRBlaster` should be provided to the SDK.

The main functions of this interface are:

* Receiving IR data or commands sent from the SDK core engine that need to be pass through to the IR Blaster hardware interface.
* Passing the received data from the IR Blaster hardware to the SDK core engine.
Once the class instance is created, you must call [`IRKit::setup`](#irkitsetup) or [`IRKit::setIRHW`](#irkitsetuphw) to assign it to the SDK.

### sendData
```cpp
int sendData(
	const std::vector<uint8_t>& irUartData)
```

#### description
* invoked whenever the SDK has data to be transmit to the IR Blaster hardware (the MCU). It is the implementor's responsibility to relay these data bytes to the UART port of the IR Blaster.

#### input
* `irUartData`: data to be sent to the IR Blaster hardware.

#### output
* `BIRError.BIROK` if succeeded, or the error code if failed.

### setReceiveDataCallback
```cpp
void setReceiveDataCallback(
	BIRReceiveDataCallback* pCallback)
```
#### description
* This is used for sending back data come from the IR Blaster to the SDK. It's the implementor's responsibility to keep this passing callback, and call `BIRReceiveDataCallback::onDataReceived(const std::vector<uint8_t> &receivedDataBytes)` to pass the incoming data from IR Blaster to the SDK.

#### input

* callback: the callback passed from the SDK for the user to send back the data come from the IR Blaster hardware.

#### output
* n/a

#### isConnection
```cpp
int isConnection()
```

#### description
* to report the IR Blaster hardware connection status to the SDK.

#### input
* n/a

#### output
* `BIRError.BIROK` if the connection is established, or the error code if failed.

##### Sample Implementation
```cpp
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#include <BomeansIRSDK.h>
#include <deque>

std::string APIKEY="my-api-key";
BOMEANS_LIB_NAME::BIRReceiveDataCallback* mpMyCallback;
class MYIRBlaster : public BOMEANS_LIB_NAME::BIRIRBlaster
{ 
public:
	MYIRBlaster() { }
	~MYIRBlaster() { }

	// When there are data to be sent to the underlying hardware (Bomeans MCU), SDK will
	// call this function with the data as the parameter.
	virtual int sendData(const BomArray<uint8_t>& irUartData)
	{
		const uint8_t* pIrUartData = &irUartData[0];
		int irUartDataLen = irUartData.size();

		// those data bytes should be forwarded to the hardware via user-defined protocol
		printf(">>>Sent %d bytes: ", irUartDataLen);
		for (int i = 0; i < irUartDataLen; i++)
		{
			printf("%02X,", pIrUartData[i]);
		}
		printf("\n");
		return  BIRError::BIRNoError;
	}

	// return true if the underlying hw is connected, or false otherwise.
	virtual int isConnection()
	{
		return true;	// assume the hardware is always connected.
	}

	// This is where the SDK pass the callback function for the underyling hw to send back 
	// data to the SDK.
	// Keep the pasisng callback function, and whenere there are data to be sent back to SDK, 
	// call the onDataReceived() of the passing callback.
	virtual void setReceiveDataCallback(BOMEANS_LIB_NAME::BIRReceiveDataCallback* pCallback)
	{
		mpMyCallback = pCallback;
	}

	// test code, simulating the received data from Bomeans MCU(e.g. IR Learning data), 
	// and pass these data back to the SDK for further handling
	void sendDummy()
	{
		if (NULL != mpMyCallback)
		{
			BomArray<uint8_t> v = {
				0xff, 0x61, 0x00, 0x54, 0x00, 0x92, 0x09, 0x86, 
				0x2c, 0x06, 0x08, 0x20, 0xc0, 0x00, 0x90, 0x03, 
				0xd4, 0x08, 0x0c, 0x03, 0x00, 0x0b, 0x1c, 0x05, 
				0xa0, 0x05, 0x48, 0x07, 0xcc, 0x35, 0x18, 0x04, 
				0x5d, 0xa7, 0x6b, 0x99, 0xcc, 0x07, 0x00, 0x10, 
				0x20, 0x30, 0x40, 0x50, 0x60, 0x70, 0x80, 0x90, 
				0xa0, 0xb0, 0x48, 0x00, 0x10, 0x32, 0x44, 0x65, 
				0x07, 0x26, 0x82, 0x25, 0x92, 0x10, 0x32, 0x44, 
				0x65, 0x07, 0x62, 0x82, 0x25, 0x92, 0x10, 0x32, 
				0x44, 0x65, 0x07, 0x62, 0x82, 0x25, 0xa2, 0x10, 
				0x32, 0x44, 0x65, 0x07, 0xb3, 0x82, 0x25, 0x92, 
				0xe6, 0xf0
			};
			mpMyCallback->onDataReceived(v);
		}
	}
};

int main()
{
	USING_BOMEANS

	// The SDK required curl library, for accessing the web APIs
	curl_global_init(CURL_GLOBAL_ALL);
	
	// setup API Key
	IRKit::setup(APIKEY, false, nullptr);

	// setup the underlying hardware
	IRKit::setIRHW(&getMyIRBlaster());
	
	// get types
	BomArray<Web::TypeItem> typeItems;
	bool r = Web::getTypeList("tw", false, typeItems, nullptr);
	printf("\nTypes:\n");
	for (Web::TypeItem typeItem : typeItems)
	{
		printf("%s (%s, %s)\n", 
			typeItem.Id.c_str(), 
			typeItem.LocalizedName.c_str(), 
			typeItem.EnglishName.c_str());
	}	

	// get brands	
	BomArray<Web::BrandItem> brandItems;
	Web::getTopBrandList(typeItems[0].Id, 0, 10, "tw", false, brandItems, nullptr);
	printf("\nBrands:\n");
	for (Web::BrandItem brandItem : brandItems)
	{
		printf("%s (%s, %s)\n", 
			brandItem.Id.c_str(), 
			brandItem.LocalizedName.c_str(), 
			brandItem.EnglishName.c_str());
	}

	// get models
	BomArray<Web::ModelItem> modelItems;
	Web::getRemoteModelList(typeItems[0].Id, brandItems[1].Id, false, modelItems, nullptr);
	printf("\nModels (Remote IDs):\n");
	for (Web::ModelItem modelItem : modelItems)
	{
		printf("%s (%s)\n", modelItem.Id.c_str(), modelItem.MachineModel.c_str());
	}

	// create remote - AC
	brandItems.clear();
	Web::getTopBrandList("2", 0, 10, "tw", false, brandItems, nullptr);
	Web::getRemoteModelList(
		"2", brandItems[1].Id, 
		false, 
		modelItems, 
		nullptr);
	auto remote = IRKit::createRemote(
		"2", 
		brandItems[1].Id, 
		modelItems[0].Id, 
		false, 
		nullptr);
	printf("Create Remote: type: %s, brand: %s, model: %s\n", 
		"2", brandItems[1].Id.c_str(), modelItems[0].Id.c_str());

	printf("Supported Keys:\n");
	auto keys = remote->getAllKeys();
	for (auto key : keys)
	{
		printf("%s\n", key.c_str());
	}

	// send IR
	for (auto key : keys)
	{
		if (strcmp(key.c_str(), "IR_ACKEY_POWER") == 0)
		{
			printf("Send POWER Key\n");
			remote->transmitIR("IR_ACKEY_POWER", "IR_ACOPT_POWER_ON");
		}
	}

	for (auto key : keys)
	{
		if (strcmp(key.c_str(), "IR_ACKEY_MODE") == 0)
		{
			printf("Send MODE Key\n");
			remote->transmitIR("IR_ACKEY_MODE", "");
		}
	}

	// need manually delete the created remote
	delete remote;
	
	curl_global_cleanup();
}
```

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

* `language`: language(location) code such as "cn", "tw", "en", etc.
* `getNew`: true to pull the data from cloud, false to read the previous downloaded data (cached data) first if any.
* `items`: the retrieved `TypeItem` list.
* `userIf`: for user interaction, can be set to `nullptr`.

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

* `typeId`: type id which can be retrieved from ```getTypeList```
* `start`: start index of the brand list
* `number`: number of brand entries to be returned
* `language`: language code such as "cn", "tw", etc.
* `brandName`: brand name keyword to filter the returned list. Can be null to get the whole list. 
* `getNew`: true to pull the data from cloud, false to read the previous downloaded data (cached data) first if any.
* `items`: the retrieved `BrandItem` list.
* `userIf`: for user interaction, can be set to `nullptr`.

##### output
* true if succeeded, or false if failed.

##### remark
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

* `typeId`: type id which can be retrieved from `getTypeList`
* `start`: start index of the brand list
* `number`: number of brand entries to be returned
* `language`: language code such as "cn", "tw", etc.
* `getNew`: true to pull the data from cloud, false to read the previous downloaded data (cached data) first if any.
* `items`: the retrieved `BrandItem` list.
* `userIf`: for user interaction, can be set to `nullptr`.

##### output

* true if succeeded, or false if failed.

##### remark
* See `getBrandList` for the `BrandItem` information.

### Web::getRemoteModelList
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

* `typeId`: type id which can be retrieved from `getTypeList`
* `brandId`: brand id which can be retrieved from `getBrandList` or `getTopBrandList`
* `getNew`: true to pull the data from cloud, false to read the previous downloaded data (cached data) first if any.
* `items`: the reteieved `Web::ModelItem` list.
* `userIf`: for user interaction, can be set to `nullptr`.

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


### Web::getKeyName
```cpp
bool getKeyName(
	const std::string &typeId, 
	const std::string &language, 
	bool getNew, 
	BomArray<KeyName> &items, 
	IAPIProgress *userIf);
```

##### description
* get the key(button) list of the specified type

##### include
* `Web.h`

##### input
* `typeId`: type id which can be retrieved from webGetTypeList
* `language`: language code such as "tw", "cn", "en", etc.
* `getNew`: true to pull the data from cloud, false to read the previous downloaded data (cached data) first if any.
* `items`: the retrieved `Web::KeyName` list.
* `userIf`: for user interaction, can be set to `nullptr`.

##### output
* true if succeeded, false otherwise

##### remark
* `KeyName` has following has following members:

| member | type | description |
| --- | --- | --- |
| `TypeId` | `std::string` | type id |
| `Id` | `std::string` | key id for identofying each key(button) |
| `LocalizedName` | `std::string` | localized name of the key |


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

* `type`: type id which can be retrieved from `getTypeList`
* `brand`: brand id which can be retrieved from `getBrandList` or `getTopBrandList`
* `model`: model id, or the remote id, which can be retrieved from `getModelList`. If null is passing, a universal remote controller will be created. (see note)
* `getNew`: true to pull the data from cloud, false to read the previous downloaded data (cached data) first if any.
* `userIf`: callback thats will be invoked when download is completed. The `Remote` will be returned if succeeded.

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

## setKeyOption
```cpp
int setKeyOption(
	const std::string &keyID, 
	const std::string &optionId)
```

##### description
Set the remote internal state for AC type remote

##### input
+ `keyID`: key id of the target key(button) of the remote
+ `optionId`: key option id of the target key state(option)

##### output
+ 0(`BIRError::BIRNoError` / `BIRError::BIROK`) if succeeded, or error code if failed

##### availability
| <sub>TV-like</sub> | <sub>TV-Like Universal</sub> | <sub>AC</sub> | <sub>AC Universal</sub>
|:---:|:---:|:---:|:---:
| | |V|V


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
* `keyID`: the target key id
 
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
















					 
					 


