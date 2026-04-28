# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 언어 및 소통 원칙

- 모든 응답은 **한국어**로 작성합니다.
- IT 전문 용어는 그대로 사용하되, 필요 시 괄호 안에 영문 병기합니다 (예: "생성형 AI(Generative AI)").
- 명확하고 간결한 어조를 유지하며, 핵심만 전달합니다.

## 위험 작업 처리

아래 작업은 반드시 실행 전 사용자에게 확인을 요청합니다.

- 파일 또는 디렉토리 삭제
- `git force push` / `reset` / `rebase`
- 환경 변수 또는 설정 파일 수정
- 패키지 전역 설치 또는 제거

## 프로젝트 개요

**영수증 OCR 지출관리 앱** — 영수증 이미지(JPG/PNG/PDF)를 업로드하면 Upstage Vision LLM으로 파싱해 지출 내역을 자동 기록하는 웹 앱.

- 저장소: JSON 파일(`backend/data/expenses.json`), DB 없음
- 배포: Vercel (서버리스, 프론트+백엔드 통합)
- 외부 API: Upstage Vision API (`UPSTAGE_API_KEY` — `.env`에 설정됨)

## 기술 스택

| 레이어 | 기술 |
|--------|------|
| Frontend | React 18 + Vite 5 + TailwindCSS 3 + Axios + React Router |
| Backend | Python FastAPI + Uvicorn + LangChain 1.2.15 + Pillow |
| OCR/LLM | Upstage Vision API (`document-digitization-vision`) |
| 배포 | Vercel (serverless) |

## 개발 명령어

### 백엔드

```bash
# 프로젝트 루트에서 실행 (venv는 루트에 위치)
python -m venv venv
source venv/Scripts/activate        # Windows: venv\Scripts\activate
pip install -r backend/requirements.txt
PYTHONPATH=. venv/Scripts/uvicorn backend.main:app --reload  # http://localhost:8000
```

### 프론트엔드

```bash
cd frontend
npm install
npm run dev      # http://localhost:5173
npm run build
npm run preview
```

## 아키텍처

```
브라우저 (React)
  └─ 영수증 업로드
       ↓
FastAPI 백엔드
  ├─ 파일 검증 (10MB 한도, JPG/PNG/PDF)
  ├─ Base64 변환
  ├─ LangChain → Upstage Vision API
  ├─ JSON 파싱 (가게명, 날짜, 항목, 합계)
  └─ expenses.json 저장
       ↓
프론트엔드 대시보드
  ├─ 지출 목록 카드 표시
  ├─ 날짜 필터 / 합계 요약
  └─ 항목 수정 / 삭제
```

### API 엔드포인트

| 메서드 | 경로 | 기능 |
|--------|------|------|
| POST | `/api/upload` | 영수증 업로드 → OCR 파싱 → JSON 반환 |
| GET | `/api/expenses` | 지출 목록 조회 (날짜 필터 지원) |
| GET | `/api/summary` | 합계 및 카테고리별 통계 |
| PUT | `/api/expenses/{id}` | 지출 항목 수정 |
| DELETE | `/api/expenses/{id}` | 지출 항목 삭제 |

### 지출 데이터 스키마

```json
{
  "id": "uuid-v4",
  "created_at": "ISO8601",
  "store_name": "이마트 강남점",
  "receipt_date": "YYYY-MM-DD",
  "receipt_time": "HH:MM",
  "category": "식료품|외식|교통|쇼핑|의료|기타",
  "items": [
    { "name": "상품명", "quantity": 1, "unit_price": 1000, "total_price": 1000 }
  ],
  "total_amount": 10300,
  "payment_method": "신용카드"
}
```

## 주요 파일

| 파일 | 설명 |
|------|------|
| `PRD_영수증_지출관리앱.md` | 전체 요구사항 문서 (UI 스펙, 컬러 팔레트, 8단계 개발 계획 포함) |
| `프로그램개요서_영수증_지출관리앱.md` | 프로젝트 개요 및 아키텍처 문서 |
| `.env` | `UPSTAGE_API_KEY` 설정 (git 제외) |
| `images/` | 테스트용 영수증 이미지 14종 + UI 목업 9종 |
| `backend/main.py` | FastAPI 앱 진입점 |
| `backend/routers/` | upload.py, expenses.py, summary.py |
| `backend/services/ocr_service.py` | LangChain + Upstage 연동 |
| `backend/data/expenses.json` | 데이터 파일 (git 제외 권장) |
| `frontend/src/pages/` | Dashboard, UploadPage, ExpenseDetail |
| `frontend/src/components/` | DropZone, ExpenseCard, SummaryCard 등 |
| `vercel.json` | Vercel 서버리스 라우팅 설정 |
