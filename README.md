# NU BLE CS HALO Zephyr 보드 정의

NUCODE의 nRF54LM20B 기반 NU BLE CS HALO를 nRF Connect SDK v3.4.2에서 사용하는 외부 보드 정의입니다. NU54DK와 같은 `BOARD_ROOT` 방식으로 등록하며, 보드 타깃은 `nrf54lm20dk/nrf54lm20b/cpuapp/nu_ble_cs_halo`입니다. 회로도는 [NU BLE CS HALO Schematic](./NU_BLE_CS_HALO_Schematic%20PDF.pdf)을 참조하세요.

개발 절차, Nordic DK 대비 변경 사항, 커넥터 핀맵, 실물 검증 방법은 [HALO 보드 기술 문서](00_Docs/README.md)에 정리했습니다.

## 빌드와 플래싱

NCS v3.4.2 도구 환경의 PowerShell에서 실행합니다. 아래 예시는 기존 NUS 프로젝트를 별도 빌드 디렉터리에 빌드합니다.

```powershell
$Sdk = 'C:/ncs/v3.4.2'
$BoardRoot = 'C:/Users/eidos/GitHub/NU_BLECS_HALO_Zephyr_DTS'
$App = 'C:/Users/eidos/GitHub/peripheral_nus'
$Target = 'nrf54lm20dk/nrf54lm20b/cpuapp/nu_ble_cs_halo'
Set-Location $Sdk
west build --sysbuild -b $Target -s $App -d "$App/build-halo" -- "-DBOARD_ROOT=$BoardRoot"
west flash -d "$App/build-halo" -r jlink
```

VS Code의 nRF Connect 빌드 구성에서 위 타깃을 선택하고 Extra CMake arguments에 `-DBOARD_ROOT=C:/Users/eidos/GitHub/NU_BLECS_HALO_Zephyr_DTS`를 지정합니다. CLI에서 `west boards --board-root $BoardRoot --board nrf54lm20dk -f '{qualifiers}'`로 HALO variant를 확인할 수 있습니다. J4는 외부 프로브용 SWD 헤더입니다. 디버그 명령은 `west debug -d "$App/build-halo" -r jlink`입니다.

## USB 콘솔과 안테나

J1 USB-C의 D+/D−는 보호소자 U1을 거쳐 X1의 USB HS 핀에 연결됩니다. 보드 기본값으로 CDC ACM을 부팅 시 활성화하며, USB 제품 문자열은 `NU BLE CS HALO`입니다. J1의 가상 COM 포트를 열면 콘솔 로그를 볼 수 있습니다. BLE 광고 이름은 애플리케이션의 `CONFIG_BT_DEVICE_NAME`으로 정합니다. J2에서는 별도로 UART20을 사용할 수 있습니다.

RFIO는 U3의 RFC에 연결됩니다. 부팅 시 GPIO hog가 아래의 ANTA 조합을 설정하므로 일반 BLE 광고와 단일 안테나 채널 사운딩에 해당 경로를 사용합니다. 2안테나 CS를 사용하는 애플리케이션은 SoftDevice Controller 설정에서 안테나 수와 지원 경로 수를 활성화해야 합니다. 보드의 `cs_antenna_switch` 노드는 두 포트를 컨트롤러에 같은 순서로 제공합니다. [SKY13370-374LF 데이터시트, 표 4](https://www.skyworksinc.com/-/media/skyworks/documents/products/501-600/sky13370_374lf_201408g.pdf)

| RF 경로 | V1 · P1.17 | V2 · P1.18 |
| --- | --- | --- |
| RF1 → ANTA (부팅 기본값) | High | Low |
| RF2 → ANTB | Low | High |

## 주변장치와 핀

| 기능 | 회로 연결 | Zephyr 설정 |
| --- | --- | --- |
| RGB LED D1 | R P1.22, G P1.23, B P1.24 | `led0`, `led1`, `led2`, active high |
| 버튼 | SW1 P0.08, SW2 하드웨어 RESET | `sw0`, 내부 풀업, active low |
| J2 UART | TX P1.10, RX P1.11 | UART20, 115200, TX/RX 2선 |
| U4 ICM-42670-P | SCK P2.01, MOSI P2.02, MISO P2.04, CS P2.05 | SPI00, `accel0` / `gyro0` |
| IMU 인터럽트 | INT1 P0.09, INT2 P1.14 | 드라이버는 INT1 사용. INT2 연결은 SB1 실장 상태에 따름 |
| J3 및 U2 MS5637-02BA03 | SDA P3.02, SCL P3.03 | I²C22, 100 kHz. U2의 7비트 주소는 `0x76`. NCS v3.4.2에는 전용 센서 드라이버가 없어 센서 노드는 기본 비활성 |
| P1 아날로그 확장 | 3~10번 패드가 순서대로 AIN0~AIN7 | `p1_analog`, `zephyr,user`의 `io-channels` |
| P2 디지털 확장 | 3~10번 패드 순으로 P0.03, P3.00, P0.04, P3.01, P0.06, P3.04, P0.07, P3.05 | `p2_digital`. 두 확장 헤더 모두 물리 패드 번호 사용 |
| 전원 경로 U6 LM66200 | `IDEAL_ST` P0.05 | `zephyr,user`의 `ideal-st-gpios`. High는 3V3_SYS, Low는 VBAT |

## 클록과 전원 설정

Y1은 P1.20/XL1과 P1.21/XL2에 연결된 32.768 kHz, 부하 용량 9 pF의 크리스털입니다. 초기 내부 커패시터 값은 핀당 기생 용량을 1 pF로 가정한 17 pF입니다. 최종값은 실물의 주파수 측정으로 보정합니다. [Nordic LFXO devicetree 가이드](https://docs.nordicsemi.com/r/bundle/ngl_001/page/gl/ngl_001/lfxo_devicetree.html)

X1 모듈 내부의 32 MHz 크리스털과 부하 회로는 메인보드 회로도에 표시되지 않습니다. HFXO는 초기 검증용으로 Nordic DK의 내부 15 pF 설정을 사용합니다. 양산 설정은 모듈 사양 또는 측정 결과와 대조해야 합니다. VREGMAIN은 SoC 기본 LDO 모드를 사용합니다.

## 출처

보드 정의 구조는 [NU54DK Zephyr DTS](https://github.com/Nucode01/NU54DK_Zephyr_DTS)를 따릅니다. nRF54LM20B CPUAPP SoC와 RRAM 파티션은 NCS v3.4.2에 포함된 Zephyr 정의를 사용합니다. 보드 소스의 라이선스는 [Apache-2.0](LICENSES/Apache-2.0.txt)입니다.
