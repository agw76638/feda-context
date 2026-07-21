# 프론트엔드 스터디 커뮤니티 — 프로젝트 개요

🚀 **AI 기반 오픈 학습 아카이브 커뮤니티** — 프론트엔드/CS 학습 주제를 검색하면 AI가 자료를 요약해주고, 개인 학습노트로 정리하는 서비스

---

## 📌 문제 (핵심 아이디어)

취업 준비생/개발 학습자는 학습 자료를 찾고 정리하는 데 반복적인 비용을 씁니다:

- 같은 주제를 여러 사람이 각자 검색함 (중복 리서치)
- 검색한 자료가 개인 브라우저 기록/북마크에 흩어짐
- 배운 내용을 정리해도 공유되지 않고 개인 소유로 끝남
- 실력을 스스로 점검할 방법이 마땅치 않음

➡️ **AI가 검색·요약한 학습 자료를 공동 아카이브로 쌓고, 개인 학습노트와 연결해 정리할 수 있는 오픈 커뮤니티를 제공한다.**

---

## 🧑‍💻 유저

| 페르소나          | 니즈                                             |
| ----------------- | ------------------------------------------------ |
| 취업 준비생       | 기술면접/코딩테스트 대비 자료를 빠르게 찾고 정리 |
| 프론트엔드 학습자 | 개념 정리 + 스스로 이해도 확인                   |
| 커뮤니티 참여자   | 다른 사람이 올린 자료를 둘러보고 참고            |

---

## ✨ 핵심 기능

### A) 페이지 구성 (MVP)

1. 헤더/푸터 (공통 레이아웃, 사이드바 포함)
2. 로그인 페이지 (이메일+비밀번호, 구글/카카오 소셜로그인)
3. 회원가입 페이지
4. 가입완료 페이지
5. 홈 — AI 홈페이지 스타일의 인풋 전용 화면 (카테고리 선택 + 키워드 입력으로 주제 등록/검색만 담당, 피드 없음)
6. 요약본 노트 페이지 — AI 개념 요약 + 참고자료(출처) + 좋아요/북마크 + 학습노트 생성 버튼. 연결된 학습노트가 없을 때만 삭제 가능(확인 모달)
7. 학습노트 작성 페이지 — 개인이 직접 정리하는 글, 원본 게시글(topic) 참조. 완료 시 학습노트 상세로 이동
8. 학습노트 상세 페이지 — 작성한 학습노트 뷰 + 수정/삭제 + 퀴즈 진입 버튼. 삭제 시 확인 모달 → 요약본 노트 페이지로 이동
9. 전체 요약 노트 페이지 — AI 요약본(topics) 전체 목록 보기
10. 마이페이지 — 내 학습노트(리스트) / 내가 생성한 요약노트(카드) / 북마크·책갈피(카드), 3섹션 구성

**모달**

- 퀴즈 모달 — 학습노트 상세 페이지에서 진입 (노트 작성을 완료해야 퀴즈 접근 가능). 1회성 — 제출 후 다시 풀 수 없음
- 로그인 권장 모달 — 비로그인 상태로 요약본 생성 시도 시 노출

### B) 카테고리 (주제 제한)

자유 입력이 아니라 카테고리 선택 + 세부 키워드 입력 방식으로 제한한다 (예: CS기초 / 알고리즘 / 프론트엔드 / 백엔드 / 네트워크).
→ Alan 응답 품질 예측 가능성 확보, 요청 수(하루 100회) 절약, 콘텐츠 쏠림으로 아카이브 효과 강화.

### C) 인증

- 이메일 + 비밀번호
- 구글, 카카오 소셜로그인 **[필수 — 명세 반영]**
  - ⚠️ 카카오 로그인은 Supabase 기본 OAuth 제공자가 아니라 커스텀 OIDC 설정이 필요할 수 있음 — 구현 난이도 사전 확인 필요

### D) AI 기능 (Alan API)

- 학습 주제 검색 → 실시간 웹검색 기반 개념 요약 생성
- 이해도 확인 퀴즈 생성 (문제/보기/정답 형식 프롬프트 고정)
- 참고자료(출처) 함께 반환

```
GET /api/v1/question?content={프롬프트}&client_id={ID}
응답: { answer: "마크다운 텍스트", references: [{source, title, content}, ...] }
```

- 하루 요청 100회 / client_id 제한 → 동일 주제 재검색 방지(캐싱) 설계 필수
- 이미지 생성 기능 없음 (텍스트 전용 API)
- 응답 형식이 매번 100% 동일 보장되지 않음 (LLM 생성) → 파싱 실패 시 원문 마크다운 그대로 렌더링하는 폴백 필수

### E) 기타 (명세 필수 요구사항)

- 비동기 처리: 로딩 UI(스피너/스켈레톤), AI 생성 중 진행상태 안내, 중복 요청 방지(버튼 비활성화)
- 예외처리: 네트워크 오류 안내, AI API 실패 시 재시도, 로그인 실패 안내, 빈 데이터("등록된 데이터가 없습니다") 안내, 잘못된 입력값 처리

---

## 🗄️ 데이터 모델 (Supabase / PostgreSQL 초안)

> 초안이며 개발 중 변경될 수 있음

```sql
-- 사용자 (Supabase Auth 기본 테이블 활용, 프로필 확장)
create table profiles (
  id uuid references auth.users primary key,
  nickname text,
  created_at timestamp default now()
);

-- AI가 생성한 학습 게시글
create table topics (
  id uuid primary key default gen_random_uuid(),
  title text not null,
  category text not null,          -- 'CS기초' | '알고리즘' | '프론트엔드' 등
  alan_answer text not null,       -- Alan 응답 마크다운
  references jsonb,                -- 출처 목록
  created_by uuid references profiles(id),
  like_count int default 0,
  created_at timestamp default now()
);

-- 개인 학습노트 (topics 참조 = "연결")
create table study_notes (
  id uuid primary key default gen_random_uuid(),
  user_id uuid references profiles(id),
  topic_id uuid references topics(id),
  content text not null,
  created_at timestamp default now()
);

-- 좋아요 / 북마크
create table bookmarks (
  user_id uuid references profiles(id),
  topic_id uuid references topics(id),
  created_at timestamp default now(),
  primary key (user_id, topic_id)
);

-- 퀴즈 결과 (선택 — 퀴즈 유지 확정 시)
create table quiz_results (
  id uuid primary key default gen_random_uuid(),
  user_id uuid references profiles(id),
  topic_id uuid references topics(id),
  score int,
  answered_at timestamp default now()
);
```

---

## 🧱 기술 스택

| 구분         | 선택                  | 비고                        |
| ------------ | --------------------- | --------------------------- |
| Framework    | React (Vite)          | Next.js 필수 여부 **TBD**   |
| Language     | JavaScript            | TypeScript TBD              |
| DB / Backend | Supabase (PostgreSQL) | 명세에 명시됨               |
| CSS/UI       | TBD                   | **TBD**                     |
| AI           | Alan API              | 실시간 웹검색 기반 질의응답 |

---

## 🎨 UI / UX

## 🧭 화면 흐름 (초안)

```mermaid
flowchart TD
  Signup["회원가입"] --> Done["가입완료"]
  Done --> Login["로그인"]
  Login --> Home["홈: AI 인풋 (카테고리+키워드)"]
  Home -->|"주제 등록/검색 제출"| Alan["Alan API 호출"]
  Alan --> Summary["요약본 노트 페이지"]
  Home -->|"헤더/사이드바 내비게이션"| AllSummary["전체 요약 노트 페이지"]
  AllSummary -->|"카드 클릭"| Summary
  Home -->|"비로그인 상태 요약본 생성 시도"| AuthModal["로그인 권장 모달"]
  AuthModal --> Login
  Summary --> NoteWrite["학습노트 작성"]
  NoteWrite --> NoteDetail["학습노트 상세"]
  NoteDetail --> QuizModal["퀴즈 모달 (1회성)"]
  Summary --> Like["좋아요 / 북마크"]
  NoteDetail --> My["마이페이지"]
  Like --> My
  QuizModal --> My
  My --> Home
```

---

## 🧭 로드맵

### MVP

- 인증 (이메일+비번, 구글, 카카오)
- 홈 (AI 인풋)
- 주제 등록 → Alan 호출 → 요약본 노트 생성
- 요약본 노트 (개념요약, 참고자료, 좋아요/북마크)
- 학습노트 작성/상세 (topics 참조) + 퀴즈 모달
- 전체 요약 노트 페이지 (카테고리 필터, 무한스크롤 피드)
- 마이페이지 (학습노트/생성한 요약노트/북마크)
- 로딩·에러 처리 (명세 필수 항목)

### 스트레치

- 댓글 기능
- 관리자 페이지 (명세상 옵션)
- 통계/차트 (chart.js 등)

---

## 📌 미확정 사항 (TBD 목록)

- [ ] Next.js ?
- [ ] CSS ?

---

## 📌 상태

- 기획 논의 중 (MVP 초안 확정, 화면 흐름 정리 중)
