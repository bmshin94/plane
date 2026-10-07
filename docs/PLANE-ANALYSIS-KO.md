# Plane 전수조사 분석 및 수익화 전략 (한국어)

> 이 문서는 `bmshin94/plane` 저장소를 코드 레벨까지 전수조사하여 작성한 분석 보고서입니다.
> 작성일: 2026-10-07

## 📌 저장소 정보

| 항목 | 내용 |
|---|---|
| 내 저장소 | https://github.com/bmshin94/plane |
| 원본(업스트림) | https://github.com/makeplane/plane |
| 공식 웹사이트 | https://plane.so |
| 제품 문서 | https://docs.plane.so |
| 개발자 문서 | https://developers.plane.so |
| 셀프호스팅 가이드 | https://developers.plane.so/self-hosting/overview |
| 커뮤니티 포럼 | https://forum.plane.so |
| GitHub Discussions | https://github.com/orgs/makeplane/discussions |
| 버전 | 1.4.2 |
| 라이선스 | AGPL-3.0 |
| 작업 브랜치 | `claude/vibrant-keller-qiwb9z` |

---

## 1. 이게 뭔가 (전수조사 결과)

**Plane** — 지라(Jira)·리니어(Linear)를 대체하는 **오픈소스 프로젝트/이슈 관리 소프트웨어의 전체 소스코드**.
플러그인이나 작은 도구가 아니라, **SaaS 제품 한 개의 본체**입니다.

### 규모
- 추적 파일 **5,245개**
- Python **658개** (백엔드) / TypeScript·TSX **3,280개** (프론트엔드)
- DB 마이그레이션 **123개**
- 다국어 **20개** (한국어 `ko` 포함)

### `apps/` — 배포되는 6개 독립 서비스

| 폴더 | 정체 | 기술 스택 | 설명 |
|---|---|---|---|
| `apps/api` | 백엔드 두뇌 | Python / Django + DRF | 모든 데이터·권한·비즈니스 로직 |
| `apps/web` | 메인 앱 | React + React Router + Vite | 칸반/간트/스프린트 화면 (:3000) |
| `apps/admin` | God Mode 관리자 | React | 인스턴스 전체 설정 (:3001) |
| `apps/space` | 외부 공개 뷰 | React | 로그인 없이 보는 공개 보드 |
| `apps/live` | 실시간 협업 서버 | Node + Hocuspocus(Yjs) | 문서 동시 편집, Redis 동기화, PDF 내보내기 |
| `apps/proxy` | 게이트웨이 | Caddy | 라우팅 + Let's Encrypt SSL 자동 발급 |

### `packages/` — 16개 공용 라이브러리 (모노레포)

`ui`(디자인시스템+Storybook) · `editor`(리치텍스트) · `propel`(차트/시각화) · `types` ·
`services`(API 클라이언트) · `shared-state`(MobX) · `i18n`(20개 언어) · `constants` ·
`hooks` · `utils` · `logger` · `decorators` · `codemods` · `tailwind-config` · `typescript-config`

### `apps/api/plane/` — 백엔드 내부 구조

- **`db/models/`** — 데이터 모델 30여 개
  `issue`, `cycle`(스프린트), `module`, `project`, `page`, `workspace`,
  `intake`(외부 요청 수집함), `estimate`(스토리포인트), `draft`, `webhook`,
  `api`(API 토큰), `analytic`, `notification`, `favorite`, `importer`, `exporter`, `asset`
- **`api/`** — 외부 공개 REST API (API 키 인증)
- **`app/`** — 웹앱 전용 내부 API (세션 인증)
- **`space/`** — 공개 보드용 API
- **`authentication/`** — 이메일·매직링크 + OAuth 4종 (Google, GitHub, GitLab, **Gitea**)
- **`bgtasks/`** — Celery 백그라운드 작업 **35개**
  (이메일 알림, CSV/Excel 내보내기, 웹훅 발송, 이슈 자동화, 문서 버전 관리, 정리 작업)
- **`license/`** — 유료(EE) 기능 연동 지점
- **`middleware/`, `throttles/`** — API 레이트리밋 (기본 `60/minute`)

### 인프라 — `docker compose up` 시 기동되는 컨테이너 13개

```
web · admin · space · api · worker(Celery) · beat-worker(스케줄러)
migrator · live · plane-db(PostgreSQL) · plane-redis
plane-mq(RabbitMQ) · plane-minio(S3 호환) · proxy(Caddy)
```

배포 방식 4가지 (`deployments/`):
- **aio/community** — 올인원 단일 컨테이너 (supervisor)
- **cli/community** — 대화형 설치기 + 백업/복구 + **에어갭 복구** + 0.13→0.14 마이그레이션
- **kubernetes** — Helm 차트
- **swarm** — Docker Swarm

### `.claude/` — Claude Code 커스텀 스킬 5개 (이미 체크인되어 있음)

| 스킬 | 역할 |
|---|---|
| `branch-name` | `<type>/<이슈ID>-<설명>` 규칙 브랜치명 생성 |
| `create-pull-request` | PR 템플릿 맞춰 자동 PR 작성 |
| `release-notes` | 커밋 분석 → 릴리즈 노트 자동 생성 |
| `translate` | 20개 언어 번역 키 관리 (복수형·플레이스홀더 검증) |
| `react-doctor` | React 린트·접근성·번들·구조 진단 |

→ Plane 팀은 이미 **AI 에이전트를 공식 개발 워크플로우에 도입**했습니다.
`AGENTS.md`(AI용 개발 지침서)와 `CLAUDE.md`도 루트에 존재합니다.

---

## 2. 쉬운 비유로 이해하기

지라/리니어가 **임대 사무실**(매달 월세, 집주인 규칙)이라면,
Plane은 **IKEA 사무실 조립 키트 + 설계도면 전체**입니다. 공짜로 받아 내 땅에 직접 세웁니다.

### 폴더를 "식당"으로 바꾸면

| 폴더 | 식당의 무엇 |
|---|---|
| `apps/web` | 🍽 손님이 앉는 홀 (화면) |
| `apps/api` | 👨‍🍳 주방 — 실제로 일하는 곳 |
| `plane-db` (PostgreSQL) | 🧊 냉장고 — 모든 데이터 보관 |
| `worker` (Celery) | 🏃 설거지·배달 — 뒤에서 처리하는 일 |
| `plane-mq` (RabbitMQ) | 📋 주문표 꽂이 — 할 일 쪽지 큐 |
| `plane-redis` | 🗒 포스트잇 — 빠르지만 휘발성 |
| `plane-minio` | 📦 창고 — 사진·첨부파일 |
| `apps/live` | ✌️ 같은 메모지에 동시 낙서 (실시간 공동 편집) |
| `apps/admin` | 🔑 사장님 전용 방 (God Mode) |
| `apps/space` | 🪟 쇼윈도 — 외부 공개용 |
| `packages/*` | 🧂 공용 양념통 — 버튼·번역·아이콘 공유 |

### 제품 기능

- **Work Items** = 할 일 쪽지 한 장
- **Cycles** = 2주짜리 달리기 + 번다운 차트로 납기 경고
- **Modules** = 큰 덩어리를 쪼갠 서랍
- **Views** = 내 안경 ("내 긴급 건만 보여줘" 저장)
- **Pages** = 설명서. 한 줄을 클릭해 작업으로 변신
- **Intake** = 민원 접수함. 승인한 것만 정식 작업으로 승격
- **Estimate** = 난이도 점수 → 팀 속도를 숫자로
- **Analytics** = 성적표. 병목을 그래프로

### 겉으로 안 보이는 사실 5개

1. 프론트 코드가 백엔드의 **5배** (Python 658 : TS 3,280) — 화면에 손이 훨씬 많이 들어감
2. **한국어가 이미 포함** (`packages/i18n/src/locales/ko`) — 빈 키 채우는 게 가장 쉬운 기여 루트
3. **API 문이 3개** — `api/`(외부, API 키) / `app/`(웹앱, 세션) / `space/`(공개). 혼동하면 반나절 날림
4. **이미 AI와 함께 개발 중** — `.claude/skills/` 5개 + `AGENTS.md` 체크인
5. **RAM 12GB 사실상 필수** — 8GB는 도커 빌드 중 터짐 (CONTRIBUTING.md 명시)

---

## 3. 핵심 질문 7가지

### ① 설치 및 사용법

**사전 요구사항**
- Docker Engine 실행 중
- Node.js — 문서는 20+, 실제 `package.json engines`는 **>= 22.22.0**
- pnpm **11.10.0** 고정 (`packageManager`)
- Python 3.8+, PostgreSQL 14, Redis 6.2.7
- **RAM 최소 12GB 권장** (8GB 실패)

**경로 A — 프로덕션 자체 호스팅 (추천)**
```bash
git clone https://github.com/makeplane/plane.git plane
cd plane
chmod +x setup.sh && ./setup.sh     # .env 자동 생성
docker compose up -d                 # 13개 컨테이너 기동
```
- `.env`에서 `POSTGRES_PASSWORD`, `AWS_SECRET_ACCESS_KEY`, `SECRET_KEY` **반드시 변경**
- 도메인: `SITE_ADDRESS=plane.내도메인.com`, `CERT_EMAIL=내메일` → Caddy가 SSL 자동 발급
- **첫 접속은 `http://서버/god-mode/`** → 인스턴스 관리자 등록 (이게 먼저!)
- 그 다음 `http://서버` → 같은 계정 로그인

**경로 B — 개발**
```bash
./setup.sh
docker compose -f docker-compose-local.yml up   # 백엔드만 도커
pnpm dev                                        # 프론트 로컬 (web:3000, admin:3001)
```
`docker-compose-local.yml`에는 web/admin/space/live/proxy가 **없습니다**.

**주요 명령어 (AGENTS.md)**

| 명령 | 역할 |
|---|---|
| `pnpm dev` | 전체 개발 서버 |
| `pnpm build` | 전체 빌드 (turbo) |
| `pnpm check` | 포맷+린트+타입 검사 |
| `pnpm fix` | 자동 수정 |
| `pnpm turbo run <cmd> --filter=<패키지>` | 특정 패키지만 |
| `pnpm --filter=@plane/ui storybook` | 디자인시스템 (:6006) |
| `pnpm doctor` | react-doctor 진단 |

**백엔드 테스트 (Docker 격리)**
```bash
docker compose -f docker-compose-test.yml up --build \
  --abort-on-container-exit --exit-code-from api-tests
docker compose -f docker-compose-test.yml run --rm api-tests pytest -m unit
docker compose -f docker-compose-test.yml down -v
```

---

### ② 플러그인? 스킬? MCP? → 전부 아님. "완성된 서버 애플리케이션"

| 분류 | 맞나 | 설명 |
|---|---|---|
| 플러그인 | ❌ | 끼워넣는 부품이 아니라 **자기 자신이 본체** |
| 스킬 | ❌ | 단, `.claude/skills/` **5개가 안에 들어있음** |
| MCP 서버 | ❌ (CE엔 없음) | 떡밥 발견 — 아래 참고 |
| **정체** | ✅ | **Django + React + Celery 셀프호스팅 SaaS 제품** |

**🔍 MCP 떡밥 (중요한 발견)**

`packages/i18n/src/locales/*/tour.json`에 20개 언어 **전부** 이 문구가 있습니다:
```json
"mcp_connectors": {
  "step_zero": {
    "title": "탭 전환은 그만. 당신의 세계를 연결하세요.",
    "description": "GitHub, Slack을 연결하여 PR을 추적하고 Plane AI에서 직접 채팅을 요약하세요."
  }
}
```
→ **번역 문자열은 있는데 기능 코드는 이 저장소에 없습니다.**
MCP 커넥터 + Plane AI는 **유료(EE)/클라우드 전용**이고, CE에는 UI 텍스트만 남았습니다.
(`apps/api/plane/license/`가 그 연동 지점)

➡️ **셀프호스팅 사용자는 MCP를 쓸 수 없습니다. 직접 만들어야 하고, 그게 가장 가치 있는 작업입니다.**

---

### ③ API 토큰을 사용해야 돼? → 외부 자동화는 필수

**토큰 형태** (`apps/api/plane/db/models/api.py`)
```python
def generate_token():
    return "plane_api_" + uuid4().hex
```
→ `plane_api_` + 32자 hex

**인증 헤더** (`api_authentication.py`)
```python
auth_header_name = "X-Api-Key"
```
→ ⚠️ **`Authorization: Bearer`가 아닙니다.** `X-Api-Key`입니다.

```bash
curl -H "X-Api-Key: plane_api_xxxxxxxx" \
     https://내서버/api/v1/workspaces/<slug>/projects/
```

**APIToken 모델이 지원하는 것**

| 필드 | 의미 |
|---|---|
| `user_type` | `0=Human`, `1=Bot` → **봇 전용 토큰 개념 존재** |
| `workspace` | 워크스페이스 단위 격리 |
| `expired_at` | 만료일 (null = 무기한) |
| `is_active` | 즉시 비활성화 |
| `last_used` | 호출마다 갱신 |
| `allowed_rate_limit` | 토큰별 레이트리밋 (기본 `60/min`) |
| `is_service` | 서비스 계정 플래그 |

**감사 로그 (`APIActivityLog`)**
`path`, `method`, `query_params`, `headers`, `body`, `response_code`, `response_body`,
`ip_address`, `user_agent` 전부 DB 기록.
→ ⚠️ **요청/응답 본문이 저장됩니다.** 민감정보 정책 검토 필요. 반면 에이전트 디버깅엔 최고.

**세 가지 인증 경로 (핵심)**

| 경로 | 인증 | 용도 |
|---|---|---|
| `/api/v1/...` (`plane/api/`) | **X-Api-Key** | 외부 프로그램·에이전트 ⭐ |
| `/api/...` (`plane/app/`) | 세션 쿠키 | 웹앱 내부 전용 |
| `plane/space/` | 공개/토큰 | 외부 공개 보드 |

**API 문서 자체 제공** — `drf-spectacular` 내장
- `/schema/` → OpenAPI 스펙 JSON
- `/schema/swagger-ui/` → Swagger UI
- `/schema/redoc/` → Redoc

➡️ **OpenAPI 스펙 → LLM 툴 정의 자동 변환 가능.** 에이전트 연결에 이상적.

**웹훅도 있음** (`Webhook` 모델)
- `url`, `secret_key`(HMAC 서명), `WebhookLog`(발송 이력)
- 이벤트 토글 5개: **`project` / `issue` / `module` / `cycle` / `issue_comment`**
- 로컬 URL(`localhost`, `127.0.0.1`) 차단 — SSRF 방어
- ➡️ **토큰 = 에이전트 → Plane (Push), 웹훅 = Plane → 에이전트 (Pull).** 둘 다 쓰면 양방향 완성.

---

### ④ AI 에이전트 구축에 도움이 될까? → 매우 큼

**역할 1: 에이전트의 장기 기억 + 작업 상태 저장소**
- 상태(`state`), 담당자, 마감일, 우선순위, 의존관계(`IssueRelation`)가 DB에 영구 저장
- `IssueActivity`가 모든 변경 이력 기록 → **에이전트 행동 감사 추적**
- `estimate`로 "내 처리 속도"를 숫자로 파악

**역할 2: 멀티 에이전트의 공유 작업 큐 (칸반)**
```
[기획 에이전트] → Backlog에 작업 생성
        ↓ 웹훅
[개발 에이전트] → In Progress 이동, 코드 작성, PR을 Issue Link로 연결
        ↓ 웹훅
[리뷰 에이전트] → 코멘트 작성 → Done
```
→ 사람과 에이전트가 **같은 칸반 보드를 공유**. 사람이 "AI가 지금 뭘 하는지" 눈으로 봅니다.

**역할 3: 즉시 만들 수 있는 것 — Plane MCP 서버**
```
/schema/ (OpenAPI JSON) → MCP 툴 정의 자동 생성 → Claude/GPT 연결
```
만들 툴: `create_work_item`, `update_work_item`, `search_work_items`, `list_cycles`,
`add_to_cycle`, `create_module`, `add_comment`, `list_members`, `assign`,
`create_intake_request`, `upload_asset`

**참고할 패턴 (이미 코드에 존재)**

| 패턴 | 위치 |
|---|---|
| 비동기 에이전트 작업 큐 | `bgtasks/` 35개 Celery 태스크 |
| 에이전트 간 메시지 버스 | `plane-mq` (RabbitMQ) |
| 사람+AI 동시 문서 편집 | `apps/live` (Hocuspocus/Yjs) |
| **Human-in-the-loop 승인 플로우** | **`intake` 모델** ⭐ |

---

### ⑤ 수익화 아이디어 → 아래 4장 참고

**라이선스 경계선 요약 (AGPL-3.0)**

| 하는 일 | 결과 |
|---|---|
| 설치·구축·마이그레이션 대행 | ✅ 자유 |
| 운영·유지보수 구독 | ✅ 자유 |
| 교육/강의/책 | ✅ 자유 |
| 사내용 개조 (외부 미제공) | ✅ 자유 |
| **고친 Plane을 SaaS로 서비스** | ⚠️ 수정 소스 공개 의무 |
| **API만 호출하는 별도 제품** | ✅ 파생물 아님 → 상업화 자유 ⭐ |

---

### ⑥ React나 PHP로 만들 수 있어?

**React → 이미 React입니다** ✅
프론트 전체가 React + React Router + Vite + TypeScript + Tailwind + MobX.
- 화면 추가 → `apps/web/app` + `core`
- 컴포넌트 → `packages/ui` (Storybook)
- API 호출 → `packages/services`
- 상태 → `packages/shared-state` (MobX)
- ⚠️ 규칙: TS **strict**, 포맷 `oxfmt`, 린트 `oxlint`, 내부 패키지 `workspace:*`, 외부 `catalog:`

**PHP → 재구현은 비현실, 연동은 쉬움**

| 목표 | 가능성 |
|---|---|
| PHP(Laravel)에서 Plane API 호출 | ✅ 쉬움 (Guzzle + `X-Api-Key`) |
| PHP로 웹훅 수신 → 슬랙/카톡 알림 | ✅ 하루면 됨 |
| PHP로 Plane 데이터 대시보드 | ✅ **별도 제품 → 상업화 자유** ⭐ |
| PHP로 Plane 자체 재구현 | ❌ 비현실 (모델 30개 + 마이그레이션 123개 + Celery 35 + Yjs 서버) |

```php
// Laravel에서 Plane에 작업 생성
Http::withHeaders(['X-Api-Key' => config('plane.token')])
    ->post("https://plane.내서버/api/v1/workspaces/{$slug}/projects/{$id}/issues/", [
        'name' => '고객 문의: 결제 실패',
        'description_html' => '<p>...</p>',
        'priority' => 'urgent',
    ]);
```
➡️ **기존 PHP 사내 시스템(ERP/그룹웨어/CS)과 Plane을 잇는 어댑터**가 가장 수요 큼.

---

### ⑦ 유튜브 강의 영상 제작 가능할까? → 최적의 소재 ✅

**왜 좋은가**
- 검색 수요 확실 ("지라 대안", "무료 프로젝트 관리", "셀프호스팅")
- 결과가 눈에 보임 (칸반이 움직이는 화면 = 썸네일·리텐션 유리)
- 한국어 콘텐츠가 거의 비어있음
- 한국어 UI 기본 지원 → 진입장벽 낮음
- 난이도 스펙트럼 넓음: 설치(초급) → API(중급) → AI 에이전트(고급)

**추천 시리즈 15편**

*Season 1 — 설치와 활용 (초급, 조회수용)*
1. 지라 월 40만원 vs 0원 — Plane 소개 + 라이브 데모
2. 10분 설치: Docker로 내 서버에 올리기
3. 도메인 + HTTPS 자동 발급 (Caddy)
4. 팀 세팅: 워크스페이스/프로젝트/권한 + God Mode
5. 실전 애자일: Cycles + 번다운 차트 해석
6. Pages + Intake로 "노션 + 지라" 동시 대체

*Season 2 — 자동화 (중급, 전환율용)*
7. API 토큰 발급 + `X-Api-Key` 첫 호출 (+ Swagger UI 활용)
8. 웹훅으로 Slack/카톡 알림 (HMAC `secret_key` 검증까지)
9. 구글시트/노션/엑셀 ↔ Plane 양방향 동기화

*Season 3 — AI (고급, 차별화용) ⭐*
10. **Plane MCP 서버 직접 만들기** (OpenAPI → MCP 툴 자동 생성)
11. **Claude/GPT가 내 칸반을 직접 운영하게 하기** (Intake로 Human-in-the-loop)
12. **Plane 팀의 `.claude/skills/` 해부** — 실무 AI 개발 파이프라인

*Season 4 — 비즈니스*
13. 백업/복구/업그레이드 (`restore.sh`, 에어갭 복구)
14. AGPL 라이선스 — 뭘 팔 수 있고 뭘 팔면 안 되는지
15. Plane 구축 대행으로 월 수익 만들기

**영상에서 반드시 다룰 함정 8개 (신뢰도의 원천)**
1. ⚠️ RAM 12GB 미만이면 빌드 실패 — 1번 이탈 지점
2. ⚠️ 첫 접속은 `/god-mode/` 먼저. 모르면 로그인 자체가 안 됨
3. ⚠️ 헤더는 `Authorization: Bearer`가 아니라 **`X-Api-Key`**
4. ⚠️ `/api/v1/`(외부)과 내부 `app/` API 혼동 금지
5. ⚠️ `.env` 기본 비밀번호(`plane`/`plane`) 노출 = 사고
6. ⚠️ `OPENAI_API_KEY`/`GPT_ENGINE`은 **`# deprecated`** — 옛 튜토리얼 따라하면 안 됨
7. ⚠️ Node 22+, pnpm 11.10.0 버전 안 맞추면 설치 실패
8. ⚠️ API 요청/응답 본문이 `APIActivityLog`에 그대로 DB 저장 — 민감정보 주의

➡️ "Plane 설치법"은 영어로 몇 개 있지만, **"Plane + AI 에이전트"(10~12편)는 전 세계적으로 비어있습니다.**

---

## 4. 수익화 전략 (상세)

### 4.0 라이선스 지도 — 전략의 전제

AGPL-3.0의 핵심 조항:
> **"수정한 코드를 네트워크로 서비스하면, 그 수정 소스를 사용자에게 공개해야 한다."**

**🟢 안전지대 (소스 공개 의무 없음)**

| 행위 | 이유 |
|---|---|
| 설치/구축/마이그레이션 대행 | **노동** 판매 |
| 운영·유지보수·모니터링 구독 | **서비스** 판매 |
| 교육·강의·책·컨설팅 | **지식** 판매 |
| 사내 전용 개조 (외부 미서비스) | 배포가 아님 |
| **API만 호출하는 별도 제품** | **파생물이 아님** ⭐⭐ |

**🔴 위험지대**

| 행위 | 결과 |
|---|---|
| Plane을 고쳐서 SaaS 서비스 | 수정 소스 전부 공개 의무 |
| Plane 코드를 폐쇄 제품에 끼워 판매 | AGPL 위반 |
| 브랜딩만 바꿔 판매 | 가능하나 소스 공개 의무 |

> ⭐ **전략 축: "Plane을 팔지 마라. Plane 주변의 노동·지식·연결장치를 팔아라."**
> 특히 **API만 호출하는 독립 제품(MCP 서버, 봇, 리포트, 어댑터)은 AGPL에 전혀 묶이지 않습니다.**

---

### 티어 1 — 당장 시작 (자본 0원, 1~4주)

#### 아이디어 1. Plane 구축 대행

| 상품 | 포함 | 가격(제안) |
|---|---|---|
| 베이직 | 설치, HTTPS, 백업 설정, 1시간 교육 | **80만 원** |
| 스탠다드 | + 지라/트렐로/노션 데이터 이관, 워크플로우 설계 | **250만 원** |
| 엔터프라이즈 | + SSO(Google/GitHub/GitLab/Gitea), K8s, 사내 연동 | **600만 원+** |

**경쟁 우위 (전수조사로 확보)**
- `deployments/cli/community/`에 **백업(`restore.sh`), 에어갭 복구(`restore-airgapped.sh`), 버전 마이그레이션** 스크립트 존재 → "데이터 안전" 설득 가능
- RAM 12GB 함정, `/god-mode/` 선등록 순서 등 **실패 포인트를 아는 것이 곧 전문성**
- OAuth 4종에 **Gitea** 포함 → 사내 Gitea 쓰는 한국 기업에 바로 적용

**타겟:** 개발팀 5~50명, 지라에 연 300만~1,500만 원 쓰는 회사
**세일즈 한 문장:** "1년 구독료보다 싼 일회성 비용으로 영구 해방"

#### 아이디어 2. 운영 관리(MSP) 월 구독 ⭐가장 안정적

| 플랜 | 내용 | 월 가격 |
|---|---|---|
| Care | 모니터링, 자동 백업 검증, 보안 패치 | **15만 원** |
| Care+ | + 버전 업그레이드 대행, 월 2시간 지원 | **35만 원** |
| Managed | + 전담 운영, 4시간 내 장애 대응 SLA | **80만 원+** |

**왜 팔리나:** 컨테이너 13개를 혼자 돌보는 건 중소기업 개발자에게 지옥.
설치는 공짜여도 **운영은 아무도 못 합니다.** 고객 10곳 = 월 150~350만 원 반복 수익.

#### 아이디어 3. 유튜브 + 디지털 상품

```
유튜브 Season 1 (무료)              → 신뢰 + 리드
    ↓
유료 강의 (Season 2~3): 15만원 × 200명 = 3,000만 원
    ↓
전자책 "Plane 완전 정복": 2.5만 원
    ↓
설치 스크립트/템플릿 번들: 9만 원
    ↓
구축 대행 리드 유입 (아이디어 1·2) ⭐ 진짜 돈은 여기
```
**핵심:** 강의 수익보다 **"강의 → 구축 문의" 전환**이 본체. 영상 1편 = 영구 영업사원.

---

### 티어 2 — 제품 만들기 (1~3개월) ⭐AGPL 자유지대

#### 아이디어 4. Plane MCP 서버 (상업용) ⭐⭐⭐ 최고 추천

**왜 지금인가 (전수조사로 확인한 사실)**
- `tour.json`에 `mcp_connectors` 문자열이 20개 언어 전부 존재
- 그런데 **기능 코드는 CE 저장소에 없음** → MCP/Plane AI는 유료 EE/클라우드 전용
- **셀프호스팅 사용자는 MCP를 못 씁니다.** 시장은 비었고, 수요는 Plane이 직접 증명

**왜 만들기 쉬운가**
- `drf-spectacular`가 `/schema/`로 OpenAPI 스펙 자동 노출 → 툴 정의 자동 생성
- 인증이 헤더 한 줄 (`X-Api-Key`)
- API 호출만 → **AGPL 무관, 내 라이선스로 판매 자유**

| 버전 | 내용 | 가격 |
|---|---|---|
| OSS | 읽기 전용 툴 | 무료 (마케팅) |
| Pro | 쓰기 툴 + 멀티 워크스페이스 + 감사로그 | **$19/월** |
| Team | + 역할별 권한, 승인 플로우(Intake) | **$99/월** |
| Self-hosted | 온프레미스 라이선스 | **연 $2,000** |

**킬러 기능 — Human-in-the-loop (이미 Plane에 구현됨)**
`intake` 모델이 "외부 요청 접수 → 승인/거부/스누즈" 구조.
AI가 만든 작업을 Intake에 넣으면 **사람이 승인한 것만 정식 작업**이 됩니다.
기업이 AI 도입에서 가장 무서워하는 "AI가 멋대로 일 저지름"을 구조적으로 해결 → **세일즈 1순위**

#### 아이디어 5. 양방향 연동 어댑터 (Integration Hub)

웹훅 5개 이벤트(`project`/`issue`/`module`/`cycle`/`issue_comment`)를 받아 흘려보내는 커넥터.

| 커넥터 | 수요 |
|---|---|
| **카카오워크 / 네이버웍스 / 잔디** | 🇰🇷 한국 기업 필수. **전 세계에서 아무도 안 만듦** ⭐ |
| Slack / Teams / Discord | 범용 |
| GitHub·GitLab·Gitea 양방향 (커밋↔작업) | 개발팀 필수 |
| Google Sheets / 엑셀 자동 리포트 | 비개발 경영진용 |
| ERP·그룹웨어 (PHP/Java 레거시) | 고단가 SI |

**가격:** 커넥터당 월 $9~29, 또는 구축형 건당 150만~500만 원
**차별화:** `secret_key` HMAC 서명 검증을 제대로 구현 → "보안 검증된 커넥터"

#### 아이디어 6. 경영진용 분석 대시보드 (React/PHP 자유)

Plane의 `Analytics`는 실무자용. 경영진은 다른 걸 원합니다 —
"분기 팀별 처리량", "병목 위치", "번다운 추세로 본 납기 리스크".

**근거 데이터가 이미 전부 존재:** `analytic.py`, `estimate.py`, `IssueActivity`(변경 이력), `cycle.py`
→ **리드타임·사이클타임·처리량·CFD(누적흐름도) 전부 계산 가능**

API만 읽으므로 React/Next.js든 **PHP/Laravel이든 자유** → AGPL 무관 독립 제품
**가격:** 월 $29~99 또는 구축 300만~800만 원

#### 아이디어 7. 한국형 배포판 "Plane Korea Edition"

- 한국어 번역 완성본 (`locales/ko` 빈 키 전부)
- 공휴일 반영 캘린더 / 근무일 기준 번다운
- 카카오워크·네이버웍스 알림 내장
- 네이버클라우드·KT클라우드·NHN클라우드 원클릭 설치
- 한국어 매뉴얼 + 영상

⚠️ **코드 수정 배포 → 수정분 소스 공개 의무**
✅ **해법:** 코드는 공개하고, **돈은 설치+지원+교육 구독으로** (Red Hat 모델)
**가격:** 패키지 무료 + 지원 구독 연 300만~1,200만 원

---

### 티어 3 — 장기 사업 (6개월+)

#### 아이디어 8. 산업 특화 버전 (버티컬 SaaS)

| 산업 | 필요한 추가 | 왜 돈이 되나 |
|---|---|---|
| **건설/시공** | 공정표, 자재, 현장사진(Asset), 안전점검 | 지라를 못 쓰는 산업, 경쟁 적음 |
| **병원/임상연구** | IRB 승인 플로우, 환자 비식별, 감사추적 | **데이터 외부 유출 불가 → 셀프호스팅 필수** ⭐ |
| **법무법인** | 사건 단위, 기한 자동계산, 타임시트 | 시간당 과금과 직결 |
| **공공/국방** | 망분리, 에어갭 운영 | `restore-airgapped.sh` 이미 존재 ⭐⭐ |
| **영상/광고 제작** | 촬영 일정, 피드백 라운드, 납품 승인 | 노션으로 버티는 시장 |

**결정적 근거:** 이 산업들은 법규·보안 때문에 **클라우드 SaaS를 못 씁니다.**
지라 클라우드/리니어가 애초에 진입 불가. **셀프호스팅이 선택이 아니라 요구조건인 시장** = Plane의 독점 영역.
**가격:** 연 1,500만~1억 원 (SI 성격)

#### 아이디어 9. "AI 프로젝트 매니저" 제품 ⭐가장 큰 그림

Plane을 **데이터베이스**로만 쓰고 그 위에 AI PM을 올립니다.

```
[매일 아침] 에이전트가 전체 작업 스캔
   → "이 3건은 3일째 정지. 담당자 확인 필요?"
   → "번다운 추세상 이번 사이클 2건 미달 예상. 범위 조정 제안"
   → "A님 11건, B님 2건. 재분배안 생성"
   → Intake에 제안 등록 → 사람이 승인 ✅
```

| 필요 기능 | Plane의 어디 |
|---|---|
| 작업 데이터 읽기 | `/api/v1/` + OpenAPI 스펙 |
| 변경 감지 | `Webhook` 5개 이벤트 |
| 이력 분석 | `IssueActivity` |
| 속도 계산 | `estimate` + `cycle` |
| **승인 플로우** | **`intake`** ⭐ |
| 비동기 처리 패턴 | `bgtasks/` 35개 Celery 태스크 |
| 문서 공동 편집 | `apps/live` (Hocuspocus/Yjs) |

**가격:** 사용자당 월 $15~30 — "PM 연봉의 1/20로 PM 보조 고용"
**AGPL:** API만 호출 → **완전 자유** ✅

#### 아이디어 10. 교육·자격 사업
- 기업 출장 교육: 1일 **200만 원**
- 온라인 부트캠프: 80만 원 × 50명 = **4,000만 원**
- "Plane 공인 파트너" 인증 + 리드 배분 수수료 20%

---

### 4.1 실행 로드맵

```
Month 1-2   유튜브 Season 1 공개 (무료)           → 신뢰 + 리드
            └ 동시에 구축 대행(①) 수임 시작         → 즉시 현금

Month 2-4   Plane MCP 서버 OSS 공개 (④)           → 글로벌 인지도 ⭐
            └ 유튜브 Season 3가 그대로 홍보물

Month 3-6   MCP Pro/Team 유료화 + 카카오워크 커넥터(⑤)
            └ 구축 고객이 그대로 첫 구독자

Month 4-8   MSP 구독 전환 (②)                     → 반복 수익 안정화

Month 6-12  버티컬 1개 선택 (⑧)                   → 고단가
            추천: 병원 또는 공공 (셀프호스팅 필수 시장)

Year 2      AI PM 제품 (⑨)                        → 확장
```

### 4.2 수익 규모 시뮬레이션 (보수적)

| 시점 | 구성 | 월 매출 |
|---|---|---|
| 3개월 | 구축 2건 | 약 300만 원 (일회성) |
| 6개월 | 구축 2건 + MSP 5곳 + MCP 20구독 | 약 500만 원 |
| 12개월 | 구축 3건 + MSP 15곳 + MCP 150구독 + 강의 | 약 1,500만 원 |
| 24개월 | + 버티컬 1건 + AI PM 초기 | 약 3,500만 원 |

### 4.3 리스크와 대응

| 리스크 | 대응 |
|---|---|
| Plane이 MCP를 CE에 무료 공개 | 한국형 커넥터·버티컬·승인플로우로 차별화 유지 |
| AGPL 위반 리스크 | **코드 수정 배포 금지. 독립 제품 + 서비스로 설계** |
| Plane 프로젝트 중단 | AGPL이라 포크 가능. MSP/교육 수익은 영향 적음 |
| 중소기업 지불 능력 | "지라 1년 구독료 < 영구 구축비" 프레임 |

### 💎 한 문장 결론

> **Plane 자체로 돈 벌려 하지 마세요. Plane이 비워둔 세 칸을 채우세요 —**
> **① 셀프호스팅용 MCP 서버, ② 한국 기업용 연동(카카오워크·네이버웍스), ③ 클라우드를 못 쓰는 산업(병원·공공).**
>
> 이 세 칸은 AGPL에 묶이지 않고, 경쟁자가 없고, Plane이 수요를 직접 증명해줬습니다.

---

## 5. 빠른 참조 (치트시트)

### 설치
```bash
git clone https://github.com/makeplane/plane.git plane && cd plane
chmod +x setup.sh && ./setup.sh
docker compose up -d
# 1) http://서버/god-mode/ → 인스턴스 관리자 등록
# 2) http://서버 → 로그인
```

### API 호출
```bash
curl -H "X-Api-Key: plane_api_xxxx" \
  https://내서버/api/v1/workspaces/<slug>/projects/
```

### 주요 엔드포인트 (`apps/api/plane/api/urls/`)
`work_item` · `project` · `cycle` · `module` · `state` · `label` · `member` ·
`estimate` · `intake` · `invite` · `asset` · `sticky` · `user` · `schema`

### API 문서
- `/schema/` — OpenAPI JSON
- `/schema/swagger-ui/` — Swagger UI
- `/schema/redoc/` — Redoc

### 웹훅 이벤트
`project` · `issue` · `module` · `cycle` · `issue_comment` (+ HMAC `secret_key` 서명)

### 체크리스트 — 반드시 확인
- [ ] RAM 12GB 이상
- [ ] Node >= 22.22.0, pnpm 11.10.0
- [ ] `.env` 기본 비밀번호 전부 변경
- [ ] `/god-mode/` 먼저 등록
- [ ] 헤더는 `X-Api-Key` (Bearer 아님)
- [ ] `OPENAI_API_KEY`/`GPT_ENGINE`은 deprecated — 쓰지 말 것
- [ ] `APIActivityLog`에 요청/응답 본문 저장됨 (민감정보 정책 검토)

---

*이 문서는 저장소 전체(5,245개 파일)를 코드 레벨까지 조사하여 작성되었습니다.*
*저장소: https://github.com/bmshin94/plane · 원본: https://github.com/makeplane/plane*
