[NOTE] Bomeans IR SDK for Linux is still in alpha phase.

# Linux SDK Documentation

## Initialization

### IRKit::setup
`void setup(const std::string &apiKey, 
	bool useChinaServer, 
	BIRIRBlaster *myIrBlaster);`
	
description

* Initialize the SDK.

include

* `IRKit.h`

input

* `apiKey`:  api key applied from Bomeans Design
* `useChinaServer`: switch to China Server
* `myIrBlaster`: user-defined class instance which inherits from BIRIRBlaster class

### IRKit::setUseChineseServer
'void setUseChineseServer(bool cn)'

descripton

* Switch between China/International Server.

include

* `IRKit.h`

input

* `cn`: true to switch to China Server, false to International Server (default) 

### IRKit::setIRHW 
`void setIRHW(BIRIRBlaster *irBlaster)`

description

* Assign a user-defined class instance which processing all the in/out traffic between the SDK and the underlying IR blaster hardware.

include

* `IRKit.h`

input

* `irBlaster`: user-defined class instance which inherits from BIRIRBlaster class


### Web::getTypeList
`bool getTypeList(const std::string &language, bool getNew, BomArray<TypeItem>& items, IAPIProgress *userIf)`

description

*

include

* `Web.h`

input

*



### Web::getBrandList
`bool  getBrandList(const std::string &typeId, int start, int number, const std::string &language, const std::string &brandName, bool getNew, BomArray<BrandItem>& items, IAPIProgress *userIf)`


description

*

include

* `Web.h`

input

*

### Web::getTopBrandList
`bool getTopBrandList(const std::string &typeId, int start, int number, const std::string &language, bool getNew, BomArray<BrandItem>& items, IAPIProgress *userIf)`


description

*

include

* `Web.h`

input

*

### getRemoteModelList
`bool getRemoteModelList(const std::string &typeId, const std::string &brandId, bool getNew, BomArray<ModelItem>& items, IAPIProgress *userIf)`


description

*

include

* `Web.h`

input

*

### IRKit::createRemote
`Remote* createRemote(const std::string &type, 
					 const std::string &brand, 
					 const std::string &model,
					 bool getNew, Web::IAPIProgress *userIf)`

description

*

include

* `IRKit.h`

input

*

### IRKit::createIRReader
`IRReader* createIRReader(bool newData)`

description

*

include

* `IRKit.h`

input

*


					 
					 


