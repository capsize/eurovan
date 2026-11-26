# 슬라이드 도어 열면 전등 켜기

슬라이드 도어가 열리면 실내 전등을 자동으로 켠다. **ESP32가 RS485로 조명 명령을 보낼 수 있게 된 뒤**(엿듣기로 조명 켜기·끄기 명령 확인)에 붙인다.

```
슬라이드 도어 센서 ──(BLE)──▶ ESP32-S3 보드 ──RS485 "조명 켜기"──▶ 제어 박스 ──▶ 전등
```

## 문 센서 고르기

| 방법 | 설치 | 장점 | 단점 |
|---|---|---|---|
| **① 블루투스 문 센서 (선택)** | 문틀에 양면테이프 | 배선 없음, 차량 손 안 댐 | 배터리 교체(1년 이상) |
| ② 유선 자석 센서(리드 스위치) | 보드까지 선을 깖 | 배터리 없음, 빠름 | 배선 필요, 보드 남는 입력 핀 확인 필요 |
| ③ 차량 문 신호 따기 | 스타리아 실내등·도어 스위치 배선 | 센서 추가 없음 | **비추천.** 하이브리드 차량 전장 건드림, 보증·고장 위험 |

## ESP32-S3의 블루투스

- 칩에 **Wi-Fi(2.4GHz)와 블루투스 5(BLE)**가 같이 있어 부품 추가가 필요 없다.
- **BLE만** 된다. 문 센서·온습도계는 BLE라 괜찮다. 스피커·이어폰 같은 클래식 블루투스는 안 된다.
- Wi-Fi와 동시에 쓸 수 있다. 안테나를 나눠 쓰므로 금속 차체에서는 외장 안테나(`-U`)가 유리하다.
- ESPHome 기능: `esp32_ble_tracker`.

## 센서: 샤오미 문·창문 센서 2 (MCCGQ02HL)

- ESPHome 공식 지원 플랫폼 `xiaomi_mccgq02hl`.
- **주변광 감지** 내장 → 어두울 때만 켜기를 센서 하나로 처리.
- 배터리 CR2032 × 2. 신호가 암호화돼 **bindkey**가 필요하다.

### 준비 (보드 도착 전에 해 둘 수 있음)

1. 아이폰 **Mi Home(샤오미 홈)** 앱에 센서 등록. CN 버전이라 앱 지역을 **중국(China)**으로 둔다.
2. "Xiaomi Cloud Tokens Extractor" 등으로 센서의 **MAC 주소와 bindkey(32자리)**를 뽑는다(샤오미 계정 로그인 필요).
3. MAC·bindkey는 `esphome/secrets.yaml`에 넣는다(git에 올리지 않음).

### 설치

- 본체는 차체 문틀, 자석은 슬라이드 도어. 닫혔을 때 간격 **약 2cm 이내**.
- 슬라이드 도어는 옆으로 밀리며 닫히므로, 닫힌 위치에서 둘이 나란히 마주 보게 붙인다.
- 열림 신호는 거의 즉시, 가끔 1~2초 늦을 수 있다.

## ESPHome 설정 (초안)

조명 스위치(`light_main`)는 RS485 명령을 알아낸 뒤 만든다. 아래는 형태만 잡은 초안이다. 필드 이름은 [ESPHome Xiaomi BLE 문서](https://esphome.io/components/sensor/xiaomi_ble.html)에서 확인한다.

```yaml
esp32_ble_tracker:

binary_sensor:
  - platform: xiaomi_mccgq02hl
    mac_address: !secret door_mac
    bindkey: !secret door_bindkey
    opening:
      id: door_open
      name: "슬라이드 도어"
      on_press:        # 열림
        - if:
            condition:
              and:
                - binary_sensor.is_off: door_bright   # 어두울 때만
                - switch.is_off: light_main           # 이미 켜져 있으면 손대지 않음
            then:
              - switch.turn_on: light_main
              - globals.set: { id: door_lit, value: "true" }
      on_release:      # 닫힘
        - delay: 3min
        - if:
            condition:
              and:
                - binary_sensor.is_off: door_open
                - lambda: return id(door_lit);
            then:
              - switch.turn_off: light_main
              - globals.set: { id: door_lit, value: "false" }
    light:
      id: door_bright
      name: "도어 주변 밝음"

globals:
  - id: door_lit        # 문 때문에 켠 경우만 자동으로 끈다
    type: bool
    initial_value: "false"
```

## 동작 규칙

- 문 열림 + 어두움 → 조명 켜기.
- 문 닫힘 → 3분 뒤 끄기. 단 **문 때문에 켠 경우만**(직접 켜 둔 조명은 그대로).
- 처음엔 실내등 하나로 시험하고, 잘 되면 어닝등 등으로 넓힌다.

## 순서

1. 핀 측정 → 엿듣기 → **조명 켜기·끄기 명령 확인**
2. MCCGQ02HL 구매, Mi Home 등록, bindkey 추출
3. 위 설정 추가, 시험
