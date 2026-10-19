# nextstep-skills

[넥스트스텝(NextStep)](https://edu.nextstep.camp) 미션의 Git/GitHub 절차를 도와주는 AI 코딩 에이전트용 스킬 모음입니다. [Claude Code](https://claude.com/claude-code) 플러그인으로 설치하며, 같은 SKILL.md 형식을 읽는 Codex·pi에서도 설치할 수 있습니다.

넥스트스텝 미션은 fork → 아이디 브랜치 기준 작업 → PR → merge 후 upstream 동기화라는, 처음엔 꽤 헷갈리는 Git 절차를 요구합니다. 이 스킬들은 그 절차에서 **본인 아이디·저장소 이름 같은 빈칸을 Claude가 직접 읽어 채우고**, 위험한 명령은 확인을 받으며 진행해 줍니다.

## 설치

> Claude Code, Codex, pi 모두 **CLI 환경에서만 테스트**했습니다. 데스크톱 앱·IDE 확장 등 다른 환경에서는 동작이 다를 수 있습니다.

<details>
<summary><b>Claude Code</b></summary>

터미널에서 실행하세요.

```
claude plugin marketplace add parkchu/nextstep-skills
claude plugin install nextstep@nextstep-skills
```

**업데이트** — 새 버전은 자동으로 들어오지 않습니다. 아래 명령으로 받은 뒤 Claude Code를 다시 시작하세요. 자동 업데이트를 원하면 Claude Code 대화창에서 `/plugin` → Marketplaces → `nextstep-skills` → **Enable auto-update**를 켜세요.

```
claude plugin update nextstep@nextstep-skills
```

**삭제**

```
claude plugin marketplace remove nextstep-skills
```

</details>

<details>
<summary><b>Codex</b></summary>

터미널에서 실행하세요.

```
codex plugin marketplace add parkchu/nextstep-skills
codex plugin add nextstep@nextstep-skills
```

**업데이트**

```
codex plugin marketplace upgrade nextstep-skills
```

**삭제**

```
codex plugin remove nextstep@nextstep-skills
codex plugin marketplace remove nextstep-skills
```

</details>

<details>
<summary><b>pi</b></summary>

터미널에서 실행하세요.

```
pi install git:github.com/parkchu/nextstep-skills
```

**업데이트**

```
pi update git:github.com/parkchu/nextstep-skills
```

**삭제**

```
pi remove git:github.com/parkchu/nextstep-skills
```

</details>
