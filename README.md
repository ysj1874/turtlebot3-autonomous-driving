# 🤖 TurtleBot3 Autonomous Driving System
### UKF + LiDAR + DNN + NMPC 기반 지능형 자율주행

> **Graduation Project** — ROS1 기반 자율주행 시스템  
> 비선형 상태추정 · 센서 융합 · 위험도 예측 · 최적 제어의 완전 통합 구현

---

## 📌 프로젝트 개요

TurtleBot3 (Waffle Pi)를 플랫폼으로, **UKF 상태추정 → LiDAR 인지 → DNN 위험도 예측 → NMPC 최적 제어**의 파이프라인을 설계·구현한 자율주행 시스템입니다.

| 항목 | 내용 |
|------|------|
| 플랫폼 | TurtleBot3 Waffle Pi |
| 환경 | ROS1 (Melodic / Noetic) |
| 언어 | Python · C++ |
| 핵심 알고리즘 | UKF · DNN (PyTorch/Keras) · NMPC |

---

## 🏗️ 시스템 아키텍처

```
[Sensors]
  LiDAR │ IMU │ Odometry
        │
        ▼
[UKF State Estimator]          ← IMU + Odom + LiDAR 센서 융합
  → (x, y, yaw, velocity)
        │
        ▼
[DNN Risk Predictor]           ← 최근 N초 상태 히스토리 입력
  → Risk Score (0 ~ 1)
        │
        ▼
[NMPC Controller]              ← 경로 오차 + 위험도 기반 최적화
  → Optimal (v, ω)
        │
        ▼
[TurtleBot3 Motion Execution]
```

---

## 🚀 개발 마일스톤

### Milestone 1 — UKF 상태 추정기 구축
> **IMU + Odometry + LiDAR 데이터를 융합하는 비선형 칼만 필터 구현**

- [x] 비선형 차량 운동 모델 (Motion Model) 설계
- [x] IMU / Odometry 측정 모델 정의
- [x] LiDAR 스파이크 제거 및 안정적 거리 추정
- [x] Sigma Point 생성 및 UKF 예측·업데이트 루프 완성
- [x] Raw LiDAR 대비 변동성 **유의미하게 감소** 실험 확인

```
📈 결과: Raw LiDAR(0~1.2m 스파이크) → UKF 필터 후 안정적 곡선 유지
```

---

### Milestone 2 — LiDAR 기반 환경 인지
> **노이즈 제거 및 위험 Feature 생성 모듈 구현**

- [x] `/scan` 토픽 전처리 파이프라인 구축 (C++)
- [x] 이상값(스파이크) 필터링 알고리즘 적용
- [x] 근접 장애물 판단 로직 설계
- [x] DNN 입력용 위험도 Feature 벡터 생성

---

### Milestone 3 — DNN 위험도 예측 모델
> **시계열 상태 히스토리를 입력받아 위험도를 0~1로 예측하는 신경망 학습**

- [x] 학습 데이터셋 수집 및 전처리
- [x] 시계열 입력 기반 DNN 모델 설계 (PyTorch / Keras)
- [x] Risk Score (0~1) 출력 레이어 구성
- [x] 모델 학습 및 검증 (`train_risk_model.ipynb`)
- [x] ROS1 노드로 실시간 추론 파이프라인 연결

---

### Milestone 4 — NMPC 최적 제어기
> **경로 추종 + 위험도 가중치 기반 비선형 모델 예측 제어 구현**

- [x] 비용 함수(Cost Function) 설계: 경로 오차 · 속도 변화 · 위험도 가중치
- [x] 제어 제약 조건(속도/조향 한계) 정의
- [x] DNN Risk Score를 NMPC 비용항에 실시간 반영
- [x] 최적화 솔버 연동 (`nmpc_solver.py`)
- [x] Raw 명령 대비 제어 입력 **부드러움 대폭 향상** 확인

```
📈 결과: cmd_vel_raw(급격한 진동) → UKF+NMPC(부드러운 기울기 변화)
```

---

### Milestone 5 — ROS1 전체 시스템 통합
> **모든 모듈을 실시간으로 연결하는 통합 launch 시스템 제작**

- [x] 모듈별 ROS 노드 분리 설계
- [x] `ukf.launch` · `nmpc.launch` · `full_system.launch` 제작
- [x] 토픽 연결 및 메시지 흐름 검증
- [x] DNN + NMPC 조합 연산 시간 **약 23% 단축** 확인
- [x] 실시간 자율주행 실험 및 결과 분석 완료

---

## 📊 실험 결과 요약

| 지표 | 개선 전 | 개선 후 |
|------|---------|---------|
| LiDAR 거리 변동성 | 스파이크 빈번 (0~1.2m) | 안정적 곡선 유지 |
| 제어 입력 진동 | `/cmd_vel_raw` 급격한 진동 | UKF+NMPC 부드러운 전환 |
| 연산 시간 | NMPC 단독 기준 | DNN+NMPC 조합 **~23% 단축** |
| 장애물 인지 신뢰도 | Raw LiDAR 의존 | UKF 융합으로 향상 |

---

## 🗂️ 디렉터리 구조

```
turtlebot-autonomous-driving/
├── ukf/
│   ├── ukf_node.py            # UKF 메인 노드
│   ├── motion_model.py        # 비선형 운동 모델
│   └── measurement_model.py   # 센서 측정 모델
├── lidar/
│   ├── lidar_preprocess.cpp   # LiDAR 노이즈 제거
│   └── obstacle_detector.cpp  # 장애물 인지
├── dnn/
│   ├── risk_model.py          # DNN 추론 노드
│   └── train_risk_model.ipynb # 모델 학습 노트북
├── nmpc/
│   ├── nmpc_solver.py         # NMPC 최적화 솔버
│   └── cost_function.py       # 비용 함수 정의
├── launch/
│   ├── ukf.launch
│   ├── nmpc.launch
│   └── full_system.launch
└── README.md
```

---

## 🔧 기술 스택

![ROS](https://img.shields.io/badge/ROS1-Melodic%2FNoetic-22314E?style=flat-square&logo=ros)
![Python](https://img.shields.io/badge/Python-3.x-3776AB?style=flat-square&logo=python)
![C++](https://img.shields.io/badge/C++-14-00599C?style=flat-square&logo=cplusplus)
![PyTorch](https://img.shields.io/badge/PyTorch-DNN-EE4C2C?style=flat-square&logo=pytorch)

- **상태추정**: UKF (Unscented Kalman Filter)
- **위험도 예측**: Deep Neural Network (PyTorch / Keras)
- **최적 제어**: NMPC (Nonlinear Model Predictive Control)
- **플랫폼**: TurtleBot3 Waffle Pi

---

## 👤 기여 내역

본 프로젝트의 **인지·추정·제어 핵심 파트 전체**를 직접 설계 및 구현하였습니다.

- ✅ UKF 상태 추정 전체 모델 구축
- ✅ LiDAR 전처리 및 위험 Feature 설계
- ✅ DNN Risk Model 학습 및 연동
- ✅ NMPC 비용항 / 제약조건 설계
- ✅ ROS1 기반 전체 시스템 통합 및 launch 제작
- ✅ 주행 실험 수행 및 결과 분석

---

## 🔮 향후 발전 방향

```
현재                          →  향후
─────────────────────────────────────────────────────
UKF                          →  Factor-Graph 기반 Back-End 확장
DNN                          →  Transformer 기반 위험도 예측
NMPC                         →  강화학습 기반 Adaptive MPC
TurtleBot3 (소형 로봇)        →  실차 규모 플랫폼 확장
```

---

<div align="center">
  <sub>Graduation Project · ROS1 Autonomous Driving · TurtleBot3 Waffle Pi</sub>
</div>
