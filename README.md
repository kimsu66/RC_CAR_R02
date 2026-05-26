# RC_CAR_R02

STM32F411 마이크로컨트롤러 기반의 자율 주행 DC 카 프로젝트. UART 통신 및 블루투스 컨트롤러를 통한 원격 제어를 지원

## 개요

- STM32CubeIDE 환경에서 개발한 임베디드 RC카 펌웨어 
- 블루투스 모듈(HC-06)을 통해 스마트폰 또는 컨트롤러와 무선 통신하며, DC 모터 PWM 제어로 전진·후진·좌회전·우회전 동작을 수행한다.
- 자율 주행 모드에서는 센서 입력을 기반으로 장애물 회피 또는 라인 트래킹을 수행한다.

## 하드웨어 구성

| 구성 요소 | 모델 | 설명 |
|---|---|---|
| MCU | STM32F411RETx | ARM Cortex-M4, 100MHz |
| 모터 드라이버 | L298N | DC 모터 2채널 PWM 제어 |
| 블루투스 모듈 | HC-05 / HC-06 | UART 기반 무선 통신 |
| 거리 센서 | HC-SR04 (옵션) | 초음파 장애물 감지 |
| 전원 | Li-Po 7.4V / 배터리 | 모터 및 MCU 전원 공급 |

## 주요 기능

- **블루투스 제어**: UART를 통한 HC-06 연동, 스마트폰 앱으로 실시간 방향 제어
- **DC 모터 PWM 제어**: TIM 채널을 통한 속도 및 방향 제어
- **자율 주행 모드**: 센서 입력 기반 장애물 회피 / 라인 트래킹 전환
- **UART 디버그**: USART를 통한 상태 메시지 출력 및 명령 수신
- **STM32 HAL 기반**: STM32CubeMX로 생성된 HAL 드라이버 활용

## 프로젝트 구조

```
RC_CAR_R02/
├── Core/
│   ├── Inc/                    # 헤더 파일
│   │   └── main.h
│   └── Src/                        # 소스 및 헤더 파일
│       │
│       │  [사용자 구현 - 차량 제어]
│       ├── main.c                  # 메인 루프: 센서 모니터링, UART 명령 처리, 자율주행 분기
│       ├── car.c / car.h           # 차량 고수준 제어 API (전/후/좌/우/대각선, percent 기반)
│       ├── direction.c / direction.h  # 모터 방향 GPIO 제어 (IN1~IN4)
│       ├── speed.c / speed.h       # TIM2 PWM 듀티비 기반 속도 제어 (0~100%)
│       ├── autodrive.c / autodrive.h  # 자율주행 로직: 초음파 3방향 기반 장애물 회피 상태머신
│       │
│       │  [사용자 구현 - 센서]
│       ├── ultrasonic.c / ultrasonic.h  # HC-SR04 초음파 센서 3개(좌/중/우) 논블로킹 거리 측정
│       ├── temp.c / temp.h         # NTC 온도센서 (ADC DMA, Beta 방정식, 50°C 경고 / 70°C 위험)
│       ├── gas.c / gas.h           # MQ135 가스센서 (ADC DMA, ppm 근사 계산, SAFE/WARNING/DANGER)
│       ├── ina219.c / ina219.h     # INA219 전류·전압 센서 (I2C3, 과전류 800mA 기준 감속)
│       │
│       │  [사용자 구현 - 기타]
│       ├── ledbar.c / ledbar.h     # 74HC595 시프트 레지스터 기반 8비트 LED 바 제어
│       ├── delay.c / delay.h       # TIM11 기반 마이크로초(µs) 딜레이
│       │
│       │  [CubeMX 자동 생성 - 주변장치 초기화]
│       ├── adc.c / adc.h           # ADC1 초기화 (12비트, DMA, CH6=PA6 온도 / CH12=PC2 가스)
│       ├── dma.c / dma.h           # DMA2 초기화 (ADC1 연속 전송)
│       ├── gpio.c / gpio.h         # GPIO 초기화 (모터 방향 핀, 초음파 트리거 핀 등)
│       ├── i2c.c / i2c.h           # I2C3 초기화 (100kHz, PA8=SCL / PB4=SDA)
│       ├── tim.c / tim.h           # TIM 초기화 (TIM2=PWM, TIM3/4=IC, TIM11=µs 타이머)
│       └── usart.c / usart.h       # USART1(9600bps 블루투스) / USART2(115200bps 디버그)
├── Drivers/
│   ├── STM32F4xx_HAL_Driver/   # STM32 HAL 드라이버
│   └── CMSIS/                  # ARM CMSIS 코어
├── Debug/                      # 빌드 출력 파일
├── RC_CAR_R02.ioc              # STM32CubeMX 설정 파일
├── STM32F411RETX_FLASH.ld      # Flash 링커 스크립트
└── STM32F411RETX_RAM.ld        # RAM 링커 스크립트
```

## 제어 명령 (블루투스 UART)

| 명령 | 동작 |
|---|---|
| `F` | 전진 |
| `B` | 후진 |
| `L` | 좌회전 |
| `R` | 우회전 |
| `S` | 정지 |
| `A` | 자율 주행 모드 ON |
| `M` | 수동 제어 모드 전환 |

> 명령어는 `main.c` 내 UART 수신 콜백에서 파싱하며, 필요에 따라 수정 가능하다.

## 핀 설정 (STM32F411RETx 기준)

| 기능 | 핀 | 설명 |
|---|---|---|
| USART2 TX | PA2 | 블루투스 / 디버그 송신 |
| USART2 RX | PA3 | 블루투스 / 디버그 수신 |
| TIM3 CH1 | PA6 | 모터A PWM |
| TIM3 CH2 | PA7 | 모터B PWM |
| GPIO OUT | PB0, PB1 | 모터 방향 제어 |
| GPIO OUT | PB4, PB5 | 모터 방향 제어 |
| TIM2 CH1 | PA0 | 초음파 트리거 |
| TIM2 IC | PA1 | 초음파 에코 캡처 |

> 실제 핀 번호는 `.ioc` 파일을 STM32CubeMX로 열어 확인할 것.

## 빌드 및 플래시

### 환경

- **IDE**: STM32CubeIDE 1.x 이상
- **SDK**: STM32CubeF4 HAL 라이브러리
- **디버거**: ST-LINK V2

### CubeMX 설정 변경 시

`.ioc` 파일을 STM32CubeMX에서 열어 핀·클럭·주변장치 설정 변경 후 코드 재생성 가능하다.

## 라이선스

개인 프로젝트
