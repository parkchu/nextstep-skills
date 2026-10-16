# nextstep-skills

[넥스트스텝(NextStep)](https://edu.nextstep.camp) 미션의 Git/GitHub 절차를 도와주는 AI 코딩 에이전트용 스킬 모음입니다. [Claude Code](https://claude.com/claude-code) 플러그인으로 설치하며, 같은 SKILL.md 형식을 읽는 Codex·pi에서도 설치할 수 있습니다.

넥스트스텝 미션은 fork → 아이디 브랜치 기준 작업 → PR → merge 후 upstream 동기화라는, 처음엔 꽤 헷갈리는 Git 절차를 요구합니다. 이 스킬들은 그 절차에서 **본인 아이디·저장소 이름 같은 빈칸을 Claude가 직접 읽어 채우고**, 위험한 명령은 확인을 받으며 진행해 줍니다.

## 스킬 4개와 미션 흐름

```
┌─ 미션 시작 ─────────────────────────────────────────────┐
│                                                          │
│  /nextstep:start   fork · clone · 최초 브랜치(step1) 생성  │
│        │                                                 │
│        ▼                                                 │
│     구현 (직접)  ◀──────────────────────────┐             │
│        │                                   │             │
│        ▼                                   │             │
│  /nextstep:check  미션 조건 충족 여부 체크리스트│             │
│        │                                   │             │
│        ▼                                   │             │
│   커밋 → /nextstep:pr  push + PR 생성 링크   │             │
│        │                                   │             │
│        ▼  (리뷰 → merge 후)                 │             │
│  /nextstep:advance  upstream 동기화 +       │             │
│                     다음 step 브랜치 생성 ────┘             │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

| 스킬 | 언제 | 하는 일 |
|------|------|---------|
| **/nextstep:start** | 새 미션을 처음 시작할 때 | fork+clone(gh CLI, 없으면 브라우저 폴백), upstream remote 세팅, 아이디 브랜치 확인, 최초 작업 브랜치(step1) 생성 |
| **/nextstep:check** | 구현 중·커밋 전 수시로 | 붙여넣은 미션 요구사항을 diff와 대조해 조건별 ✅충족/❌미충족/⏸판단보류 체크리스트로 피드백 |
| **/nextstep:pr** | 단계 구현을 마쳤을 때 | push 후 base(본인 아이디 브랜치)/head가 미리 연결된 PR 생성 링크 제공. 제목·본문·PR 생성은 본인이 |
| **/nextstep:advance** | PR이 merge된 뒤 | 끝난 브랜치 정리, upstream fetch·rebase 동기화, 다음 step 브랜치 생성. 충돌 해결 절차 포함 |

## 설치

Claude Code 플러그인으로 설치합니다. Claude Code 대화창에서 실행하세요.

```
/plugin marketplace add parkchu/nextstep-skills
/plugin install nextstep@nextstep-skills
```

사용자 전역(user scope)에 설치되므로 어느 미션 저장소에서든 쓸 수 있습니다.

**업데이트** — 새 버전은 자동으로 들어오지 않습니다. 아래 명령으로 받거나, `/plugin` → Marketplaces → `nextstep-skills` → **Enable auto-update**로 자동 업데이트를 켜세요.

```
/plugin marketplace update nextstep-skills
```

**삭제**

```
/plugin uninstall nextstep@nextstep-skills
```

> 예전에 `~/.claude/skills/`에 `nextstep-*` 폴더를 복사해 설치했다면, 중복되지 않도록 그 폴더들은 지워주세요.

## 사용법

Claude Code 대화에서 `/nextstep:스킬이름`으로 직접 호출합니다.

```
/nextstep:start 미션 저장소 URL   ← 예: /nextstep:start https://github.com/next-step/java-racingcar
/nextstep:check 미션 조건         ← 요구사항 텍스트를 붙여넣거나 캡처 이미지를 첨부
/nextstep:pr                     ← 단계 구현을 마치고 커밋한 뒤
/nextstep:advance                ← PR이 merge된 뒤
```

인자를 빼먹어도 괜찮습니다 — 스킬이 필요한 정보(저장소 URL, 미션 조건)를 물어봅니다. \
자연어로 요청해도 해당 스킬이 자동으로 잡힙니다. (예: "미션 시작하려는데 저장소 세팅해줘")

**/nextstep:start 사용 시 알아둘 것**

- **미션 저장소 주소가 필요합니다.** 넥스트스텝 미션 페이지에 있는 GitHub 저장소 URL(예: `https://github.com/next-step/java-racingcar`)을 함께 알려주세요. 안 알려주면 스킬이 어느 미션인지 물어봅니다.
- **clone 위치는 Claude Code를 실행한 현재 디렉토리 바로 아래**(`./저장소이름`)입니다. 위치를 따로 묻지 않으니, 미션 저장소들을 모아둘 디렉토리에서 실행하세요.

## 필요한 것

- [Claude Code](https://claude.com/claude-code)
- [gh CLI](https://cli.github.com) (권장 — 없어도 브라우저 fork 안내로 폴백합니다)
- 넥스트스텝 웹의 **`미션 시작`**·**`리뷰 요청`** 버튼은 스킬이 대신 눌러줄 수 없습니다. 스킬이 시점마다 안내하니 직접 누르면 됩니다.

## 설계 원칙

- **빈칸은 Claude가 채운다**: 본인 아이디, 저장소 이름, base 브랜치를 외울 필요 없이 `git remote -v` 등에서 자동 추출합니다.
- **위험한 명령은 확인 후 실행**: `push`, `rebase`, `reset --hard`, `branch -D`는 실행 전에 무엇을 하는지 알리고 동의를 받습니다.
- **재실행 안전**: 이미 fork/clone돼 있으면 지우거나 덮어쓰지 않고 재사용해 이어서 진행합니다.
- **각 스킬은 자기 책임까지만**: 코드 구현·수정, 커밋 메시지, PR 제목·본문은 학습을 위해 본인 몫으로 남깁니다.
