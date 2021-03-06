## Best Practices for Interacting with a Remote Peripheral Device

핵심 블루투스 프레임워크를 통해 중앙 집중식 통신의 많은 부분이 앱에 투명하게 이루어집니다. 즉, 앱은 중앙 역할의 대부분을 제어하고 구현하는 역할을 담당하며, 기기 검색 및 연결, 원격 주변 데이터 탐색 및 상호 작용 등을 수행합니다. 이 장에서는 특히 iOS기기용 앱을 개발할 때 책임 있는 방식으로 이 수준의 제어를 활용하기 위한 지침과 모범 사례를 제공합니다.

### Be Mindful of Radio Usage and Power Consumption

블루투스 장치와 상호 작용하는 앱을 개발할 때, 블루투스 저 에너지 통신이 장치의 라디오를 공유하여 공기를 통해 신호를 전송한다는 점을 기억하십시오. 다른 형식의 무선 통신에서는 장치의 라디오를 사용해야 할 수도 있습니다. 예를 들어 Wi-Fi, 클래식 블루투스는 물론, BluetoothLowEnergy.를 사용하는 다른 앱도 라디오 사용을 최소화하여 앱을 개발합니다.

iOS기기용 앱을 개발할 때 무선 사용을 최소화하는 것은 특히 중요합니다. 무선 사용은 iOS기기의 배터리 수명에 부정적인 영향을 미치기 때문입니다. 다음의 지침은 당신이 당신 기기의 라디오를 잘 시청하는데 도움이 될 것이다. 결과적으로 애플리케이션의 성능이 향상되고 기기의 배터리가 더 오래 지속됩니다.