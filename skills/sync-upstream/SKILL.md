---
name: sync-upstream
description: 넥스트스텝(NextStep) 미션 리뷰가 끝나 PR이 merge된 뒤, next-step 저장소에 통합된 코드와 동기화(upstream rebase)한다. 먼저 merge된 변경이 있는지 확인해, 있으면 끝난 브랜치 삭제·upstream remote 추가·fetch·rebase·충돌 해결까지 진행하고, 아직 merge 전이라 동기화할 게 없으면 동기화를 건너뛴다. 다음 단계 브랜치 생성은 선택 사항으로, 끝에 만들지 묻는다(merge 전이면 마지막 스텝 브랜치 기준). 미션을 다 끝낸 뒤에도 쓴다. 사용자가 "리뷰 끝났어", "PR merge 됐어", "미션 끝났는데 정리해줘", "upstream 동기화", "다음 step 브랜치 만들어줘", "/nextstep:sync-upstream" 등을 요청할 때 사용한다.
---

# /nextstep:sync-upstream — merge된 코드 동기화

리뷰가 끝나 PR이 merge되면, next-step에 통합된 코드와 **동기화(upstream rebase)** 한다.
가장 먼저 **merge된 변경이 있는지** 확인하고, 아직 merge 전이라 동기화할 게 없으면 동기화를 건너뛴다.
**다음 단계 브랜치 생성은 선택 사항**이라, 마지막 단계까지 끝낸 뒤에도 이 스킬로 정리만 하고 끝낼 수 있다.
넥스트스텝 절차 중 가장 헷갈리는 구간이라, 빈칸을 직접 채워 안내하고 위험 명령은 확인 후 실행한다.

## 1단계 — 컨텍스트 파악 (먼저 실행)

```bash
git remote -v               # origin(본인 fork), upstream(next-step) 존재 여부
git branch -a               # 로컬/원격 브랜치
git branch --show-current   # 지금 브랜치
git status                  # 미커밋 변경 여부
```
추출할 값:
- **본인_아이디**: `origin` URL의 owner. → 동기화 기준 브랜치(`upstream/{본인_아이디}`).
- **저장소(repo)**: repo 이름.
- **upstream 유무**: 없으면 2단계에서 추가한다.
- **마지막_스텝_브랜치**: 현재 브랜치가 본인 아이디 브랜치가 아니면 현재 브랜치. 아이디 브랜치에 있으면 로컬 `step{N}` 중 N이 가장 큰 것. 후보가 없거나 여러 개라 애매하면 사용자에게 묻는다.

> 미커밋 변경이 있으면 먼저 정리(커밋/스태시)하도록 안내한다. rebase/reset 전에 작업이 날아가지 않게.

## 2단계 — upstream(next-step) remote 추가 (없을 때만)

`git remote -v`에 upstream이 없으면 추가한다. 이미 있으면 건너뛴다.
```bash
git remote add upstream https://github.com/next-step/{repo}.git
git remote -v
```

## 3단계 — upstream에서 본인 아이디 브랜치 가져오기

fetch는 로컬 브랜치를 바꾸지 않으니 merge 여부와 상관없이 실행한다.
```bash
git fetch upstream {본인_아이디}
```

## 4단계 — merge 여부 확인 (분기)

upstream에 로컬 아이디 브랜치에 없는 커밋이 있는지, 곧 **아직 동기화하지 않은 merge가 있는지** 본다.
```bash
git rev-list --count {본인_아이디}..upstream/{본인_아이디}
```
> 로컬에 아이디 브랜치가 없으면 위 명령이 에러난다. 이때는 `git branch {본인_아이디} upstream/{본인_아이디}`로 만들고, 동기화는 그걸로 끝난 것으로 보고 7단계로 간다(base는 아이디 브랜치).

- **1 이상** → merge된 변경이 있다. 결과를 한 줄로 알리고 **5~6단계(동기화)** 로 간다.
- **0** → 동기화할 게 없다. 이유가 둘이라 마지막 스텝 브랜치가 merge됐는지 한 번 더 본다.
  ```bash
  git merge-base --is-ancestor {마지막_스텝_브랜치} upstream/{본인_아이디} && echo merged
  ```
  - `merged`가 나오면 **이미 merge되고 동기화도 끝난 상태**다. 5~6단계를 건너뛰고 7단계로 간다(base는 아이디 브랜치).
  - 안 나오면 squash merge일 수도 있으니, gh가 있으면 PR 상태로 확인한다.
    ```bash
    gh pr list --repo next-step/{repo} --base {본인_아이디} --head {마지막_스텝_브랜치} --state merged
    ```
    결과가 있으면 위와 같이 이미 동기화된 상태로 본다. 결과가 없으면 **아직 merge 전**이다. 5~6단계를 건너뛰고 7단계로 간다(base는 마지막 스텝 브랜치).
  - gh가 없거나 인증이 안 돼 있으면, 마지막 스텝 PR이 merge됐는지 사용자에게 묻는다.

## 5단계 — 끝난 브랜치 정리 (merge됐을 때만, 확인 후)

본인 아이디 브랜치로 이동하고, merge가 끝난 이전 기능 브랜치를 삭제한다. `-D`는 되돌리기 어려우니 **어느 브랜치를 지울지 확인 후** 실행한다. 아직 merge 전인 브랜치(리뷰 중인 다음 단계 등)는 지우지 않는다.
```bash
git checkout {본인_아이디}
git branch -D {삭제할_브랜치}     # 예: git branch -D step1
```

## 6단계 — 동기화 (merge됐을 때만, rebase 확인 후)

merge된 최신 코드를 본인 브랜치에 맞춘다. 히스토리를 바꾸는 동작이라 **실행 전 한 줄로 알리고 동의를 받는다.**
```bash
git rebase upstream/{본인_아이디}
```
- 충돌이 나면 아래 "충돌 해결"로 간다.
- rebase 후 로컬 아이디 브랜치가 origin(본인 fork)보다 앞서게 된다. fork의 아이디 브랜치도 최신으로 맞춰두려면(선택) push한다:
```bash
git push origin {본인_아이디}
```

## 7단계 — 다음 단계 브랜치 생성 (선택)

브랜치를 만들지 **먼저 묻는다.** 미션을 다 끝냈거나 아직 필요 없다고 하면 여기서 마친다.
base는 4단계 판정에 따라 다르다.

- **merge됨(동기화했거나 이미 동기화된 상태)** → 본인 아이디 브랜치 기준.
  ```bash
  git checkout {본인_아이디}
  git checkout -b {새_브랜치}      # 예: git checkout -b step2
  ```
- **아직 merge 전** → 리뷰를 기다리며 다음 단계를 이어 할 수 있게, **마지막 스텝 브랜치 기준**으로 만들지 묻는다.
  ```bash
  git checkout {마지막_스텝_브랜치}
  git checkout -b {새_브랜치}      # 예: step1 기준으로 step2
  ```
  > 이 브랜치로 PR을 올리면, 이전 단계가 merge되기 전까지는 이전 단계 커밋도 PR diff에 함께 보인다고 안내한다.

완료 후, 어떤 브랜치에서 작업하면 되는지 한 줄로 알려준다.

## 충돌 해결 (rebase가 꼬일 때)

본인 아이디 브랜치를 upstream으로 깔끔히 맞춘 뒤 기능 브랜치에 merge한다.
`reset --hard`는 로컬 변경을 지우므로 **실행 전 반드시 확인한다.**
```bash
git checkout {본인_아이디}
git reset --hard upstream/{본인_아이디}    # ⚠️ 로컬 변경 사라짐
git checkout {기능_브랜치}
git merge {본인_아이디}
```
예) `git checkout javajigi` → `git reset --hard upstream/javajigi` → `git checkout step2` → `git merge javajigi`

## 하지 말 것
- placeholder(`{본인_아이디}` 등)를 그대로 두고 안내하지 않는다. `git remote -v`로 실제 값을 채운다.
- `rebase`·`reset --hard`·`branch -D`를 사용자 확인 없이 실행하지 않는다.
- 아직 merge 전이라고 판정했으면 `branch -D`·`rebase`·`reset --hard`를 실행하지 않는다.
- 미커밋 변경이 남아 있는데 rebase/reset을 강행하지 않는다.
- 다음 단계 브랜치를 묻지 않고 만들지 않는다.
