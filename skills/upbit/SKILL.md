---
name: upbit
description: >-
  Upbit 암호화폐 거래소 CLI. TRIGGER when: 암호화폐 시세 조회, 코인 거래, "upbit", "코인 가격", "호가",
  "비트코인 시세", 잔고 확인, BTC/ETH/XRP 등 코인 매수/매도, 캔들 데이터 조회, 실시간 시세 확인 요청이 있을 때.
  DO NOT TRIGGER when: 일반 금융/주식 거래, 해외 거래소(Binance, Coinbase 등), 블록체인 개발, 스마트 컨트랙트.
---

# upbit — Upbit 암호화폐 거래소 CLI

## 설치 확인

```bash
which upbit || echo "설치 필요"
upbit --version 2>/dev/null || echo "버전 확인 불가"
```

설치가 안 되어 있으면:

```bash
# Shell Script (빠른 설치)
curl -sSL https://kyungw00k.dev/upbit/install.sh | sh

# Homebrew
brew install kyungw00k/cli/upbit
```

## 인증 설정

시세 조회 명령어는 **인증 없이** 사용할 수 있습니다.
거래, 잔고 조회, 입출금, 실시간 개인 스트림은 **인증이 필요합니다**.

```bash
export UPBIT_ACCESS_KEY=your_access_key
export UPBIT_SECRET_KEY=your_secret_key
```

API 키는 환경변수에서만 읽으며 디스크에 저장되지 않습니다.
Upbit API 키는 [Upbit 개발자 센터](https://upbit.com/mypage/open_api_management)에서 발급받을 수 있습니다.

### 인증 필요 여부 요약

| 카테고리 | 인증 필요 여부 |
|---------|--------------|
| 시세 (`ticker`, `candle`, `orderbook`, `trades`, `market`) | 불필요 |
| 거래 (`buy`, `sell`, `balance`, `order`) | **필요** |
| 입출금 (`wallet`, `deposit`, `withdraw`) | **필요** |
| 실시간 공개 (`watch ticker/orderbook/trade/candle`) | 불필요 |
| 실시간 개인 (`watch my-order`, `watch my-asset`) | **필요** |

## 핵심 명령어

### 시세 조회 (인증 불필요)

```bash
upbit ticker KRW-BTC                           # 비트코인 현재가
upbit ticker KRW-BTC KRW-ETH KRW-XRP          # 복수 코인 현재가
upbit candle KRW-BTC                           # 캔들 데이터 (기본 1일봉)
upbit candle KRW-BTC -i 1d --from 2025-01-01   # 특정 기간 일봉
upbit candle KRW-BTC -i 1m                     # 1분봉
upbit orderbook KRW-BTC                        # 호가창 스냅샷
upbit trades KRW-BTC                           # 최근 체결 내역
upbit market                                   # 전체 거래 가능 마켓 목록
upbit tick-size KRW-BTC                        # 호가 단위 확인
```

### 거래 (인증 필요)

```bash
upbit balance                                  # 포트폴리오 잔고 (KRW 평가액)
upbit buy KRW-BTC -p 100000000 -V 0.001        # 지정가 매수 (1억원에 0.001 BTC)
upbit buy KRW-BTC -p now -V 0.001              # 현재가로 0.001 BTC 매수
upbit buy KRW-BTC -p now -V 50%               # 현재가로 KRW 잔고 50% 매수
upbit sell KRW-BTC -V 100%                     # 전량 시장가 매도
upbit sell KRW-BTC -V 50%                      # 보유량의 50% 매도
upbit sell KRW-BTC -p 120000000 -V 0.001       # 지정가 매도
```

### 주문 관리 (인증 필요)

```bash
upbit order list                               # 미체결 주문 목록
upbit order show <uuid>                        # 특정 주문 상세
upbit order cancel <uuid>                      # 주문 취소
upbit order replace <uuid> -p 110000000        # 주문 가격 수정
```

### 입출금 (인증 필요)

```bash
upbit wallet                                   # 지갑 정보
upbit deposit list                             # 입금 내역
upbit deposit show <uuid>                      # 특정 입금 상세
upbit deposit address BTC                      # BTC 입금 주소
upbit withdraw list                            # 출금 내역
upbit withdraw show <uuid>                     # 특정 출금 상세
upbit withdraw request BTC --amount 0.01 --address <address>  # 출금 신청
```

### 실시간 TUI

```bash
upbit watch ticker KRW-BTC                     # 실시간 가격 (TUI)
upbit watch ticker KRW-BTC KRW-ETH KRW-XRP    # 복수 코인 실시간 가격
upbit watch orderbook KRW-BTC                  # 실시간 호가창
upbit watch trade KRW-BTC                      # 실시간 체결 스트림
upbit watch candle KRW-BTC -i 1m               # 실시간 캔들스틱 차트
upbit watch my-order                           # 내 미체결 주문 모니터링 (인증 필요)
upbit watch my-asset                           # 내 자산 실시간 모니터링 (인증 필요)
```

### 유틸리티

```bash
upbit tool-schema                              # AI Agent용 JSON Schema
upbit tool-schema buy                          # 특정 명령어 Schema
upbit cache                                    # 캔들 캐시 정보
upbit cache --clear                            # 캐시 삭제
upbit update                                   # 최신 버전으로 업데이트
upbit update --check                           # 업데이트 확인만
```

## 시나리오별 사용법

### 1. 특정 코인의 현재가를 알고 싶을 때

```bash
# 비트코인 현재가
upbit ticker KRW-BTC

# 여러 코인 동시 조회
upbit ticker KRW-BTC KRW-ETH KRW-XRP

# JSON으로 파이프 처리
upbit ticker KRW-BTC | jq '.[] | {market, trade_price, change_rate}'
```

출력 예시:
```
MARKET    PRICE          CHANGE  CHANGE_RATE  VOLUME
KRW-BTC   142,000,000    ▲       +1.23%       1234.5678
```

### 2. 비트코인을 시장가로 매수할 때

```bash
# 현재가 기준으로 10만원어치 매수
upbit buy KRW-BTC -p now -t 100000

# KRW 잔고 전체로 매수
upbit buy KRW-BTC -p now -V 100%

# KRW 잔고의 30%로 매수
upbit buy KRW-BTC -p now -V 30%
```

> `-p now` — 현재 시장가 사용 (내부적으로 최유리 지정가 주문)
> `-V` — 코인 수량 (절대값 또는 잔고 퍼센트)
> `-t` — KRW 금액 기준 주문

### 3. 잔고를 확인할 때

```bash
# 전체 포트폴리오 (KRW 평가액 포함)
upbit balance

# JSON으로 파이프 처리
upbit balance | jq '.[] | {currency, balance, avg_buy_price}'
```

출력 예시:
```
CURRENCY  BALANCE     AVG_BUY_PRICE  CURRENT_PRICE  P&L
KRW       500,000     -              -              -
BTC       0.00100000  142,000,000    145,000,000    +2.11%
```

### 4. 호가창을 실시간으로 볼 때 (TUI)

```bash
# 비트코인 실시간 호가창
upbit watch orderbook KRW-BTC
```

- `q` 또는 `Ctrl+C`로 종료
- 매수(초록) / 매도(빨강) 호가가 스프레드 중심으로 표시됨

호가 스냅샷만 필요한 경우:
```bash
upbit orderbook KRW-BTC
upbit orderbook KRW-BTC | jq '.[] | .orderbook_units[:5]'
```

### 5. 캔들 데이터를 JSON으로 추출할 때

```bash
# 2025년 1월 1일 이후 일봉 전체
upbit candle KRW-BTC --from 2025-01-01 | jq .

# 1분봉 최근 200개
upbit candle KRW-BTC -i 1m | jq .

# CSV로 내보내기
upbit candle KRW-BTC -i 1d --from 2025-01-01 -o csv > btc_daily.csv

# 특정 필드만 추출
upbit candle KRW-BTC | jq '.[] | {timestamp: .candle_date_time_kst, open: .opening_price, close: .trade_price}'
```

캔들 인터벌 옵션 (`-i`):
- 분봉: `1m`, `3m`, `5m`, `10m`, `15m`, `30m`
- 시봉: `1h`, `4h`
- 일봉: `1d` (기본값)
- 주봉: `1w`
- 월봉: `1M`

캐시 관련:
```bash
upbit candle KRW-BTC --from 2025-01-01 --no-cache   # 캐시 사용 안 함
upbit cache                                           # 캐시 상태 확인
upbit cache --clear                                   # 캐시 전체 삭제
```

### 6. 퍼센트 기반 주문 (전체 잔고의 50% 매수)

```bash
# KRW 잔고의 50%로 이더리움 매수 (현재가)
upbit buy KRW-ETH -p now -V 50%

# 보유 BTC의 30%를 시장가 매도
upbit sell KRW-BTC -V 30%

# 보유 BTC 전량 매도
upbit sell KRW-BTC -V 100%
```

가격 키워드 (`-p` 옵션):
| 키워드 | 의미 |
|--------|------|
| `now` | 현재 시장가 (최유리 지정가) |
| `open` | 당일 시가 |
| `low` | 당일 저가 |
| `high` | 당일 고가 |
| 숫자 | 지정가 (KRW) |

### 7. 지정가 주문

```bash
# 1억 5천만원에 0.001 BTC 지정가 매수
upbit buy KRW-BTC -p 150000000 -V 0.001

# 예약 주문 (조건 충족 시 자동 체결 감시)
upbit buy KRW-BTC -p 130000000 -V 0.001 --watch
```

## 출력 형식

### auto 모드 동작

| 실행 환경 | 출력 형식 |
|-----------|-----------|
| 터미널 직접 실행 | `table` (사람이 읽기 좋은 형식) |
| 파이프로 연결 | `json` (compact) |

출력 형식 강제 지정:
```bash
upbit ticker KRW-BTC              # 테이블 (터미널)
upbit ticker KRW-BTC | jq .       # JSON (파이프, 자동)
upbit ticker KRW-BTC -o csv       # CSV
upbit ticker KRW-BTC -o jsonl     # JSON Lines
upbit ticker KRW-BTC -o json      # JSON (강제)
upbit ticker KRW-BTC -o table     # 테이블 (강제)
```

필드 선택:
```bash
upbit ticker KRW-BTC --json price  # 특정 필드만 JSON 출력
```

### 주요 JSON 필드 (ticker)

| 필드 | 설명 |
|------|------|
| `market` | 마켓 코드 (예: `KRW-BTC`) |
| `trade_price` | 현재가 |
| `opening_price` | 당일 시가 |
| `high_price` | 당일 고가 |
| `low_price` | 당일 저가 |
| `acc_trade_volume_24h` | 24시간 거래량 |
| `signed_change_rate` | 전일 대비 변동률 |
| `change` | 변동 방향 (`RISE`/`FALL`/`EVEN`) |

## 주의사항

- **호가 단위 자동 보정**: 주문 가격은 Upbit 호가 단위에 맞게 자동 반올림됩니다. `upbit tick-size KRW-BTC`로 단위 확인 가능.
- **퍼센트 주문 계산**: `-V 50%`는 현재 잔고 기준. 수수료(0.05%)를 고려한 실제 체결 금액은 약간 다를 수 있습니다.
- **실시간 TUI 종료**: `q`, `Esc`, 또는 `Ctrl+C`로 종료.
- **복수 마켓 TUI**: `watch ticker` 등에서 여러 마켓 지정 시 `Tab` 또는 `←/→` 키로 전환.
- **캔들 캐시**: SQLite 캐시 사용. 경로는 `$XDG_CACHE_HOME/upbit/` 또는 `~/.cache/upbit/`. `--no-cache` 플래그로 비활성화.
- **인증 없이 거래 명령어 실행 시**: 환경변수 미설정 시 에러 반환. `echo $UPBIT_ACCESS_KEY`로 설정 확인.
- **API Rate Limit**: Upbit REST API 분당 제한이 있음. 대량 조회 시 에러 발생 가능.
- **Non-TTY 환경**: 파이프/CI 환경에서는 자동으로 JSON 출력. AI 에이전트 파이프라인에 최적화됨.
