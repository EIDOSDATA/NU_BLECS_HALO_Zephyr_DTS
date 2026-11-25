# NU BLE CS HALO 보드 문서

이 문서는 nRF54LM20B 기반 HALO를 빌드하는 펌웨어 개발자와 회로를 검증하는 보드 개발자가 함께 사용한다. 빌드 예제는 NCS v3.4.2 / Zephyr 4.4.2, application core(`cpuapp`) 기준이다.

| 하려는 작업 | 문서 |
| --- | --- |
| HALO 타깃으로 NUS를 빌드하고 USB 콘솔과 BLE 광고를 확인한다 | [보드 개발 매뉴얼](01_BOARD_DEVELOPMENT.md) |
| nRF54LM20 DK용 앱이나 overlay를 HALO로 옮긴다 | [nRF54LM20 DK 대비 변경 사항](02_NRF54LM20DK_DELTA.md) |
| 커넥터의 물리 핀, GPIO, 신호 방향과 극성을 찾는다 | [핀맵과 공유 핀](03_PINOUT.md) |
| USB가 잡히지 않거나 광고가 보이지 않을 때 측정할 위치를 찾는다 | [하드웨어 검증 노트](04_HARDWARE_NOTES.md) |

보드 타깃은 `nrf54lm20dk/nrf54lm20b/cpuapp/nu_ble_cs_halo`이다. 기본 콘솔은 J1 USB CDC ACM, 디버그 연결은 J4의 외부 SWD 프로브다. NUS, IPT Initiator, IMU 샘플은 이 타깃으로 빌드했다. USB 열거, BLE 무선 송수신, 센서 측정값은 실물 시험으로 확인할 항목이다.

## 자료와 변경 이력 관리

배선과 부품 번호는 [HALO 회로도](../NU_BLE_CS_HALO_Schematic%20PDF.pdf), 현재 소프트웨어 설정은 [보드 소스](../boards/nucode/nu_ble_cs_halo/)를 기준으로 한다. 보드 타깃명이나 빌드 방법을 변경하면 [저장소 사용 안내](../README.md)와 개발 매뉴얼을 함께 갱신한다. 핀 배치가 바뀌면 핀맵과 DK 차이표를, 실장 상태나 측정 결과가 확정되면 하드웨어 노트를 갱신한다.

문서 구성은 [NU54DK 기술 문서](https://github.com/Nucode01/NU54DK_Zephyr_DTS/tree/main/00_Docs)를 참고했다.
