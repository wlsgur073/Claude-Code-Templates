# Guide & Template Coverage Expansion

**Date:** 2026-04-03
**Branch:** `feat/coverage-expansion`
**Status:** In Progress

## Context

Claude Code 생태계의 프로덕션급 플러그인/프로젝트들의 설정 패턴을 분석한 결과, 우리 가이드와 템플릿에서 보완이 필요한 6개 영역을 식별했다. MCP 통합 가이드가 완전히 부재하고, hook/agent/skill 예제의 다양성이 부족하며, CLAUDE.md 고급 조직화 패턴이 미비하다. 이 plan은 각 영역별 구체적인 수정 사항과 대상 파일을 정리한다.

**이전 plan과의 관계:** `2026-04-02-qw-guide-improvements.md`의 Pending 항목 중 HI-2(에이전트 카탈로그)와 NH-3(훅 기반 세션 상태)이 이번 작업과 일부 겹친다. 해당 항목들은 이 plan으로 흡수하고, 이전 plan의 Pending에서 제거한다.

---

## P1. MCP 통합 가이드 신규 작성

현재 7개 가이드 중 MCP를 전담하는 가이드가 없다. `advanced-features-guide.md`에서 hooks/agents/skills는 다루지만 MCP 설정은 빠져 있다. MCP는 Claude Code의 핵심 확장 메커니즘이므로 독립 가이드가 필요하다.

### MCP 가이드 작업 내용

**신규 파일: `docs/guides/mcp-guide.md`** (~130줄, frontmatter v1.0.0)

다룰 내용:

1. **MCP란 무엇인가** — Model Context Protocol의 역할, Claude Code에서의 위치 (2-3줄)
2. **`.mcp.json` 파일 구조** — 프로젝트 루트에 위치, 설정 형식 예제

   ```json
   {
     "mcpServers": {
       "server-name": {
         "command": "npx",
         "args": ["-y", "@example/mcp-server"],
         "env": { "API_KEY": "..." }
       }
     }
   }
   ```

3. **MCP 서버 등록 방법** — 3가지 실행 방식:
   - `npx` (Node.js 기반 서버)
   - `uvx` (Python 기반 서버)
   - `docker` (컨테이너 기반 서버)
4. **ToolSearch와 deferred tool loading** — MCP 도구가 즉시 로드되지 않고 ToolSearch로 활성화되는 패턴 설명
5. **플러그인에서 MCP 연동** — `plugin.json`의 `mcpServers` 필드, `${CLAUDE_PLUGIN_ROOT}` 경로 변수
6. **보안 고려사항** — 신뢰할 수 있는 서버만 등록, env에 민감 정보 주의, `.mcp.json`의 git 커밋 여부 판단 기준
7. **실전 사용 사례** — DB 조회, 문서 검색, 외부 API 연동 등 (TaskFlow 예제 활용)

**기존 파일 수정:**

- `docs/guides/getting-started.md` — "What's Next" 섹션에 MCP 가이드 크로스링크 추가 (1줄), version bump → 1.1.0
- `docs/guides/advanced-features-guide.md` — "Further Reading" 섹션에 MCP 가이드 링크 추가 (1줄), version bump → 1.1.0

**Advanced 템플릿:**

- `templates/advanced/.mcp.json` — TaskFlow 프로젝트 맥락의 MCP 서버 예제 (가상의 DB 조회 서버)

**한국어 번역:**

- `docs/i18n/ko-KR/guides/mcp-guide.md` — 영문 가이드 완성 후 번역

---

## P2-A. Hook 패턴 강화

현재 가이드와 템플릿의 hook 예제는 모두 인라인 shell 명령이다. 실제 프로덕션 프로젝트에서는 외부 스크립트 기반 hook이 일반적이며, 이 패턴을 가르쳐야 한다. `UserPromptSubmit`은 현재 1줄 언급에 그치고 있어 실전 예제가 필요하다.

**주의:** QW-2에서 이미 PreToolUse(`exit 2` 파일 보호) + PostToolUse(eslint) + UserPromptSubmit 언급 + hook types(command/prompt)는 구현됨. 겹치지 않는 부분만 추가.

### Hook 패턴 작업 내용

**`docs/guides/advanced-features-guide.md`** — Hook 섹션에 추가 (~15줄)

현재 Hook 섹션(line 11-58) 뒤에 "Script-Based Hooks" 소섹션 추가:

1. **인라인 vs 스크립트 비교표**
   | 방식 | 장점 | 단점 | 적합한 상황 |
   | 인라인 | 설정 파일 하나로 완결 | 복잡한 로직 어려움 | 간단한 검사/포맷팅 |
   | 스크립트 | 복잡한 로직, 테스트 가능 | 별도 파일 관리 필요 | 다단계 검증, 조건부 로직 |

2. **스크립트 기반 hook 예제**

   ```json
   {
     "UserPromptSubmit": [{
       "hooks": [{
         "type": "command",
         "command": "bash \"${CLAUDE_PROJECT_DIR}/scripts/validate-prompt.sh\"",
         "timeout": 5000,
         "statusMessage": "Validating input"
       }]
     }]
   }
   ```

3. **timeout 가이드라인**
   - 입력 검증/키워드 감지: 3-5초
   - 린트/포맷팅: 10-15초
   - 빌드/테스트: 30초 이상 (필요 시 `timeout` 명시)

**Advanced 템플릿 수정:**

- `templates/advanced/.claude/settings.json` — 기존 2개 hook에 `UserPromptSubmit` 스크립트 hook 1개 추가
- `templates/advanced/scripts/validate-prompt.sh` — TaskFlow 맥락의 프롬프트 검증 스크립트 (예: "migration" 키워드 감지 시 경고)

**한국어 번역 동기화:**

- `docs/i18n/ko-KR/guides/advanced-features-guide.md` — 영문 변경분 반영
- `docs/i18n/ko-KR/templates/advanced/.claude/settings.json` — 동기화

**Version bumps:** `advanced-features-guide.md` → 1.1.0 (P1의 MCP 링크 추가와 합산)

---

## P2-B. Agent 설계 패턴 강화

현재 1개 에이전트 예제(backend-developer.md)만 있어 모델 티어별 활용 패턴을 보여주지 못한다. QW-1에서 4섹션 구조와 모델 선택표는 구현했으나, 실제 다양한 에이전트 예제가 없다. 이전 plan의 HI-2(에이전트 카탈로그)를 여기서 해소한다.

### Agent 패턴 작업 내용

**신규 파일 2개 — Advanced 템플릿 에이전트:**

1. **`templates/advanced/.claude/agents/security-reviewer.md`**
   - model: `opus` (보안 리뷰는 단일 실수 비용이 높으므로)
   - tools: Read, Grep, Glob (읽기 전용)
   - Scope: 전체 `src/` — 인증/인가, 입력 검증, 의존성 보안
   - Rules: OWASP Top 10 체크, SQL injection/XSS 검사, 민감 데이터 노출 확인
   - Constraints: 코드 수정 금지 (분석 전용), 보안 이슈 발견 시 심각도 분류 (Critical/High/Medium/Low)
   - Verification: 발견된 이슈를 파일명:줄번호와 함께 보고, 오탐 최소화
   - YAML 주석: `# opus: single mistake in security review can be costly`

2. **`templates/advanced/.claude/agents/test-writer.md`**
   - model: `haiku` (테스트 코드 생성은 빠른 반복이 중요, 비용 효율)
   - tools: Read, Edit, Write, Bash, Glob
   - Scope: `tests/` 디렉터리만 수정, `src/`는 읽기만
   - Rules: 기존 테스트 패턴 따름 (factories 사용, Supertest 패턴), 엣지 케이스 포함, 테스트 이름은 행동 기술
   - Constraints: `src/` 코드 수정 금지, 기존 테스트 삭제 금지
   - Verification: `npm test` 통과, 커버리지 감소하지 않음
   - YAML 주석: `# haiku: fast iteration for test generation, cost-effective for high-volume output`

**`docs/guides/advanced-features-guide.md`** — Agent 섹션 보강 (~10줄)

모델 선택표(line 103-111) 뒤에 추가:

- **역할 경계 원칙**: 하나의 에이전트 = 하나의 관점. 구현/리뷰/테스트를 하나의 에이전트에 몰지 않는다.
- **읽기 전용 에이전트 패턴**: tools에서 Edit/Write를 제외하면 분석 전용 에이전트가 된다. 보안 리뷰, 아키텍처 분석, 코드 탐색에 적합.
- **에이전트 조합 예시**: `backend-developer`(구현) → `security-reviewer`(리뷰) → `test-writer`(테스트) 파이프라인

**한국어 번역:**

- `docs/i18n/ko-KR/templates/advanced/.claude/agents/security-reviewer.md`
- `docs/i18n/ko-KR/templates/advanced/.claude/agents/test-writer.md`
- `docs/i18n/ko-KR/guides/advanced-features-guide.md` — 변경분 반영

---

## P3-A. Skill 설계 패턴 강화

현재 1개 스킬 예제(add-endpoint)만 있다. 다양한 스킬 설계 패턴(폴백, 검증 워크플로)을 보여주는 추가 예제가 필요하다.

### Skill 패턴 작업 내용

**신규 파일 — Advanced 템플릿 스킬:**

`templates/advanced/.claude/skills/run-checks/SKILL.md`

- name: `run-checks`
- description: "Runs build, lint, and test checks in sequence with clear pass/fail reporting"
- argument-hint: `[--fix]`
- 단계:
  1. `$ARGUMENTS`에서 `--fix` 플래그 파싱
  2. Step 1: `npm run build` — 컴파일 확인
  3. Step 2: `npm run lint` (--fix 시 `npm run lint -- --fix`) — 스타일 확인
  4. Step 3: `npm test` — 테스트 통과
  5. Step 4: 결과 요약 (✓ pass / ✗ fail + 에러 메시지)
- **폴백 패턴 시연**: CI 환경(`$CI` 변수) 감지 시 `npm run ci:check`으로 대체

**`docs/guides/advanced-features-guide.md`** — Skill 섹션 보강 (~8줄)

현재 Skill 섹션(line 115-153) 뒤, "Further Reading" 전에 추가:

- **`$ARGUMENTS` 파싱 패턴**: `$ARGUMENTS`에서 플래그와 위치 인자를 추출하는 방법
- **참조 파일 활용**: `references/` 디렉터리에 프로젝트 컨벤션 문서를 두고 스킬이 읽게 하는 패턴
- **폴백 패턴**: 도구가 있으면 Path A, 없으면 Path B로 분기하는 설계

**한국어 번역:**

- `docs/i18n/ko-KR/templates/advanced/.claude/skills/run-checks/SKILL.md`
- `docs/i18n/ko-KR/guides/advanced-features-guide.md` — 변경분 반영

---

## P3-B. CLAUDE.md 고급 조직화 패턴

현재 `claude-md-guide.md`는 기본 작성법과 @import을 잘 설명하지만, 프로젝트 규모가 커질 때의 조직화 전략이 부족하다.

### CLAUDE.md 조직화 작업 내용

**`docs/guides/claude-md-guide.md`** — "Pruning Your CLAUDE.md" 섹션(line 89) 앞에 신규 섹션 삽입 (~12줄)

### Organizing at Scale

- **Skill/Command Dispatch Table**: 프로젝트에 여러 커스텀 스킬이 있을 때 CLAUDE.md에 매핑표를 두어 빠른 참조 제공

  ```markdown
  ## Available Skills
  | Skill | Purpose |
  | /add-endpoint | Scaffold new API endpoint with handler, service, tests |
  | /run-checks | Run build, lint, and test suite in sequence |
  ```

- **Agent Reference List**: 프로젝트에 여러 에이전트가 있을 때 이름과 용도를 한 줄씩 나열. 에이전트 정의 자체는 `.claude/agents/`에 두되, CLAUDE.md에서 어떤 에이전트가 있는지 알 수 있게 한다.
- **Folder-Level CLAUDE.md 전략 리마인더**: 200줄 초과 시 rules 분리 외에, 서브디렉터리별 CLAUDE.md로 컨텍스트를 분산하는 전략 재강조 (이미 "Folder-Level CLAUDE.md" 섹션에서 다루지만, "Organizing at Scale"에서 전략적 관점 추가)

**TaskFlow Advanced 템플릿:**

- `templates/advanced/CLAUDE.md` — "Available Skills" 및 "Available Agents" 테이블 추가 (add-endpoint, run-checks, backend-developer, security-reviewer, test-writer)

**한국어 번역:**

- `docs/i18n/ko-KR/guides/claude-md-guide.md` — 변경분 반영
- `docs/i18n/ko-KR/templates/advanced/CLAUDE.md` — 동기화

**Version bump:** `claude-md-guide.md` → 1.1.0

---

## P4. Brownfield 프로젝트 가이드 보완

기존 프로젝트에 Claude Code를 도입할 때의 가이드가 부족하다. `/generate` 스킬은 advanced 경로에서 기존 설정을 감지하지만, 가이드 레벨에서는 이를 명시적으로 가르치지 않는다.

### Brownfield 가이드 작업 내용

**`docs/guides/effective-usage-guide.md`** — "Common Failure Patterns" 섹션(line 92) 앞에 추가 (~8줄)

### Adopting Claude Code in Existing Projects

기존 프로젝트에 Claude Code를 도입할 때의 권장 순서:

1. **기존 컨벤션 탐색 먼저** — 린터 설정(`.eslintrc`, `ruff.toml`), 테스트 프레임워크(`jest.config`, `pytest.ini`), 빌드 도구를 먼저 확인하고 CLAUDE.md에 반영
2. **`/init` 또는 `/claude-code-template:generate`의 advanced 경로 활용** — 기존 설정을 자동 감지해서 CLAUDE.md 초안 생성
3. **점진적 확장** — CLAUDE.md + settings.json부터 시작, 필요에 따라 rules → hooks → agents → skills 순서로 추가

**한국어 번역:**

- `docs/i18n/ko-KR/guides/effective-usage-guide.md` — 변경분 반영

**Version bump:** `effective-usage-guide.md` → 1.1.0

---

## 이전 Plan Pending 항목 정리

`2026-04-02-qw-guide-improvements.md`의 Pending에서 이 plan으로 흡수되는 항목:

- **HI-2 에이전트 카탈로그 패턴** → P2-B에서 해소 (security-reviewer + test-writer 추가, 역할 경계 원칙, 조합 패턴)
- **NH-3 훅 기반 세션 상태 관리** → P2-A에서 부분 해소 (스크립트 기반 hook 패턴)

여전히 남는 항목 (별도 plan 필요):

- **HI-1 READ-ONLY 에이전트 패턴** → P2-B의 security-reviewer가 읽기 전용 패턴을 시연하므로 부분 해소. 별도 가이드까지는 불필요할 수 있음.
- **HI-3 커밋 프로토콜 패턴** → 미해소, 별도 작업
- **NH-1 AGENTS.md 경계 명시** → 미해소, memory에 기록됨
- **NH-2 AI 생성 코드 리뷰 체크리스트** → 미해소, 별도 작업

---

## Constraints

- 가이드 줄 제한: ~130줄 (신규 MCP 가이드), ~160줄 (advanced-features-guide.md — QW에서 완화)
- 모든 템플릿은 TaskFlow 프로젝트 컨텍스트 유지
- 한국어 번역(`docs/i18n/ko-KR/`)은 영문과 동기화
- 파일 수정 시 해당 파일의 frontmatter version bump
- CLAUDE.md 200줄 이하 유지
- 커밋 전 반드시 사용자 승인

## 수정 대상 파일 전체 목록

**신규 생성 (8개):**

- `docs/guides/mcp-guide.md`
- `docs/i18n/ko-KR/guides/mcp-guide.md`
- `templates/advanced/.mcp.json`
- `templates/advanced/scripts/validate-prompt.sh`
- `templates/advanced/.claude/agents/security-reviewer.md`
- `templates/advanced/.claude/agents/test-writer.md`
- `templates/advanced/.claude/skills/run-checks/SKILL.md`
- `docs/i18n/ko-KR/templates/advanced/.claude/skills/run-checks/SKILL.md`

**수정 (10개+):**

- `docs/guides/advanced-features-guide.md` (hook + agent + skill 보강, MCP 링크)
- `docs/guides/claude-md-guide.md` (고급 조직화 패턴)
- `docs/guides/getting-started.md` (MCP 가이드 크로스링크)
- `docs/guides/effective-usage-guide.md` (Brownfield 팁)
- `templates/advanced/CLAUDE.md` (dispatch table)
- `templates/advanced/.claude/settings.json` (UserPromptSubmit hook)
- 한국어 번역 파일 6개+ (위 수정 파일의 ko-KR 미러)

**이전 Plan 업데이트:**

- `docs/plans/2026-04-02-qw-guide-improvements.md` — Pending에서 HI-2, NH-3 제거, 이 plan으로 이관 표기

## Verification

구현 완료 후 확인 사항:

- [ ] 모든 가이드의 줄 수 확인 (MCP ~130줄, advanced ~160줄, claude-md ~140줄, effective ~140줄)
- [ ] `getting-started.md`의 "What's Next"에서 MCP 가이드 링크가 작동하는지 확인
- [ ] TaskFlow advanced 템플릿이 일관된 프로젝트 컨텍스트를 유지하는지 확인
- [ ] 한국어 번역이 영문과 구조적으로 일치하는지 확인
- [ ] 모든 수정 파일의 frontmatter version이 bump되었는지 확인
- [ ] CLAUDE.md (루트)가 200줄 이하인지 확인
