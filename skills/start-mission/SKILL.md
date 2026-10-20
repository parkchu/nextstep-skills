---
name: start-mission
description: 넥스트스텝(NextStep) 미션을 처음 시작할 때, 미션 저장소를 fork·clone하고 최초 작업 브랜치(step1 등)까지 만들어 준다. gh CLI로 fork하고 clone까지 이어서 처리하며, gh가 없거나 인증이 안 돼 있으면 브라우저 Fork 안내로 폴백한다. 이미 fork/clone돼 있으면 재사용하고 이어서 진행한다(재실행 안전). 사용자가 "미션 시작하려는데 저장소 세팅해줘", "새 미션 시작할래", "fork부터 해줘", "/nextstep:start-mission" 등으로 요청할 때 사용한다.
---

# /nextstep:start-mission — 넥스트스텝 미션 최초 시작

새 미션을 처음 시작할 때 **fork → clone → 최초 작업 브랜치 생성**까지 세팅한다.

## 1단계 — 미션 저장소 확인

호출할 때 미션 저장소 URL이나 이름(예: `https://github.com/next-step/java-racingcar`, `java-racingcar`)이 주어졌으면 거기서 repo 이름을 쓴다.
없으면 사용자에게 **어느 미션 저장소인지** 묻는다. fork할 원본(upstream)의 owner는 항상 `next-step`이다.

## 2단계 — fork + clone (현재 디렉토리 아래)

clone은 **이 스킬을 호출한 현재 디렉토리 바로 아래**(`./{repo}`)에 받는다. 위치를 따로 묻지 않는다.

먼저 재실행 안전 확인: `./{repo}` 디렉토리가 **이미 있으면 새로 받지 않고** 그 안으로 이동해 3단계로 간다.

없으면 gh 사용 가능 여부를 확인하고 fork와 clone을 **나눠서** 실행한다:
```bash
gh auth status                                                      # gh 설치·인증 확인
gh repo fork next-step/{repo} --clone=false --default-branch-only   # fork만 만든다
gh api user -q .login                                               # 본인 아이디 확인
git clone https://github.com/{본인_아이디}/{repo}.git
cd {repo}
```
- `--default-branch-only`로 **기본 브랜치만** 복사한다. 원본에는 수강생마다 아이디 브랜치가 있어서 전부 복사하면 다른 사람 브랜치 수백 개가 fork에 딸려 온다. 본인 아이디 브랜치는 3단계에서 따로 올린다.
- `--clone`을 쓰지 않는다. `gh repo fork --clone`은 clone 뒤 upstream을 추가하면서 브랜치 지정 없이 `git fetch upstream`을 돌려, fork는 기본 브랜치만이어도 **로컬 `upstream/*`에 다른 사람 브랜치가 전부** 들어온다. upstream은 3단계에서 직접 추가하고, 본인 아이디 브랜치만 지정해서 받는다.
- **fork가 이미 있으면** gh가 기존 fork를 그대로 재사용한다(에러 아님). 그대로 진행한다.

### gh를 못 쓸 때 (미설치·미인증) — 브라우저 폴백

막히지 않고 진행한다. Fork 버튼 URL을 안내하고, 사용자가 fork를 마치면 clone부터 잇는다.
clone URL에 본인 아이디가 들어가는데 gh 없이는 알아낼 방법이 없으니, **이 경로에서만 사용자에게 GitHub 아이디를 물어** 입력받은 값으로 채운다:
```
https://github.com/next-step/{repo}/fork      ← 브라우저에서 Fork
```
Fork 화면의 "Copy the `{기본 브랜치}` branch only" 체크박스는 기본값(체크) 그대로 둔다. gh 경로와 같게 기본 브랜치만 복사하고, 본인 아이디 브랜치는 3단계에서 올린다.
```bash
git clone https://github.com/{본인_아이디}/{repo}.git
cd {repo}
```
> 여유가 될 때 `brew install gh` + `gh auth login`을 해두면 다음부터 한 번에 된다고 덧붙인다.

## 3단계 — remote 확인·보장

어느 경로로 왔든(gh/폴백/기존 clone 재사용), remote 두 개를 보장한다:
```bash
git remote -v     # origin(본인 fork) + upstream(next-step 원본) 둘 다 있는지
```
**본인_아이디**는 `origin` URL `github.com/{아이디}/{repo}`의 owner로 확정한다. clone이 끝났으면 어느 경로든 여기서 자동으로 알 수 있으니 따로 묻지 않는다.

upstream이 없으면 추가한다:
```bash
git remote add upstream https://github.com/next-step/{repo}.git
```

이어서 origin(본인 fork)에 **본인 아이디 브랜치**를 보장한다. 이 브랜치가 이후 PR의 base이자 작업 브랜치의 시작점이다:
```bash
git fetch origin
git branch -r --list "origin/{본인_아이디}"   # 보이면 이미 있음 → 4단계로
```
기본 브랜치만 fork했으니 처음에는 없는 게 정상이다. upstream에서 가져와 origin에 올린다:
```bash
git fetch upstream {본인_아이디}
git push origin refs/remotes/upstream/{본인_아이디}:refs/heads/{본인_아이디}
```
- push는 원격에 반영되는 동작이니 **무엇을 어디로 올리는지 한 줄로 알리고 동의를 받은 뒤** 실행한다.
- `git fetch upstream {본인_아이디}`부터 실패하면 next-step에 브랜치 자체가 없는 것이다. 미션 시작 버튼을 안 누른 경우이니, 넥스트스텝 사이트(edu.nextstep.camp)에서 해당 미션의 **`미션 시작` 버튼**을 누르고 이 스킬을 다시 실행하도록 안내한다. 이 버튼을 눌러야 next-step 저장소에 본인 아이디 브랜치가 생기고 리뷰어가 배정된다. 다시 실행하면 이미 받은 fork·clone은 재사용된다.

## 4단계 — 최초 작업 브랜치 생성

관례적 기본값 **`step1`을 제안하고 확인**받은 뒤 생성한다(미션에 따라 다르면 사용자가 이름을 정정).
시작점은 반드시 **본인 아이디 브랜치**다(PR base가 이 브랜치라서, 기본 브랜치에서 갈라지면 diff가 오염될 수 있다).
로컬 아이디 브랜치도 함께 만든다. 나중에 [/nextstep:sync-upstream]이 이 로컬 브랜치로 merge 여부를 판정하고 rebase한다(이미 있으면 첫 줄은 건너뛴다):
```bash
git branch {본인_아이디} origin/{본인_아이디}
git checkout -b step1 {본인_아이디}
```

## 5단계 — 마무리 안내

- 작업 위치(`{현재 위치}/{repo}`)와 브랜치(`step1`)를 한 줄로 알려준다.
- IDE(IntelliJ 등)에서 이 디렉토리를 **Import/Open** 해서 구현을 시작하면 된다고 안내한다.
- 이후 흐름 소개: 구현 → [/nextstep:check-requirements]로 미션 조건 확인 → 커밋 → [/nextstep:create-pr]로 리뷰 요청.

## 하지 말 것

- placeholder(`{repo}`, `{본인_아이디}` 등)를 그대로 두고 안내하지 않는다. URL·`gh auth status` 등 실제 값으로 채운다.
- 이미 있는 clone 디렉토리를 지우거나 덮어쓰지 않는다. 재사용하고 이어서 진행한다.
- gh가 없다고 설치를 강요하며 멈추지 않는다. 브라우저 폴백으로 진행한다.
- 미션 구현·커밋·push·PR까지 넘보지 않는다. 이 스킬은 **브랜치 생성까지**, 이후는 각 담당 스킬의 몫.
