# kyungw00k/skills Marketplace Design

## 목적

kyungw00k의 공개 CLI 도구들을 AI 에이전트가 활용할 수 있도록 Claude Code skills marketplace로 구성한다. 각 스킬은 시나리오 기반 사용 가이드로, 에이전트가 언제/어떻게 해당 CLI를 사용할지 판단할 수 있게 한다.

## 레포 구조

```
skills/                              # github.com/kyungw00k/skills
├── .claude-plugin/
│   └── marketplace.json             # marketplace + plugin 정의
├── skills/
│   ├── juso/SKILL.md                # 한국 주소/우편번호 검색
│   ├── upbit/SKILL.md               # Upbit 암호화폐 거래소
│   ├── stealth-wright/SKILL.md      # 브라우저 자동화
│   └── apkpure/SKILL.md             # APK 다운로드
└── README.md
```

## marketplace.json

```json
{
  "name": "kyungw00k-skills",
  "owner": { "name": "kyungw00k" },
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

## SKILL.md 공통 포맷

각 스킬은 동일한 구조를 따른다:

```markdown
---
name: <cli-name>
description: >-
  <한 줄 설명>. TRIGGER when: <트리거 조건들>.
  DO NOT TRIGGER when: <비트리거 조건들>.
---

# <cli-name> — <한 줄 설명>

## 설치 확인
(설치 여부 체크 및 설치 방법)

## 핵심 명령어
(주요 명령어 + 옵션 레퍼런스)

## 시나리오별 사용법
(구체적 상황 → 명령어 매핑)

## 출력 형식
(에이전트가 파싱할 수 있는 형식 안내)

## 주의사항
(API 키, 제한, 인증 등)
```

## 스킬별 상세

### juso

- **TRIGGER**: 주소 검색, 우편번호 조회, "juso", "주소", "zipcode"
- **DO NOT TRIGGER**: 지도/네비게이션, 해외 주소
- **핵심 시나리오**: 우편번호 찾기, 영문 주소 변환, 지번↔도로명, JSON 파이프
- **출력**: table(기본), json, jsonl, csv
- **주의**: Postcodify API 기반, 캐시 24시간 TTL

### upbit

- **TRIGGER**: 암호화폐 시세, 거래, "upbit", "코인 가격", 호가
- **DO NOT TRIGGER**: 일반 금융/주식, 해외 거래소
- **핵심 시나리오**: 현재가 조회, 매수/매도, 잔고 확인, 실시간 모니터링(TUI)
- **출력**: table(기본), json, jsonl, csv
- **주의**: 거래 기능은 `UPBIT_ACCESS_KEY`, `UPBIT_SECRET_KEY` 환경변수 필요

### stealth-wright

- **TRIGGER**: 웹 자동화, 브라우저 조작, 스크린샷, "sw", 폼 입력
- **DO NOT TRIGGER**: 일반 HTTP 요청(curl), API 호출
- **핵심 시나리오**: 페이지 스냅샷, 폼 자동입력, 로그인 자동화, 어노테이트 스크린샷
- **출력**: ARIA snapshot(텍스트), 스크린샷(이미지), PDF
- **주의**: Playwright 기반, 데몬 모드 지원 (Unix socket)

### apkpure

- **TRIGGER**: APK 다운로드, 안드로이드 앱 파일, "apkpure"
- **DO NOT TRIGGER**: Play Store 리뷰/정보 조회, iOS 앱
- **핵심 시나리오**: 특정 앱 APK 다운로드, 버전 지정 다운로드, 배치 다운로드
- **출력**: 다운로드된 APK 파일
- **주의**: APKPure 사이트 의존

## 각 CLI 레포 업데이트

skills marketplace를 만든 후, 각 CLI 레포(juso, upbit, stealth-wright, apkpure)에도 스킬 설치 가이드를 추가한다.

### README.md 추가 섹션

각 CLI 레포의 README.md에 "Claude Code Skill" 섹션을 추가:

```markdown
## Claude Code Skill

AI 에이전트가 이 CLI를 자동으로 활용할 수 있는 Claude Code 스킬이 제공됩니다.

### 설치

marketplace 추가 후 plugin 설치:

\`\`\`bash
claude /install kyungw00k-skills
\`\`\`

또는 settings.json에 직접 추가:

\`\`\`json
{
  "permissions": {
    "additionalMarketplaces": ["kyungw00k/skills"]
  }
}
\`\`\`
```

### GitHub Pages 추가

각 CLI의 GitHub Pages 랜딩 페이지에도 Claude Code Skill 설치 안내를 추가한다. (GitHub Pages가 있는 레포에 한해)

## 범위 외

- **private CLI** (unmillie, kyobo-cli, ridi-cli, toon-dl, nxziperto, nsw-dl): 별도 private 레포로 관리
- **go-cli-dev** (개발 가이드): private skills 레포로 관리
- 개발 가이드, 코드 컨벤션, 아키텍처 문서는 이 marketplace에 포함하지 않음
