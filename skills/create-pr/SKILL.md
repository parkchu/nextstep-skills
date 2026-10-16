---
name: create-pr
description: 넥스트스텝(NextStep) 미션의 작업 브랜치를 push하고, 저장소·브랜치 주소를 확인한 뒤 PR 생성 링크(브라우저)를 제공한다. 링크에는 PR 제목만 채운다 — 호출과 함께 준 제목이 있으면 그것을, 없으면 `{작업브랜치} - {본인_아이디}`(예: step1 - parkchu)를 쓴다. 본문은 채우지 않는다. PR 자체는 사용자가 브라우저에서 만든다. 사용자가 "미션 끝냈으니 PR 보내줘", "리뷰 요청 보내줘", "PR 올려줘", "/nextstep:create-pr", "/nextstep:create-pr 1단계 - 자동차 경주" 등으로 요청할 때 사용한다.
---

# /nextstep:create-pr — 넥스트스텝 PR 생성

넥스트스텝 미션 작업을 마친 뒤 **push → PR 생성 링크 제공**까지 돕는다.
PR 생성 자체는 사용자가 브라우저에서 한다(이 스킬은 base/head와 제목이 채워진 링크만 준비해 준다). 본문은 채우지 않고 사용자가 GitHub 화면에서 직접 작성한다.

> 넥스트스텝 PR의 특수성: base는 next-step 저장소의 **`master`가 아니라 "본인 아이디" 브랜치**다. 헷갈리기 쉬우니 반드시 맞춘다.

## 1단계 — 컨텍스트 파악 (먼저 실행)

빈칸을 사용자가 외우지 않도록, 현재 저장소에서 값을 직접 읽어 채운다.
```bash
git remote -v               # origin(=본인 fork), upstream(=next-step) URL
git branch --show-current   # 작업 브랜치 (예: step1)
git status                  # 커밋 안 된 변경이 있는지
git log origin/{작업브랜치}..HEAD --oneline   # 아직 push 안 된 커밋 (있으면 push 필요)
```
> 최초 push 전에는 `origin/{작업브랜치}` ref가 없어서 위 `git log`가 에러난다. 에러 = **아직 한 번도 push 안 함** = 전부 push 대상으로 해석하고 그대로 진행한다.
여기서 추출할 값:
- **본인_아이디**: `origin` URL `github.com/{아이디}/{repo}`에서. → PR의 **base 브랜치 이름**이자 head 소유자.
- **저장소(repo)**: origin/upstream의 repo 이름.

## 2단계 — 커밋 상태 확인

- 커밋 안 된 변경이 있으면 먼저 커밋하도록 안내한다. (이 스킬은 커밋을 대신하지 않는다.)
- 모든 작업이 커밋되어 있어야 push가 의미 있다.

## 3단계 — Push (실행 전 확인)

push는 외부(원격)에 반영되는 동작이므로, **무엇을 어디로 올리는지 한 줄로 알리고 동의를 받은 뒤** 실행한다.
```bash
git push origin {작업브랜치}      # 예: git push origin step1
```
> 이미 같은 브랜치로 PR이 열려 있으면, push만으로 기존 PR에 자동 반영된다. **새 PR을 또 만들 필요 없다.** 이 경우 4단계는 건너뛰고 "기존 PR에 반영됨"만 알린다. 기존 PR의 제목은 바꾸지 않는다(호출과 함께 제목을 줬다면, 제목은 GitHub에서 직접 고쳐야 한다고 알린다).

## 4단계 — base/head·제목 확인 후 PR 생성 링크 제공

**PR 제목 정하기**
- 호출과 함께 준 텍스트가 있으면 그대로 제목으로 쓴다. (예: `/nextstep:create-pr 1단계 - 자동차 경주` → `1단계 - 자동차 경주`)
- 없으면 기본값 `{작업브랜치} - {본인_아이디}`를 쓴다. (예: `step1 - parkchu`)

push 후, base/head·제목 값을 **표로 정리해 사용자에게 보여주고 OK를 받는다.** (틀리기 쉬운 부분이라 반드시 확인. 사용자가 제목을 고치면 고친 값으로 링크를 만든다)

| 항목 | 값 |
|------|----|
| base 저장소 | `next-step/{repo}` |
| base 브랜치 | `{본인_아이디}` |
| compare(head) | `{본인_아이디}:{작업브랜치}` |
| PR 제목 | `{제목}` |

확인이 끝나면, **base/head와 제목만 채워진** GitHub PR 생성 링크를 만들어 준다. 사용자는 클릭 후 GitHub 화면에서 본문을 직접 작성하고 **Create pull request**를 누르면 된다. (제목도 그 화면에서 얼마든지 고칠 수 있다)

링크 형식 — `title` 값은 **URL 인코딩**해서 넣는다(공백 → `%20`, 한글·`[`·`]`·`&`·`#` 등도 인코딩):
```
https://github.com/next-step/{repo}/compare/{본인_아이디}...{본인_아이디}:{작업브랜치}?expand=1&title={URL 인코딩한 제목}
```
예) `https://github.com/next-step/java-racingcar/compare/javajigi...javajigi:step1?expand=1&title=step1%20-%20javajigi`

마지막으로 안내: PR 생성 후, 넥스트스텝(edu.nextstep.camp) 강의 화면 우측 상단의 **`리뷰 요청`** 버튼을 눌러야 리뷰어에게 전달된다. *(이건 웹에서만 가능)*

## 하지 말 것
- base 브랜치를 next-step의 `master`로 두지 않는다. 반드시 **본인 아이디 브랜치**.
- push를 사용자 확인 없이 실행하지 않는다.
- PR 제목을 임의로 지어내지 않는다. 호출과 함께 준 제목이나 기본값 `{작업브랜치} - {본인_아이디}`만 쓴다.
- PR 본문을 대신 작성하거나 링크에 채워 넣지 않는다. (사용자가 GitHub 화면에서 직접 작성)
- `gh pr create` 등으로 PR을 대신 만들지 않는다. (이 스킬은 push까지만, PR 생성은 사용자 몫)
