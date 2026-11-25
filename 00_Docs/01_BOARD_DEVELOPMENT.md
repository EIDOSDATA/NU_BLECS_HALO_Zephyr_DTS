# NU BLE CS HALO 보드 개발 매뉴얼

## 1. 보드 타깃과 소스 구조

HALO는 Zephyr의 `nrf54lm20dk`에 `nrf54lm20b/cpuapp/nu_ble_cs_halo` variant를 추가한다. 애플리케이션 코드와 보드 정의는 별도 저장소에 둘 수 있다. `BOARD_ROOT`에는 최상위 `boards` 폴더를 **포함하는 디렉터리**를 지정한다. [보드 정의 디렉터리](../boards/nucode/nu_ble_cs_halo/)

| 바꾸려는 항목 | 수정할 파일 | 확인할 결과 |
| --- | --- | --- |
| 타깃 등록·sysbuild runner 그룹 | `board.yml`, `Kconfig.nrf54lm20dk` | HALO qualifier와 `SOC_NRF54LM20B_CPUAPP` 선택 |
| CPU, 클록, 메모리, 콘솔 | `nrf54lm20dk_nrf54lm20b_cpuapp_nu_ble_cs_halo.dts` | model, chosen, USB 및 클록 노드 |
| 시리얼, USB 스택, 부팅 시 CDC ACM | 같은 접두어의 `_defconfig` | 최종 이미지의 `.config` |
| 보드 메타데이터 | 같은 접두어의 `.yaml` | identifier, Twister용 메모리 및 기능 목록 |
| 커넥터, 센서, LED, 버튼, ADC | `halo_peripherals.dtsi` | 버스·주소·극성·alias |
| UART·SPI·I²C 핀과 절전 시 상태 | `halo-pinctrl.dtsi` | 각 컨트롤러의 default 및 sleep PSEL |
| 부팅 안테나와 CS 안테나 순서 | `nu_ble_cs_halo_rf.dtsi` | V1·V2의 초기 출력과 `ant-gpios` 순서 |

최상위 DTS는 Nordic SoC의 `nrf54lm20b_cpuapp.dtsi`와 CPUAPP 파티션 정의를 포함한다. 여기에 HALO의 주변장치 DTSI와 RF DTSI를 포함하고, 주변장치 파일에서 pinctrl 파일을 불러온다. SDK를 바꾸면 SoC와 파티션, runner 변경이 HALO에도 영향을 준다. [DK 대비 변경 사항](02_NRF54LM20DK_DELTA.md)

## 2. 보드 검색과 NUS 빌드

NCS v3.4.2 도구 환경의 PowerShell에서 HALO 저장소 루트를 현재 디렉터리로 열고 실행한다. 다음 예제는 SDK에 포함된 NUS 샘플을 빌드한다.

```powershell
$BoardRoot = (Get-Location).Path
$Sdk = 'C:/ncs/v3.4.2'
$Target = 'nrf54lm20dk/nrf54lm20b/cpuapp/nu_ble_cs_halo'
$App = Join-Path $Sdk 'zephyr/samples/bluetooth/peripheral_nus'
$Build = Join-Path $env:TEMP 'halo-build/peripheral_nus'
Set-Location $Sdk

west boards --board-root "$BoardRoot" --board nrf54lm20dk -f '{qualifiers}'
west build --sysbuild -b $Target -s "$App" -d "$Build" -- "-DBOARD_ROOT=$BoardRoot"
```

첫 번째 명령 결과에 `nrf54lm20b/cpuapp/nu_ble_cs_halo`가 있어야 한다. 기존 사용자 NUS 프로젝트로 빌드할 때는 `$App`을 해당 프로젝트 경로로 바꾸고 전용 빌드 디렉터리를 선택한다. 보드나 overlay를 바꿔 기존 디렉터리를 재사용하려면 `west build -p always`로 다시 구성한다.

VS Code에서는 nRF Connect의 SDK를 v3.4.2로 선택한 뒤 빌드 구성에 전체 HALO 타깃명과 `-DBOARD_ROOT=<HALO 저장소 경로>`를 지정한다. 보드 목록에 타깃이 보이지 않으면 위 CLI 검색으로 SDK와 보드 루트를 먼저 확인한다.

## 3. 앱 설정과 overlay

보드에서 USB CDC ACM을 이미 선언하고 부팅 시 활성화한다. NUS 앱은 `printk()`로 J1에 출력할 수 있다. 스마트폰에 표시할 제품 이름은 앱의 `prj.conf`에서 바꾼다.

```conf
CONFIG_BT_DEVICE_NAME="HALO NUS"
```

특정 앱에서만 J2 UART20으로 콘솔을 보내려면 아래 내용을 그 앱의 `boards/nrf54lm20dk_nrf54lm20b_cpuapp_nu_ble_cs_halo.overlay`에 둔다. 로그를 볼 때에는 USB–UART 어댑터의 RX를 보드 TX에, GND를 공통으로 연결하고 115200 bps를 선택한다. J2의 물리 핀 번호는 [핀맵](03_PINOUT.md)에 있다.

```dts
/ {
	chosen {
		zephyr,console = &uart20;
		zephyr,shell-uart = &uart20;
	};
};
```

J2만 사용하는 앱에서는 `CONFIG_CDC_ACM_SERIAL_INITIALIZE_AT_BOOT=n`을 함께 적용해 USB 콘솔의 자동 초기화를 해제한다. 기본 J1 구성을 복원할 때는 앱의 두 변경을 함께 제거한다. 기존 DK용 overlay를 이식할 때는 특히 UART20 RX/RTS가 RF 스위치의 P1.17/P1.18을 점유하는지 확인한다.

## 4. 생성 결과 확인

예제의 sysbuild에서 애플리케이션 이미지 이름은 `peripheral_nus`다. 다른 앱은 빌드 디렉터리의 `domains.yaml`에서 이미지 이름을 확인한다.

```powershell
$Image = Join-Path $Build 'peripheral_nus'
$Z = Join-Path $Image 'zephyr'
Get-Content "$Build/domains.yaml"
Select-String -Path "$Image/build_info.yml" -Pattern 'qualifiers|nu_ble_cs_halo'
Select-String -Path "$Z/zephyr.dts" -Pattern 'NU BLE CS HALO|zephyr,console|rfsw-v[12]-boot|ant-gpios'
Select-String -Path "$Z/.config" -Pattern '^CONFIG_(BOARD_TARGET|SOC_NRF54LM20B_CPUAPP|GPIO_HOGS|CDC_ACM_SERIAL_ENABLE_AT_BOOT|BT_DEVICE_NAME)='
Get-Content "$Z/runners.yaml"
```

| 확인 위치 | HALO NUS 기본 구성의 기대값 |
| --- | --- |
| `build_info.yml` | 보드 경로에 SDK의 Nordic 보드와 외부 HALO 디렉터리 모두 포함 |
| `zephyr.dts` | model `NU BLE CS HALO`, console `&cdc_acm_uart0`, 부모 컨트롤러 `usbhs` |
| RF GPIO hog | GPIO1 17번 `output-high`, 18번 `output-low`로 ANTA 선택 |
| `.config` | B형 CPUAPP, GPIO hog, 부팅 시 CDC ACM 활성. SDK 기본 NUS의 BLE 이름은 `Zephyr` |
| `runners.yaml` | 기본 flash `nrfutil`, 기본 debug `jlink`. SDK가 이 B형 보드에 사용하는 J-Link 장치 문자열은 `nRF54LM20A_M33` |

생성 결과와 다르면 먼저 빌드 로그의 `Found BOARD.dts`, 적용된 overlay 목록, Kconfig 병합 파일을 확인한다. 수정할 위치는 보드 소스나 앱 설정이다.

## 5. 외부 J-Link와 실물 검증

J4에 외부 J-Link를 연결한다. 플래싱과 디버깅에 같은 빌드 디렉터리를 사용한다.

```powershell
west flash -d "$Build" -r jlink
west debug -d "$Build" -r jlink
```

위 두 명령은 각각 펌웨어 기록과 디버그 세션 시작 용도다. 플래싱 후 아래 순서로 J1과 무선 경로를 개별 확인한다.

1. **USB 열거** — PC와 J1을 USB 데이터 케이블로 연결한다. Windows에서 새 가상 COM 포트와 USB 제품 문자열을 확인한다. J-Link의 COM 포트도 함께 표시될 수 있다.
2. **앱 실행** — J1 COM 포트를 열고 NUS 샘플의 주기적 `Data send - Result` 로그를 확인한다. PC에서 포트를 늦게 열면 초기 부트 로그를 놓칠 수 있다.
3. **BLE 광고** — nRF Connect for Mobile의 Scanner에서 이미지의 `CONFIG_BT_DEVICE_NAME`을 찾는다. J1 열거와 BLE 광고는 서로 다른 확인 항목이다.
4. **NUS 수신** — 연결 후 NUS RX 특성에 `HALO`를 쓴다. J1에서 같은 문자열을 확인한다. 반대 방향은 NUS TX 알림을 구독해 주기적인 `Hello World!`를 확인한다.
5. **RF 진단** — 광고 시작이 성공했는데 스캐너에 장치가 보이지 않으면 U3의 V1/V2에서 부팅 기본 상태를 측정한다. 측정 지점과 경로 선택표는 [하드웨어 검증 노트](04_HARDWARE_NOTES.md)에 있다.

## 6. 샘플별 검증 범위

| NCS v3.4.2의 샘플 경로 | HALO 구성에서 확인한 결과 | 실물에서 판정할 항목 |
| --- | --- | --- |
| `zephyr/samples/bluetooth/peripheral_nus` | sysbuild 링크 및 이미지 생성, USB 콘솔과 RF 핀 매핑 | J1 COM 열거, 광고 수신, NUS RX/TX |
| `nrf/samples/bluetooth/channel_sounding/ipt_initiator` | 단일 안테나 기본 구성으로 이미지 생성 | 호환 reflector 연결, CS 절차와 거리 출력 |
| `zephyr/samples/sensor/accel_polling` | `CONFIG_ADC=y` 추가 구성으로 이미지 생성 | IMU 축별 측정값, 알려진 입력 전압의 ADC 변환값 |

IPT 기본 검증 이미지에서 `CONFIG_BT_CTLR_SDC_CS_NUM_ANTENNAS=1`이다. 2안테나 시험은 별도 앱 설정으로 진행하고 V1/V2 전환과 실제 CS 결과를 함께 기록한다.

## 7. 보드 변경과 SDK 갱신

- **배선 변경** — 회로도 net을 SoC 포트 및 물리 커넥터 번호와 대조한다. pinctrl의 default/sleep과 버스의 CS·IRQ·극성을 함께 바꾸고 핀맵을 갱신한다.
- **장치 추가** — SDK binding과 드라이버 지원 여부를 확인한다. I²C 주소와 SPI chip select는 같은 버스의 기존 센서와 대조한다. MS5637의 현재 상태와 SB1 조건은 하드웨어 노트에 기록한다.
- **SDK 갱신** — 새 SDK의 `boards/nordic/nrf54lm20dk`, CPUAPP 파티션, USB 스택 설정 및 CS 안테나 binding을 비교한다. 세 샘플을 새 빌드 디렉터리에 빌드한 뒤 이전 `zephyr.dts`, `.config`, `runners.yaml`과 비교한다.
- **다중 이미지** — HALO의 `board.yml`은 우선순위 20으로 HALO 타깃의 1회 실행 그룹을 선언한다. 여러 보드 타깃을 함께 플래싱하는 구성을 추가하면 그룹 적용 범위와 마지막 리셋 순서를 별도로 검증한다.
