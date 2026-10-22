# Naver Shopping Promotion Audit

공식몰의 네이버 쇼핑검색 광고 소재 내부에 등록된 `PROMOTION` 확장소재를 직접 수정하기 위해, 수정 대상과 현재 등록 정보를 Google Sheets로 불러오는 Google Apps Script 프로젝트입니다.

## 프로젝트 배경

공식몰 쇼핑검색 캠페인이 많아지면 광고관리 화면에서 **소재 → 상세보기 → 확장소재**를 하나씩 열어 현재 홍보문구를 확인하고 수정 대상을 찾아야 합니다. 이 프로젝트는 네이버 검색광고 API를 이용해 운영 가능한 소재만 선별하고, 소재 내부의 `PROMOTION` 정보와 확장소재 미등록 소재를 한 번에 확인할 수 있는 작업 시트를 생성합니다.

생성된 시트는 확장소재를 API로 자동 변경하기 위한 것이 아니라, 광고관리 화면에서 사람이 직접 수정하기 전에 대상 소재와 현재 값을 빠르게 파악하기 위한 운영 도구입니다.

## 주요 기능

- 활성 `SHOPPING` 캠페인만 조회
- 활성 광고그룹과 활성 광고 소재만 처리
- 운영 규칙에 따라 이름이 `h-s`로 끝나는 광고그룹 제외
- 광고 소재 ID를 `ownerId`로 사용해 소재 단위 확장소재 조회
- `SHOPPING_EXTRA`는 점검 대상에서 제외
- `PROMOTION`이 등록된 소재와 대상 확장소재가 없는 소재를 구분
- 소재 ID와 상품명을 함께 제공해 광고관리 화면에서 수정 대상을 빠르게 검색
- 목록 응답에서 상품명이 없으면 소재 상세 API로 보완 조회
- HTTP 429 응답 시 지수 백오프로 자동 재시도
- 동시 실행 방지를 위한 Apps Script Lock 적용
- 재실행 시 기존 시트를 초기화해 중복 방지

## 처리 흐름

```text
Campaigns
   ↓ 활성 SHOPPING 캠페인
Adgroups
   ↓ 활성 상태 + h-s 제외
Ads
   ↓ 활성 소재
Ad Extensions(ownerId = nccAdId)
   ↓ SHOPPING_EXTRA 제외
PROMOTION / 확장소재 없음 분류
   ↓
Google Sheets
```

## 출력 결과

실행 시 `네이버_PROMOTION확인` 시트가 생성됩니다.

| 캠페인 | 광고그룹 | 소재ID | 상품명 | 확장소재ID | 확장소재유형 | 확장소재내용 | 확장소재상태 | 확장소재검토상태 |
|---|---|---|---|---|---|---|---|---|

`확장소재유형`에는 다음 두 값만 출력됩니다.

- `PROMOTION`
- `확장소재 없음` — `SHOPPING_EXTRA`를 제외하면 대상 확장소재가 없는 소재

## 실제 활용 방법

1. 스크립트를 실행해 공식몰의 활성 쇼핑검색 소재를 조회합니다.
2. 시트에서 `PROMOTION`의 현재 문구와 검토 상태를 확인합니다.
3. `확장소재 없음` 소재를 신규 등록 대상으로 분류합니다.
4. 소재 ID와 상품명을 이용해 네이버 광고관리 화면에서 해당 소재를 찾습니다.
5. **소재 → 상세보기 → 확장소재**에서 홍보문구를 직접 수정하거나 등록합니다.

## 설치 및 설정

1. Google Spreadsheet에서 **확장 프로그램 → Apps Script**를 엽니다.
2. [`Code.gs`](./Code.gs)의 내용을 Apps Script에 붙여넣습니다.
3. **프로젝트 설정 → 스크립트 속성**에 다음 값을 등록합니다.

| 속성 | 설명 |
|---|---|
| `NAVER_API_KEY` | 네이버 검색광고 API License |
| `NAVER_SECRET_KEY` | Secret Key |
| `NAVER_CUSTOMER_ID` | 광고계정 Customer ID |

4. 함수 목록에서 `buildNaverPromotionCheck`를 실행합니다.
5. 최초 실행 시 Google Sheets 및 외부 요청 권한을 승인합니다.

## 기술적 포인트

- HMAC-SHA256 기반 네이버 검색광고 API 인증
- `UrlFetchApp.fetchAll` 기반 제한적 병렬 처리
- 429 Too Many Requests 대응 지수 백오프
- 다양한 확장소재 JSON 구조의 재귀 파싱
- 목록 API와 상세 API를 결합한 결측 상품명 보완
- Script Lock을 이용한 동시 실행 제어

## 파일 구성

```text
.
├── Code.gs          # API 조회, 필터링, 시트 적재
├── appsscript.json  # Apps Script 런타임 설정
├── LICENSE
└── README.md
```

## 보안

API 키와 고객 ID는 코드에 직접 입력하지 않습니다. 모든 인증정보는 Apps Script의 스크립트 속성에서만 읽으며, 실제 인증정보가 포함된 파일은 Git에 올리지 않습니다.

## License

MIT
