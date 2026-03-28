# Skills Marketplace Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** kyungw00k/skills 레포에 Claude Code skills marketplace를 구성하고, 4개 CLI(juso, upbit, stealth-wright, apkpure)의 시나리오 기반 사용 가이드 스킬을 작성한다. 각 CLI 레포에도 스킬 설치 안내를 추가한다.

**Architecture:** `.claude-plugin/marketplace.json` 하나로 marketplace + plugin(cli-tools)을 정의. 각 스킬은 `skills/<name>/SKILL.md`에 YAML frontmatter + 시나리오 기반 마크다운으로 작성. 각 CLI 레포의 README와 GitHub Pages에 설치 가이드 섹션 추가.

**Tech Stack:** Claude Code Plugin System, Markdown, JSON

---

## File Structure

```
skills/                                          # kyungw00k/skills repo
├── .claude-plugin/
│   └── marketplace.json                         # marketplace + plugin 정의
├── skills/
│   ├── juso/SKILL.md                            # juso 사용 가이드 스킬
│   ├── upbit/SKILL.md                           # upbit 사용 가이드 스킬
│   ├── stealth-wright/SKILL.md                  # stealth-wright 사용 가이드 스킬
│   └── apkpure/SKILL.md                         # apkpure 사용 가이드 스킬
└── README.md                                    # 레포 소개 및 설치 안내

# 외부 레포 변경 (각 CLI 레포)
# kyungw00k/juso       — README.md 수정, docs/index.md 수정
# kyungw00k/upbit      — README.md 수정, docs/index.md 수정
# kyungw00k/apkpure    — README.md 수정, docs/index.md 수정
# kyungw00k/stealth-wright — README.md 수정 (GitHub Pages 없음)
```

---

### Task 1: Marketplace 설정

**Files:**
- Create: `.claude-plugin/marketplace.json`

- [ ] **Step 1: marketplace.json 작성**

```json
{
  "name": "kyungw00k-skills",
  "owner": {
    "name": "kyungw00k"
  },
  "metadata": {
    "description": "AI agent usage guides for kyungw00k CLI tools"
  },
  "plugins": [
    {
      "name": "cli-tools",
      "description": "AI agent usage guides for kyungw00k public CLI tools — juso, upbit, stealth-wright, apkpure",
      "source": "./",
      "skills": [
        "./skills/juso",
        "./skills/upbit",
        "./skills/stealth-wright",
        "./skills/apkpure"
      ]
    }
  ]
}
```

- [ ] **Step 2: JSON 유효성 확인**

Run: `python3 -m json.tool .claude-plugin/marketplace.json > /dev/null && echo "valid"`
Expected: `valid`

- [ ] **Step 3: Commit**

```bash
git add .claude-plugin/marketplace.json
git commit -m "feat: add marketplace.json for kyungw00k-skills"
```

---

### Task 2: juso 스킬 작성

**Files:**
- Create: `skills/juso/SKILL.md`

**참고:** 스킬 내용 작성 시 kyungw00k/juso README를 참조한다.
Run: `gh api repos/kyungw00k/juso/readme --jq .content | base64 -d`

- [ ] **Step 1: SKILL.md 작성**

juso의 README에서 명령어, 옵션, 출력 형식을 추출하여 시나리오 기반 스킬을 작성한다. 아래 구조를 따른다:

```markdown
---
name: juso
description: >-
  한국 주소/우편번호 검색 CLI. TRIGGER when: 사용자가 주소 검색, 우편번호 조회,
  "juso", "주소", "zipcode", "우편번호" 언급 시.
  DO NOT TRIGGER when: 지도/네비게이션, 해외 주소, 길찾기 질문.
---

# juso — 한국 주소/우편번호 검색 CLI

## 설치 확인

(juso --version 또는 which juso로 확인, 없으면 brew install kyungw00k/cli/juso)

## 핵심 명령어

(juso <keyword> 기본 검색, --lang, --jibun, -f 옵션 등)

## 시나리오별 사용법

- 주소로 우편번호를 찾고 싶을 때
- 영문 주소가 필요할 때
- 지번 주소가 필요할 때
- 검색 결과를 JSON으로 파이프해서 처리할 때
- 캐시 관리가 필요할 때

## 출력 형식

(TTY: table, 비TTY: json 자동 전환, -f 옵션으로 json/jsonl/csv 지정 가능)

## 주의사항

(Postcodify API 기반, 캐시 24h TTL, tool-schema 명령 존재)
```

각 섹션은 README의 실제 명령어와 옵션을 포함하여 에이전트가 바로 실행할 수 있는 수준으로 작성한다.

- [ ] **Step 2: 스킬 내용 검증**

README의 명령어/옵션과 SKILL.md의 내용이 일치하는지 확인한다.

- [ ] **Step 3: Commit**

```bash
git add skills/juso/SKILL.md
git commit -m "feat: add juso skill — Korean address/zipcode lookup guide"
```

---

### Task 3: upbit 스킬 작성

**Files:**
- Create: `skills/upbit/SKILL.md`

**참고:** 스킬 내용 작성 시 kyungw00k/upbit README를 참조한다.
Run: `gh api repos/kyungw00k/upbit/readme --jq .content | base64 -d`

- [ ] **Step 1: SKILL.md 작성**

upbit의 README에서 명령어 그룹(시세/거래/입출금/실시간), 옵션, 인증 방법을 추출하여 시나리오 기반 스킬을 작성한다:

```markdown
---
name: upbit
description: >-
  Upbit 암호화폐 거래소 CLI. TRIGGER when: 암호화폐 시세 조회, 코인 거래,
  "upbit", "코인 가격", "호가", "비트코인 시세", 잔고 확인 시.
  DO NOT TRIGGER when: 일반 금융/주식, 해외 거래소(Binance 등), 블록체인 개발.
---

# upbit — Upbit 암호화폐 거래소 CLI

## 설치 확인

(upbit --version, brew install kyungw00k/cli/upbit)

## 인증 설정

(UPBIT_ACCESS_KEY, UPBIT_SECRET_KEY 환경변수, 읽기 전용 vs 거래 권한)

## 핵심 명령어

시세 조회: ticker, candle, orderbook, trades, market
거래: buy, sell, balance, order
실시간: watch ticker/orderbook/trade/candle

## 시나리오별 사용법

- 특정 코인의 현재가를 알고 싶을 때
- 비트코인을 시장가로 매수할 때
- 잔고를 확인할 때
- 호가창을 실시간으로 볼 때 (TUI)
- 캔들 데이터를 JSON으로 추출할 때
- 퍼센트 기반 주문 (전체 잔고의 50% 매수)

## 출력 형식

(TTY: table, 비TTY: json 자동 전환, -f 옵션)

## 주의사항

(API 키 필요(거래), 읽기 전용은 키 불필요, 원화 마켓 기본, TUI는 비-에이전트용)
```

- [ ] **Step 2: 스킬 내용 검증**

README의 명령어/옵션과 SKILL.md의 내용이 일치하는지 확인한다.

- [ ] **Step 3: Commit**

```bash
git add skills/upbit/SKILL.md
git commit -m "feat: add upbit skill — cryptocurrency exchange CLI guide"
```

---

### Task 4: stealth-wright 스킬 작성

**Files:**
- Create: `skills/stealth-wright/SKILL.md`

**참고:** 스킬 내용 작성 시 kyungw00k/stealth-wright README를 참조한다.
Run: `gh api repos/kyungw00k/stealth-wright/readme --jq .content | base64 -d`

- [ ] **Step 1: SKILL.md 작성**

stealth-wright의 README에서 명령어(네비게이션/스냅샷/인터랙션/세션), ref 기반 주소 지정, semantic 로케이터를 추출하여 시나리오 기반 스킬을 작성한다:

```markdown
---
name: stealth-wright
description: >-
  스텔스 브라우저 자동화 CLI. TRIGGER when: 웹 자동화, 브라우저 조작, 스크린샷,
  "sw", 폼 입력, 웹 스크래핑, 페이지 캡처, 로그인 자동화 시.
  DO NOT TRIGGER when: 일반 HTTP 요청(curl/wget), REST API 호출, 서버사이드 작업.
---

# stealth-wright (sw) — 스텔스 브라우저 자동화 CLI

## 설치 확인

(sw --version, brew install kyungw00k/cli/stealth-wright)

## 핵심 명령어

네비게이션: open, goto, close, go-back, reload
스냅샷: snapshot, screenshot --annotate, pdf
인터랙션: click, fill, check, hover, type, press
세션: list, kill-all, daemon

## 시나리오별 사용법

- 웹 페이지의 구조를 파악할 때 (ARIA snapshot)
- 스크린샷을 찍어 시각적으로 확인할 때
- 어노테이트 스크린샷으로 요소를 식별할 때 (멀티모달 AI용)
- 로그인 폼을 자동으로 입력할 때
- ref 기반으로 특정 요소를 클릭할 때
- semantic 로케이터로 요소를 찾을 때

## 출력 형식

(snapshot: ARIA 텍스트, screenshot: PNG 파일, pdf: PDF 파일)

## 주의사항

(Playwright 기반, 데몬 모드, 스텔스 모드 기본 활성화)
```

- [ ] **Step 2: 스킬 내용 검증**

README의 명령어/옵션과 SKILL.md의 내용이 일치하는지 확인한다.

- [ ] **Step 3: Commit**

```bash
git add skills/stealth-wright/SKILL.md
git commit -m "feat: add stealth-wright skill — browser automation CLI guide"
```

---

### Task 5: apkpure 스킬 작성

**Files:**
- Create: `skills/apkpure/SKILL.md`

**참고:** 스킬 내용 작성 시 kyungw00k/apkpure README를 참조한다.
Run: `gh api repos/kyungw00k/apkpure/readme --jq .content | base64 -d`

- [ ] **Step 1: SKILL.md 작성**

apkpure의 README에서 명령어, 옵션(버전 지정, 아키텍처, 배치)을 추출하여 시나리오 기반 스킬을 작성한다:

```markdown
---
name: apkpure
description: >-
  APKPure에서 APK 다운로드 CLI. TRIGGER when: APK 다운로드, 안드로이드 앱 파일,
  "apkpure", "apk 받기", 특정 앱 APK 필요 시.
  DO NOT TRIGGER when: Play Store 리뷰/정보 조회, iOS 앱, 앱 개발.
---

# apkpure — APK 다운로드 CLI

## 설치 확인

(apkpure --version, brew install kyungw00k/cli/apkpure)

## 핵심 명령어

(apkpure -a <앱ID> <출력경로>, -l 버전 목록, -c CSV 배치, @버전 지정)

## 시나리오별 사용법

- 특정 앱의 최신 APK를 다운로드할 때
- 특정 버전의 APK가 필요할 때
- 여러 앱을 한꺼번에 다운로드할 때 (CSV 배치)
- 특정 아키텍처(arm64-v8a)의 APK가 필요할 때
- 사용 가능한 버전 목록을 확인할 때

## 출력 형식

(다운로드된 APK 파일, -l은 버전 목록 텍스트 출력)

## 주의사항

(APKPure 사이트 의존, 앱 ID는 Google Play 패키지명)
```

- [ ] **Step 2: 스킬 내용 검증**

README의 명령어/옵션과 SKILL.md의 내용이 일치하는지 확인한다.

- [ ] **Step 3: Commit**

```bash
git add skills/apkpure/SKILL.md
git commit -m "feat: add apkpure skill — APK download CLI guide"
```

---

### Task 6: README.md 작성

**Files:**
- Create: `README.md`

- [ ] **Step 1: README.md 작성**

```markdown
# kyungw00k-skills

AI agent usage guides for kyungw00k CLI tools.

## Skills

| Skill | Description |
|-------|-------------|
| [juso](skills/juso/SKILL.md) | 한국 주소/우편번호 검색 |
| [upbit](skills/upbit/SKILL.md) | Upbit 암호화폐 거래소 시세/거래 |
| [stealth-wright](skills/stealth-wright/SKILL.md) | 스텔스 브라우저 자동화 |
| [apkpure](skills/apkpure/SKILL.md) | APK 다운로드 |

## Install

Add the marketplace and enable the plugin:

```bash
claude /install kyungw00k-skills
```

Or manually add to your Claude Code settings:

```json
{
  "permissions": {
    "additionalMarketplaces": ["kyungw00k/skills"]
  }
}
```

## License

MIT
```

- [ ] **Step 2: Commit**

```bash
git add README.md
git commit -m "feat: add README with skill list and install instructions"
```

---

### Task 7: 각 CLI 레포 README에 스킬 설치 안내 추가

**Files:**
- Modify: `kyungw00k/juso` — README.md
- Modify: `kyungw00k/upbit` — README.md
- Modify: `kyungw00k/stealth-wright` — README.md
- Modify: `kyungw00k/apkpure` — README.md

각 레포를 clone하여 README에 "Claude Code Skill" 섹션을 추가한다.

- [ ] **Step 1: juso 레포 README 수정**

kyungw00k/juso를 clone하고, README.md의 적절한 위치 (보통 Install 섹션 뒤)에 다음 섹션을 추가:

```markdown
## Claude Code Skill

AI 에이전트가 juso를 자동으로 활용할 수 있는 Claude Code 스킬이 제공됩니다.

```bash
claude /install kyungw00k-skills
```
```

커밋하고 PR을 생성하거나 직접 push.

- [ ] **Step 2: upbit 레포 README 수정**

kyungw00k/upbit에 동일한 "Claude Code Skill" 섹션 추가.

- [ ] **Step 3: stealth-wright 레포 README 수정**

kyungw00k/stealth-wright에 동일한 "Claude Code Skill" 섹션 추가.

- [ ] **Step 4: apkpure 레포 README 수정**

kyungw00k/apkpure에 동일한 "Claude Code Skill" 섹션 추가.

- [ ] **Step 5: 각 레포 커밋**

각 레포에서:
```bash
git add README.md
git commit -m "docs: add Claude Code skill install guide"
git push
```

---

### Task 8: GitHub Pages에 스킬 설치 안내 추가

**Files:**
- Modify: `kyungw00k/juso` — docs/index.md (또는 해당 Pages 소스)
- Modify: `kyungw00k/upbit` — docs/index.md
- Modify: `kyungw00k/apkpure` — docs/index.md
- stealth-wright는 GitHub Pages 없음 — 스킵

각 레포의 GitHub Pages 소스 파일을 확인하고, 랜딩 페이지에 "Claude Code Skill" 설치 안내를 추가한다.

- [ ] **Step 1: juso GitHub Pages 소스 확인 및 수정**

Run: `gh api repos/kyungw00k/juso/pages --jq '.source'`
해당 경로의 index 파일에 스킬 설치 안내 섹션 추가.

- [ ] **Step 2: upbit GitHub Pages 소스 확인 및 수정**

동일하게 upbit의 Pages 소스 확인 후 수정.

- [ ] **Step 3: apkpure GitHub Pages 소스 확인 및 수정**

동일하게 apkpure의 Pages 소스 확인 후 수정.

- [ ] **Step 4: 각 레포 커밋 및 push**

```bash
git add docs/
git commit -m "docs: add Claude Code skill section to GitHub Pages"
git push
```
