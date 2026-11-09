# 🐟 붕어빵 굽는 로봇팔 (Doosan Robotics + ROS 2)

두산 로봇팔로 붕어빵을 만드는 전체 과정(반죽 붓기 → 속재료 넣기 → 반죽 덮기 → 굽기 → 꺼내기)을 자동화하는 미니 프로젝트입니다.
손님이 오면 주문(팥 / 슈크림)을 받고, 주문대로 로봇팔이 붕어빵을 만듭니다.

> 발표: 2026-11-14 / 팀: 이대형, 권해림, 송승은, 지수진

## 현재 상태

- 로봇팔 동작은 **RViz2(3D 화면) 위의 가상 두산 m1013 로봇팔**로 시뮬레이션합니다. [확인 필요: 실제 e0509 로봇팔 연동 범위]
- 그리퍼(손) 제어는 아직 실제 명령 대신 화면 표시와 대기 시간으로 대체되어 있습니다. [확인 필요]
- 시연이 빠르도록 대기 시간을 0.1배로 줄여 두었습니다. (굽기 180초 → 약 18초)

## 만드는 순서 (동작 흐름)

1. 손님이 오면 감지합니다. (웹캠 얼굴 인식, 또는 키보드 Enter로 대체)
2. 주문을 받습니다. (음성 또는 키보드: 팥 / 슈크림, 틀 3개 분량)
3. 홈 자세에서 출발합니다.
4. 주전자로 1차 반죽을 틀 3개에 붓습니다.
5. 주문에 따라 팥 / 슈크림 소스통을 집어 속재료를 짜 넣습니다.
6. 2차 반죽으로 덮습니다.
7. 뚜껑을 닫고 굽습니다. 다 구워지면 뚜껑을 엽니다.
8. 붕어빵 3개를 접시에 옮기고 홈 자세로 돌아옵니다.

## 구성 (파일별 역할)

| 파일 | 역할 |
|---|---|
| `bungeoppang/pour_demo.py` | 메인 데모. 주문을 받아 위 1~8번 과정을 로봇팔로 실행 |
| `bungeoppang/customer_node.py` | 손님 감지. 도착하면 `arrived`, 떠나면 `left` 전달 (웹캠 / 키보드 모드) |
| `bungeoppang/voice_order_node.py` | 주문 받기 (음성 / 키보드) [확인 필요] |
| `bungeoppang/hud_node.py` | 현재 상태를 보여 주는 현황판 [확인 필요] |
| `bungeoppang/gripper_marker.py` | 화면에 그리퍼 모양과 소스통 색 표시 [확인 필요] |
| `bungeoppang/reach_check.py` | 로봇팔이 닿는 위치인지 점검 [확인 필요] |
| `bungeoppang/layout.py`, `geom.py` | 틀, 주전자 등의 위치와 계산 도구 [확인 필요] |
| `bungeoppang/my_node.py` | 데모의 다른 버전 [확인 필요] |
| `bungeoppang/pour_e0509.py` | 실제 e0509용 버전으로 추정 [확인 필요] |
| `bungeoppang/pour_demo_success_backup.py` | 성공했던 시점의 백업본 [확인 필요: 정리 또는 삭제 여부] |
| `start_all.sh` | 필요한 창을 한 번에 모두 켜는 실행 스크립트 |
| `run_demo.sh` | 로봇이 준비될 때까지 기다린 뒤 데모 실행 |
| `clean.sh`, `backup.sh`, `run_my.sh` | 정리 / 백업 / my_node 실행 [확인 필요] |

## 준비물

- Ubuntu + ROS 2 (Jazzy 기준) [확인 필요: 정확한 버전]
- MoveIt 2, pymoveit2
- 두산 로봇 ROS 2 패키지 (공식 문서: https://doosanrobotics.github.io/doosan-robotics-ros-manual/jazzy/)
- 웹캠 얼굴 인식을 쓰려면 OpenCV, MediaPipe
- 작업 폴더: `~/ros2_ws` (이 저장소는 `~/ros2_ws/src/` 안에 둡니다)

## 실행 방법

```bash
cd ~/ros2_ws
colcon build --packages-select bungeoppang
source install/setup.bash

# 한 번에 모두 실행 (창 5개가 열림)
./start_all.sh            # 기본: pour_demo, 키보드 입력 테스트 모드
```

- `test`(기본): 손님 도착과 주문을 키보드로 입력
- `voice`: 주문은 마이크, 손님 도착은 키보드
- `real`: 마이크와 카메라 모두 사용
- 로봇 창에 "손님 대기" 메시지가 나오면, 손님 감지 창에서 **Enter**를 누르면 시작됩니다.
- 모드 지정 방법 (실행 시 옵션 순서) [확인 필요]

## 노드 간 연결 (ROS 2 토픽)

| 이름 | 보내는 쪽 → 받는 쪽 | 내용 |
|---|---|---|
| `/bungeoppang/customer` | customer → 데모 | `arrived` / `left` |
| `/bungeoppang/order` | voice_order → 데모 | 주문 내용 (JSON, `fillings`) |
| `/bungeoppang/gripper` | 데모 → 화면 | 그리퍼 / 소스통 표시 |

## 팀 역할

| 이름 | 담당 |
|---|---|
| 이대형 | 로봇 연결 [확인 필요] |
| 권해림 | 주문 시스템, 화면 [확인 필요] |
| 송승은 | 붕어빵 틀 모형 [확인 필요] |
| 지수진 | 로봇 동작 [확인 필요] |

## 알려진 한계 / 앞으로 할 일

- 실제 그리퍼 제어는 미완성
- 연기 감지는 미구현 [확인 필요]
- 좌표가 코드 안에 들어 있어서, 자리가 바뀌면 코드 수정이 필요함
