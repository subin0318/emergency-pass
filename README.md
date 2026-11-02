# 🚑 응급패스

증상을 입력하면 **지금 가장 빨리 진료받을 수 있는 응급실**을 추천해 주는 전라남도 응급의료 길안내 서비스입니다.

단순히 가까운 병원이 아니라 **실시간 응급실 병상 + 이동 시간 + 중증질환 수용 가능 여부 + 장비 현황**을 함께 따져 "도착 후 실제 진료까지 걸리는 시간"이 가장 짧은 병원을 골라 줍니다.

## 사용 모드

화면 상단에서 세 가지 모드를 전환할 수 있습니다.

| 모드 | 대상 | 하는 일 |
|------|------|---------|
| 시민 모드 | 일반 시민 | 현재 위치(GPS·주소 검색)와 증상을 입력하면 병원 추천, 지도 경로 안내 |
| 구급대원 모드 | 119 구급대원 | 시민 모드 기능 + 환자 정보(나이·성별·의식·호흡·출혈 등) 입력, 선택한 병원으로 **이송 정보 공유 카드** 생성 |
| 의료진 모드 | 병원 의료진 | 들어온 이송 카드를 확인하고 **수용 확정**, 병원별 확정 환자 목록 확인 |

> 이송 카드는 브라우저 `localStorage`에 저장되므로 현재는 같은 브라우저 안에서만 공유됩니다(시연용).

## 주요 기능

- **증상 분석** — 자연어로 입력한 증상을 Gemini LLM이 분석해 중증도·필요 진료과·진료 경로를 추출 (LLM 실패 시 규칙 기반 추출로 대체)
- **중증 응급실 추천** — 국립중앙의료원 실시간 병상 데이터와 카카오내비 ETA로 예상 소요 시간 계산 후 상위 병원 추천
- **경증 병의원 추천** — 경증이면 건강보험심사평가원(HIRA) 데이터로 주변 일반 병·의원을 추천해 권역센터 과밀 분산
- **패널티 기반 보정** — 도서·산간 지역, 필수 장비 부족, 전용 병상 부족, 응급실 뺑뺑이 위험 등을 분 단위 패널티로 반영
- **지도 & 길찾기** — Leaflet 지도에 병원 표시, 카카오 길찾기 실제 도로 경로 표시
- **병원 상세** — 응급실·중환자실 가용 병상, CT·MRI·조영촬영기 가능 여부, 혼잡도(원활/보통/혼잡), 병상 추이

## 기술 스택

| 구분 | 사용 기술 |
|------|-----------|
| Frontend | React 19, Vite, Leaflet, lucide-react, Daum 우편번호 API |
| Backend | Python, FastAPI, Uvicorn |
| AI | Google Gemini `gemini-2.5-flash` (`google-genai`) |
| DB | SQLite |
| 외부 API | 국립중앙의료원 응급의료정보, 건강보험심사평가원 병원정보, Kakao Local / Kakao Mobility |

## 프로젝트 구조

```
응급패스_최종/
├── Back/                        # FastAPI 백엔드
│   ├── main.py                  # API 엔드포인트, 추천 로직
│   ├── llm_engine.py            # 증상 분석 (Gemini + 규칙 기반)
│   ├── routing.py               # 거리·ETA 계산 (카카오내비, 캐시)
│   └── config.py                # 패널티·임계값 등 정책 상수
├── collect_hospitals/           # 공공데이터 수집기
│   ├── collect_hospitals.py     # 병원 정보·수용 정보·실시간 병상 → hospital.db
│   ├── query_hospitals.py       # DB 내용 확인용
│   └── hospital.db              # SQLite DB
├── Front/                       # React 프론트엔드
│   ├── src/
│   │   ├── App.jsx              # 모드 전환, 이송 카드 상태 관리
│   │   ├── constants.js         # 위치 프리셋, 증상 태그
│   │   ├── utils.js
│   │   └── components/
│   │       ├── AppHeader.jsx                  # 상단 모드 전환
│   │       ├── CitizenAmbulanceDashboard.jsx  # 시민·구급대원 화면
│   │       ├── MedicalDashboard.jsx           # 의료진 화면
│   │       ├── LocationPanel.jsx              # 위치 입력
│   │       ├── SymptomPanel.jsx               # 증상 입력
│   │       ├── PatientInfoPanel.jsx           # 환자 정보 (구급대원)
│   │       ├── RoutingFilterPanel.jsx         # 경증 분산 필터
│   │       ├── RecommendationResults.jsx      # 추천 결과 목록
│   │       ├── HospitalDetailModal.jsx        # 병원 상세
│   │       ├── MapComponent.jsx               # Leaflet 지도
│   │       └── ErrorBanner.jsx
│   ├── vite.config.js           # /api → 백엔드 프록시
│   └── package.json
├── requirements.txt
├── .env.example
└── .env                         # API 키 (커밋 금지)
```

## 시작하기

### 1. API 키 준비

`.env.example`을 복사해 프로젝트 루트에 `.env`를 만들고 키를 채워 주세요.

| 변수 | 발급처 |
|------|--------|
| `SERVICE_KEY` | [공공데이터포털](https://www.data.go.kr) — 국립중앙의료원 응급의료기관 정보 조회 서비스 |
| `HIRA_SERVICE_KEY` | [공공데이터포털](https://www.data.go.kr) — 건강보험심사평가원 병원정보서비스 |
| `KAKAO_API_KEY` | [Kakao Developers](https://developers.kakao.com) — REST API 키 (로컬, 모빌리티 길찾기) |
| `GEMINI_API_KEY` | [Google AI Studio](https://aistudio.google.com) |
| `SERVER_PORT` | (선택) 백엔드 포트, 기본 8000 |

### 2. 백엔드 실행

```bash
pip install -r requirements.txt
```

병원 데이터 수집 (최초 1회, 이후 최신 병상 갱신이 필요할 때마다). 공공데이터 API 일일 호출 제한 때문에 자동 갱신 대신 직접 실행합니다.

```bash
python collect_hospitals/collect_hospitals.py
```

서버 실행:

```bash
python Back/main.py
```

### 3. 프론트엔드 실행

```bash
cd Front
npm install
npm run dev
```

개발 서버가 `/api` 요청을 `http://localhost:8000`으로 프록시합니다. 백엔드 주소가 다르면 `VITE_BACKEND_ORIGIN` 환경 변수로 바꿀 수 있습니다.

## API

| Method | Endpoint | 설명 |
|--------|----------|------|
| POST | `/api/analyze-symptoms` | 증상 텍스트 분석 (`mode`: `citizen` / `ambulance`) |
| POST | `/api/recommend` | 중증 응급실 추천 |
| POST | `/api/recommend-light` | 경증 병·의원 추천 |
| POST | `/api/route` | 길찾기 경로 (실패 시 직선) |
| GET | `/api/hospitals` | 전체 응급의료기관 목록 |
| GET | `/api/hospital-bed-history` | 병원별 병상 추이 |
| GET | `/api/geocode` | 주소 → 좌표 |
| GET | `/api/reverse-geocode` | 좌표 → 행정구역 |

서버 실행 후 `http://localhost:8000/docs`에서 Swagger 문서를 볼 수 있습니다.

## 추천 알고리즘

```
기대 시간      = 성공 확률 × ETA + (1 − 성공 확률) × (ETA + 뺑뺑이 20분)
예상 소요 시간 = 기대 시간 + Σ 패널티
```

- **ETA**: 카카오내비 실시간 경로 시간 (실패 시 직선거리 ÷ 40km/h), 3분 캐시
- **뺑뺑이 확률 모델**: 가용 병상 수로 "도착했을 때 병상이 남아 있을 확률"을 추정합니다 (10개 이상 98%, 5개 이상 85%, 3개 이상 60%, 2개 40%, 1개 20%).
- **패널티**: 도서·산간 +30분, 필수 장비 부족 +20분, 경증의 권역센터 선택 +30분, 화상·소아 전용 병상 부족 +15분, 중환자실 부족 +10분
- **혼잡도**: 응급실 가용 병상 6개 이상 원활, 1개 이상 보통, 0개 혼잡
- **경증 검색 반경**: 기본 30km, 결과가 3개 미만이면 50km로 확장
- 정책 값은 모두 [`Back/config.py`](Back/config.py)에서 조정할 수 있습니다.

## ⚠️ 면책

이 서비스는 참고용 정보를 제공하며 의료적 판단을 대신하지 않습니다. 생명이 위급한 상황에서는 즉시 **119**에 연락하세요.
