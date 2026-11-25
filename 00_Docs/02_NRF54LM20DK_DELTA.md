# nRF54LM20 DK 대비 NU BLE CS HALO 보드 정의 차이

비교 기준은 nRF Connect SDK v3.4.2의 `nrf54lm20dk/nrf54lm20b/cpuapp`과 이 저장소의 `nrf54lm20dk/nrf54lm20b/cpuapp/nu_ble_cs_halo`다. 두 타깃은 같은 nRF54LM20B 애플리케이션 코어를 사용한다. HALO에서 달라지는 부분은 보드 배선, 연결 장치, 부팅 시 활성화할 기능이다. [Nordic 기준 보드 소스](https://github.com/nrfconnect/sdk-zephyr/tree/ncs-v3.4.2/boards/nordic/nrf54lm20dk) · [HALO 회로도](../NU_BLE_CS_HALO_Schematic%20PDF.pdf)

## 보드 선택과 파일 대응

`board.yml`은 Nordic 보드에 HALO variant를 등록한다. HALO의 최상위 DTS는 Nordic SoC 파일과 CPUAPP 파티션 파일을 직접 포함하고, HALO 전용 DTSI에서 핀과 장치를 받는다. SDK 업그레이드 때는 아래 대응 관계로 변경 사항을 추적할 수 있다.

| 기능 | NCS의 nRF54LM20 DK | HALO |
| --- | --- | --- |
| 타깃 등록 및 SoC 선택 | `board.yml`, `Kconfig.nrf54lm20dk` | [variant 등록](../boards/nucode/nu_ble_cs_halo/board.yml), [nRF54LM20B CPUAPP 선택](../boards/nucode/nu_ble_cs_halo/Kconfig.nrf54lm20dk) |
| SoC·메모리·클록 | `nrf54lm20dk_nrf54lm20b_cpuapp.dts`, `nrf54lm20_a_b_cpuapp_common.dtsi` | [최상위 DTS](../boards/nucode/nu_ble_cs_halo/nrf54lm20dk_nrf54lm20b_cpuapp_nu_ble_cs_halo.dts) |
| 주변장치·커넥터 | `nrf54lm20dk_common.dtsi`, CPUAPP 공통 DTSI | [HALO 주변장치](../boards/nucode/nu_ble_cs_halo/halo_peripherals.dtsi), [안테나 스위치](../boards/nucode/nu_ble_cs_halo/nu_ble_cs_halo_rf.dtsi) |
| pinctrl | `nrf54lm20dk_nrf54lm20_a_b-pinctrl.dtsi` | [HALO UART·SPI·I²C 핀](../boards/nucode/nu_ble_cs_halo/halo-pinctrl.dtsi) |
| 기본 드라이버와 테스트 메타데이터 | CPUAPP용 `_defconfig`, `.yaml` | [기본 설정](../boards/nucode/nu_ble_cs_halo/nrf54lm20dk_nrf54lm20b_cpuapp_nu_ble_cs_halo_defconfig), [Twister 메타데이터](../boards/nucode/nu_ble_cs_halo/nrf54lm20dk_nrf54lm20b_cpuapp_nu_ble_cs_halo.yaml) |

## USB, UART와 안테나 경로

| 항목 | nRF54LM20 DK | NU BLE CS HALO |
| --- | --- | --- |
| 기본 콘솔 및 shell 대상 | `uart20` | J1의 네이티브 USB CDC ACM `cdc_acm_uart0` |
| USB 초기화 | USBHS, USB regulator 및 BC1.2 노드 활성 | 같은 노드를 활성화하고 보드 기본값으로 CDC ACM을 부팅 시 시작. USB 제품 문자열 `NU BLE CS HALO` |
| UART20 | TX P1.16, RX P1.17, RTS P1.18, CTS P1.19에 pinctrl 지정 | J2 전용 TX P1.10, RX P1.11, 115200 bps, 2선 |
| 별도 UART 서비스 | MCU Manager, Bluetooth monitor, controller-to-host 인터페이스를 `uart20`으로 지정 | 이 세 서비스가 필요한 앱은 사용할 인터페이스를 overlay에서 선택 |
| UART30 | P0.06~P0.09에 UART pinctrl 제공 | P0.08은 SW1, P0.09는 IMU INT1에 연결. HALO는 UART30 핀 그룹을 제공하지 않음 |
| RF 스위치 | HALO U3 제어 노드 없음 | SKY13370-374LF의 V1=P1.17, V2=P1.18. 부팅 시 V1=High, V2=Low로 RF1→ANTA 선택 |

DK의 UART20 RX/RTS와 HALO의 RF 제어 신호는 같은 두 핀을 사용한다. DK 타깃에서 콘솔만 USB로 옮긴 구성에는 기존 UART20 pinctrl이 남을 수 있다. HALO variant는 J2 UART 배선과 RF 초기값을 함께 적용한다. 두 안테나를 사용하는 채널 사운딩에서는 컨트롤러가 `cs_antenna_switch`의 `ant-gpios` 순서대로 ANTA, ANTB를 선택할 수 있다. 애플리케이션의 안테나 수와 경로 수 설정은 별도로 적용한다. [스위치 제어 조합과 하드웨어 확인](04_HARDWARE_NOTES.md)

## 주변장치와 핀

| 항목 | nRF54LM20 DK | NU BLE CS HALO |
| --- | --- | --- |
| LED | 녹색 LED 4개, P1.22/25/27/28. PWM LED 출력은 P1.25 | RGB D1의 R/G/B가 P1.22/23/24에 연결. `led0`~`led2` GPIO alias 제공 |
| 버튼 | `sw0`~`sw3`에 P1.26, P1.09, P1.08, P0.05 할당 | `sw0`은 P0.08의 SW1. SW2는 전용 RESET. P0.05는 전원 선택 상태 `IDEAL_ST` |
| SPI00 | SCK P2.01, MOSI P2.02, MISO P2.04, CS P2.05. MX25R64 flash 자식 노드는 기본 `disabled` | 같은 네 핀으로 U4 ICM-42670-P에 연결. `accel0`, `gyro0` alias와 INT1 P0.09 사용 |
| SERIAL22 | 확장용 SPI22 핀 그룹이 P3.00~03에 걸쳐 있음 | I²C22의 SDA P3.02, SCL P3.03을 J3와 U2에 연결. U2 MS5637의 주소는 `0x76`, 센서 노드는 기본 `disabled` |
| 확장 헤더와 ADC | Nordic expansion header의 GPIO 인덱스. ADC 컨트롤러 활성 | 2×5 패드 P1/P2에 실제 패드 번호 기반 GPIO 매핑. P1 아날로그 입력 8개에 ADC 채널 0~7 제공 |
| 기타 기본 장치 | NFCT, 온도 센서, IEEE 802.15.4 노드 활성. PWM LED 및 watchdog alias 제공 | 해당 세 SoC 노드는 기본 `disabled`. 앱에 노출하는 alias 목록은 HALO 주변장치 DTSI에서 관리 |

HALO에 DK용 SPI flash overlay를 적용하면 IMU와 flash가 같은 SPI00의 CS에 배정된다. 또한 DK용 SPI22 overlay의 SCK/CS는 HALO I²C22의 SCL/SDA와 겹치며 두 기능이 SERIAL22 하드웨어를 공유한다. 실드나 샘플을 이식할 때는 먼저 대상 장치가 어느 커넥터에 연결되는지 확인한다. [HALO 핀 배치](03_PINOUT.md)

## 메모리, 클록과 전원

두 보드는 CPUAPP SRAM, RRAM, `slot0_partition`, 128 MHz HFPLL, GPIO0~3, GPIOTE20/30과 GRTC 채널 배분을 SDK 기준으로 구성한다. HALO의 YAML에는 RAM 511 KiB, flash 968 KiB가 기재돼 있다. 비교 대상 DK YAML의 flash 값은 920 KiB다. 이는 Twister의 메타데이터이며 실제 RRAM 파티션은 두 보드 모두 SDK의 `nrf54lm20b_cpuapp_partition.dtsi`를 통해 결정된다.

| 항목 | nRF54LM20 DK | NU BLE CS HALO의 근거 |
| --- | --- | --- |
| LFXO 내부 커패시터 | 17 pF | Y1은 32.768 kHz, 부하 용량 9 pF. 핀당 1 pF 기생 용량을 가정해 초기 내부 설정값을 17 pF로 계산 |
| HFXO 내부 커패시터 | 15 pF | X1 모듈 내부 고주파 크리스털 회로는 메인보드 도면 범위 밖이므로 DK의 15 pF를 초기값으로 사용 |
| VREGMAIN | DCDC 모드를 보드에서 지정 | nRF54LM20B SoC DTS의 LDO 초기 모드 사용. VREGUSB는 양쪽 모두 활성 |

## NUS 프로젝트를 옮길 때

1. 빌드 설정의 `BOARD_ROOT`를 이 저장소로 지정하고, `nrf54lm20dk/nrf54lm20b/cpuapp/nu_ble_cs_halo`로 새 빌드 디렉터리를 만든다. 기존 DK용 overlay를 `DTC_OVERLAY_FILE`에 고정했다면 그 안의 콘솔, UART, 안테나 핀 설정을 검토한다. [빌드 명령](../README.md)
2. 생성된 `<build>/<앱>/zephyr/zephyr.dts`에서 `model = "NU BLE CS HALO"`, `zephyr,console = &cdc_acm_uart0`, `rfsw-v1-boot` 및 `rfsw-v2-boot`를 확인한다. 최종 Kconfig에서는 `CONFIG_GPIO_HOGS=y`와 `CONFIG_CDC_ACM_SERIAL_ENABLE_AT_BOOT=y`를 확인한다.
3. HALO는 SDK의 runner를 재사용한다. 기본 flash runner는 `nrfutil`, debug runner는 `jlink`다. J4에 연결한 외부 J-Link는 `west flash -d <build> -r jlink`로 선택한다. 로그 수신에는 J1의 USB COM 포트를 사용한다.
4. 스마트폰 BLE 스캐너에 표시할 광고 이름은 NUS 앱의 `CONFIG_BT_DEVICE_NAME`에서 설정한다. HALO의 USB 제품 문자열과 BLE 이름은 각각 확인한다.

SDK 업그레이드 시에는 SoC 및 파티션 include, GRTC 채널 배분, USB 초기화 Kconfig, `nordic,bt-cs-antenna-switch` binding을 새 SDK 소스와 대조한다. 새로 빌드한 NUS 및 채널 사운딩 앱의 `zephyr.dts`에서 UART20과 RF 스위치가 서로 다른 핀을 사용하는지 다시 확인한다.
