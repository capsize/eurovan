# eurovan

유로밴 캠핑카의 컨트롤 패널(난징 셰다 전기 南京协达电气 **Nova-23P-12V**)을 Waveshare 산업용 ESP32-S3 보드(RS485·CAN)에 연결해 아이폰으로 제어하는 프로젝트.

## 구성

```
아이폰 ──(Wi-Fi)── ESP32-S3 보드 ──(RS485, RJ45 랜선)── Nova-23P 패널 버스
                       │
                  캠핑카 12V (퓨즈 1~2A)
```

- **배선도(단계별, 인터랙티브): <https://claude.ai/artifact/QSS3ajDXHE48Naq5C5bWvg>** (원본 `docs/wiring.html`)
- ESP32-S3 보드: [주문한 상품 (알리익스프레스)](https://ko.aliexpress.com/item/1005012855109271.html) (2026-11-26 주문)

| 파일 | 용도 |
|---|---|
| `esphome/rs485-sniff.yaml` | 패널 버스가 RS485일 때 데이터 엿듣기 |
| `esphome/can-sniff.yaml` | 패널 버스가 CAN일 때 데이터 엿듣기 |
| `esphome/secrets.yaml.example` | Wi-Fi 비밀번호 예시 (`secrets.yaml`로 복사해 사용, git에 올리지 않음) |
| `docs/protocol.md` | 분석한 프로토콜 기록 |
| `docs/wiring.html` | 단계별 인터랙티브 배선도 ([보기](https://claude.ai/artifact/QSS3ajDXHE48Naq5C5bWvg)) |
| `docs/rs485.md` | RS485 기초: A/B, GND, RJ45와의 차이, 8가닥 판정 |
| `docs/rv-c.md` | 캠핑카 표준 RV-C(CAN) 요약 |
| `docs/research.md` | 셰다 전기·비슷한 패널 조사, CAN+ESP32 연결 사례 |

## 현재 상태 (2026-11-25)

- [x] 패널 정보 확인: 南京协达电气 Nova-23P-12V, 주문처 康派斯, 순정 앱 없음
- [x] 통신 방식 확인: **RS485** (제어 박스 `RS485 X 2` RJ45 포트 2개, 같은 버스로 추정)
- [x] 연결 자리 확인: 제어 박스의 **남는 RS485 RJ45 포트**
- [x] 퓨즈 ↔ 기능, 설정값, 적산계(Juntek TK15) 정리 → [docs/protocol.md](docs/protocol.md)
- [ ] **다음:** RJ45 단자대 케이블(컴스 TB943, 구매함)로 핀 8개 전압 측정 → 12V·GND·RS485 A/B 판정
- [x] Waveshare ESP32-S3-RS485-CAN 주문 (2026-11-26, [상품 링크](https://ko.aliexpress.com/item/1005012855109271.html))
- [ ] 보드 도착 후 `esphome/rs485-sniff.yaml` 핀 번호 수정
- [ ] RS485 엿듣기(9600bps부터) → 패킷 분석 → 아이폰 제어

## 진행 순서

### 1. 프로토콜 문서 요청 (가장 빠름)

Nova-23P는 통신 프로토콜이 공개돼 있지 않다(2026-11 검색 기준). 캠핑카 제작사나 제조사에 아래 문구로 요청한다.

> 您好，我是房车用户，使用贵公司的 Nova-23P-12V 控制面板。我想用 ESP32 做手机控制，请问能否提供该面板的通信协议文档（RS485 还是 CAN、波特率、数据帧格式、指令表）以及接口引脚定义？谢谢。

문서를 받으면 3단계로 바로 간다.

### 2. 통신선 찾기

**확인됨(2026-11-25):** 화면과 메인 제어 박스는 **RJ45(랜선)**로 연결돼 있고, 제어 박스 포트 옆에 **`RS485 X 2`**라고 적혀 있다 → 통신은 **RS485**(CAN 아님). 제어 박스의 RS485 RJ45 포트 2개 중 **위쪽이 비어 있어** 여기에 ESP32 보드를 연결한다(화면 뒤에도 빈 RJ45 포트가 있다). 제어 박스에는 `CAN`(비어 있음, 이 차에서는 미사용), `USART`(사용 중), `SINGAL`(비어 있음) 커넥터도 있다. 설정 화면에는 Wi-Fi·앱 항목이 없다. RJ45 단자대 어댑터로 빈 포트의 핀별 전압을 재는 순서는 [docs/protocol.md](docs/protocol.md#빈-rj45-포트-핀-확인-순서). 빈 포트가 같은 버스면 아래 3단계(분기)는 필요 없다.

> **주의:** 이 RJ45 포트에는 12V가 흐를 수 있다. 노트북·공유기 랜 포트에 절대 꽂지 않는다.

일반적인 방법(참고):

1. 메인 스위치를 내리고 패널 뒤 커넥터를 확인한다. 핀 수·핀 간격(mm)·모양을 사진으로 남긴다.
2. 전원을 켜고 커넥터 뒤쪽에 백프로브 핀을 꽂아 멀티미터로 잰다.
   - 두 선이 모두 약 2.5V → **CAN** (H/L)
   - 두 선 사이에 0.2V 이상 차등 전압 → **RS485** (A/B)
3. 12V, GND, 통신선 2가닥의 핀 위치를 `docs/protocol.md`에 적는다.

### 3. 선을 자르지 않고 분기(탭)

**Y자 케이블(추천)**: 패널 커넥터와 같은 수·암 커넥터를 사서 1:1로 잇고, 통신선 2가닥 + GND만 보드로 뺀다. 뽑으면 원상태로 돌아간다.

커넥터별 검색어:

| 커넥터 | 국내(쿠팡) | 알리(영문) | 타오바오(중문) |
|---|---|---|---|
| RJ45 | RJ45 분배기 1:2, RJ45 단자대 어댑터 | RJ45 splitter 1 to 2, RJ45 to screw terminal | RJ45分线器 一分二 |
| RJ11/RJ12 | RJ11 분배기, RJ12 6P6C 분배기 | RJ11 splitter, 6P6C splitter | RJ11 一分二, 6P6C 分线器 |
| JST XH (2.5mm) | JST XH 2.54 커넥터 세트 | JST XH 2.54 extension cable male female | XH2.54 公母对插 延长线 |
| JST PH (2.0mm) | JST PH 2.0 커넥터 세트 | JST PH 2.0 extension cable male female | PH2.0 公母 延长线 |
| Molex Micro-Fit (3.0mm) | 마이크로핏 3.0 커넥터 | Molex Micro-Fit 3.0 extension cable | 5557 5559 公母对插线 |
| 자동차 방수형 | DT 커넥터, 방수 커넥터 | Deutsch DT connector kit | 汽车防水连接器 公母 |

타오바오에서 `协达 Nova 房车 控制面板 线束`로 순정 배선을 찾아 Y자로 개조하는 방법도 있다.

대안: Posi-Tap(피복 관통형, 거의 원상 복구 가능). 스카치락(T-탭)은 진동에 약해 쓰지 않는다.

배선 주의:
- 분기선은 짧게(CAN은 30cm 이하), 두 가닥은 꼬아서(랜선 한 쌍 사용 가능).
- 보드 GND를 패널 GND와 꼭 연결한다.
- 보드의 120Ω 종단 저항은 **끈다**(분기 지점은 버스 끝이 아님).
- 보드 전원은 12V에서 퓨즈(1~2A)를 거쳐 넣는다. 입력 전압 범위는 Waveshare 위키에서 확인.

### 4. 엿듣고 분석

1. [ESPHome](https://esphome.io/) 설치: `pip install esphome` 또는 ESPHome Desktop 앱.
2. `esphome/secrets.yaml.example`을 `esphome/secrets.yaml`로 복사하고 비밀번호를 채운다.
3. Waveshare 위키에서 보드 핀맵을 확인해 yaml의 `TODO` 핀 번호를 고친다.
4. 버스 종류에 맞는 파일을 올리고 로그를 본다.
   ```bash
   esphome run esphome/rs485-sniff.yaml   # Nova-23P는 RS485로 확인됨. can-sniff.yaml은 참고용
   ```
5. 패널 버튼(조명, 물펌프 등)을 하나씩 누르며 어떤 바이트가 바뀌는지 비교해 `docs/protocol.md`에 적는다.
   - 데이터가 깨져 보이면 속도를 바꾼다. RS485: 9600 → 19200 → 38400 → 115200. CAN: 250k → 125k → 500k.
   - 패킷이 Modbus RTU 형식(주소, 기능코드 03/06/10, CRC16)이면 ESPHome `modbus_controller`로 바로 다룰 수 있다.

### 5. 아이폰으로 제어

알아낸 명령을 ESPHome `switch`/`button`, 상태를 `sensor`로 등록한 뒤 아래 단계로 넓혀 간다. 설정은 그대로 두고 기능만 더하면 된다.

| 단계 | 방식 | 필요한 것 | 원격 |
|---|---|---|---|
| ① | ESPHome `web_server` + 보드 Wi-Fi AP. Safari로 `http://192.168.4.1` 접속 → 공유 → **홈 화면에 추가** | 없음 | 차 안만 |
| ② | Home Assistant + iOS **Home Assistant** 앱 | 라즈베리파이 등, LTE 라우터 + Tailscale(또는 HA Cloud) | 가능 |
| ③ | ②에서 **HomeKit Bridge** 켜기 → 아이폰 홈 앱·Siri | ② | 홈 허브(HomePod/Apple TV) 있으면 가능 |

HomeKit만 쓰고 싶으면 Home Assistant 없이 ESP32에 HomeSpan(Arduino 라이브러리)을 올리는 방법도 있지만, ESPHome 대신 코드를 직접 짜야 한다.

## 안전 수칙

- **처음엔 읽기만.** 명령 보내기는 조명처럼 위험 없는 것부터 하나씩 시험한다. 히터·인버터·워터펌프 오작동 주의.
- 패널이 주기적으로 상태를 묻는(폴링) 구조면, 보드가 같은 주소로 응답해 충돌하지 않게 패널이 쉬는 틈에만 보낸다.
- 기존 배선은 자르지 않는다. 작업 전 메인 스위치·퓨즈를 내린다.
- 제조사 보증 조건을 미리 확인한다.

## 참고

- [ESPHome UART debug](https://esphome.io/components/uart.html#debugging)
- [ESPHome CAN bus](https://esphome.io/components/canbus/)
- [ESPHome Modbus Controller](https://esphome.io/components/modbus_controller.html)
- [RV-Bridge (RV-C CAN ↔ HomeKit)](https://github.com/rubillos/RV-Bridge)
- [ESP32로 제어 패널 시리얼 프로토콜 분석 사례](https://community.home-assistant.io/t/made-a-dumb-dishwasher-smart-an-esp32-in-the-door-that-speaks-the-control-panels-own-serial-protocol/1028827)
- ESPHome 스타터 키트(ESP32-C6, USB-C 5V)는 RS485/CAN·12V 입력이 없어 패널 연결용으로는 맞지 않다. 연습용이나 실내 온습도 센서로는 쓸 만하다.
