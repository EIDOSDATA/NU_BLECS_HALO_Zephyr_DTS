# NU BLE CS HALO 핀 구성

대상은 `nrf54lm20dk/nrf54lm20b/cpuapp/nu_ble_cs_halo`다. 아래에서 `P1.10`은 nRF54LM20B의 GPIO 포트 1, 핀 10을 뜻하고 `P1-10`은 보드의 P1 확장 패드 10번을 뜻한다. J1~J4와 TP1은 회로도에 인쇄된 접점 번호를 따른다. [HALO 회로도, PDF 1~3쪽](<../NU_BLE_CS_HALO_Schematic PDF.pdf>)

## J1 USB-C와 콘솔

기본 콘솔과 셸은 J1에 연결된 MCU의 USB CDC ACM 포트(`cdc_acm_uart0`)를 사용한다. J1은 모듈 X1의 USB HS 차동 핀과 직접 통신한다. [보드 DTS](../boards/nucode/nu_ble_cs_halo/nrf54lm20dk_nrf54lm20b_cpuapp_nu_ble_cs_halo.dts)

| J1 접점 | 신호 | 보드에서의 연결 |
| --- | --- | --- |
| A6, B6 | D+ | 보호소자 U1의 1→6번 경로 → X1-G1 `HS_D+` |
| A7, B7 | D− | U1의 3→4번 경로 → X1-F1 `HS_D-` |
| A4, A9, B4, B9 | VBUS_RAW | FB1을 거쳐 `VBUS`로 연결. X1-H1은 VBUS, X1-J10은 VMCU |
| A5, B5 | CC1, CC2 | R1, R2가 각 선을 5.1 kΩ으로 GND에 연결 |
| A1, A12, B1, B12 | GND | 회로 접지 |
| A8, B8 | SBU1, SBU2 | 회로도상 미연결 |

## J2 UART와 J3 I²C

| 물리 핀 | J2 UART | J3 MODULO I²C |
| --- | --- | --- |
| 1 | GND | GND |
| 2 | VMCU | VMCU |
| 3 | `UART_TX` · P1.10 · 보드에서 송신 | `MODULO_SDA` · P3.02 |
| 4 | `UART_RX` · P1.11 · 보드에서 수신 | `MODULO_SCL` · P3.03 |

J2의 UART20은 115200 bps, TX/RX 2선으로 구성된다. USB–UART 어댑터로 사용할 때에는 J2-3을 어댑터 RX에, J2-4를 어댑터 TX에 연결하고 GND를 공유한다. J1 콘솔과 J2 UART는 펌웨어에서 각각 `cdc_acm_uart0`와 `uart20`으로 접근한다. [핀 설정](../boards/nucode/nu_ble_cs_halo/halo-pinctrl.dtsi)

J3과 기압 센서 U2(MS5637-02BA03)는 `i2c22`(라벨 `modulo_i2c`)를 공유한다. 기본 속도는 100 kHz이고 R8(SDA), R7(SCL)이 각각 2.1 kΩ으로 VMCU에 풀업한다. U2의 7비트 주소는 `0x76`이며 보드 정의에서 `ms5637` 자식 노드는 `disabled`다. [주변장치 DTSI](../boards/nucode/nu_ble_cs_halo/halo_peripherals.dtsi)

## J4와 TP1 디버그 커넥터

J4는 10핀 SWD 헤더이고 TP1은 TC2030-IDC-NL-FP 접점이다. 두 커넥터는 같은 타깃 신호를 서로 다른 핀 번호로 제공한다. VMCU는 디버거의 타깃 전압 기준이다. [HALO 회로도, PDF 1쪽 J4 및 2쪽 TP1](<../NU_BLE_CS_HALO_Schematic PDF.pdf>)

| 신호 | J4 핀 | TP1 핀 | 모듈 쪽 연결 |
| --- | --- | --- | --- |
| VMCU | 1 | 1 | X1의 VDD 전원 레일 |
| SWDIO | 2 | 2 | X1-A9 |
| SWCLK | 4 | 4 | X1-A8 |
| SWO | 6 | 6 | P2.07, X1-F10 |
| RESET | 10 | 3 | X1-A10, SW2를 누르면 Low |
| GND | 3, 5, 9 | 5 | 회로 접지 |
| 미연결 | 7, 8 | — | 회로도에 NC 표시 |

## P1·P2 확장 패드

아래 번호는 두 2×5 패드의 물리 핀 번호다. DTS의 GPIO nexus `p1_analog`와 `p2_digital`도 3~10을 그대로 사용한다. 예를 들어 `P1-7`은 AIN4이고 `P2-7`은 P0.06이다. [HALO 회로도, PDF 3쪽 Mounting layout](<../NU_BLE_CS_HALO_Schematic PDF.pdf>) · [GPIO 및 ADC 매핑](../boards/nucode/nu_ble_cs_halo/halo_peripherals.dtsi)

| 핀 | P1 아날로그 확장 | ADC 입력 | P2 디지털 확장 |
| --- | --- | --- | --- |
| 1 | VMCU | — | VMCU |
| 2 | GND | — | GND |
| 3 | P1.00 | AIN0 | P0.03 |
| 4 | P1.31 | AIN1 | P3.00 |
| 5 | P1.30 | AIN2 | P0.04 |
| 6 | P1.29 | AIN3 | P3.01 |
| 7 | P1.06 | AIN4 | P0.06 |
| 8 | P1.05 | AIN5 | P3.04 |
| 9 | P1.04 | AIN6 | P0.07 |
| 10 | P1.03 | AIN7 | P3.05 |

ADC 채널 0~7은 해당 번호의 AIN을 단일 종단 입력으로 사용한다. 기본 정의는 12비트, 내부 기준, 이득 1/4, 기본 획득 시간이다. 애플리케이션은 `zephyr,user`의 `io-channels`에서 채널을 가져올 수 있다.

## RGB LED, 버튼, 전원 상태 및 크리스털

| 회로 | 신호 및 핀 | 애플리케이션에서의 의미 |
| --- | --- | --- |
| D1 적색 | `RED` · P1.22 | `led0` / `led_red`. High에서 점등 |
| D1 녹색 | `GREEN` · P1.23 | `led1` / `led_green`. High에서 점등 |
| D1 청색 | `BLUE` · P1.24 | `led2` / `led_blue`. High에서 점등 |
| SW1 | P0.08 | `sw0` / `button0`. 내부 풀업, 누르면 Low |
| SW2 | 전용 RESET | SWD 커넥터와 같은 하드웨어 리셋 입력 |
| U6 LM66200 | `IDEAL_ST` · P0.05 | `zephyr,user`의 `ideal-st-gpios`. High는 3V3_SYS, Low는 VBAT가 선택된 상태 |
| Y1 | P1.20/XL1, P1.21/XL2 | 32.768 kHz 외부 크리스털의 두 단자 |

## SPI00 IMU

온보드 U4(ICM-42670-P)의 SPI 칩 선택 인덱스는 0이고 최대 클록 설정은 8 MHz다. Zephyr의 `accel0`와 `gyro0`가 같은 `imu` 노드를 가리킨다. [SPI 핀 설정](../boards/nucode/nu_ble_cs_halo/halo-pinctrl.dtsi) · [IMU 노드](../boards/nucode/nu_ble_cs_halo/halo_peripherals.dtsi)

| U4 신호 | MCU 핀 | 연결 및 동작 |
| --- | --- | --- |
| SCLK | P2.01 | MCU → U4, `IMU_SCLK` |
| MOSI | P2.02 | MCU → U4, `IMU_MOSI` |
| MISO | P2.04 | U4 → MCU, `IMU_MISO` |
| CS | P2.05 | MCU → U4, Low에서 선택 |
| INT1 | P0.09 | U4 → MCU. 드라이버의 `int-gpios`가 active high로 선언 |
| INT2 | P1.14 | U4 → SB1 → MCU. SB1을 닫았을 때 연결되며, `zephyr,user`의 `imu-int2-gpios`로 노출 |

## RF 안테나 선택

모듈 X1의 RFIO는 C9를 거쳐 U3(SKY13370-374LF)의 공통 포트 RFC에 연결된다. 부팅 시 DTS의 GPIO hog가 V1과 V2를 설정한다. HALO의 UART20은 J2의 P1.10/P1.11을 사용하므로, 두 안테나 제어 핀을 UART에 재할당하지 않는다. [RF 스위치 DTSI](../boards/nucode/nu_ble_cs_halo/nu_ble_cs_halo_rf.dtsi) · [HALO 회로도, PDF 4~6쪽](<../NU_BLE_CS_HALO_Schematic PDF.pdf>)

| 선택 경로 | RFSW_V1 · P1.17 · U3-1 | RFSW_V2 · P1.18 · U3-3 | RF 출력 |
| --- | --- | --- | --- |
| ANTA (부팅 기본값) | High | Low | U3-6 / RF1 → AFEA |
| ANTB | Low | High | U3-4 / RF2 → AFEB |

두 안테나를 사용하는 채널 사운딩에서는 컨트롤러가 `cs_antenna_switch`의 `ant-gpios` 순서로 포트를 선택한다. 핀이나 극성을 바꿀 때는 부팅용 GPIO hog와 컨트롤러용 배열을 함께 검토한다. 실장 조건과 측정 항목은 [하드웨어 노트](04_HARDWARE_NOTES.md)에 정리했다.
