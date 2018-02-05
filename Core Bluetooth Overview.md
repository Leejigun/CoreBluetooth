## Core Bluetooth Overview

핵심 블루투스 프레임워크를 통해 iOS및 Mac앱이 블루투스 장치와 통신할 수 있습니다. 예를 들어 앱은 심박 수 모니터, 디지털 온도 조절기 및 기타 iOS기기와 같은 낮은 에너지 주변 장치를 검색, 탐색 및 상호 작용할 수 있습니다.

프레임워크는 주변 장치를 이용하기 위한 블루투스 4.0사양을 추상화하고 있습니다. 다시 말해서, 개발자에게 제공하는 많은 하위 수준의 세부 정보를 숨기기 때문에 블루투스 장치와 상호 작용하는 앱을 훨씬 쉽게 개발할 수 있습니다. 프레임워크는 블루투스 4.0을 기반으로 하기 때문에, 규격의 일부 개념과 용어가 채택되어 사용되고 있습니다. 이 장에서는 핵심 블루투스 프레임워크를 사용하여 우수한 앱을 개발하기 위해 알아야 하는 주요 용어와 개념을 소개합니다.

**<Important:** iOS 10 이상의 버전에서는 블루투스를 사용하려면 반드시 `Info.plist` 파일에 사용하는데 필요한 데이터 유형에 대한 description keys 를 포함하고 있어야 합니다. 블루투스 주변 장치의 데이터에 특별히 접근하려면 다음 링크의 description keys를 읽어보시기를 바랍니다.  [NSBluetoothPeripheralUsageDescription](https://developer.apple.com/library/content/documentation/General/Reference/InfoPlistKeyReference/Articles/CocoaKeys.html#//apple_ref/doc/uid/TP40009251-SW20)>

### Central and Peripheral Devices and Their Roles in Bluetooth Communication

모든 블루투스 통신에는 두개의 주요 플레이어, 즉 중앙(central)과 주변(peripheral) 장치가 포함되어 있습니다. 다소 전통적인 클라이언트-서버 아키텍처에 기초하여, 주변 장치는 일반적으로 다른 장치에 필요한 데이터를 가지고 있습니다. 중앙 장치는 일반적으로 주변 장치가 제공하는 정보를 사용하여 특정 작업을 수행합니다. 예를 들어 그림 1-1에 표시된 것처럼 심박 수 모니터에는 사용자의 심박 수를 사용자 친화적으로 표시하기 위해 Mac이나 iOS앱에 필요한 유용한 정보가 있을 수 있습니다.

![Figure 1-1  Central and peripheral devices](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/Art/CBDevices1_2x.png)

### Centrals Discover and Connect to Peripherals That Are Advertising

주변 장치들은 그들이 가지고 있는 일부 데이터를 광고 패킷(advertising packets)의 형태로 방송한다. 광고 패킷은 주변 장치의 이름,기본 기능 등 주변 장치가 제공할 수 있는 유용한 정보를 포함하고 있느 작은 데이터 묶음입니다. 예를 들어 디지털 온도 조절기는 실내의 현재 온도를 제공한다고 광고할 수 있다. 블루투스에서는, 광고가 주변 장치들이 그들의 존재를 알리는 주된 방법입니다.

반면, 중앙 기기는 그림 1-2에 나타난 것처럼 관심 있는 정보를 광고하는 주변 기기를 검색하고 찾을 수 있습니다. 중앙기기는 자신이 광고를 발견한 주변 장치에 연결해 달라고 요청할 수 있다.

![**Figure 1-2**  Advertising and discovery](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/Art/AdvertisingAndDiscovery_2x.png)



### How the Data of a Peripheral Is Structured

주변 장치에 연결하는 목적은 그것이 제공하는 데이터를 탐색하고 상호 작용을 시작하는 것입니다. 그러나 이렇게 하기 전에 주변 장치의 데이터가 어떻게 구조화되어 있는지 이해가 필요합니다.

주변 장치는 하나 이상의 **서비스**를 포함하거나 연결된 신호 강도에 대한 유용한 정보를 제공할 수 있습니다. 서비스는 기기(또는 기기의 일부)의 기능이나 기능을 달성하기 위한 관련된 행동과 데이터의 모음이다. 예를 들어 심박 수 모니터의 서비스는 모니터의 심박 수 센서에서 나오는 심박 수 데이터를 노출하는 것일 수 있습니다.

서비스 자체는 **특성** 또는 **포함된 다른 서비스**(즉, 다른 서비스에 대한 참조)로 구성됩니다. 특성은 주변 기기 서비스에 대한 추가 세부 사항을 제공합니다. 예를 들어 방금 설명한 심박 수 서비스에는 기기의 심박 수 센서의 예정된 신체 위치를 설명하는 특성과 심박 수 측정 데이터를 전송하는 다른 특성이 포함될 수 있다. 그림 1-3은 심박 수 모니터의 서비스와 특성을 보여 주는 한가지 가능한 구조를 보여 주고 있다.

![**igure 1-3**  A peripheral’s service and characteristics](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/Art/CBPeripheralData_Example_2x.png)

### How Centrals, Peripherals, and Peripheral Data Are Represented

블루투스 통신과 관련된 주요 플레이어와 데이터는 간단한 방식으로 핵심 블루투스 프레임워크에 매핑 됩니다.

#### Objects on the Central Side

로컬 중심 기기를 사용하여 주변의 원격 장치와 상호 작용하는 경우에는 블루투스 통신의 중앙 기기에서 작업을 수행합니다. 로컬 주변 장치를 설정하고 이를 사용하여 중앙에서 요청에 응답하지 않는 한 대부분의 블루투스 트랜잭션이 중앙에서 처리됩니다.

앱에서 중앙 역할을 구현하는 방법에 대한 자세한 내용은 원격 주변 장치와 상호 작용하기 위한 [Performing Common Central Role Tasks](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/PerformingCommonCentralRoleTasks/PerformingCommonCentralRoleTasks.html#//apple_ref/doc/uid/TP40013257-CH3-SW1) 및  [Best Practices for Interacting with a Remote Peripheral Device](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/BestPracticesForInteractingWithARemotePeripheralDevice/BestPracticesForInteractingWithARemotePeripheralDevice.html#//apple_ref/doc/uid/TP40013257-CH6-SW1)을 참조하십시오.

#### Local Centrals and Remote Peripherals

중앙의 측면에서 로컬 중앙 장치는 [CBCentralManager](https://developer.apple.com/documentation/corebluetooth/cbcentralmanager)개체에 의해서 나타냅니다. 이러한 개체는 검색 및 주변 장치 연결을 포함하여 검색되거나 연결된 원격 주변 장치( `CBPeripheral`Object로 나타냄)를 관리하는 데 사용됩니다. 그림 1-4는 로컬 중심 및 원격 주변 장치가 코어 블루투스 프레임워크에 어떻게 표시되어 있는지 보여 줍니다.

![img](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/Art/CBObjects_CentralSide_2x.png)



#### A Remote Peripheral’s Data Are Represented by CBService and CBCharacteristic Objects

원격 주변 장치의 데이터(CBPeripheral 객체)와 상호 작용하는 경우에는, 그 객체가 가진 서비스와 특성을 다루고 있는 것입니다. CoreBluetooth프레임워크에서는 원격 주변 장치의 서비스가 CBService개체로 표시됩니다. 마찬가지로, 원격 주변 장치 서비스의 특성은 CBCharacteristic개체로 나타냅니다. 그림 1-5는 원격 주변 기기의 서비스와 특성의 기본 구조를 보여 준다.

![img](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/Art/TreeOfServicesAndCharacteristics_Remote_2x.png)

#### Objects on the Peripheral Side

Mac및 iOS기기는 블루투스 주변 장치로 기능할 수 있으며 다른 Mac, iPhone및 iPad장치를 포함한 다른 장치에 데이터를 제공 할 수 있습니다. 주변 장치 역할을 구현하도록 장치를 설정한다면, 블루투스 통신에서 주변 장치의 작업을 수행할 수 있습니다.

#### Local Peripherals and Remote Centrals

주변 장치 측면에서 로컬 주변 장치는 CBPeripheralManager개체로 나타난다. 이러한 개체는 로컬 주변 장치의 서비스 및 특성 데이터베이스 내에서 게시된 서비스를 관리하고 이러한 서비스를 원격 중앙 장치(CBCentral개체로 표시됨)에 광고하는 데 사용됩니다. 주변 관리자 개체는 이러한 원격 중앙 집중식 읽기 및 쓰기 요청에도 응답하는 데 사용됩니다. 그림 1-6은 로컬 주변 장치와 원격 중심부가 코어 블루투스 프레임워크에 어떻게 표시되어 있는지 보여 줍니다.

![img](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/Art/CBObjects_PeripheralSide_2x.png)

#### A Local Peripheral’s Data Are Represented by CBMutableService and CBMutableCharacteristic Objects

로컬 주변 장치에서 데이터를 설정하고 상호 작용하는 경우(CBPeripheralManager개체로 나타냄)서비스 및 특성에 대한 다양한 버전을 다루고 있는 것이다. CoreBluetooth프레임워크에서는 로컬 주변 장치의 서비스가 CBMutableService개체로 표시됩니다. 마찬가지로, 로컬 주변 장치 서비스의 특성은 CB.tableCharacteristic 객체로 표현된다. 그림 1-7은 지역 주변 기기 서비스의 기본 구조와 특성을 보여 준다.

![img](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/Art/TreeOfServicesAndCharacteristics_Local_2x.png)
