# game-poor 기술 조사 보고서

## 1. 실현 가능 여부 요약

| 항목 | 가능 여부 | 비고 |
|------|-----------|------|
| Steam 할인 정보 수집 | **가능** | Steam Store API + IsThereAnyDeal API |
| Steam 위시리스트 조회 | **조건부 가능** | 프로필 공개 설정 필요, 비공식 엔드포인트 |
| Epic 무료 게임 정보 | **가능** | 공개 엔드포인트, 인증 불필요 |
| Epic 할인 정보 | **가능** | IsThereAnyDeal API 경유 |
| Epic 위시리스트 | **불가** | 공식 API 없음 |
| Nintendo 할인 정보 | **제한적** | 공식 API 없음, IsThereAnyDeal 일부 커버 |
| Nintendo 위시리스트 | **불가** | 공식 API 없음 |
| 매일 21:00 자동 실행 | **가능** | Claude Code schedule 기능 (cron 기반) |
| 이메일 발송 | **확인 필요** | Claude schedule의 결과 전달 방식 확인 필요 |

---

## 2. 플랫폼별 API 상세

### 2-1. Steam

#### Steam Web API (공식)
- **문서**: https://developer.valvesoftware.com/wiki/Steam_Web_API
- **인증**: Steam Web API Key 필요 (무료 발급)
  - 발급: https://steamcommunity.com/dev/apikey
  - Steam 계정 로그인 필요
- **제공 데이터**: 사용자 프로필, 보유 게임, 게임 뉴스, 업적
- **위시리스트**: 공식 API 미제공
- **형식**: JSON/XML
- **Rate Limit**: 약 10만 요청/일 (비공식)

#### Steam Store API (비공식/반공식)
- **인증**: 불필요
- **주요 엔드포인트**:
  - `store.steampowered.com/api/appdetails?appids={id}` — 게임 상세(가격, 할인율)
  - `store.steampowered.com/api/featuredcategories/` — 세일 카테고리
  - `store.steampowered.com/api/featured/` — 특집 게임
- **할인 정보**: `price_overview` 필드에 할인율, 최종가격 포함
- **Rate Limit**: 약 200 요청/5분 (보수적 추정)

#### Steam 위시리스트 (비공식)
- **엔드포인트**: `store.steampowered.com/wishlist/profiles/{steamid}/wishlistdata/`
- **조건**: Steam 프로필 및 위시리스트가 **공개**로 설정되어야 함
- **인증**: 불필요 (공개 프로필 한정)
- **필요 정보**: Steam ID (64비트 숫자) 또는 커스텀 URL

### 2-2. Epic Games Store

#### 무료 게임 프로모션 (반공식)
- **엔드포인트**: `store-site-backend-static.ak.epicgames.com/freeGamesPromotions`
- **인증**: 불필요
- **제공 데이터**: 현재 무료 게임 + 다음 주 예정 무료 게임
- **갱신 주기**: 매주 목요일 (한국시간 금요일 0시)
- **형식**: JSON
- **안정성**: 비교적 안정적, 다수 서드파티에서 사용 중

#### 할인 정보
- 공식 API 없음
- IsThereAnyDeal API를 통해 Epic 가격 추적 가능

#### 위시리스트
- **프로그래밍 접근 불가** — 관심 게임 직접 지정 방식으로 대체

### 2-3. Nintendo eShop

#### 공식 API
- **없음** — 가장 폐쇄적인 플랫폼

#### 대안
- **IsThereAnyDeal**: Nintendo eShop 가격 일부 커버
- **DekuDeals** (dekudeals.com): Nintendo 특화 가격 추적, 단 API 미제공
- **비공식 엔드포인트**: 존재하나 수시 변경, 의존 비권장

#### 위시리스트
- **프로그래밍 접근 불가** — 관심 게임 직접 지정 방식으로 대체

---

## 3. 핵심 서드파티 API

### 3-1. IsThereAnyDeal (ITAD) — **핵심 추천**
- **문서**: https://docs.isthereanydeal.com/
- **인증**: API Key 필요 (앱 등록 후 무료 발급)
  - 등록: https://isthereanydeal.com/dev/app/
- **요금**: 무료 (비상업적/개인 사용)
- **커버 스토어**: Steam, Epic, GOG, Humble, Fanatical 등 수십 개
- **주요 기능**:
  - 멀티스토어 가격 비교
  - 역대 최저가 조회
  - 현재 진행 중인 딜 목록
  - Steam 위시리스트 동기화
- **형식**: JSON
- **평가**: 이 프로젝트의 핵심 데이터 소스로 가장 적합

### 3-2. CheapShark — **보조 추천**
- **문서**: https://apidocs.cheapshark.com/
- **인증**: 불필요 (API 키 없이 즉시 사용)
- **요금**: 완전 무료
- **커버 스토어**: Steam, GOG, Humble, GMG, Fanatical 등
- **주요 기능**:
  - 딜 목록 조회/필터
  - 게임 검색
  - 이메일 가격 알림 내장
- **형식**: JSON
- **평가**: API 키 없이 즉시 사용 가능, 프로토타이핑에 최적

### 3-3. RAWG — **메타데이터 보조**
- **문서**: https://rawg.io/apidocs
- **인증**: API Key 필요 (무료 발급)
- **요금**: 무료 (월 20,000 요청)
- **제공 데이터**: 게임 메타데이터(장르, 설명, 평점, 출시일)
- **가격 정보**: 없음
- **평가**: 신작 출시 캘린더 및 게임 정보 보강용

---

## 4. 필요한 개인정보 및 키

| 항목 | 용도 | 필수 여부 | 취득 방법 |
|------|------|-----------|-----------|
| **Steam Web API Key** | Steam 사용자 데이터 조회 | 선택 | steamcommunity.com/dev/apikey |
| **Steam ID** | 위시리스트 조회 | 위시리스트 사용 시 필수 | Steam 프로필에서 확인 |
| **Steam 프로필 공개 설정** | 위시리스트 외부 접근 | 위시리스트 사용 시 필수 | Steam 개인정보 설정 |
| **ITAD API Key** | 멀티스토어 가격 조회 | 권장 | isthereanydeal.com/dev/app/ |
| **RAWG API Key** | 게임 메타데이터 | 선택 | rawg.io에서 발급 |
| **이메일 주소** | 알림 수신 | 필수 | 사용자 제공 |

---

## 5. 기술 구성 방안

### 방안 A: Claude Code Schedule 단독 (권장 - 1단계)
```
Claude Schedule (매일 21:00 KST)
  └─ 실행 시 API 호출 스크립트 실행
       ├─ CheapShark API (키 불필요, 즉시 사용 가능)
       ├─ Epic 무료 게임 엔드포인트
       ├─ ITAD API (키 발급 후)
       └─ 결과 정리 → 이메일 발송 or Claude 알림
```

### 방안 B: 스크립트 + Claude Schedule (2단계)
```
game-poor/
  ├─ config.json          # 관심 게임, 플랫폼 설정
  ├─ scripts/
  │   ├─ fetch_deals.py   # API 호출 및 데이터 수집
  │   ├─ fetch_epic_free.py
  │   └─ send_email.py    # 이메일 발송
  ├─ request.md
  └─ research.md

Claude Schedule → python scripts/fetch_deals.py → 이메일 발송
```

### 이메일 발송 방법 후보
| 방법 | 장점 | 단점 |
|------|------|------|
| Gmail SMTP | 무료, 안정적 | 앱 비밀번호 설정 필요 |
| SendGrid | 무료 티어(100건/일) | 계정 가입 필요 |
| Mailgun | 무료 티어 | 계정 가입 필요 |
| Claude 자체 알림 | 설정 간편 | 이메일이 아닌 Claude 내 알림 |

---

## 6. 실현 가능성 종합 평가

### 즉시 가능 (API 키 불필요)
- CheapShark로 PC 게임 딜 조회
- Epic 무료 게임 정보 조회
- Steam Store API로 개별 게임 가격/할인 조회

### API 키 발급 후 가능
- IsThereAnyDeal로 멀티스토어 가격 비교 (가장 강력)
- Steam Web API로 사용자 데이터 조회
- RAWG로 게임 메타데이터/출시 정보

### 제한적/불가
- Steam 위시리스트: 프로필 공개 필수, 비공식 엔드포인트
- Epic 위시리스트: 불가 → 수동 목록으로 대체
- Nintendo 위시리스트: 불가 → 수동 목록으로 대체
- Nintendo 가격: ITAD 일부 커버, 직접 API 없음

---

## 7. 권장 진행 순서

1. **Phase 1**: CheapShark + Epic 무료게임 (키 불필요, 즉시 시작)
2. **Phase 2**: ITAD API 키 발급 → 멀티스토어 할인 추적
3. **Phase 3**: Steam 위시리스트 연동 (프로필 공개 설정)
4. **Phase 4**: 관심 게임 목록 관리 기능 (config.json)
5. **Phase 5**: Nintendo 가격 추적 (ITAD 경유)
6. **Phase 6**: 이메일 발송 자동화
7. **Phase 7**: Claude schedule 연동 (매일 21:00 자동 실행)

---

## 8. Claude Schedule 동작 방식 (확인 완료)

Claude Code의 `schedule` 기능은 **Anthropic 클라우드에서 실행되는 원격 에이전트**입니다.
로컬 머신에서 실행되는 cron이 **아닙니다**.

### 핵심 특성
- 실행 시 **격리된 원격 세션**(CCR)이 생성됨
- 지정된 **Git 레포를 clone**하여 작업 환경 구성
- 프롬프트를 기반으로 에이전트가 자율 실행
- 사용 가능 도구: Bash, Read, Write, Edit, Glob, Grep 등
- **MCP 커넥터** 연결 가능 (Notion, Google Calendar 등)
- cron 표현식은 **UTC 기준** (KST 21:00 = UTC 12:00)
- **최소 실행 간격**: 1시간

### 원격 실행의 제약
- **로컬 파일 접근 불가** — 로컬 스크립트 직접 실행 불가
- **로컬 환경변수 접근 불가** — API 키를 로컬에만 저장하면 사용 불가
- **로컬 Python 패키지 접근 불가** — pip install 된 패키지 사용 불가
- 에이전트는 **제로 컨텍스트**로 시작 → 프롬프트가 자체 완결적이어야 함

### 환경(Environment) 선택

| 환경 | 종류 | 장점 | 단점 |
|------|------|------|------|
| **Default (anthropic_cloud)** | 클라우드 | 항상 실행 가능, 안정적 | 로컬 파일/스크립트 접근 불가 |
| **isang-giui-MacBookPro (bridge)** | 브릿지 | 로컬 머신에서 실행, 로컬 자원 접근 가능 | Mac이 켜져 있어야 실행됨 |

- **권장**: cloud 환경 (매일 밤 9시에 Mac이 꺼져 있을 수 있으므로)
- bridge 환경은 로컬 스크립트가 반드시 필요한 경우에만 사용

### 모델 선택

| 모델 | 특징 | 적합도 |
|------|------|--------|
| `claude-sonnet-4-6` (기본값) | 빠르고 비용 효율적 | API 호출 + 정리 수준이면 충분 |
| `claude-opus-4-6` | 더 정확하고 깊은 분석 | 복잡한 판단이 필요한 경우 |

- **권장**: `claude-sonnet-4-6` (할인 정보 수집/정리 작업에 충분)

### 사용 가능한 MCP 커넥터

현재 연결된 커넥터:
- **Notion** ✅ — 결과를 Notion 페이지에 작성 가능
- **Google Calendar** ✅ — 일정 연동 가능
- **Postman** ✅ — API 테스트용

**이메일 관련 커넥터 없음** — 이메일 발송을 위해 별도 방법 필요

---

## 9. Git 레포 필요 (신규)

schedule은 Git 레포를 clone하여 실행하므로, **GitHub 레포 생성이 전제조건**입니다.

- 현재 `/Users/lmagine/game-poor/`는 git repo가 아님
- GitHub에 `game-poor` 레포 생성 필요
- 레포에 포함할 항목: `config.json` (관심 게임 목록), 프롬프트 참조 문서 등

---

## 10. API 키 보관 방법 (신규)

원격 에이전트가 API 키에 접근하는 방법:

| 방법 | 보안성 | 편의성 | 비고 |
|------|--------|--------|------|
| **프롬프트에 직접 포함** | ❌ 낮음 | ✅ 높음 | 간편하지만 로그에 노출 위험 |
| **레포 내 config 파일** | ⚠ 중간 | ✅ 높음 | private 레포 필수, .gitignore 불가(clone 시 필요) |
| **키 불필요 API 우선 사용** | ✅ 높음 | ✅ 높음 | CheapShark, Epic 무료게임은 키 없이 사용 가능 |

- **권장**: Phase 1에서는 키 불필요 API(CheapShark, Epic)만 사용
- ITAD 키가 필요한 Phase 2부터 private 레포 내 config 파일에 저장

---

## 11. 이메일 발송 방법 재검토 (신규)

원격 환경에서의 이메일 발송은 로컬과 다릅니다.

### 방법 비교 (원격 환경 기준)

| 방법 | 가능 여부 | 장점 | 단점 |
|------|-----------|------|------|
| **Notion 페이지 작성** | ✅ 즉시 가능 | MCP 연결됨, 설정 불필요 | 이메일이 아닌 Notion 알림 |
| **Bash + curl로 메일 API 호출** | ✅ 가능 | SendGrid/Mailgun 무료 티어 | API 키 필요, 계정 가입 |
| **Gmail SMTP (curl)** | ⚠ 가능하나 복잡 | 무료 | 앱 비밀번호를 레포에 저장해야 함 |
| **CheapShark 내장 알림** | ✅ 즉시 가능 | 설정만 하면 자동 발송 | CheapShark 커버 게임만 가능 |

### 권장 접근
1. **1단계**: Notion 페이지에 결과 작성 (즉시 가능, 추가 설정 없음)
2. **2단계**: SendGrid 무료 티어로 이메일 발송 (API 키 1개만 필요)
3. **보조**: CheapShark 내장 이메일 알림 활용 (별도 시스템 없이 가격 알림)

---

## 12. Rate Limit 대응 (신규)

| API | 제한 | 대응 |
|------|------|------|
| Steam Store API | ~200 요청/5분 | 관심 게임 수 제한, 요청 간 딜레이 |
| CheapShark | 명시 없음 (합리적 사용) | 한 번에 목록 조회로 최소화 |
| ITAD | 명시 없음 (비상업적) | 배치 엔드포인트 활용 |
| Epic 무료게임 | 제한 없음 | 1회 호출로 충분 |

- 관심 게임 50개 이하면 대부분 제한에 걸리지 않음
- Steam Store API는 개별 appid 조회이므로, 게임 수가 많으면 ITAD 배치 조회로 대체

---

## 13. 프롬프트 설계 가이드 (신규)

원격 에이전트 프롬프트는 **자체 완결적**이어야 합니다.

### 포함해야 할 내용
- 수집 대상 API 엔드포인트와 호출 방법
- 관심 게임 목록 참조 방법 (레포 내 config.json 읽기)
- 결과 정리 형식 (마크다운 등)
- 결과 전달 방법 (Notion 페이지 작성 / 이메일 API 호출)
- 에러 발생 시 행동 (로그 남기기 등)

### 프롬프트 예시 구조
```
1. config.json에서 관심 게임 목록 읽기
2. CheapShark API로 현재 할인 정보 조회
3. Epic 무료 게임 엔드포인트 조회
4. 결과를 마크다운으로 정리
5. Notion 페이지에 오늘 날짜로 작성
```

---

## 14. 수정된 권장 진행 순서

1. **Phase 1 (사전 준비)**
   - GitHub에 `game-poor` private 레포 생성
   - `config.json`에 관심 게임 목록 작성
   - Notion에 결과 저장용 데이터베이스 생성

2. **Phase 2 (MVP — 키 불필요)**
   - CheapShark API + Epic 무료게임 수집 프롬프트 작성
   - Notion에 결과 작성하는 schedule 생성 (매일 21:00 KST)
   - 동작 확인 및 프롬프트 튜닝

3. **Phase 3 (확장 — ITAD)**
   - ITAD API 키 발급
   - 멀티스토어 할인 추적 추가
   - Nintendo eShop 가격 추적 (ITAD 경유)

4. **Phase 4 (위시리스트)**
   - Steam 프로필 공개 설정
   - Steam 위시리스트 연동

5. **Phase 5 (이메일 전환)**
   - SendGrid 또는 Mailgun 계정 생성
   - 이메일 발송으로 전환 (Notion 병행 가능)

6. **Phase 6 (고도화)**
   - 가격 이력 추적 (레포 내 JSON 파일로 누적)
   - 최저가 알림
   - 장르/태그 기반 추천
