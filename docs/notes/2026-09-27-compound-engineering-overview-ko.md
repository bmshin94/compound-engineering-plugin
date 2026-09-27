# Compound Engineering 플러그인 분석 정리 (대화 기록)

- 작성일: 2026-09-27
- 분석 대상 저장소(포크): https://github.com/bmshin94/compound-engineering-plugin
- 원본 저장소: https://github.com/EveryInc/compound-engineering-plugin
- 분석 기준 버전: 3.26.3 (MIT 라이선스)
- 참고 글: https://every.to/chain-of-thought/compound-engineering-how-every-codes-with-agents

---

## 1. 전수조사 결과: 뭐하는 건지 / 언제 쓰는지 / 나한테 어떤 도움이 되는지

### 한 줄 정의

**AI 코딩 에이전트(Claude Code, Cursor, Codex 등)에 설치하는 "일하는 방법 모음" 플러그인.**
스킬 35개가 들어 있고, 에이전트가 `기획 -> 계획 -> 구현 -> 정리 -> 리뷰 -> 배운 점 기록` 순서로 일하게 만든다.
핵심은 마지막 단계(`ce-compound`)에서 배운 것을 `docs/solutions/`에 써두고, 다음 작업 때 에이전트가 그걸 읽어서 **같은 실수를 반복하지 않게** 하는 것. 그래서 이름이 "복리(Compound) 엔지니어링".

### 폴더 구조

| 폴더/파일 | 내용 |
|---|---|
| `skills/` | 플러그인의 본체. 스킬 35개, 각 폴더에 `SKILL.md`(지시문) + `references/`(세부 지침, 리뷰어 페르소나) + `scripts/`(bash/python 보조 스크립트) |
| `src/` | Bun/TypeScript CLI. Claude Code 플러그인 형식을 다른 에이전트(Codex, Copilot, OpenCode, Pi, Kiro, Droid, Antigravity) 형식으로 변환 |
| `.claude-plugin/`, `.cursor-plugin/`, `.codex-plugin/`, `.grok-plugin/`, `.kimi-plugin/`, `.devin-plugin/`, `.omp-plugin/`, `.opencode/`, `.pi/`, `.cline/`, `.agy/` | 에이전트 호스트별 설치 매니페스트(14개 호스트 지원) |
| `docs/guides/` | 스킬별 사용자 설명서 |
| `docs/solutions/` | 이 저장소 스스로 쌓은 "배운 점" 문서 63개 (플러그인이 자기 자신에게 복리 방식을 적용한 결과물) |
| `docs/plans/`, `docs/brainstorms/`, `docs/specs/` | 기획서, 계획서, 호스트별 포맷 사양 |
| `tests/` | 테스트 파일 158개 (변환기, 스킬 규칙, 스크립트 동작 검증) |
| `site/` | Jekyll 문서 사이트 |
| `scripts/release/` | 릴리스 자동화 |
| `AGENTS.md`, `CONCEPTS.md`, `STRATEGY.md` | 개발 규칙, 용어집, 프로젝트 전략 |

### 스킬 35개 분류

| 그룹 | 스킬 | 쉽게 말하면 |
|---|---|---|
| 핵심 루프 | `ce-brainstorm` `ce-plan` `ce-work` `ce-simplify-code` `ce-code-review` `ce-compound` | 아이디어 구체화 -> 계획서 -> 구현 -> 코드 다듬기 -> 다중 리뷰어 리뷰 -> 배운 점 기록 |
| 루프 주변 | `ce-strategy` `ce-product-pulse` `ce-sweep` `ce-compound-refresh` | 전략 문서, 제품 지표 리포트, Slack/GitHub 피드백 수집, 오래된 학습 문서 정리 |
| 필요할 때 | `ce-ideate` `ce-bakeoff` `ce-pov` `ce-debug` `ce-explain` `ce-doc-review` `ce-optimize` `ce-prototype` | 아이디어 발굴, 여러 안 경쟁시키기, 의견/판정, 디버깅, 코드 설명, 문서 리뷰, 성능 최적화, 시제품 |
| Git 작업 | `ce-commit` `ce-commit-push-pr` `ce-babysit-pr` `ce-resolve-pr-feedback` `ce-worktree` | 커밋, PR 생성, PR 지켜보기, 리뷰 코멘트 반영, 워크트리 |
| 자동 파이프라인 | `lfg` | 위 과정을 사람 개입 없이 PR까지 한 번에 |
| 테스트/디자인 | `ce-test-browser` `ce-test-xcode` `ce-polish` `ce-dogfood` | 브라우저 테스트, iOS 시뮬레이터 테스트, UX 다듬기, 자동 QA |
| 협업 | `ce-proof` `ce-handoff` `ce-promote` | 문서 공유, 다른 에이전트에게 인수인계, 홍보 문구 작성 |
| 유틸리티 | `ce-setup` `ce-noslop` `ce-retune` `ce-riffrec-feedback-analysis` | 초기 설정/점검, AI 티 안 나는 글쓰기, 새 모델용 스킬 재튜닝, 화면녹화 피드백 분석 |

`ce-code-review`에는 보안, 성능, 정확성, 테스트, API 계약, 데이터 마이그레이션, iOS 등 리뷰어 페르소나가 20명 가까이 들어 있어서 한 번의 리뷰를 여러 전문가 관점으로 돌린다.

### 언제 쓰나

- 기능 하나를 제대로 만들고 싶을 때: `/ce-brainstorm` -> `/ce-plan` -> `/ce-work` -> `/ce-code-review` -> `/ce-compound`
- 알아서 끝까지 해줬으면 할 때: `/ce-brainstorm` 후 `/lfg`
- 버그가 났을 때: `/ce-debug`
- 뭘 만들지 모를 때: `/ce-ideate`
- 남의 코드가 이해 안 될 때: `/ce-explain`

### 나한테 도움이 되는 점

1. AI가 "대충 바로 코딩"하는 대신 계획과 리뷰를 먼저 하게 되어 결과물 품질이 올라간다.
2. 프로젝트마다 겪은 함정이 `docs/solutions/`에 쌓여서 다음 작업부터 AI가 알아서 피한다.
3. Claude Code, Cursor, Codex 등 도구를 바꿔도 같은 방식으로 일할 수 있다.
4. 커밋, PR, 리뷰 반영 같은 반복 작업이 줄어든다.
5. 스킬 파일 자체가 잘 쓴 프롬프트 교과서라서, 나만의 에이전트/스킬을 만들 때 참고서가 된다.

---

## 2. 더 쉽게 설명

AI 코딩 도우미를 **신입 개발자**라고 생각하면 된다. 머리는 좋은데 기억력이 없어서 매일 아침 어제 일을 다 잊고 온다.

이 플러그인은 그 신입에게 주는 **업무 매뉴얼 + 업무 일지**다.

- 매뉴얼(스킬 35개): "코딩 전에 먼저 뭘 만들지 물어보고, 계획서 쓰고, 다 만들면 리뷰받아" 같은 일하는 순서
- 업무 일지(`docs/solutions/`): "이 프로젝트에서 환경변수 설정할 때 이런 함정 있었음" 같은 기록

첫 번째 작업에서 겪은 문제를 일지에 쓰고, 두 번째 작업 때 AI가 일지를 먼저 읽는다. README의 표현 그대로 **"첫 번째 실행이 가르치고, 두 번째 실행이 기억한다."**

요리로 비유하면: 레시피 없이 매번 감으로 요리하던 사람에게 레시피북(스킬)과 실패노트(solutions)를 쥐여준 것. 요리할수록 노트가 두꺼워지고 실패가 줄어든다.

---

## 3. 자주 묻는 질문

### 설치 및 사용법

Claude Code 기준:

```text
/plugin marketplace add EveryInc/compound-engineering-plugin
/plugin install compound-engineering
```

이미 설치되어 있다면 업데이트 전에 마켓플레이스를 먼저 새로고침해야 한다(`docs/install/upgrading.md`).

Cursor: 에이전트 채팅에서 `/add-plugin compound-engineering`

Codex CLI:

```bash
codex plugin marketplace add EveryInc/compound-engineering-plugin
codex plugin add compound-engineering@compound-engineering-plugin
```

그 밖에 Copilot, Grok, Kimi, Devin, OpenCode, Pi, Qwen, Factory Droid, Cline, Antigravity도 README의 "More Install Options" 참고.

설치 후 첫 사용:

```text
/ce-setup                                   # 환경 점검 + 설정 파일 생성
/ce-brainstorm 로그인 기능에 소셜 로그인 추가
/ce-plan
/ce-work
/ce-simplify-code
/ce-code-review
/ce-compound
```

자동 모드: `/ce-brainstorm 기능 설명` 후 `/lfg`. Codex에서는 `/` 대신 `$`(예: `$ce-plan`).

선택 도구(없어도 동작, 있으면 기능 확장): `gh`(GitHub CLI), `agent-browser`(브라우저 테스트), `jq`, `ast-grep`, `ffmpeg`.

### 플러그인이야? 스킬이야? MCP야?

**플러그인이다. 그 안에 스킬 35개가 들어 있다. MCP 서버는 아니다.**

- 스킬: 에이전트가 읽는 지시문 파일(`SKILL.md`). 특정 작업 방법을 알려준다.
- 플러그인: 스킬 여러 개를 묶어서 한 번에 설치/업데이트할 수 있게 만든 패키지.
- MCP: 에이전트에 외부 도구(DB, API 등)를 연결하는 서버 프로토콜. 이 플러그인은 MCP 서버를 제공하지 않는다. 다만 일부 스킬이 이미 설치된 외부 도구(예: iOS 테스트의 XcodeBuildMCP, GitHub CLI)를 활용한다.

### API 토큰이 필요해?

**기본적으로 별도 토큰 불필요.** 쓰고 있는 AI 에이전트(Claude Code 구독 등)만 있으면 된다. 스킬은 그 에이전트 안에서 실행되고 사용량도 그 에이전트 요금제에서 차감된다. 플러그인 자체는 무료(MIT)이며 텔레메트리(사용 데이터 수집)도 없다.

선택적으로 필요한 경우:

| 기능 | 필요한 것 |
|---|---|
| PR 생성/리뷰 반영 | `gh auth login` (GitHub 로그인) |
| `ce-riffrec-feedback-analysis`, `ce-sweep`의 음성 전사 | `OPENAI_API_KEY` (없으면 전사만 건너뜀) |
| `ce-sweep` Slack 수집 | Slack 연결 |
| `ce-pov` 오라클, 크로스 모델 리뷰 | 다른 에이전트 CLI(Codex 등)가 설치/로그인되어 있을 때만 |
| `ce-proof` | Proof 서비스 |

### 왜 GitHub에서 유명할까?

1. 뉴스레터/미디어 회사 Every(every.to)가 실제 사내 개발에 쓰는 방식을 공개했고, "Compound Engineering" 글이 개발자 커뮤니티에서 많이 퍼졌다.
2. "AI는 매번 까먹는다"는 모두가 겪는 문제에 대한 구체적인 해법을 제시한다.
3. 14개 에이전트 호스트를 지원해서 도구를 가리지 않는다.
4. 새 모델이 나올 때마다 스킬을 재튜닝하는 등 유지보수가 매우 활발하다(테스트 158개, 릴리스 자동화).
5. MIT 라이선스로 누구나 가져다 쓰고 고칠 수 있다.
6. 스킬 파일 자체가 프롬프트 엔지니어링 교재로 쓸 만큼 품질이 높다.

### 로컬 에이전트 구축에 도움이 될까?

**설계 참고서로는 매우 유용, 그대로 쓰기엔 조건부.**

- 도움되는 점: 계획 -> 실행 -> 리뷰 -> 기억 저장 구조, 리뷰어 페르소나를 서브에이전트로 돌리는 방식, `docs/solutions/`를 장기 기억(간이 RAG)으로 쓰는 방식, 스킬을 한 번 작성해 여러 호스트로 변환하는 방식(`src/converters/`)을 그대로 배울 수 있다.
- 로컬 모델 사용: OpenCode나 Pi 같은 호스트는 Ollama 등 로컬 모델을 붙일 수 있고, 이 플러그인도 설치된다.
- 한계: 스킬이 최신 대형 모델 기준으로 튜닝되어 지시문이 길고 서브에이전트를 많이 쓴다. 작은 로컬 모델은 지시를 놓치거나 느릴 수 있다. 핵심 스킬(`ce-plan`, `ce-code-review`, `ce-compound`)만 골라 가볍게 쓰는 것을 추천.

### 수익화할 만한 아이디어가 있어?

있다. 상세 내용은 4장. 전제: MIT 라이선스라서 상업적 이용, 수정, 재판매 모두 가능하다. 단 저작권/라이선스 고지는 유지해야 하고, "Every"나 원작자의 공식 제품인 것처럼 보이게 하면 안 된다. 원작자는 이 플러그인 자체를 유료화하지 않는다고 `STRATEGY.md`에 명시했으므로, 돈은 **플러그인 판매가 아니라 그 위의 서비스/콘텐츠/도구**에서 만든다.

### React나 PHP로 만들 수 있어?

**가능하다. 다만 "무엇을" 만드느냐에 따라 다르다.**

- 이 플러그인으로 React/PHP 프로젝트를 개발하기: 바로 가능. 언어를 가리지 않는다. React 프로젝트에 설치하면 `ce-test-browser`, `ce-polish`, `ce-dogfood` 같은 프론트엔드 스킬까지 활용할 수 있다.
- 이 플러그인 자체를 React/PHP로 다시 만들기: 의미가 적다. 플러그인은 앱이 아니라 마크다운 지시문 + 보조 스크립트 묶음이라서, 에이전트가 읽는 형식은 언어와 무관하다.
- 이 플러그인을 활용한 제품을 React/PHP로 만들기: 추천. 예) `docs/solutions/`, `docs/plans/`를 팀이 웹에서 검색/열람하는 지식 대시보드(React 프론트 + PHP/Laravel API), 고객사별 스킬/팩 관리 콘솔, 설치/설정 마법사 웹페이지 등.
- 나만의 스킬 만들기: `SKILL.md` 형식만 따르면 되고, 예를 들어 "Laravel 규칙 리뷰어", "우리 회사 React 컨벤션 리뷰어"를 Compound Pack이나 페르소나로 추가할 수 있다.

---

## 4. 수익화 아이디어 상세

가격은 시장 조사 전 가설값이다. 실제로는 소규모 파일럿으로 검증 후 조정한다.

### 아이디어 1. 팀 도입 컨설팅 / 셋업 대행

- 대상: AI 코딩 도구는 샀는데 활용법이 제각각인 5~50인 개발팀, 에이전시
- 제공: 팀 레포에 플러그인 설치, `ce-setup` 설정, 팀 코딩 규칙을 `CODING_STANDARDS.md`와 Compound Pack으로 정리, 첫 기능 하나를 루프로 같이 완주, 사내 교육 1회
- 가격 가설: 셋업 패키지 1건 200만~800만 원, 월 유지 자문 50만~200만 원
- 시작 방법: 우리 프로젝트(React/PHP)에 먼저 적용해 전후 비교(리뷰에서 잡은 버그 수, 반복 실수 감소)를 사례로 만든다
- 리스크: 도구 변화가 빨라 교육 자료가 금방 낡음 -> 월 유지 계약으로 업데이트를 판매 포인트로

### 아이디어 2. 업종/스택별 Compound Pack 판매

- 개념: Compound Pack은 "planning이 참고하고 review가 강제하는 규칙 폴더". 이걸 스택/업종별로 잘 만들어 판다
- 예시: Laravel 보안/성능 팩, React + Next.js 컨벤션 팩, 한국 전자금융/개인정보보호 체크 팩, 쇼핑몰(PG 결제, 재고) 도메인 팩
- 판매 방식: 비공개 Git 저장소 접근권 판매(Gumroad, 자체 결제), 연 구독으로 업데이트 제공
- 가격 가설: 팩 1개 3만~10만 원, 팀 라이선스 연 30만~100만 원
- 리스크: 복제가 쉬움 -> 규칙보다 "지속 업데이트 + 사례 + 지원"을 판다

### 아이디어 3. 한국어 강의 / 콘텐츠

- 대상: AI 코딩을 제대로 배우고 싶은 국내 개발자, 비개발 창업자
- 제공: "Claude Code + Compound Engineering으로 React/PHP 서비스 만들기" 실전 강의(인프런, 클래스101, 유튜브 멤버십), 전자책, 오프라인 워크숍
- 차별점: 한국어 자료가 부족하고, 실제 서비스 하나를 루프로 완성하는 과정을 보여줄 수 있음
- 가격 가설: 온라인 강의 5만~20만 원, 워크숍 1인 20만~50만 원, 기업 출강 회당 100만 원 이상
- 시작 방법: 무료 유튜브/블로그로 설치법과 1회차를 공개해 수요 확인 후 유료화

### 아이디어 4. 팀 지식 대시보드 SaaS (React + PHP)

- 문제: `docs/solutions/`와 `docs/plans/`가 레포 안 마크다운이라 비개발자나 다른 팀이 보기 어렵다
- 제품: GitHub 레포를 연결하면 학습 문서, 계획서, 리뷰 결과를 검색/태그/통계로 보여주는 웹앱. "이번 달 팀이 새로 배운 함정 10개", "가장 많이 재사용된 학습" 같은 리포트
- 기술: React 프론트, PHP(Laravel) API, GitHub App 연동, 마크다운 + YAML frontmatter 파싱(`module`, `tags`, `problem_type` 필드가 이미 표준화되어 있어 파싱이 쉬움)
- 과금: 팀당 월 2만~10만 원 구독
- 리스크: GitHub/에이전트 회사가 비슷한 기능을 내장할 수 있음 -> 한국어 UI, 사내 설치형(온프레미스) 옵션으로 차별화

### 아이디어 5. AI 코드 리뷰 / 품질 점검 대행

- 제공: 외주 결과물이나 레거시 코드를 받아 `ce-code-review`(다중 페르소나 리뷰) + 사람 검수로 보안/성능/유지보수 리포트 작성
- 대상: 외주 개발물을 검수할 능력이 없는 스타트업, 인수 전 코드 실사가 필요한 곳
- 가격 가설: 소규모 레포 1건 50만~300만 원
- 주의: AI 결과를 그대로 넘기지 말고 반드시 사람이 검증한 내용만 리포트에 포함

### 아이디어 6. 외주 개발 생산성 무기로 사용

- 우리가 받는 React/PHP 외주 프로젝트에 이 루프를 적용해 같은 인력으로 더 많은 프로젝트를 처리
- 프로젝트가 쌓일수록 `docs/solutions/`와 자체 팩이 두꺼워져 다음 견적이 더 빨라지고 저렴해짐(말 그대로 복리)
- 고객에게는 계획서, 리뷰 리포트를 산출물로 함께 제공해 신뢰도와 단가를 올림

### 추천 실행 순서

1. 우리 프로젝트 하나에 직접 적용해 사례와 수치 확보 (2~4주)
2. 그 과정을 콘텐츠로 공개해 관심 확인 (아이디어 3)
3. 반응이 오는 스택으로 팩 제작 (아이디어 2)
4. 문의가 들어오면 컨설팅/리뷰 대행 (아이디어 1, 5)
5. 반복 요청이 보이면 대시보드 SaaS 개발 (아이디어 4)

### 공통 주의사항

- MIT 라이선스 고지 유지, 원작자/Every 브랜드로 오인되는 이름이나 로고 사용 금지
- 고객 코드를 다룰 때는 NDA와 데이터 처리 범위를 명확히
- AI 산출물은 사람 검증을 거친 뒤 제공
