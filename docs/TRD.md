# 기술 요구사항 및 설계

## 구성
프론트엔드는 Next.js App Router, TypeScript strict, Tailwind CSS, shadcn/ui, Lucide, TanStack Query 및 필요한 경우 Recharts를 사용한다. 백엔드는 Python 3.12+, FastAPI, Pydantic, httpx, PyMuPDF와 독립 실행 가능한 스케줄러를 사용한다. 저장소와 인증은 Supabase PostgreSQL 및 Auth, 스키마는 SQL 마이그레이션을 사용한다. 복잡한 에이전트 프레임워크는 필요할 때만 도입한다.

수집기 → 정규화 → 중복 제거 → 문서 처리 → 순차 분석·근거 검증 → 저장의 책임을 분리한다. 원본 저장은 AI 처리와 독립시킨다. 스케줄러는 웹 서버와 별도로 실행할 수 있어야 하며 동시 실행을 방지한다. LLM은 교체 가능한 공통 인터페이스와 Mock 구현을 제공한다.

## 데이터 및 권한
`sources`, `source_documents`, `problems`, `problem_evidence`, `problem_requirements`, `project_ideas`, `saved_problems`, `collection_jobs`, `analysis_jobs`를 마이그레이션으로 생성한다. 외래키, 원천별 고유 식별자 제약과 검색·조회 인덱스를 둔다. 원본 payload, 추출 텍스트, 분석 모델·프롬프트 버전, 작업 상태와 오류를 추적한다. 사용자 저장 데이터는 소유자 RLS를 적용하고 관리자 작업 데이터는 공개하지 않는다. 서비스 역할 키는 서버에서만 사용한다.

## 연동 및 환경
- 입찰공고: https://www.data.go.kr/data/15129394/openapi.do
- 사전규격: https://www.data.go.kr/data/15129437/openapi.do
- 선택적 기업마당: https://www.bizinfo.go.kr/apiDetail.do?id=bizinfoApi

오퍼레이션, 필드 및 인증 방식은 구현 전에 공식 문서와 실제 응답으로 검증한다. 날짜 분할은 요청당 7일 이하로 제한한다. API 실패·재시도·호출량을 기록하고 비밀값은 로그에서 제외한다.

`.env.example`에 `NEXT_PUBLIC_APP_URL`, `API_BASE_URL`, `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY`, `SUPABASE_SERVICE_ROLE_KEY`, `DATABASE_URL`, `DATA_GO_KR_SERVICE_KEY`, `BIZINFO_API_KEY`, `LLM_PROVIDER`, `LLM_API_KEY`, `LLM_MODEL`, `COLLECTION_ENABLED`, `AI_ANALYSIS_ENABLED`, `MOCK_MODE`를 문서화한다. 기본적으로 수집·AI는 비활성화하고 Mock을 활성화한다. 실제 비밀은 Git에 추가하지 않는다.

## 검증과 배포
pnpm, Vitest, Playwright, pytest, Ruff, ESLint 및 TypeScript 검사를 사용한다. 프론트엔드는 Vercel 또는 동급, DB는 Supabase, Python과 스케줄러는 별도 실행 가능한 호스팅을 목표로 한다. 로컬 Docker는 필수가 아니다. 인증 전에는 Mock으로 검증하고 실제 서비스 설정 후 마이그레이션과 실연동을 별도 검증한다.
