# game-poor 프로젝트

게임 할인 정보를 수집하여 Notion에 업데이트하는 자동화 프로젝트.

## Schedule 에이전트 실행 가이드

### 실행 목적
매일 Steam/Epic 게임 할인 정보를 수집하고, Notion 페이지를 최신 정보로 교체한다.

### 설정 파일
- `config.json`: Steam 프로필, API URL, 필터 조건, Notion 페이지 ID

### 수집 절차

#### 1. Steam 찜목록 할인
1. `config.json`의 `steam.wishlist_api`로 전체 찜목록 appid 조회
2. 각 appid에 대해 `steam.app_details_api`에 `&appids={단일appid}&filters=price_overview,basic`로 가격 조회
   - **주의: 이 API는 단일 appid만 지원** (복수 appid 전송 시 `null` 반환)
   - 10개 조회마다 1.5초 대기 (rate limit 방지)
3. 필터: `discount_percent > 0` AND `final < 1500000` (가격 단위: 원, 100으로 나누지 않음)
4. 해당되는 게임 전부 표시
5. 각 게임명에 `https://store.steampowered.com/app/{appid}` 링크 포함

#### 2. Steam 세일 이벤트
- `https://store.steampowered.com/api/featured/` 호출
- 현재 대형 세일 이벤트 진행 여부 확인 (예: 여름 세일, 겨울 세일 등)

#### 3. Epic Games 무료 게임
- `config.json`의 `epic.free_games_url` 호출
- 현재 무료 게임 + 다음 주 예정 무료 게임 추출

#### 4. Epic Games 할인
- CheapShark API: `https://www.cheapshark.com/api/1.0/deals?storeID=25&upperPrice=15&sortBy=Savings&pageSize=30`
  - storeID=25: Epic Games Store
  - upperPrice=15: $15 이하 (한화 약 15,000원 미만에 근사)
  - sortBy=Savings: 할인율 내림차순
  - pageSize=30: 상위 30개
- 할인 중인 게임만 (`savings` > 0)

### 결과 작성

Notion 페이지(`config.json`의 `notion.deals_page_id`)를 `update-page`로 교체한다.

#### 페이지 형식
```markdown
> 마지막 업데이트: YYYY-MM-DD HH:MM (KST)

## Steam 찜목록 할인

(세일 이벤트 진행 중이면 여기에 표시: 예 "현재 Steam 여름 세일 진행 중!")

| 게임명 | 원가 | 할인가 | 할인율 |
| --- | --- | --- | --- |
| [게임명](스토어링크) | ₩XX,XXX | ₩XX,XXX | XX% |

(해당 없으면 "현재 찜목록에서 조건에 맞는 할인 게임이 없습니다." 표시)

## Epic Games 무료 게임

### 현재 무료
| 게임명 | 기간 |
| --- | --- |
| [게임명](스토어링크) | M/D ~ M/D |

### 다음 주 예정
| 게임명 | 기간 |
| --- | --- |
| [게임명](스토어링크) | M/D ~ M/D |

## Epic Games 할인 TOP 30 (할인율순)

| 게임명 | 원가 | 할인가 | 할인율 | 메타크리틱 |
| --- | --- | --- | --- | --- |
| [게임명](스토어링크) | $XX.XX | $XX.XX | XX% | XX |

(해당 없으면 "조건에 맞는 할인 게임이 없습니다." 표시)
```

### API 호출 방법
- **curl을 사용할 것** (Python urllib은 macOS에서 SSL 인증서 오류 발생)
- 반드시 `--compressed` 플래그 포함 (Steam API가 gzip 응답을 반환하므로 누락 시 디코딩 실패)
- 예시: `curl -s --compressed --max-time 10 "URL"`
- JSON 파싱은 curl 결과를 python3에 파이프하여 처리

### 에러 처리
- API 호출 실패 시 해당 섹션에 "데이터 수집 실패" 표시하고 다음 섹션 진행
- 전체 실패해도 Notion 페이지에 실패 사유 기록
