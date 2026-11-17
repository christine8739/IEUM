# 이음 (IEUM)

> 시각장애인을 위한 AI 보행 보조 앱 — 실시간 객체 탐지와 TTS·햅틱 피드백으로 안전한 이동을 돕습니다.

**팀 도담도담 · 2026 한이음 드림업 공모전**


---

## 소개

이음은 카메라 영상을 실시간으로 분석해 전방의 장애물·신호등·위험 요소를 음성과 진동으로 알려주고, 계단과 경사를 피하는 접근성 기반 경로를 안내하는 앱입니다.

### 이음만의 특징
- **접근성 우선 길찾기**: 계단 제외 옵션(Tmap `searchOption=30`)으로 이동 약자에게 맞는 경로 후보 제공
- **보행 상황 중심 탐지**: 신호등 색상, 전방 객체를 보행 맥락에 맞춰 안내
- **음성 + 햅틱 이중 피드백**: 소리와 진동으로 정보 전달
- **데모 모드**: 사전 계산된 탐지 결과를 재생해 어떤 환경에서도 동일한 시연 가능

---

## 주요 기능

| 기능 | 설명 |
|---|---|
| 실시간 객체 탐지 | YOLO 기반 탐지 + ByteTrack 추적, 결과를 WebSocket으로 앱에 전달 |
| 신호등 색상 인식 | 바운딩 박스 하단 1/3 영역을 HSV 기준으로 분석해 적/녹 판별 |
| 음성·햅틱 안내 | 탐지 결과를 TTS와 진동 패턴으로 변환 |
| 접근성 길찾기 | 목적지 입력 → 경로 후보(추천/계단 제외) → 턴바이턴 음성 안내 |
| 내 경로 | 자주 쓰는 경로 저장 및 재사용 |
| 데모 모드 | letterbox 좌표 보정을 적용한 사전 탐지 결과 재생 |

---

## 시스템 아키텍처

```
┌─────────────┐  WebSocket   ┌──────────────┐  WebSocket   ┌──────────────────┐
│  AI 모듈     │ ───────────▶ │ Spring Boot  │ ───────────▶ │ React Native 앱   │
│  FastAPI     │   :8000      │ 백엔드 :8080  │              │ (Expo)           │
│  YOLO+Byte   │              │ JWT / REST    │ ◀─────────── │ TTS · 햅틱 · 지도  │
└─────────────┘              └──────┬───────┘     REST      └──────────────────┘
                                    │
                             PostgreSQL · Tmap API
```

<!-- 가능하면 위 도식을 이미지(docs/architecture.png)로 교체 -->

---

## 레포지토리 구성

| 파트 | 레포 | 내용 |
|---|---|---|
| Frontend | [frontend-ieum](https://github.com/christine8739/frontend-ieum) | React Native(Expo) 앱, 데모 모드, 길찾기·내 경로 화면 |
| Backend | [backend-ieum](https://github.com/christine8739/backend-ieum) | Spring Boot, JWT 인증, 경로 후보 조회·저장 API |
| AI | [AI-ieum](https://github.com/christine8739/AI-ieum) | YOLO 탐지, ByteTrack 추적, 신호등 인식 |

> 원본 조직 레포: [corner-IEUM](https://github.com/corner-IEUM)

---

## 기술 스택

- **Frontend**: React Native, Expo, TypeScript
- **Backend**: Spring Boot, Gradle, Spring Security + JWT, PostgreSQL
- **AI**: Python, FastAPI, YOLO(nano), ByteTrack, OpenCV
- **External API**: Tmap (경로 탐색)
- **Tools**: Git/GitHub, WebSocket

---

### Frontend
- React Native/Expo 앱 전반 구현 (탐지 화면, 길찾기, 내 경로)
- 탐지 박스 오버레이를 정확하게 맞추는 **letterbox 좌표 보정**(`computeLetterbox` / `mapBoxToScreen`) 구현
- 모바일 규격 웹 화면을 위한 `WebContainer` 구현
- 데모 모드 훅(`useDemoDetectionFeed`)과 사전 탐지 결과 생성 스크립트(`tools/precompute_demo_detections.py`)로 시연 파이프라인 완성

### Backend
- 대중교통 경로 API 구현: 경로 저장(POST), 구간 상태 갱신(leg-state PATCH), 버스 도착정보 조회(GET), 에러 응답 규격
- 길찾기 경로 후보 조회·저장 API와 프론트 화면 연동
- Spring Security + JWT 인증 환경에서 프론트-백엔드 통합 테스트

### AI 모듈
- 추적기를 **BoTSORT → ByteTrack으로 전환해 FPS 약 7 → 약 27로 개선**
- 최적 설정 확정: YOLO nano + `imgsz=768` + ByteTrack + frame skip 2
- `class_history` 상태 오염 문제를 수정해 클래스 판정 안정화
- HSV 기반 신호등 색상 인식(`core/traffic_light.py`) 구현

### 풀스택 연동
- FastAPI → Spring Boot → React Native 전체 WebSocket 파이프라인 통합 및 로컬 개발 환경 구성
- 개발보고서(아키텍처·기능표·시스템 다이어그램) 작성

---

## 실행 방법

각 레포의 README를 참고하세요. 로컬 실행 순서는 다음과 같습니다.

```bash
# 1. AI 서버 (포트 8000)
cd AI-ieum
pip install -r requirements.txt
uvicorn main:app --port 8000

# 2. 백엔드 (포트 8080)
cd backend-ieum
./gradlew bootRun

# 3. 프론트엔드
cd frontend-ieum
npm install
npx expo start
```

환경 변수는 각 레포의 `.env.example`을 복사해 사용합니다.

---

## 팀

| 이름 | 역할 |
|---|---|
| 임수영 | Frontend 리드, Backend(대중교통 경로 API), AI 모듈 디버깅, 풀스택 연동 |
| 박소현 | Backend (인증 JWT, 카카오/팀맵 API, 길찾기 API) |
---

## 향후 계획
- 사전 계산 탐지에서 실시간 카메라 인식으로 확장
- 위치 기반 AI 신뢰도 및 암묵적 개인화 기능
- 카카오맵 WebView 연동으로 지도 고도화
