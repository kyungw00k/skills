---
name: juso
description: >-
  한국 우편번호 및 주소 검색 CLI. TRIGGER when: 주소 검색, 우편번호 조회, "juso", "주소", "zipcode",
  "우편번호", 한국 주소 변환, 도로명/지번 조회 요청이 있을 때.
  DO NOT TRIGGER when: 지도/네비게이션, 해외 주소 검색, 길찾기, 위도/경도 조회.
---

# juso — 한국 우편번호/주소 검색 CLI

## 설치 확인

```bash
which juso || echo "설치 필요"
juso --version 2>/dev/null || echo "버전 확인 불가"
```

설치가 안 되어 있으면:

```bash
# Homebrew
brew install kyungw00k/cli/juso

# Shell Script
curl -fsSL https://kyungw00k.dev/juso/install.sh | sh

# Go
go install github.com/kyungw00k/juso/cmd/juso@latest
```

## 핵심 명령어

```bash
juso <키워드>                    # 우편번호 + 도로명 주소 검색
juso <키워드> --lang en          # 영문 주소 출력
juso <키워드> --lang all         # 한/영 동시 출력
juso <키워드> --jibun            # 지번 주소 출력
juso <키워드> -o json            # JSON 출력
juso <키워드> -o jsonl           # JSON Lines 출력 (스트리밍)
juso <키워드> -o csv             # CSV 출력
juso <키워드> -o table           # 테이블 출력 (강제)
```

### 옵션 레퍼런스

| 플래그 | 기본값 | 설명 |
|--------|--------|------|
| `-o, --output` | `auto` | 출력 형식: `auto`, `table`, `json`, `jsonl`, `csv` |
| `--lang` | `ko` | 언어: `ko`(한국어), `en`(영문), `all`(한/영 동시) |
| `--jibun` | `false` | 지번 주소 출력 (기본은 도로명) |

### 서브커맨드

```bash
juso cache stats        # 캐시 통계 (건수, 크기)
juso cache clear        # 캐시 전체 삭제
juso tool-schema        # AI Agent용 JSON Schema 출력
juso tool-schema search # 검색 명령어 Schema만
juso update             # 최신 버전으로 업데이트
juso update --check     # 업데이트 확인만
```

## 시나리오별 사용법

### 1. 우편번호 찾기

```bash
# 특정 도로명으로 우편번호 검색
juso 테헤란로

# 출력 예시:
# 우편번호  주소
# 06134     서울특별시 강남구 테헤란로 101
# 06134     서울특별시 강남구 테헤란로 103
```

```bash
# 건물명이나 지역명으로도 검색 가능
juso 강남역
juso 코엑스
```

### 2. 영문 주소 변환

```bash
# 도로명 → 영문 주소
juso 테헤란로 --lang en

# 출력 예시:
# 우편번호  주소
# 06134     101, Teheran-ro, Gangnam-gu, Seoul
```

```bash
# ASCII 키워드 입력 시 자동으로 영문 모드 적용
juso teheran-ro

# 한/영 동시 확인
juso 강남대로 --lang all
```

### 3. 지번 주소 조회 (도로명 ↔ 지번)

```bash
# 도로명 키워드로 지번 주소 출력
juso 테헤란로 --jibun

# 동 이름으로 지번 주소 검색
juso 역삼동 --jibun
```

### 4. JSON 파이프 처리

```bash
# JSON 출력 후 jq로 파싱
juso 강남대로 -o json | jq '.[].postcode5'
juso 강남대로 -o json | jq '.[0]'

# 파이프 연결 시 자동으로 compact JSON 출력 (auto 모드 동작)
juso 강남대로 | jq '.'

# CSV 파일로 내보내기
juso 테헤란로 -o csv > results.csv

# JSON Lines (스트리밍 처리에 유용)
juso 강남구 -o jsonl | while read line; do echo "$line" | jq '.postcode5'; done
```

## 출력 형식

### auto 모드 동작

| 실행 환경 | 출력 형식 |
|-----------|-----------|
| 터미널 직접 실행 | `table` (사람이 읽기 좋은 형식) |
| 파이프로 연결 | `json` (compact) |

### JSON 필드 레퍼런스

| 필드 | 설명 | 예시 |
|------|------|------|
| `postcode5` | 5자리 우편번호 | `"06134"` |
| `postcode6` | 6자리 우편번호 (구) | `"135080"` |
| `koAddress` | 한국어 도로명 주소 | `"서울특별시 강남구 테헤란로 101"` |
| `koJibun` | 한국어 지번 주소 | `"서울특별시 강남구 역삼동 718-1"` |
| `enAddress` | 영문 도로명 주소 | `"101, Teheran-ro, Gangnam-gu, Seoul"` |
| `enJibun` | 영문 지번 주소 | `"718-1, Yeoksam-dong, Gangnam-gu, Seoul"` |
| `buildingName` | 건물명 | `"GS타워"` |
| `kakaoMapURL` | 카카오맵 검색 링크 | `"https://map.kakao.com/..."` |
| `naverMapURL` | 네이버지도 검색 링크 | `"https://map.naver.com/..."` |

### JSON 출력 예시

```bash
$ juso 테헤란로 101 -o json
[
  {
    "postcode5": "06134",
    "postcode6": "135080",
    "koAddress": "서울특별시 강남구 테헤란로 101",
    "koJibun": "서울특별시 강남구 역삼동 718",
    "enAddress": "101, Teheran-ro, Gangnam-gu, Seoul",
    "enJibun": "718, Yeoksam-dong, Gangnam-gu, Seoul",
    "buildingName": "파르나스타워",
    "kakaoMapURL": "https://map.kakao.com/...",
    "naverMapURL": "https://map.naver.com/..."
  }
]
```

## 주의사항

- **데이터 출처**: [Postcodify](https://www.poesis.dev/postcodify/) API 기반. 인터넷 연결 필요.
- **캐시**: 검색 결과는 SQLite에 로컬 캐시됨 (TTL: 24시간).
  - 캐시 경로: `$XDG_CACHE_HOME/juso/cache.db` → `~/.cache/juso/cache.db` → `~/.juso/cache.db` 순서로 탐색
  - 캐시 초기화: `juso cache clear`
- **API 키 불필요**: 별도 인증 없이 사용 가능.
- **해외 주소 불가**: 한국 주소 전용. 해외 주소 검색은 지원하지 않음.
- **검색 정확도**: 도로명, 지번, 건물명, 동/구/시 이름 모두 키워드로 사용 가능.
