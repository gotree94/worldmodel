# 로봇 학습용 시뮬레이션·물리 엔진: Cosmos, Unity/Unreal, GENESIS

월드모델(NVIDIA Cosmos)과 물리 시뮬레이션 엔진·플랫폼의 관계, 그리고 Unity/Unreal과 GENESIS에 대한 정리입니다.

---

## 1. 먼저 구분해야 할 개념: "월드 모델" vs "물리 시뮬레이션 엔진"

| 구분 | 월드 모델 (World Model) | 물리 시뮬레이션 엔진 (Physics Engine) |
| --- | --- | --- |
| 역할 | 행동에 따른 미래를 **예측·생성** (영상/비디오 생성) | 물리 법칙을 **수치적으로 계산**해 환경 재현 |
| 방식 | 학습 기반 (딥러닝, 생성모델) | 규칙·수치 기반 (통합기, 접촉/마찰 계산) |
| 대표 예 | NVIDIA Cosmos, Genie, Sora, V-JEPA | PhysX, Newton, MuJoCo, Bullet, Chaos |
| 학습 용도 | 데이터 생성, 평가, 상상 롤아웃, 합성 데이터 | RL 정책 학습, sim-to-real, 접촉·궤적 시뮬레이션 |

> **핵심**: NVIDIA Cosmos는 물리 엔진이 아니라 "물리를 이해하는 **월드 파운데이션 모델(WFM)**"이며, 실제 물리 시뮬레이션은 **Isaac Sim/Lab(PhysX 5, Newton)**이 담당합니다. Cosmos로는 영상/월드를 **생성**하고, Isaac으로는 물리를 **계산**합니다.

---

## 2. NVIDIA Cosmos와 물리 엔진 (Newton) 접근

### Cosmos (월드 파운데이션 모델) — 물리 엔진 아님
- 물리적 상식·공간 추론·시간 역학을 학습한 생성 모델
- 광고·자율주행·로봇 학습용 합성 영상과 월드 생성
- **Cosmos 3** (Edge 4B / Nano 16B / Super 64B), Cosmos-Transfer, Cosmos-Predict, Cosmos-Reason 등
- **Cosmos 3 Edge**: 실시간 온디바이스 추론용 40억 파라미터 모델

### Newton (물리 엔진) — Cosmos의 시뮬레이션 파트너
- **개발 주체**: NVIDIA + Google DeepMind + Disney Research, Linux Foundation 관할, 오픈소스
- **특징**: GPU 가속·확장 가능·미분 가능 물리 엔진 (NVIDIA Warp 기반, MuJoCo Warp 통합)
- **위치**: Isaac Sim의 물리 백엔드로 PhysX 5와 병행 사용. Isaac Lab(RL 프레임워크)이 이를 활용
- **강점**: 접촉 역학(contact dynamics) 정확도, GPU 병렬 처리, 물리 일관성 → sim-to-real 격차 축소
- **상태**: Isaac Lab 3.0 Beta에 통합. 보행(로커모션) 정책은 실제 G1 로봇에 배포 검증 완료, 그리핑은 아직 미검증. SIGGRAPH 2026에 눈/모래/탄성체 실시간 솔버 추가
- **비교**: 기존 PhysX 5 / Warp보다 접촉 정확도·GPU 병렬 효율 우수. USD/Warp 직결

> **결론**: NVIDIA 생태계에서 "물리 엔진 접근"은 **PhysX → Newton**으로 진화 중이며, Cosmos는 별도로 월드를 생성합니다.

---

## 3. Unity / Unreal 에서의 물리 엔진 접근

### Unity
- **물리 엔진**: **PhysX 4 기반(Unity 변형)** — NVIDIA PhysX 5보다 구형. 고급 기능·GPU 가속 부족
- **로보틱스 지원**: **Unity Robotics Hub** (URDF 임포터, ROS-TCP 커넥터, 시뮬레이션 샘플)
- **강점**: 최고 수준의 시각 품질·커스텀 센서 로직 → **인지(perception)·합성 데이터 생성·HIL/HWIL**에 적합
- **한계**: 고충실도 로봇 물리 out-of-box 부족 (articulation body, CCD 등은 개선 중)
- **학습 연계**: ML-Agents 툴킷으로 RL 가능하나, Isaac Sim 대비 물리 정확도·GPU RL 규모 열위

### Unreal Engine
- **물리 엔진**: **Chaos Engine (PBD, Position Based Dynamics 솔버)** — 로봇 접촉·정밀 시뮬레이션에 부적합
- **로보틱스**: UnrealZoo, DeliveryBench 등 벤치마크·연구 용도로 사용
- **한계**: 게임용 엔진이라 정밀 접촉 역학(TGS/PGS 솔버)이 아니라 로봇 학습용으로는 부정확
- **하이브리드 접근**: Unreal 렌더링 + MuJoCo 물리 결합 (예: "Unreal Robotics Lab") — 통합 복잡도 증가
- **최신**: SIGGRAPH 2026에서 Unreal Editor에 **MCP(MCP 연결)로 AI 클라이언트 연동** 발표 (AI 에이전트가 씬·에셋 조작)

### Unity/Unreal과 전용 로봇 시뮬레이터 비교 요약
| 항목 | Unity | Unreal | Isaac Sim | MuJoCo |
| --- | --- | --- | --- | --- |
| 물리 엔진 | PhysX 4 변형 | Chaos (PBD) | **PhysX 5 + Newton** | MuJoCo (MJX GPU) |
| 시각 품질 | 최상 | 최상 | 최상(RTX) | 기본 |
| 로봇 물리 정확도 | 보통 | 낮음 | **높음** | **높음** |
| GPU RL 병렬 | 제한적 | 제한적 | **우수** | 우수(MJX) |
| 주 용도 | 인지/데이터 생성 | 비주얼/게임 | **로봇 학습 전용** | 접촉 고충실도 RL |

> **결론**: Unity/Unreal은 **렌더링·데이터 생성**에 강하고, **정밀 로봇 물리**는 Isaac Sim(PhysX/Newton) 또는 MuJoCo가 담당합니다. 물리 학습 중심이라면 Unity/Unreal보다 Isaac Sim·MuJoCo·GENESIS가 더 적합합니다.

---

## 4. GENESIS (Genesis World)

### 개요
- **오픈소스 범용·생성형 물리 엔진 + 시뮬레이션 플랫폼** (2024년 12월 학술 프로젝트 시작, 현재 Genesis AI가 공식 지원, 2026년 5월 Genesis World 1.0)
- 20+ 연구실이 24개월 협력, CMU 주도
- 이후 **Genesis AI** (범용 로봇 "Eno" + 파운데이션 모델 "GENE")로 상업화 확장

### 핵심 특징
- **Python 기반 전체**: 인터페이스와 코어 모두 Python, `pip install` 로 간편 설치
- **통합 멀티-물리 엔진**: Rigid, FEM(유한요소), MPM(물질점), Particle(PBD/SPH), uipc, SAP 등 다양한 솔버를 하나의 씬·상태로 통합
- **초고속**: 실시간 대비 **최대 430,000배 빠른 시뮬레이션** (실제 로봇 암 4,300만 FPS 달성 사례), Isaac Gym·MuJoCo MJX보다 10~80배 빠름
- **생성형(Generative)**: VLM이 자연어 프롬프트를 **4D 동적 월드·상호작용 씬·캐릭터 모션**으로 변환, 물리 법칙에 따르는 카메라 이동·물체 거동 생성
- **사진現實적 렌더링**: Nyx(로보틱스 전용 렌더러), Luisa 레이트레이싱, Pyrender 래스터라이저
- **미분 가능(Differentiable)**: 물리 시뮬레이션 최적화 지원
- **컴파일러**: Quadrants (크로스플랫폼 컴파일러)로 랩톱 CPU부터 데이터센터 GPU까지 확장

### 월드모델·로봇 학습과의 관계
- **물리 시뮬레이션** (GENESIS) + **생성형 4D 월드 생성** (월드모델 기능) 두 가지를 겸함 → 영상 생성 기반 월드모델과 달리 **실제 물리 계산 가능**
- RL/모방학습 훈련용 가상 환경, 합성 데이터 생성, 정책 평가에 사용
- 자동화된 데이터 생성으로 "자생적 데이터 생태계" 지향

### 참고 자료
- GitHub: `Genesis-Embodied-AI/genesis-world`, `Genesis-Embodied-AI/Genesis`
- 문서: genesis-world.readthedocs.io · genesis.ai

---

## 5. 종합 비교 (로봇 학습 관점)

| 플랫폼/엔진 | 성격 | 물리 | 생성(월드모델) | 속도 | 로봇 학습 적합도 |
| --- | --- | --- | --- | --- | --- |
| **NVIDIA Cosmos** | 월드 파운데이션 모델 | ✗ (생성) | ✓✓✓ | - | 합성 영상·데이터 생성 |
| **Isaac Sim/Lab + Newton** | 로봇 시뮬레이션 플랫폼 | ✓✓✓ | 일부 | GPU 병렬 | **RL·sim-to-real 최적화** |
| **Unity** | 게임 엔진 | ✓(PhysX4) | ✗ | 보통 | 인지·데이터 생성 |
| **Unreal** | 게임 엔진 | △(Chaos PBD) | ✗ | 보통 | 비주얼·연구 벤치마크 |
| **MuJoCo(MJX)** | 물리 엔진 | ✓✓✓ | ✗ | GPU 우수 | 접촉 고충실도 RL |
| **GENESIS(Genesis World)** | 생성형 범용 물리 플랫폼 | ✓✓✓ | ✓✓ (4D 생성) | **430,000x** | **생성+물리 통합 학습** |

**요약**
- Cosmos는 물리 엔진이 아닌 **생성형 월드모델** → 물리는 Isaac Sim의 **Newton(PhysX 5 대체/병행)** 담당
- Unity/Unreal은 **렌더링·데이터 생성**에 강하고, 정밀 로봇 물리에는 부적합 (하이브리드 결합 필요)
- **GENESIS(Genesis World)** 는 물리 엔진이면서 생성형 4D 월드 생성까지 겸하는 차별화 플랫폼으로, 초고속·멀티물리·생성이 모두 가능한 최신 대안

---

## 6. 추가 추천: 그 외 주목할 만한 시뮬레이터·프레임워크 (2026)

앞서 다룬 Cosmos/Isaac/Unity/Unreal/MuJoCo/GENESIS 외에, 로봇 학습에 실무적으로 많이 쓰이는 도구들입니다. 용도별로 정리했습니다.

### 6-1. 조작(Manipulation) 벤치마크 중심
- **Robosuite** (MIT, MuJoCo 기반)
  - 표준 조작 벤치마크에 특화: **LIBERO, MimicGen, RoboMimic**
  - 모방학습·조작 연구에서 논문 게재 시 표준으로 사용
  - **선택 시점**: 조작 정책을 LIBERO/MimicGen/RoboMimic 기준으로 결과를 내야 할 때
- **RLBench / RoboSuite 계열**: 멀티태스크 조작 학습 벤치마크

### 6-2. JAX 네이티브 미분 가능 RL (신호: 미분(diff)물리 수요 증가)
- **Brax** (Google, Apache 2.0)
  - JAX 기반 미분 가능 로코모션·물리. GPU 병렬로 4096+ 환경
  - **선택 시점**: JAX 스택에서 미분 가능 물리로 그라디언트 기반 제어 연구할 때

### 6-3. 모델기반 제어·궤적 최적화 (RL이 아닌 제어 이론 중심)
- **Drake** (MIT Russ Tedrake 그룹)
  - 엄밀한 접촉 물리(SDF, 정리 가능한 time-stepping 솔버)
  - 궤적 최적화·모션 플래닝·그래스프 분석에 강함
  - **선택 시점**: 학습 정책을 모델기반 컨트롤러 안의 컴포넌트로 쓸 때 / 접촉 물리를 수학적으로 증명 가능하게 다룰 때

### 6-4. 교육·모바일 프로토타이핑 / ROS 2 생태계
- **Webots** (아파치 2.0, 무료)
  - GUI 중심, C/C++/Python/Java/MATLAB API, 내장 로봇·센서 라이브러리
  - **선택 시점**: 교육·스웜 로봇·모바일 프로토타이핑
- **Gazebo Harmonic** (아파치 2.0)
  - **ROS 2 네이티브 통합**이 가장 강점, 멀티로봇 시스템 시뮬레이션
  - 순수 RL에는 적합도 낮음
- **CoppeliaSim** (구 V-REP)
  - 한 씬에서 **여러 물리 엔진(Bullet/ODE/Vortex/Newton/뮤허스)** 스위칭 가능
  - 접촉 많은 조작·학술 연구에 다용도

### 6-5. 레거시·입문
- **PyBullet** (zlib, 무료)
  - `pip install` 원스톱, URDF 지원, 교육·기존 벤치마크 재현용
  - **신규 연구에는 MuJoCo 권장** (접촉 정확도·GPU 병렬 열위)

### 6-6. 월드모델/생성형 시뮬레이션 계열 (물리 엔진 아님, 보완용)
- **Google Genie 3** (DeepMind, 2025.8 공개 / Project Genie)
  - 텍스트 프롬프트로 **실시간 탐색 가능한 3D 상호작용 월드** 생성 (720p, 24fps)
  - 물리·물·조명·날씨 시뮬레이션, 일관성 유지, 프롬프트로 이벤트 주입
  - 20만 시간 인터넷 비디오로 훈련
  - **로보틱스 활용**: 로봇 훈련용 **1인칭(ego-centric) 대규모 데이터 생성**, 강화학습 제어 가능
  - **한계**: 지리적 정확성 낮음, 단일 에이전트 중심, 일관성 수 시간 미만
  - **활용 사례**: Waymo가 자율주행 월드모델로 Genie 3 채택 (2026.2), Street View 연동(2026.5)
- **Waymo World Model / Street View 연동 Genie**
  - 실제 도로(Street View)를 생성 월드에 앵커링 → 현실 기반 상호작용 환경
  - 자율주행·실외 로봇 내비게이션 학습에 유용

### 6-7. 데이터·참고 리소스 (로봇 학습 생태계)
- **Open X-Embodiment**: 21개 기관, 22개 플랫폼, 100만+ 실제 로봇 궤적 오픈 데이터셋
- **DROID**: 76,000 시연 궤적(약 350시간), 로봇 조작 학습용
- **MuJoCo Menagerie**: 출시된 로봇 모델(MJCF) 라이브러리
- **Hugging Face LeRobot**: 여러 시뮬레이터에 공통 래퍼로 정책 배포 지원
- **Genesis AI / Eno + GENE**: Genesis World의 상용화 스택 (범용 로봇 + 파운데이션 모델)

---

## 7. 최종 추천 로드맵 (2026, 용도 기반)

| 목적 | 추천 |
| --- | --- |
| 휴머노이드·로코모션 RL (NVIDIA GPU 보유) | **Isaac Lab** (PhysX 5/Newton) |
| 학술 조작·VLA 평가 | **MuJoCo(MJX) + Robosuite** |
| 표준 조작 벤치마크 출판 | **LIBERO / MimicGen / RoboMimic (Robosuite)** |
| 생성형 월드 데이터·1인칭 데이터 수집 | **GENESIS, Genie 3, Cosmos** |
| 물리 이미지 사실성 + 대규모 합성 데이터 | **Isaac Sim** |
| ROS 2 풀스택·멀티로봇 | **Gazebo Harmonic** |
| 제어 이론·궤적 최적화 | **Drake** |
| JAX 미분 가능 RL | **Brax** |
| 교육·입문·빠른 프로토타입 | **PyBullet → Webots** |
| 실외·자율주행 월드 시뮬레이션 | **Waymo World Model / Genie 3 + Street View** |

**시사점**: 2026년에는 "하나의 툴을 고르라"기보다, **물리(Isaac/MuJoCo/GENESIS)·생성(Cosmos/Genie)·데이터(Open X, DROID)** 를 결합하는 하이브리드 파이프라인이 표준이 되어가고 있습니다. 시뮬레이터는 일반적으로 **2~3개를 병행하는 것**이 권장됩니다.
