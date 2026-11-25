# 조사: 셰다 전기 패널과 비슷한 연결 사례

2026-11 웹 검색 결과 정리. Nova-23P를 직접 다룬 자료나 연동 사례는 찾지 못했다.

## 셰다 전기(南京协达电气)

- 특수차·캠핑카·요트용 부품을 만드는 난징 회사. 차량용 통합 제어 시스템, 인버터 등을 만든다.
- 제어 시스템은 자체 개발이고, **휴대폰 앱·태블릿으로 Wi-Fi 원격 조회·제어**가 된다고 소개돼 있다. 제어 화면 3~10인치, B형·C형 캠핑카와 트레일러용. 부품은 차량용 등급, 작동 온도 -25~55°C. ([21세기房车, 2019년경](https://m.21rv.com/channel/cms/article/detail/834428))
- Nova-23P의 모델명, 앱 이름, 통신 프로토콜은 공개 자료에 없다.
- 구글 플레이의 블루투스 앱 [I-NOVA](https://play.google.com/store/apps/details?id=com.yq.ble.nova&hl=en_US)(`com.yq.ble.nova`)가 셰다 제품용인지는 확인되지 않았다.

## 비슷한 중국 캠핑카 제어기

- **森鹏(Senpeng)**: 캠핑카용 제어 화면과 도메인 컨트롤러를 만든다. 중국 캠핑카 제어가 CAN 기반 통합 제어기 방식으로 가는 흐름이다. ([智能汽车资源网](https://www.smartautoclub.com/p/90671/))
- 중국 캠핑카 제어 패널은 대부분 자체 CAN이나 RS485 프로토콜을 쓰고 문서를 공개하지 않는다. 직접 엿들어 분석하는 게 일반적이다.

## 참고할 만한 연결 사례 (CAN + ESP32 + ESPHome)

| 사례 | 내용 | 참고할 점 |
|---|---|---|
| [Espar 히터 CAN 분석](https://github.com/TrooperDuper/espar-airtronic-esphome) | 순정 컨트롤러를 ESP32로 바꾸고 CAN 프로토콜을 분석해 Home Assistant 온도조절로 연동 | 가장 비슷한 사례. 분석 → 문서화 → ESPHome 연동 과정 |
| [Jeep CAN 분석](https://community.home-assistant.io/t/esphome-can-bus-and-a-jeep-grand-cherokee-2012/758376) | `cansniffer`로 깜빡이·문을 하나씩 조작하며 바뀌는 메시지를 찾음 | 버튼별 분석 방법 |
| [Redarc (Truma/Alde 스레드)](https://community.home-assistant.io/t/truma-alde-motorhome-rv-integration/178409) | M5Stack Atom + CAN 수신기 + ESPHome으로 캠핑카 충전기 CAN 분석 | 캠핑카 전원 장치 사례 |
| [Lippert 컨트롤러](https://brinkleyowners.com/t/home-automation-but-in-an-rv-turned-up-to-11/9296) | 캠핑카 제어 패널이 CAN으로 연결돼 있어 ID 파악이 쉬웠다는 경험담 | 캠핑카 제어 패널 + CAN |
| [RV-Bridge](https://github.com/rubillos/RV-Bridge) | ESP32로 RV-C(CAN) ↔ HomeKit | 패널이 RV-C면 거의 그대로 사용 가능 ([rv-c.md](./rv-c.md)) |
| [Nissan Leaf 연동기](https://blog.jingo.uk/notes-on-integrating-a-nissan-leaf-ze1-and-home-assistant/) | ESPHome `on_frame`으로 바이트를 해석해 센서 만들기 | ESPHome 코드 예시 |
| [WiCAN](https://hackaday.com/2026/06/04/connecting-your-car-to-home-assistant/) | ESP32-S3 기반 오픈소스 CAN 어댑터, MQTT로 Home Assistant 연동 | 완성품 대안 |
| [Vevor 히터](https://github.com/zatakon/vevor_heater_control) | 중국 히터, CAN이 아닌 단선 버스를 분석 | 중국 제품 분석 사례 |

## 결론과 다음 할 일

1. **순정 앱 먼저 확인**: 유로밴이나 셰다에 Nova-23P용 휴대폰 앱이나 Wi-Fi 모듈이 있는지 묻는다. 있으면 그 앱을 쓰거나, 앱 통신(Wi-Fi/BLE)을 분석하는 게 CAN보다 쉬울 수 있다.
2. **없으면 CAN 엿듣기**: Espar 사례처럼 `esphome/can-sniff.yaml`을 250kbps(RV-C 확인)부터 돌리고, 결과를 [protocol.md](./protocol.md)에 적는다.
3. **분석 안전**: 차량 주행 관련 버스(엔진·섀시)에는 연결하지 않는다. 캠핑카 실내 제어 버스에서만 시험한다.
