# RV-C 요약

RV-C는 미국 RVIA가 만든 캠핑카 장치 통신 표준이다. CAN 버스(250kbps, 29비트 확장 ID) 위에서 돌아가며, SAE J1939 형식을 따른다.

> 아래 DGN 번호와 바이트 배치는 기억에 기대어 정리한 것이다. 명령을 보내기 전에 RVIA 공식 RV-C 문서(<https://www.rv-c.com/>, 무료 공개)에서 꼭 확인한다. Nova-23P가 RV-C가 아니라 자체 프로토콜을 쓸 수도 있다.

## CAN ID 구조

```
우선순위(3비트) | DGN(17비트) | 보낸 장치 주소(8비트)
```

예: 조명 명령 DGN `0x1FEDB`를 우선순위 6으로 보내면 ID는 `0x19FEDBxx`(`xx` = 보낸 장치 주소).

## 자주 쓰는 DGN

| 기능 | 상태 (읽기) | 명령 (쓰기) |
|---|---|---|
| 조명·디머 | DC_DIMMER_STATUS_3 `0x1FEDA` | DC_DIMMER_COMMAND_2 `0x1FEDB` |
| 배터리 전압·전류 | DC_SOURCE_STATUS_1 `0x1FFFD` | — |
| 물탱크·오수탱크 | TANK_STATUS `0x1FFB7` | — |
| 워터펌프 | WATER_PUMP_STATUS `0x1FFB3` | WATER_PUMP_COMMAND `0x1FFB2` |
| 온도조절(에어컨·히터) | THERMOSTAT_STATUS_1 `0x1FFE2` | THERMOSTAT_COMMAND_1 `0x1FEF9` |
| 인버터 | INVERTER_STATUS `0x1FFD4` | — |
| 충전기 | CHARGER_STATUS `0x1FFC7` | — |
| 발전기 | GENERATOR_STATUS_1 `0x1FFDC` | — |
| 날짜·시간 | DATE_TIME_STATUS `0x1FFFF` | — |

## 예: 조명 명령 (`0x1FEDB`, 데이터 8바이트)

| 바이트 | 뜻 |
|---|---|
| 0 | 조명 번호(instance) |
| 1 | 그룹(보통 `FF`) |
| 2 | 밝기(0~200, 200 = 100%) |
| 3 | 명령: `00` 밝기 설정, `03` 끄기, `05` 토글 |
| 4~7 | 대부분 `FF` |

## Nova-23P가 RV-C인지 확인하는 법

1. `esphome/can-sniff.yaml`의 `bit_rate`를 `250kbps`로 두고 엿듣는다.
2. 로그의 ID가 29비트이고, 가운데 DGN 부분이 `1FFxx`·`1FExx` 형태(예: `0x19FFB7xx`)로 반복되면 RV-C일 가능성이 높다.
3. 맞으면 위 표의 상태 DGN부터 해석해 보고(배터리 `0x1FFFD`, 탱크 `0x1FFB7`), 결과를 `docs/protocol.md`에 적는다.

## 참고

- RVIA RV-C 공식 사이트: <https://www.rv-c.com/>
- [RV-Bridge (RV-C CAN ↔ HomeKit, ESP32)](https://github.com/rubillos/RV-Bridge)
