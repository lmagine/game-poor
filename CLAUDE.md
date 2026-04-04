# game-poor 프로젝트

게임 할인 정보를 수집하여 Notion에 업데이트하는 자동화 프로젝트.

## Schedule 에이전트 실행 가이드

### 실행 목적
매일 Steam/Epic 게임 할인 정보를 수집하고, Notion 페이지를 최신 정보로 교체한다.

### 설정 파일
- `config.json`: Steam 프로필, API URL, 필터 조건, Notion 페이지 ID

### 수집 절차

#### 1. Steam 찜목록 할인
1. `config.json`의 `steam.wishlist_url`에 `?p=0`, `?p=1` 등 페이지를 붙여 전체 찜목록 조회
2. 응답이 빈 배열/객체면 마지막 페이지
3. 필터: `discount_percent > 0` AND `final_price < 15000` (가격 단위: 원, API 응답은 센트 단위이므로 주의 — 한국 원화는 그대로 원 단위)
4. 해당되는 게임 전부 표시

#### 2. Steam 세일 이벤트
- `https://store.steampowered.com/api/featured/` 호출
- 현재 대형 세일 이벤트 진행 여부 확인 (예: 여름 세일, 겨울 세일 등)

#### 3. Epic Games 무료 게임
- `config.json`의 `epic.free_games_url` 호출
- 현재 무료 게임 + 다음 주 예정 무료 게임 추출

#### 4. Epic Games 할인
- CheapShark API: `https://www.cheapshark.com/api/1.0/deals?storeID=25&upperPrice=15&sortBy=Metacritic&pageSize=20`
  - storeID=25: Epic Games Store
  - upperPrice=15: $15 이하 (한화 약 15,000원 미만에 근사)
  - sortBy=Metacritic: 인기도(메타크리틱 점수)순
  - pageSize=20: 상위 20개
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
| ... | ... | ... | ... |

(해당 없으면 "현재 찜목록에서 조건에 맞는 할인 게임이 없습니다." 표시)

## Epic Games 무료 게임

### 현재 무료
| 게임명 | 기간 |
| --- | --- |
| ... | ... |

### 다음 주 예정
| 게임명 | 기간 |
| --- | --- |
| ... | ... |

## Epic Games 할인 TOP 20

| 게임명 | 원가 | 할인가 | 할인율 | 메타크리틱 |
| --- | --- | --- | --- | --- |
| ... | ... | ... | ... | ... |

(해당 없으면 "조건에 맞는 할인 게임이 없습니다." 표시)
```

### 에러 처리
- API 호출 실패 시 해당 섹션에 "데이터 수집 실패" 표시하고 다음 섹션 진행
- 전체 실패해도 Notion 페이지에 실패 사유 기록
