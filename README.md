# RC_CAR_R02

STM32F411 마이크로컨트롤러 기반의 자율 주행 DC 카 프로젝트. UART 통신 및 블루투스 컨트롤러를 통한 원격 제어를 지원

## 개요

- STM32CubeIDE 환경에서 개발한 임베디드 RC카 펌웨어 
- 블루투스 모듈(HC-05/HC-06)을 통해 스마트폰 또는 컨트롤러와 무선 통신하며, DC 모터 PWM 제어로 전진·후진·좌회전·우회전 동작을 수행한다.
- 자율 주행 모드에서는 센서 입력을 기반으로 장애물 회피 또는 라인 트래킹을 수행한다.

## 하드웨어 구성

| 구성 요소 | 모델 | 설명 |
|---|---|---|
| MCU | STM32F411RETx | ARM Cortex-M4, 100MHz |
| 모터 드라이버 | L298N | DC 모터 2채널 PWM 제어 |
| 블루투스 모듈 | HC-05 / HC-06 | UART 기반 무선 통신 |
| 거리 센서 | HC-SR04 (옵션) | 초음파 장애물 감지 |
| IR 센서 | IR 라인 센서 (옵션) | 라인 트래킹 |
| 전원 | Li-Po 7.4V / 배터리팩 | 모터 및 MCU 전원 공급 |

## 주요 기능

- **블루투스 제어**: UART2를 통한 HC-05/HC-06 연동, 스마트폰 앱으로 실시간 방향 제어
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
│   └── Src/                    # 소스 파일
│       ├── main.c              # 메인 애플리케이션 로직
│       ├── stm32f4xx_hal_msp.c # HAL MSP 초기화
│       └── stm32f4xx_it.c      # 인터럽트 핸들러
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
| TIM2 CH1 | PA0 | 초음파 트리거 (옵션) |
| TIM2 IC | PA1 | 초음파 에코 캡처 (옵션) |

> 실제 핀 번호는 `.ioc` 파일을 STM32CubeMX로 열어 확인할 것.

## 빌드 및 플래시

### 환경

- **IDE**: STM32CubeIDE 1.x 이상
- **SDK**: STM32CubeF4 HAL 라이브러리
- **디버거**: ST-LINK V2

### 빌드 방법

1. STM32CubeIDE에서 프로젝트 Import
   - `File → Import → Existing Projects into Workspace`
   - 이 레포 루트 디렉토리 선택
2. `Project → Build Project` (또는 `Ctrl+B`)
3. ST-LINK 연결 후 `Run → Debug` (또는 `F11`)로 플래시 및 디버그 실행

### CubeMX 설정 변경 시

`.ioc` 파일을 STM32CubeMX에서 열어 핀·클럭·주변장치 설정 변경 후 코드 재생성 가능하다.

## 블루투스 연결 방법

1. HC-05 모듈 기본 설정: 보드레이트 `9600bps`, 페어링 PIN `1234`
2. 스마트폰에서 HC-05 페어링
3. Bluetooth Serial 앱(예: Serial Bluetooth Terminal)으로 연결
4. 위 제어 명령 전송으로 RC카 동작 확인

## 라이선스

개인 프로젝트
