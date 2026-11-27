# 추가 센서: 수평계, 외부 온도

대시보드 시안([캠퍼 평면](https://claude.ai/artifact/5ZoANdN2CLNDCxWX687srY))의 운전석 자리에 수평계를, 차 바깥에 외부 온도를 넣었다. 시안의 값은 예시이고, 실제로 쓰려면 아래 센서를 보드에 달아야 한다. **아직 주문하지 않았다.**

| 표시 | 센서 | 알리 검색어 | 보드 연결 | 필요한 GPIO |
|---|---|---|---|---|
| 수평계(앞뒤·좌우 기울기) | MPU6050 (GY-521 모듈) | `GY-521 MPU6050` | I2C, 선 4가닥 | 2개 (SDA, SCL) |
| 외부 온도 | DS18B20 방수형(선 달린 것) | `DS18B20 waterproof` | 1-Wire, 선 3가닥 | 1개 |

**먼저 확인할 것:** 주문한 Waveshare ESP32-S3-RS485-CAN 보드에 남는 GPIO가 3개 있는지. 보드가 도착하면 핀 배열을 보고 확인한다.

## 수평계 (MPU6050)

캠핑카는 주차할 때마다 수평을 맞춘다. 앞뒤·좌우가 몇 도 기울었는지 보여 주면 받침목을 어디에 댈지 바로 알 수 있다.

### 배선

| 센서 핀 | 보드 |
|---|---|
| VCC | 3.3V |
| GND | GND |
| SDA | 남는 GPIO |
| SCL | 남는 GPIO |

나머지 핀(XDA, XCL, AD0, INT)은 연결하지 않는다.

### 설치

- **차체에 단단히 고정한다.** 가구 문짝이나 흔들리는 판에 붙이면 값이 틀어진다.
- 센서를 완전히 평평하게 붙이기는 어렵다 → 수평인 곳에 차를 세우고 그때 값을 0°로 저장하는 **영점 보정**을 한 번 한다.
- 주행 중에는 가속·감속 때문에 값이 흔들린다. 주차 후에만 쓰므로 문제없다.

### ESPHome (초안, 미검증)

```yaml
i2c:
  sda: GPIOxx   # 보드 도착 후 정함
  scl: GPIOxx

sensor:
  - platform: mpu6050
    address: 0x68
    update_interval: 1s
    accel_x: { id: ax, internal: true }
    accel_y: { id: ay, internal: true }
    accel_z: { id: az, internal: true }

  - platform: template
    name: "앞뒤 기울기"
    unit_of_measurement: "°"
    accuracy_decimals: 1
    update_interval: 1s
    lambda: |-
      return atan2(id(ax).state, sqrt(pow(id(ay).state, 2) + pow(id(az).state, 2))) * 57.2958;
    filters:
      - sliding_window_moving_average: { window_size: 5, send_every: 1 }
      - offset: 0.0   # 영점 보정값

  - platform: template
    name: "좌우 기울기"
    unit_of_measurement: "°"
    accuracy_decimals: 1
    update_interval: 1s
    lambda: |-
      return atan2(id(ay).state, id(az).state) * 57.2958;
    filters:
      - sliding_window_moving_average: { window_size: 5, send_every: 1 }
      - offset: 0.0   # 영점 보정값
```

어느 축이 앞뒤인지는 센서를 붙인 방향에 따라 달라진다. 붙인 뒤 차 앞을 들었을 때 어느 값이 변하는지 보고 축과 부호를 맞춘다.

## 외부 온도 (DS18B20 방수형)

### 배선

| 센서 선 | 보드 |
|---|---|
| 빨강 (VCC) | 3.3V |
| 검정 (GND) | GND |
| 노랑 (DATA) | 남는 GPIO |

**DATA와 3.3V 사이에 4.7kΩ 저항**을 하나 연결한다(풀업). 저항이 붙어 나오는 모듈형도 있다.

### 설치

- 센서 끝을 차 밖 그늘진 곳에 둔다. 햇빛을 직접 받거나 엔진·배기 열이 닿는 곳은 피한다.
- 선이 차체를 지나는 구멍은 방수 처리한다.

### ESPHome (초안, 미검증)

```yaml
one_wire:
  - platform: gpio
    pin: GPIOxx   # 보드 도착 후 정함

sensor:
  - platform: dallas_temp
    name: "외부 온도"
    update_interval: 30s
```

## 순서

1. 보드 도착 → 남는 GPIO 확인
2. MPU6050, DS18B20 방수형, 4.7kΩ 저항 주문
3. 책상 위에서 보드에 연결해 값이 나오는지 확인
4. 차에 설치, 수평계 영점 보정
