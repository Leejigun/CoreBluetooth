## Performing Common Central Role Tasks

블루투스 통신에서 중심 역할을 구현하는 장치는 여러가지 공통적인 태스크를 수행합니다. 예를 들어, 사용 가능한 주변 장치를 검색 및 연결하고 주변 장치가 제공하는 데이터를 탐색 및 상호 작용합니다. 주변 역할을 구현하는 기기는 또한 여러가지 일반적이지만 다른 태스크를 수행합니다. 예를 들어, 게시 및 광고 서비스를 수행하고 연결된 중앙 집중식의 읽기, 쓰기 및 구독 요청에 응답합니다.

이 장에서는 코어 블루투스 프레임워크를 사용하여 중앙 측에서 가장 일반적인 유형의 블루투스 작업을 수행하는 방법에 대해 배우게 됩니다. 다음에 나오는 코드 기반의 예는 당신이 당신의 로컬 기기에서 중심적인 역할을 구현하기 위해서 당신의 앱을 개발하는데 도움을 줄것이다. 

구체적으로 다음을 수행하는 방법에 대해 알아봅니다.

* 중앙 관리자 개체 시작
* 광고하고 있는 주변 장치 검색 및 연결
* 주변 장치에 연결한 후 주변 장치에서 데이터 탐색
* 주변 장치 서비스의 특성 값으로 읽기 및 쓰기 요청 전송
* 특성 값을 구독하면 특성 값이 업데이트될 때 알림을 받을 수 있습니다.



다음 챕터에서는 로컬 장치에서 주변 장치 역할을 구현하는 앱을 개발하는 방법에 대해 알아보겠습니다.이 장의 코드 예시는 간단하고 추상적입니다. 실제 환경의 앱에 적용하려면 적절한 변경 작업을 수행해야 할 수도 있습니다.

###Starting Up a Central Manager

[CBCentralManager](https://developer.apple.com/documentation/corebluetooth/cbcentralmanager)개체는 로컬 중앙 장치의 핵심 블루투스 개체 지향 표현이기 때문에 블루투스 트랜잭션을 수행하기 전에 중앙 관리자 인스턴스를 할당하고 초기화할 수 있습니다. 중앙 관리자는 initialization(초기화) 하기 위해 `InitWithDelegate:queue:options:`메서드를 호출해야 합니다.

```
myCentralManager = [[CBCentralManager alloc] initWithDelegate:self queue:nil options:nil];
```

이 예에서는, 자체적으로 중앙 역할 이벤트를 받도록 설정되어 있습니다. CentralManager는 발송 대기 열을 nil로 지정하여 중앙 역할 이벤트를 전송합니다.

CentralManager를 생성하면 CentralManager가 [centralManagerDidUpdateState:](https://developer.apple.com/documentation/corebluetooth/cbcentralmanagerdelegate/1518888-centralmanagerdidupdatestate) 델리게이트 객체의 메소드를 호출합니다. 당신은 메소드를 구현하여 중앙 장치에서 블루투스를 지원하고 사용할 수 있도록 하십시오. 이 델리게이트 메소드를 구현하는 방법에 대한 자세한 내용은 *CBCentralManagerDelegate Protocol Reference* 를 참조하십시오.

### Discovering Peripheral Devices That Are Advertising

일단 중앙 관리자가 초기화되면, 중앙 관리자의 첫번째 작업은 주변 장치 검색입니다. 주변 장치는 광고로 자신들의 존재를 알립니다. 앱이 중앙 관리자의 검색 도구 프로그램을 호출해 광고하고 있는 근처의 주변 장치를 검색합니다.

```
[myCentralManager scanForPeripheralsWithServices:nil options:nil];
```

**Note:** 첫번째 파라미터에 nil을 지정하면, 중앙 관리자가 지원되는 서비스에 관계 없이 검색된 모든 주변 장치를 반환합니다. 실제 앱에서는 대개 [CBUUID](https://developer.apple.com/documentation/corebluetooth/cbuuid) 객체의 배열을 지정하며, 각 객체는 주변 장치가 광고하고 있는 서비스의 UUID(범용 고유 식별자)를 나타냅니다. 여러개의 서비스 UUID를 지정하면 중앙 관리자가 이러한 서비스를 광고하는 주변 장치만 반환하여 사용자가 관심 있는 디바이스만 검색할 수 있도록 합니다.

중앙 장치 관리자가 주변 장치를 찾아낼 때 마다 델리게이트 객체는  `centralManager: didDiscoverPeripheral: advertisementData: RSSI:` 를 호출합니다. 새로 발견된 주변 장치는 [CBPeripheral](https://developer.apple.com/documentation/corebluetooth/cbperipheral)개체로 반환됩니다. 검색된 주변 장치에 연결하려는 경우 시스템이 이를 할당 취소하지 않도록 이에 대한 강력한 참조(Strong)를 유지하십시오. 다음 예에서는 클래스 속성을 사용하여 검색된 주변 장치에 대한 참조를 유지하는 시나리오를 보여 줍니다.

```
- (void)centralManager:(CBCentralManager *)central didDiscoverPeripheral:(CBPeripheral *)peripheral advertisementData:(NSDictionary *)advertisementData RSSI:(NSNumber *)RSSI {
  
  NSLog(@"Discovered %@", peripheral.name);
  self.discoveredPeripheral = peripheral;
}
...
```

여러 장치에 연결하려는 경우 대신 NSArray의 검색된 주변 장치를 유지할 수 있습니다. 연결하려는 주변 장치를 모두 찾았으면 다른 장치에 대한 스캔을 중지하여 전력을 절약하십시오.

```
[myCentralManager stopScan];
```

### Connecting to a Peripheral Device After You’ve Discovered It

관심 있는 주변 장치 광고 서비스를 발견한 후에는 중앙 관리자의 연결 장치를 호출해 주변 장치에 대한 연결을 요청합니다.

```
[myCentralManager connectPeripheral:peripheral options:nil];
```

연결 요청이 성공적으로 이루어지면 중앙 관리자는 [centralManager:didConnectPeripheral:](https://developer.apple.com/documentation/corebluetooth/cbcentralmanagerdelegate/1518969-centralmanager) 를 호출합니다. 주변 장치와 상호 작용을 시작하기 전에 델리게이트에 적절한 콜백을 수신하도록 하기 위해 대리인을 설정합니다.

```
- (void)centralManager:(CBCentralManager *)central didConnectPeripheral:(CBPeripheral *)peripheral {
	NSLog(@"Peripheral connected");
	peripheral.delegate = self;
	...
}
```

### Discovering the Services of a Peripheral That You’re Connected To

주변 장치에 대한 연결을 설정한 후에는 해당 장치의 데이터를 탐색할 수 있습니다. 주변 기기가 제공하는 것을 탐색하기 위한 첫번째 단계는 이용 가능한 서비스를 발견하는 것입니다. 주변 장치가 광고할 수 있는 데이터의 양에는 크기 제한이 있기 때문에, 주변 장치가 광고하는 것보다 더 많은 서비스를 가지고 있다는 것을 알 있다. 주변 장치의  `discoverServices:`메서드를 사용하면 주변 장치에서 제공하는 모든 서비스를 검색할 수 있습니다.

```
[peripheral discoverServices:nil];
```

**Note:** 실제 응용 프로그램에서는 대개 매개 변수로 nil로 전달하지 않습니다. 그렇게 하면 주변 장치에서 사용할 수 있는 모든 서비스가 반환되기 때문입니다. 주변 장치는 사용자가 관심 있는 것보다 더 많은 서비스를 포함할 수 있기 때문에, 모든 서비스를 발견하는 것은 배터리 수명을 낭비하고 불필요한 시간 사용이 될 수 있습니다. 대신, [Explore a Peripheral’s Data Wisely](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/BestPracticesForInteractingWithARemotePeripheralDevice/BestPracticesForInteractingWithARemotePeripheralDevice.html#//apple_ref/doc/uid/TP40013257-CH6-SW6) 에서처럼 이미 알고 있는 서비스의 UUID를 지정하는 것이 좋습니다.

지정된 서비스가 검색되면 연결된 주변 장치(CBPeripheral)는  `peripheral:didDiscoverServices:` 메서드를 호출합니다. CoreBluetooth는 주변 장치에서 발견되는 각 서비스에 대해 CBService객체의 배열을 생성합니다. 다음과 같이 이 델리게이트 메소드를 구현하여 검색된 다양한 서비스에 액세스 할 수 있습니다.

```
- (void)peripheral:(CBPeripheral *)peripheral didDiscoverServices:(NSError *)error {
  for (CBService *service in peripheral.services){
    NSLog(@"Discovered service %@", service);
    ...
  }
}
...
```

### Discovering the Characteristics of a Service

관심 있는 서비스를 찾게 되면 다음 단계는 해당 서비스의 모든 특성을 발견하는 것입니다. 서비스의 모든 특성을 검색하는 것은 주변 장치의  `discoverCharacteristics:forService:`메서드로, 다음과 같이 적절한 서비스를 지정하는 것만큼이나 간단하게 검색합니다.

```
NSLog(@"Discovering characteristics for service %@", interestingService);
[peripheral discoverCharacteristics:nil forService:interestingService];
```

**Note:** 실제 앱에서는 대개 첫번째 파라미터로 nil을 전달하지 않습니다. 그렇게 하면 주변 장치 서비스의 모든 특성이 반환되기 때문입니다. 주변 장치의 서비스가 관심 있는 것보다 더 많은 특성을 포함할 수 있기 때문에, 모든 것을 발견하는 것은 배터리 수명을 낭비하고 불필요한 시간 사용이 될 수 있다. 대신 검색하려는 특성의 UUID를 지정하는 것이 좋습니다.

주변 장치는 지정된 서비스의 특성이 검색될 때 위임 개체의  `peripheral: didDiscoverCharacteristicsForService: error:`메서드를 호출합니다. CoreBluetooth는 검색된 각 특성에 대해 [CBCharacteristic](https://developer.apple.com/documentation/corebluetooth/cbcharacteristic)객체의 배열을 생성합니다. 다음 예에서는 이 방법을 구현하여 검색된 모든 특성을 기록하는 방법을 보여 줍니다.

```
- (void)peripheral:(CBPeripheral *)peripheral didDiscoverCharacteristicsForService:(CBService *)service error:(NSError *)error {
  for (CBCharacteristic *characteristic in service.characteristics){
    NSLog(@"Discovered characteristic %@", characteristic);
    ...
  }
}
...
```



### Retrieving the Value of a Characteristic

특성에는 주변 장치 서비스에 대한 정보를 나타내는 단일 값이 포함됩니다. 예를 들어, 건강 온도계 서비스의 온도 측정 특성은 섭씨로 온도를 나타내는 값을 가질 수 있습니다. 특성을 직접 읽거나 구독하여 특성의 값을 검색할 수 있습니다.

#### Reading the Value of a Characteristic

관심 있는 서비스의 특성을 찾은 후에는 주변 장치의 `readValueForCharacteristic:` 메서드를 호출하여 특성 값을 읽을 수 있습니다.

```
NSLog(@"Reading value for characteristic %@", interestingCharacteristic);
[peripheral readValueForCharacteristic:interestingCharacteristic];
```

특성의 값을 읽으려고 하면 주변 장치가 값을 검색하기 위해 [peripheral:didUpdateValueForCharacteristic:error:](https://developer.apple.com/documentation/corebluetooth/cbperipheraldelegate/1518708-peripheral) 메서드를 호출합니다. 값을 성공적으로 검색한 경우 다음과 같이 특성의 값 속성을 통해 값에 액세스 할 수 있습니다.

```
- (void)peripheral:(CBPeripheral *)peripheral didUpdateValueForCharacteristic:(CBCharacteristic *)characteristic error:(NSError *)error {
  NSData *data = characteristic.value;
  // parse the data as needed
  ...
}
```

**Note:** 모든 특성을 읽을 수 있는 것은 아닙니다. 특성에 `CBCharacteristicPropertyRead` 상수가 포함되는지 여부를 확인하여 특성을 읽을 수 있는지 여부를 확인할 수 있습니다. 읽을 수 없는 특성의 값을 읽으려고 하면 `peripheral: didUpdateValueForCharacteristic: error:` 메서드가 적절한 오류를 반환합니다.

### Subscribing to a Characteristic’s Value

 `readValueForCharacteristic:` 메소드를 사용하여 특성 값을 읽을 경우, 정적 값에 대해서는 효과적일 수 있지만 동적 값을 검색하는 가장 효율적인 방법이 아닙니다. 시간에 따라 변하는 특성 값은 구독해야 합니다. 예를 들어, 심장 박동이 빨라지는 경우 이를 구독하면 됩니다. 특성 값을 구독하면 값이 변경될 때 주변 장치로부터 알림을 받습니다.

주변 장치의  `setNotifyValue:forCharacteristic:`메서드로, 첫번째 매개 변수를 YES로 지정하여 관심이 있는 특성 값을 구독합니다.

```
[peripheral setNotifyValue:YES forCharacteristic:interestingCharacteristic];
```

특성 값에 구독하거나 이 값에서 구독을 취소하면 주변 장치에서 [peripheral: didUpdateNotificationStateForCharacteristic: error:](https://developer.apple.com/documentation/corebluetooth/cbperipheraldelegate/1518768-peripheral)메서드를 호출합니다. 어떤 이유로든 구독 요청이 실패할 경우 다음 예와 같이 이 메소드를 구현하여 오류의 원인에 액세스 할 수 있습니다.

```
- (void)peripheral:(CBPeripheral *)peripheral didUpdateNotificationStateForCharacteristic:(CBCharacteristic *)characteristic error:(NSError *)error{
  if (error){
     NSLog(@"Error changing notification state: %@",[error localizedDescription]);
  }
}
...
```

**Note:** 모든 특성이 구독을 제공하는 것은 아닙니다. 특성에  `CBCharacteristicPropertyNotify`또는 [CBCharacteristicPropertyIndicate](https://developer.apple.com/documentation/corebluetooth/cbcharacteristicproperties/1519085-indicate)상수 중 하나가 포함되어 있는지 여부를 확인하여 구독을 제공할 수 있습니다.

특성 값에 구독한 후에는 값이 변경되면 주변 장치에서 사용자의 앱에 알립니다. 값이 변경될 때마다 주변 장치는 [peripheral: didUpdateValueForCharacteristic: error:](https://developer.apple.com/documentation/corebluetooth/cbperipheraldelegate/1518708-peripheral)메서드를 델리게이트 개체로 호출합니다. 업데이트된 값을 검색하려면 [Reading the Value of a Characteristic](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/PerformingCommonCentralRoleTasks/PerformingCommonCentralRoleTasks.html#//apple_ref/doc/uid/TP40013257-CH3-SW12).에서 설명하는 것과 동일한 방법으로 메소드를 구현할 수 있습니다.

### Writing the Value of a Characteristic

때때로 특성의 값을 변경하는 것이 필요합니다. 예를 들어 앱이 블루투스 상호 작용하는 경우 서모 스탯에 실내 온도를 설정할 수 있는 값을 제공할 수 있습니다. 특성 값이 쓰기 가능한 경우 주변 장치의 write(쓰기)에 다음과 같은 경우 NSData(인스턴스)값을 쓸 수 있습니다.

```
NSLog(@"Writing value for characteristic %@", interestingCharacteristic);
[peripheral writeValue:dataToWrite forCharacteristic:interestingCharacteristictype: CBCharacteristicWriteWithResponse];
```

특성 값을 기록할 때 수행할 쓰기 모드를 지정할 수 있습니다. 위의 예에서 쓰기 유형은 `CBCharacteristicWriteWithResponse`로, 이 애플리케이션에서 다음과 같은 요소를 호출하여 입력하도록 지시하는 것입니다. 다음 예에서 볼 수 있듯이, 이 위임 방법을 구현하여 오류 조건을 처리합니다.

```
- (void)peripheral:(CBPeripheral *)peripheral didWriteValueForCharacteristic:(CBCharacteristic *)characteristic error:(NSError *)error{
  if (error){
    NSLog(@"Error writing characteristic value: %@", [error localizedDescription]);
  }
}
...
```

만약  `CBCharacteristicWriteWithoutResponse`,로 쓰기 유형을 지정하는 경우에는 쓰기 작업이 best-effort로 수행되며, 전송이 보장되지도 보고되지도 않습니다. 주변 장치는 어떤 델리게이트 메소드도 호출하지 않는다. 코어 블루투스 프레임워크에서 지원되는 쓰기 유형에 대한 자세한 내용은 [CBPeripheralClassReference](https://developer.apple.com/documentation/corebluetooth/cbperipheral)의 [CBCharacteristicWriteType](https://developer.apple.com/documentation/corebluetooth/cbcharacteristicwritetype) 열거 값을 참조하십시오.

**Note:** 특성은 특정 유형의 쓰기만 지원하거나, 아무 쓰기도 지원하지 않을 수 있습니다. [CBCharacteristicPropertyeWiteWiteWatWartsponse](https://developer.apple.com/documentation/corebluetooth/cbcharacteristicproperties/cbcharacteristicpropertywritewithoutresponse)또는  [CBCharacteristicPropertyWrite](https://developer.apple.com/documentation/corebluetooth/cbcharacteristicproperties/1519089-write)중 하나에 대한 속성 속성 속성을 확인하여 쓰기 유형( 있는 경우)을 지원할 수 있는지 확인합니다.