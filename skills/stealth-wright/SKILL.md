---
name: sw
description: >-
  브라우저 자동화 CLI (스텔스 기능 포함). TRIGGER when: 웹 자동화, 브라우저 조작, 스크린샷, "sw",
  폼 입력, 웹 스크래핑, 페이지 캡처, 로그인 자동화, 요소 클릭, 페이지 탐색 요청이 있을 때.
  DO NOT TRIGGER when: 일반 HTTP 요청(curl/wget), REST API 호출, 서버사이드 작업.
---

# sw (Stealth Wright) — 스텔스 브라우저 자동화 CLI

## 설치 확인

```bash
which sw || echo "설치 필요"
sw --version 2>/dev/null || echo "버전 확인 불가"
```

설치가 안 되어 있으면:

```bash
# Homebrew
brew install kyungw00k/cli/sw

# 소스 빌드
git clone https://github.com/kyungw00k/stealth-wright.git
cd stealth-wright
go build -o build/sw ./cmd/sw
make install
```

## 핵심 명령어

### 탐색 (Navigation)

```bash
sw open <url>              # 브라우저 열고 URL로 이동
sw goto <url>              # 현재 세션에서 URL로 이동
sw go-back                 # 뒤로 가기
sw go-forward              # 앞으로 가기
sw reload                  # 페이지 새로고침
sw close                   # 브라우저 세션 닫기
```

### 스냅샷 & 스크린샷

```bash
sw snapshot                         # ARIA 트리 + 요소 ref 생성
sw screenshot                       # 스크린샷 저장
sw screenshot --annotate            # ref 번호가 오버레이된 어노테이트 스크린샷
sw screenshot --full-page           # 전체 페이지 스크린샷
sw screenshot --filename out.png    # 특정 파일명으로 저장
sw pdf [filename]                   # 페이지를 PDF로 저장
```

### 상호작용 — ref 기반 (snapshot 선행 필요)

```bash
sw click <ref>             # 요소 클릭 (예: sw click e5)
sw dblclick <ref>          # 더블클릭
sw hover <ref>             # 마우스 오버
sw fill <ref> <text>       # 입력 필드에 텍스트 입력
sw check <ref>             # 체크박스/라디오 체크
sw uncheck <ref>           # 체크 해제
sw select <ref> <value>    # 드롭다운 선택
```

### 상호작용 — semantic 로케이터 (snapshot 불필요)

```bash
sw click --role button --text "Submit"
sw click --label "닫기"
sw fill --label "Email" "user@example.com"
sw fill --placeholder "검색어를 입력하세요" "query"
sw hover --role link --text "About"
sw check --label "로그인 상태 유지"

# 요소 탐색
sw find --role button
sw find --text "로그인"
sw find --label "이메일"
sw find --placeholder "비밀번호"
```

### Semantic 로케이터 플래그

| 플래그 | 설명 |
|--------|------|
| `--role <aria-role>` | ARIA 역할: button, link, textbox, checkbox, heading 등 |
| `--text <string>` | 보이는 텍스트 (부분 일치) |
| `--label <string>` | aria-label 속성 (부분 일치) |
| `--placeholder <string>` | input placeholder (부분 일치) |
| `--exact` | 정확한 문자열 매칭 요구 |

### 전역 옵션

| 플래그 | 기본값 | 설명 |
|--------|--------|------|
| `-s, --session <name>` | `default` | 세션 이름 |
| `-b, --browser <type>` | chromium | 브라우저: chromium, firefox, webkit |
| `--headed` | false | 헤드 모드(화면 표시) |
| `--stealth` | true | 스텔스 모드 활성화 |
| `--no-stealth` | — | 스텔스 모드 비활성화 |
| `--persistent` | false | 프로필 디스크 저장 |

## 시나리오별 사용법

### 1. 웹 페이지의 구조를 파악할 때 (ARIA snapshot)

페이지의 요소 구조를 파악하고 `[ref=eN]` 참조를 얻는 기본 워크플로우:

```bash
sw open https://example.com
sw snapshot
```

출력 예시:
```yaml
- heading "Example Domain" [level=1] [ref=e1]
- paragraph [ref=e2]: This domain is for use in illustrative examples.
- link "More information..." [ref=e3]
```

이후 ref를 사용해 요소와 상호작용:
```bash
sw click e3           # "More information..." 링크 클릭
sw fill e1 "텍스트"   # heading에 텍스트 입력
```

### 2. 스크린샷을 찍어 시각적으로 확인할 때

```bash
sw open https://example.com

# 기본 스크린샷
sw screenshot

# 전체 페이지 스크린샷
sw screenshot --full-page

# 특정 파일명으로 저장
sw screenshot --filename result.png
```

### 3. 어노테이트 스크린샷으로 요소를 식별할 때 (멀티모달 AI용)

ref 번호가 각 요소 위에 오버레이된 스크린샷 — 멀티모달 AI 모델이 요소를 시각적으로 식별하는 데 유용:

```bash
sw open https://example.com
sw screenshot --annotate
# → 각 요소에 [e1], [e2], [e3] 등 ref 번호가 표시된 이미지 저장
```

어노테이트 스크린샷을 확인한 뒤 ref 번호로 바로 조작:
```bash
sw click e5
sw fill e2 "입력할 내용"
```

### 4. 로그인 폼을 자동으로 입력할 때

**방법 A — snapshot + ref 기반:**

```bash
sw open https://example.com/login
sw snapshot
# → - textbox "이메일" [ref=e1]
# → - textbox "비밀번호" [ref=e2]
# → - button "로그인" [ref=e3]

sw fill e1 "user@example.com"
sw fill e2 "mypassword"
sw click e3
```

**방법 B — semantic 로케이터 (snapshot 불필요):**

```bash
sw open https://example.com/login
sw fill --label "이메일" "user@example.com"
sw fill --placeholder "비밀번호" "mypassword"
sw click --role button --text "로그인"
```

### 5. ref 기반으로 특정 요소를 클릭할 때

```bash
sw open https://example.com
sw snapshot

# 링크 클릭
sw click e3

# 우클릭 (컨텍스트 메뉴)
sw click e3 right

# 더블클릭
sw dblclick e5

# 체크박스
sw check e7
sw uncheck e7

# 드롭다운 선택
sw select e4 "옵션값"
```

### 6. semantic 로케이터로 요소를 찾을 때

snapshot 없이도 ARIA 속성으로 요소를 직접 찾고 조작:

```bash
sw open https://example.com

# 모든 버튼 탐색
sw find --role button
# → ### Found 3 element(s)
# → - [ref=e3] <button> "로그인"
# → - [ref=e7] <button> "회원가입"
# → - [ref=e12] <button> "비밀번호 찾기"

# 텍스트로 탐색
sw find --text "로그인"

# label로 탐색
sw find --label "이메일 주소"

# 찾은 후 바로 조작
sw click --role button --text "로그인"
sw fill --label "이메일 주소" "user@example.com"
```

### 7. 전형적인 AI 에이전트 워크플로우

open → snapshot → 요소 식별 → 상호작용 → 검증:

```bash
# 1. 브라우저 열기
sw open https://example.com

# 2. 구조 파악
sw snapshot

# 3. 어노테이트 스크린샷으로 시각 확인 (멀티모달 모델용)
sw screenshot --annotate

# 4. ref 또는 semantic으로 상호작용
sw fill e1 "검색어"
sw click --role button --text "검색"

# 5. 결과 검증
sw snapshot
sw screenshot
```

### 8. 다중 세션 관리

```bash
# 별도 세션으로 브라우저 열기
sw open https://site-a.com -s session-a
sw open https://site-b.com -s session-b

# 세션 목록 확인
sw list

# 세션 전환 (각 명령어에 -s 플래그 사용)
sw snapshot -s session-a
sw click e3 -s session-b

# 세션 닫기
sw close -s session-a
sw kill-all   # 모든 세션 종료
```

### 9. 상태 저장 및 복원 (로그인 세션 유지)

```bash
# 로그인 후 상태 저장
sw open https://example.com/login
sw fill --label "이메일" "user@example.com"
sw fill --placeholder "비밀번호" "mypassword"
sw click --role button --text "로그인"
sw state-save ./login-state.json

# 다음 세션에서 로그인 상태 복원
sw open https://example.com
sw state-load ./login-state.json
# → 로그인 상태로 바로 사용 가능
```

## 출력 형식

### snapshot 출력 (ARIA 트리)

```yaml
- heading "페이지 제목" [level=1] [ref=e1]
- paragraph [ref=e2]: 본문 텍스트
- textbox "이메일" [ref=e3]
- textbox "비밀번호" [ref=e4]
- button "로그인" [ref=e5]
- link "비밀번호 찾기" [ref=e6]
```

`[ref=eN]` 형식의 참조값은 이후 `sw click`, `sw fill` 등 명령어에서 바로 사용 가능.

### find 출력

```
### Found 3 element(s)
- [ref=e3] <button> "로그인"
- [ref=e7] <button> "회원가입"
- [ref=e12] <button> "비밀번호 찾기"
```

## 주의사항

- **스텔스 모드 기본 활성화**: WebDriver 마커 숨김, 핑거프린트 스푸핑, WebRTC 누출 방지 등 봇 탐지 우회 기능이 기본으로 켜져 있음. 비활성화 시 `--no-stealth` 사용.
- **데몬 방식 동작**: CLI와 브라우저 프로세스가 Unix 소켓을 통한 JSON-RPC 2.0으로 통신. `sw open` 시 데몬이 자동 시작됨.
- **ref는 snapshot 후 유효**: `[ref=eN]` 참조는 마지막 `sw snapshot` 기준. 페이지 변경 후엔 다시 snapshot 필요.
- **semantic 로케이터 우선**: snapshot 없이 바로 조작할 때는 `--role`, `--text`, `--label`, `--placeholder` 플래그 활용.
- **세션 격리**: `-s <name>` 플래그로 독립적인 브라우저 세션 관리. 기본 세션명은 `default`.
- **브라우저 바이너리 설치**: 처음 사용 시 브라우저 바이너리가 없으면 `sw install-browser` 실행.
