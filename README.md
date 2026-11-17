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

| 분류 | 기능 | 설명 | 파트 |
|------|------|------|------|
| 탐지 | 실시간 객체 탐지 | 카메라 프레임을 AI 서버로 보내 YOLO 탐지와 ByteTrack 추적 결과를 수신 | Frontend · AI |
| 탐지 | 근접도·방향 판단 | 객체의 근접도와 좌/정면/우를 판단하고, 위험도가 높은 것을 우선 안내 | AI |
| 탐지 | 신호등 색상 인식 | HSV 분석으로 적색/녹색 구분 | AI |
| 안내 | 음성(TTS)·진동 피드백 | 위험 장면, 경로 이탈, 대중교통 상태 전환 시 안내 | Frontend |
| 안내 | 음성 명령(STT) | 화면별 명령(선택, 반복, 시작, 보행 종료 등)과 확인 응답 | Frontend |
| 길찾기 | 접근성 우선 도보 길찾기 | 추천 경로와 계단 없는 경로를 비교해 제공 (Tmap) | Frontend · Backend |
| 길찾기 | 대중교통 길찾기 | 후보 조회, 구간 상태 관리(대기 → 탑승 → 하차), GPS 기반 하차 안내 | Frontend · Backend |
| 길찾기 | 경로 이탈 감지·재탐색 | 경로에서 벗어나면 음성·진동으로 알리고 재탐색 | Frontend |
| 길찾기 | 지도 표시 | 카카오맵에 현재 위치와 경로 표시 | Frontend |
| 접근성 | 횡단보도 접근성 정보·경고 | PostGIS로 주변 횡단보도의 턱, 점자블록, 음향신호기 정보를 집계하고 실시간으로 푸시 | Backend · Frontend |
| 계정·기록 | 회원가입·로그인 | BCrypt, JWT | Frontend · Backend |
| 계정·기록 | 보행·길찾기 기록 | 보행 및 길찾기 기록 저장·조회 | Frontend · Backend |
| 시연 | 데모 모드 | 서버가 데모 영상을 실시간 추론하고 앱이 같은 영상을 재생 | Frontend · AI |

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
| Frontend | [frontend-ieum](https://github.com/corner-IEUM/frontend-ieum) | React Native(Expo) 앱, 데모 모드, 길찾기·내 경로 화면 |
| Backend | [backend-ieum](https://github.com/corner-IEUM/backend-ieum) | Spring Boot, JWT 인증, 경로 후보 조회·저장 API |
| AI | [AI-ieum](https://github.com/corner-IEUM/AI-ieum) | YOLO 탐지, ByteTrack 추적, 신호등 인식 |

> 원본 조직 레포: [corner-IEUM](https://github.com/corner-IEUM)

---

## 기술 스택

- **Frontend**: React Native, Expo, TypeScript
- **Backend**: Java 17, Spring Boot, Spring Security(JWT), JPA, Flyway, WebSocket(STOMP), PostgreSQL(PostGIS)
- **AI**: Python, Ultralytics YOLO, ByteTrack, OpenCV
- **External API**: Tmap(보행자·대중교통 경로), Kakao(지도, 장소 검색), 서울시 버스 정보 API(공공데이터포털)
- **Tools**: Git/GitHub, Gradle, Expo/EAS

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
