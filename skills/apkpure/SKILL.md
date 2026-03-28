---
name: apkpure
description: >-
  APKPure에서 안드로이드 APK 파일을 다운로드하는 CLI. TRIGGER when: APK 다운로드, 안드로이드 앱 파일,
  "apkpure", "apk 받기", 특정 앱 APK 필요, 안드로이드 앱을 파일로 받고 싶을 때.
  DO NOT TRIGGER when: Play Store 리뷰/정보 조회, iOS 앱, 앱 개발, 앱 사용법 문의.
---

# apkpure — APKPure APK 다운로드 CLI

## 설치 확인

```bash
which apkpure || echo "설치 필요"
apkpure -V 2>/dev/null || echo "버전 확인 불가"
```

설치가 안 되어 있으면:

```bash
# Homebrew
brew install kyungw00k/cli/apkpure

# Shell Script
curl -fsSL https://kyungw00k.dev/apkpure/install.sh | sh

# Go
go install github.com/kyungw00k/apkpure/cmd/apkpure@latest
```

## 핵심 명령어

```bash
apkpure -a <패키지ID> <출력경로>                        # 최신 APK 다운로드
apkpure -a <패키지ID>@<버전> <출력경로>                 # 특정 버전 APK 다운로드
apkpure -l -a <패키지ID>                               # 사용 가능한 버전 목록 확인
apkpure -c apps.csv <출력경로>                          # CSV 파일에서 일괄 다운로드
apkpure -a <패키지ID> -o arch=arm64-v8a <출력경로>      # 특정 아키텍처 APK 다운로드
```

### 옵션 레퍼런스

| 플래그 | 기본값 | 설명 |
|--------|--------|------|
| `-a, --app` | | 앱 패키지 ID (예: `com.instagram.android` 또는 `com.instagram.android@1.2.3`) |
| `-c, --csv` | | 앱 ID 목록이 담긴 CSV 파일 경로 |
| `-f, --field` | `1` | CSV에서 앱 ID가 담긴 필드 번호 |
| `-v, --version-field` | | CSV에서 버전이 담긴 필드 번호 |
| `-l, --list-versions` | | 사용 가능한 버전 목록 출력 |
| `-o, --options` | | 추가 옵션 (예: `arch=arm64-v8a,language=ko-KR`) |
| `-r, --parallel` | `4` | 병렬 다운로드 수 |
| `-s, --sleep-duration` | `0` | 다운로드 사이 대기 시간 (밀리초) |
| `-V, --version` | | CLI 버전 출력 |

### `-o` 옵션 레퍼런스

| 키 | 설명 | 예시 |
|----|------|------|
| `arch` | CPU 아키텍처 | `arm64-v8a`, `armeabi-v7a`, `x86`, `x86_64` |
| `language` | 언어 코드 | `ko-KR`, `en-US` |
| `os_ver` | Android OS 버전 | `35` (Android 15) |
| `output_format` | 목록 명령어 출력 형식 | `plaintext`, `json` |

## 시나리오별 사용법

### 1. 특정 앱의 최신 APK를 다운로드할 때

```bash
# Instagram 최신 APK 다운로드
apkpure -a com.instagram.android ./downloads

# YouTube 최신 APK 다운로드
apkpure -a com.google.android.youtube ./downloads
```

### 2. 특정 버전의 APK가 필요할 때

```bash
# 먼저 사용 가능한 버전 목록 확인
apkpure -l -a com.instagram.android

# 특정 버전 지정하여 다운로드
apkpure -a com.instagram.android@150.0.0.0 ./downloads
```

### 3. 여러 앱을 한꺼번에 다운로드할 때 (CSV 배치)

```bash
# apps.csv 파일 준비 (한 줄에 패키지 ID 하나)
# com.instagram.android
# com.google.android.youtube
# com.kakao.talk

# CSV 파일로 일괄 다운로드 (기본 4개 병렬)
apkpure -c apps.csv ./downloads

# 병렬 다운로드 수 조정
apkpure -c apps.csv -r 2 ./downloads

# 다운로드 간격 추가 (서버 부하 방지, 1초 대기)
apkpure -c apps.csv -s 1000 ./downloads
```

CSV에 버전 정보도 포함된 경우:

```bash
# apps_with_version.csv 예시:
# com.instagram.android,150.0.0.0
# com.google.android.youtube,18.32.39

apkpure -c apps_with_version.csv -f 1 -v 2 ./downloads
```

### 4. 특정 아키텍처(arm64-v8a)의 APK가 필요할 때

```bash
# arm64-v8a 아키텍처 APK 다운로드
apkpure -a com.instagram.android -o arch=arm64-v8a ./downloads

# 여러 옵션 동시 적용 (아키텍처 + 언어)
apkpure -a com.instagram.android -o arch=arm64-v8a,language=ko-KR ./downloads

# x86 에뮬레이터용 APK
apkpure -a com.instagram.android -o arch=x86 ./downloads
```

### 5. 사용 가능한 버전 목록을 확인할 때

```bash
# 기본 텍스트 형식으로 버전 목록 출력
apkpure -l -a com.instagram.android

# JSON 형식으로 버전 목록 출력
apkpure -l -a com.instagram.android -o output_format=json

# JSON 출력 후 jq로 파싱
apkpure -l -a com.instagram.android -o output_format=json | jq '.'
```

## 출력 형식

### 다운로드 완료 시

다운로드가 완료되면 지정한 출력 경로에 APK 파일이 저장됩니다.

```
<출력경로>/<패키지ID>-<버전>.apk
```

예시:
```
./downloads/com.instagram.android-150.0.0.0.apk
```

### 버전 목록 출력 (plaintext)

```
150.0.0.0
149.0.0.0
148.0.0.0
...
```

### 버전 목록 출력 (JSON)

```json
[
  "150.0.0.0",
  "149.0.0.0",
  "148.0.0.0"
]
```

## 주의사항

- **인터넷 연결 필요**: APKPure 서버에서 직접 다운로드하므로 인터넷 연결이 필요합니다.
- **패키지 ID 확인**: 패키지 ID는 Google Play Store URL의 `id=` 파라미터에서 확인할 수 있습니다. (예: `play.google.com/store/apps/details?id=com.instagram.android`)
- **아키텍처 미지정 시**: `-o arch=` 옵션 없이 다운로드하면 APKPure가 범용(universal) APK를 제공합니다. 특정 기기 호환성이 필요하면 아키텍처를 명시하세요.
- **버전 표기**: `@` 뒤에 버전을 붙여 특정 버전을 지정합니다. 정확한 버전 문자열은 `-l` 옵션으로 먼저 확인하세요.
- **병렬 다운로드**: `-r` 기본값은 4입니다. 네트워크 상황에 따라 조절하세요.
- **이용 약관**: APKPure 서비스 이용 약관을 준수하여 사용하세요.
