# NU BLE CS HALO 하드웨어 확인 사항

이 문서는 nRF54LM20B용 HALO 보드 정의를 실물에서 처음 확인할 때 사용한다. 회로 위치는 **PDF 뷰어의 페이지 번호**로 표기한다. 5쪽과 6쪽은 각각 안테나 A와 B에 쓰인 같은 하위 시트의 인스턴스다. [보드 회로도](../NU_BLE_CS_HALO_Schematic%20PDF.pdf) · [커넥터 및 핀맵](03_PINOUT.md)

## 전원 경로와 상태 신호

| 전원 | 회로 경로 | 확인 지점 |
| --- | --- | --- |
| USB | J1 → VBUS_RAW → FB1 → VBUS → U5 TPS7A2033 → 3V3_SYS → U6 VIN1 | J1에 USB를 연결한 상태에서 VBUS, 3V3_SYS 순서로 측정 |
| 코인셀 | BT1 → VBAT → U6 VIN2 | BT1 양극과 GND 사이의 전압 측정 |
| 모듈 전원 | U6 LM66200 VOUT → VMCU → X1 VDD 및 센서 전원 | J4-1의 VMCU를 J4-3 GND 기준으로 측정 |

U6의 활성 로우 ON 핀은 GND에 연결되어 있다. U6이 선택한 입력은 ST 핀에서 읽는다. R13(10 kΩ)이 이 신호를 VMCU로 풀업하고 X1의 P0.05가 `IDEAL_ST`로 읽는다. 회로가 정상 동작할 때 High는 VIN1(3V3_SYS), Low는 VIN2(VBAT)다. USB 삽입 여부를 판별할 때는 X1의 VBUS도 확인한다. BT1 경로에는 충전 회로가 없다. [TI LM66200 데이터시트 §7.3.1](https://www.ti.com/lit/ds/symlink/lm66200.pdf)

## USB 콘솔과 SWD

J1의 VBUS는 FB1을 거쳐 X1의 VBUS 핀에도 공급된다. 데이터 경로는 J1의 D+/D− → U1 USBLC6-2P6 → X1의 HS_D+/HS_D−다. J4의 외부 J-Link는 SWD로 연결된다. Windows에서 프로브의 COM 포트만 표시될 때는 다음 순서로 확인한다.

1. 데이터 전송이 되는 케이블을 **HALO의 J1**에 연결한다. X1이 코인셀로 동작하거나 J-Link가 VMCU를 감지해도 J1의 VBUS와 데이터선은 별도로 확인해야 한다.
2. 해당 이미지의 `zephyr.dts`에서 `zephyr,console`이 `cdc_acm_uart0`를 가리키고 USB HS 컨트롤러가 활성인지 확인한다. `.config`에서는 `CONFIG_USB_DEVICE_STACK_NEXT=y`와 `CONFIG_CDC_ACM_SERIAL_ENABLE_AT_BOOT=y`를 확인한다. [빌드 및 생성 파일 확인](01_BOARD_DEVELOPMENT.md)
3. Windows 장치 관리자에서 J1을 뺐다 꽂을 때 사라졌다 다시 생기는 COM 포트를 확인한다. 보드 기본 USB 제품 문자열은 `NU BLE CS HALO`이며 포트 번호는 PC가 할당한다. 해당 포트에서 주기 로그를 확인하거나 NUS 연결 후 알려진 문자열을 보내 수신 로그를 확인한다.
4. 장치 인식 자체가 안 되면 J1의 CC1/CC2 풀다운 R1/R2(각 5.1 kΩ), FB1 뒤 VBUS, U1 앞뒤의 데이터선 연결을 회로도 1~2쪽과 대조한다. 전원이 꺼진 상태에서 연속성을 측정한다. [Nordic nRF54LM20B USB 핀 동작 조건](https://docs.nordicsemi.com/r/bundle/ps_nrf54lm20a/page/usbhs.html-concept_usb_pins)

J2의 UART20은 TX P1.10, RX P1.11, 기본 115200 bps다. UART 어댑터를 사용할 때는 VMCU 논리 레벨과 공통 GND를 맞추고 TX/RX를 교차 연결한다. J2에서 콘솔 로그를 받으려면 애플리케이션에서 콘솔 대상을 UART20으로 선택해야 한다.

## RF 스위치와 두 안테나

회로도 4~6쪽에서 모듈 RFIO는 C9를 거쳐 U3 SKY13370-374LF의 RFC로 들어간다. RF1은 ANTA, RF2는 ANTB로 이어진다. 각 경로에는 47 pF 직렬 C10, 0 Ω 직렬 R9, SR4W089 안테나가 있으며 병렬 C11/C12는 DNP로 표기되어 있다. 실제 RF 경로를 확인할 때는 시험 보드의 R9 실장과 매칭 부품 상태를 먼저 대조한다.

| 상태 | V1 (P1.17 / U3-1) | V2 (P1.18 / U3-3) | 연결 |
| --- | --- | --- | --- |
| 부팅 후 일반 BLE 또는 단일 안테나 CS | High | Low | RFC → RF1 → ANTA |
| ANTB를 선택하는 구간 | Low | High | RFC → RF2 → ANTB |

동일 레벨 조합은 제조사가 경로를 보장하지 않는다. GPIO hog가 ANTA를 설정하는 시점은 펌웨어 초기화 중이다. BLE 광고를 확인할 때는 초기화 이후 V1/V2를 측정하고 스마트폰의 BLE 스캐너에서 앱의 광고 이름을 찾는다. 2안테나 CS를 활성화한 이미지에서는 오실로스코프로 두 제어선을 함께 관측한다. 멀티미터의 평균 전압으로는 안테나 전환을 판정하기 어렵다. [RF DTSI](../boards/nucode/nu_ble_cs_halo/nu_ble_cs_halo_rf.dtsi) · [Skyworks 데이터시트 표 4](https://www.skyworksinc.com/-/media/skyworks/documents/products/501-600/sky13370_374lf_201408g.pdf)

기본 안테나에서 광고가 보이고 USB 콘솔이 열리면, NUS RX 특성에 `HALO` 4바이트를 써서 USB에 같은 텍스트가 나오는지 확인한다. 이 시험은 BLE 수신부터 USB 출력까지 확인한다. 거리값은 채널 사운딩 전용 펌웨어와 호환 Reflector를 사용해 별도로 확인한다.

## 클록과 모듈 내부 설정

- **32.768 kHz LFXO**. 회로도 2쪽 Y1은 X1의 XL1/P1.20과 XL2/P1.21에 연결되며 부품 표기는 부하 용량 9 pF이다. DTS의 내부 커패시터 17 pF는 핀당 기생 용량 1 pF를 가정한 초깃값이다. 보드 리비전 확정 시 주파수 측정 결과로 보정한다. [Nordic LFXO devicetree 가이드](https://docs.nordicsemi.com/r/bundle/ngl_001/page/gl/ngl_001/lfxo_devicetree.html)
- **32 MHz HFXO 및 VREGMAIN**. 메인보드 회로도는 X1 모듈의 내부 부품을 보여 주지 않는다. DTS의 HFXO 내부 15 pF는 SDK 기준 보드에서 가져온 초깃값이다. VREGMAIN은 SoC 기본 LDO 모드를 사용한다. HFXO 부하를 확정하거나 DC/DC로 변경하려면 해당 X1 리비전의 크리스털 사양과 전원 수동소자 실장을 확인한다. [보드 DTS](../boards/nucode/nu_ble_cs_halo/nrf54lm20dk_nrf54lm20b_cpuapp_nu_ble_cs_halo.dts)

## 센서 및 확장 패드

| 대상 | 보드에서 확인할 사항 |
| --- | --- |
| SPI IMU U4 ICM-42670-P | SPI00의 CS P2.05와 SCK P2.01에서 트랜잭션을 확인한다. INT1은 P0.09로 직결되고, INT2와 P1.14 사이는 SB1을 지난다. INT2를 사용하는 하드웨어에서는 SB1의 실장 상태를 기록한다. |
| I²C 압력 센서 U2 MS5637 및 J3 | SDA P3.02, SCL P3.03을 공유하며 R7/R8이 각각 2.1 kΩ으로 VMCU에 풀업한다. U2 주소는 7비트 `0x76`이다. I²C22는 활성이고 전용 센서 드라이버가 없는 NCS v3.4.2에서는 U2 노드를 비활성으로 둔다. J3 장치를 추가할 때 주소와 외부 풀업을 확인한다. |
| P1/P2 확장 패드 | 두 커넥터 모두 1번이 VMCU, 2번이 GND다. P1의 신호 패드를 ADC와 GPIO로 함께 사용할 때는 해당 핀을 구동하는 주변장치를 먼저 확인한다. [패드별 신호표](03_PINOUT.md) |

## 실물 시험 기록

NCS v3.4.2에서 HALO 타깃으로 `peripheral_nus`, `ipt_initiator`, `accel_polling` 빌드가 완료됐다. 아래 항목의 실물 결과는 아직 기록되지 않았다.

| 시험 | 기록할 결과 |
| --- | --- |
| USB 전원 및 콘솔 | 보드 리비전, J1 VBUS, VMCU, 인식된 COM 포트, NUS 문자열 수신 로그 |
| RF 및 채널 사운딩 | 광고 시 V1/V2 실측, 스캔한 BLE 이름, 연결한 Reflector, 거리 로그 및 사용 안테나 |
| 센서와 전원 상태 | IMU 측정값, INT2 사용 시 SB1 상태, U2 응답, 두 전원 조건에서 IDEAL_ST |
| 클록 | X1 모듈 사양, LFXO 보정값과 측정 근거, HFXO 부하 확인 결과 |
