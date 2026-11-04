# Career Radar — 데이터 소스 조사 및 수집 설계

- 갱신일: 2026-11-04
- 대상: 데이터 엔지니어 채용공고의 기술 요구 빈도·우대사항·준비 우선순위 분석
- 상태: 공개 명세와 대화 중 확인한 자료를 반영한 조사 기록. 인증키 발급·실제 API 호출·자동 수집·저장·분석 권한 검증은 아직 완료하지 않음

## 1. 소스 선택과 검증 상태

| 소스 | 제공 정보와 역할 | 접근 및 제약 | 판정 |
|---|---|---|---|
| 고용24 | 목록·상세 API. 직무내용, 우대사항, 자격면허·전공·외국어·컴퓨터 활용 등 본문 분석 후보 | 기업회원 및 심사. 공고 제공 동의·이용 기관 조건 확인 필요 | 본문 소스 후보. 실제 접근·이용·응답 검증 대기 |
| 사람인 | 공고 제목·기업·직무·경력·일정·키워드. 키워드 분석 보완 | 공개 API 명세에 상세 본문·별도 우대사항 필드 없음. 승인과 access-key 필요 | 보완 소스 후보. 본문 소스 대체 불가 |
| 점핏 | 공개 공고에 기술 태그·주요업무·자격요건·우대사항 | 외부 수집용 공식 공개 API를 확인하지 못함. 웹 자동 수집·저장·분석·결과 제공 조건 미확인 | 웹 수집 후보, 이용 가능 여부 보류 |
| 원티드 | 공개 공고에 주요업무·자격요건·우대사항·기술스택/툴 항목 | 전체 공고 수집용 공식 공개 API를 확인하지 못함. ATS 연동 API를 전체 공고 수집 권한으로 해석하지 않음 | 웹 수집 후보, 이용 가능 여부 보류 |
| 잡코리아 | 채용정보 API와 공개 공고 | API 조건에서 프로젝트·개인 사용·DB 수집 등의 목적은 승인하지 않는다고 명시 | 현재 프로젝트의 API 후보에서 제외. 웹 수집 허용 여부는 별도 미확인 |

최소 한 소스에서 충분한 본문과 사용 가능 범위를 확인하고 MVP를 시작합니다. 한 소스가 불가하면 다른 후보를 검증합니다. robots.txt의 미차단 경로, 브라우저에서 보이는 JSON 응답, 공개 페이지 존재만으로 수집·보관·분석·재제공 권한을 확정하지 않습니다.

## 2. 공식 근거

### 고용24

- [Open API 소개·신청 절차](https://www.work24.go.kr/cm/e/a/0110/selectOpenApiIntro.do?bbsClCd=OosccI71O3P2dBxVz5A40Q%3D%3D)
- [채용정보 목록 명세](https://www.work24.go.kr/cm/e/a/0110/selectOpenApiSvcInfo.do?fullApiSvcId=000000000000000000000000000000)
- [채용정보 상세 명세](https://m.work24.go.kr/cm/e/a/0110/selectOpenApiSvcInfo.do?fullApiSvcId=000000000000000000000000000000%5E000000000000000000000000000001%5E000000000000000000000000000003)
- [채용정보 API 이용 기관 FAQ](https://m.work24.go.kr/cm/c/g/0100/customerSearch.do?keyword=%EC%B1%84%EC%9A%A9%EA%B3%B5%EA%B3%A0)

목록 endpoint는 `callOpenApiSvcInfo210L01.do`, 상세는 `callOpenApiSvcInfo210D01.do`입니다. `authKey`로 인증하고 채용정보는 XML 응답을 지정합니다. 목록은 `display` 최대 100, `startPage` 최대 1000으로 명시되어 있습니다. 일일 호출량은 확인한 안내만으로 확정하지 않습니다.

목록에는 `wantedAuthNo`, `company`, `title`, `jobsCd`, `career`, `regDt`, `smodifyDtm`, `closeDt`, `wantedInfoUrl` 등이 있습니다. 상세에는 `jobCont` 직무내용, `pfCond`·`etcPfCond` 우대사항, `certificate`, `major`, `forLang`, `compAbl`, `keywordList` 등이 있습니다. 자격요건 전체가 하나의 독립 본문으로 제공된다고 가정하지 않습니다.

상세 표시 안내에는 원문 이동 버튼과 출처 명기가 요구됩니다. 원본 장기 보관, 기술 통계 생성, 공개 서비스 제공 범위는 실제 신청 조건과 정책을 확인해야 합니다. 웹 검색에서 확인되는 타기관 연계 공고가 API에서도 모두 반환된다고 가정하지 않습니다.

### 사람인

- [사용 안내](https://oapi.saramin.co.kr/guide/info)
- [채용공고 검색 명세](https://oapi.saramin.co.kr/guide/job-search)
- [공고 번호 조회 명세](https://oapi.saramin.co.kr/guide/job-search-id)
- [FAQ](https://oapi.saramin.co.kr/help)
- [이용 제한](https://oapi.saramin.co.kr/caution)
- [이용 신청](https://oapi.saramin.co.kr/join)

`GET https://oapi.saramin.co.kr/job-search`에 `access-key`를 전달합니다. JSON/XML을 지원하고 하루 최대 500회, `count` 최대 110입니다. `start`는 0부터 시작하는 페이지 번호입니다. 이는 하루 500개 공고 제한이 아니며, 500 × 110은 반복·중복을 포함한 이론적 응답 항목 수입니다.

`job_cd`, `job_mid_cd`, `keywords`로 범위를 좁힐 수 있으며 등록일 `published_min/max`, 수정일 `updated_min/max` 조건이 있습니다. 공개 명세에는 별도 경력 검색 파라미터가 없으므로 경력은 응답을 받은 뒤 분류하는 설계를 기본으로 합니다.

API 응답에는 제목, 기업, 직무·업종, 경력, 학력, 키워드, 게시·수정·접수·마감 시각과 원문 URL 등이 있습니다. **상세 직무내용 및 별도 자격요건·우대사항 본문은 공개 응답 명세에 없습니다.** 키워드로 필수·우대 여부를 확정하지 않습니다.

FAQ는 현재 진행 중인 공고만 조회 가능하다고 안내합니다. 등록일 검색이 있어도 이미 마감된 과거 N개월 전체를 확보할 수 있다는 뜻은 아닙니다. 응답의 `active` 필드 존재만으로 마감 공고 조회가 가능하다고 가정하지 않습니다.

이용 제한에는 API 이용 서비스의 재판매·이용요금 및 이용자에게 대가를 받는 행위 금지가 있습니다. 장기 보관·통계 생성·결과 공개 가능 범위는 추가 검증 대상입니다.

### 점핏·원티드 웹 수집 후보

- [점핏 공고 예시](https://jumpit.saramin.co.kr/position/51015919)
- [점핏 robots.txt](https://jumpit.saramin.co.kr/robots.txt)
- [점핏 공식 기업 안내](https://cdn.jumpit.co.kr/jumpit/jumpit_biz_introduce_202304.pdf)
- [원티드 공고 예시](https://www.wanted.co.kr/wd/293438)
- [원티드 고객센터](https://help.wanted.co.kr/hc/ko)
- [원티드 robots.txt](https://www.wanted.co.kr/robots.txt)
- [잡코리아 API 조건](https://www.jobkorea.co.kr/network/Popup_Xml.asp)

대화에서 제공된 점핏 robots.txt 내용은 `GPTBot` 전체 차단, 일반 크롤러의 이력서·회원 영역 차단을 포함하고 `/position/`는 차단 목록에 없었습니다. 이는 사용자 제공 스냅샷이며 운영 전 다시 확인합니다. 원티드 robots.txt와 양사의 자동 수집·저장·분석 정책은 아직 검증하지 않았습니다. 특정 봇으로 신분을 위장하거나 차단을 회피하지 않습니다.

점핏의 2023년 기업 안내에는 사람인 동시 게재를 신청할 수 있다고 설명되어 있습니다. 모든 점핏 공고가 사람인에 자동 게재되거나 사람인 API에서도 제공된다고 가정하지 않습니다. 현재 정책과 실제 중복률은 샘플로 확인해야 합니다.

## 3. 최소 수집 필드 매핑

| 공통 필드 | 고용24 | 사람인 | 미제공 시 처리 |
|---|---|---|---|
| external_id | wantedAuthNo | id | 안정적 식별 불가 시 격리 |
| title | title / wantedTitle | position.title | 누락 오류 |
| company_name | company / corpNm | company.name 또는 JSON company.detail.name | 결측 표시, 기업 중복 판정 제한 |
| role | jobsCd / jobsNm / relJobsNm | position.job-code / job-mid-code | 제목·본문 분류와 추론 근거 기록 |
| experience | career / enterTpNm | position.experience-level | 미상 유지 |
| published_at | regDt | posting-timestamp | 기간 통계에서 제외, 최초 수집일로 대체 금지 |
| source_updated_at | smodifyDtm | modification-timestamp | 본문 해시로 변경 감지 |
| deadline / status | closeDt / receiptCloseDt | expiration-timestamp / close-type / active | 상시·채용시·날짜·미상을 구분 |
| job_url | wantedInfoUrl 또는 공식 상세 링크 | url | 출처 표시 정책 확인 |
| description | jobCont | 미제공 | 본문 통계 대상에서 제외 |
| requirements_text | 독립 전체 필드 미확인. 자격·역량 조건은 제공 필드로 보존 | 미제공 | 필수/우대 추론 금지 |
| preferred_text | pfCond / etcPfCond | 미제공 | 미상을 우대 없음으로 해석하지 않음 |
| qualification_fields | certificate / major / forLang / compAbl | 독립 상세 필드 없음 | 원문 의미 유지 |
| keywords | keywordList | keyword | 키워드 전용 분석으로 구분 |

점핏·원티드는 실제 수집이 허용되고 샘플을 확보한 뒤 매핑합니다. 회사 소개·복리후생에만 등장하는 기술은 업무 요구로 자동 처리하지 않습니다.

`source`, `first_collected_at`, `last_collected_at`, `run_id`, `raw_location`, `content_hash`, 추출 버전·근거 문장은 수집·처리 시스템에서 생성합니다. 지역·급여·담당자 연락처 등은 초기 최소 수집 범위에서 제외합니다.

## 4. 수집·갱신 계획

1. 초기 검색어는 데이터 엔지니어·데이터엔지니어·Data Engineer를 후보로 삼고 실제 직무 코드를 확인합니다. ETL·DW·데이터 플랫폼은 본문을 검토해 포함합니다.
2. 교육생 모집, 무관 직무, 수정·재게시, 타 사이트 동시 게재를 구분합니다. 최초 발견과 신규 등록을 같은 값으로 세지 않습니다.
3. 하루 1회 배치를 초기 설계안으로 삼습니다. API 제공 날짜 필터, 수집량, 이용 조건을 확인해 확정합니다.
4. 고정된 조회 구간·페이지 단위로 원본을 기록하고 마지막 성공 구간부터 증분 처리합니다. 중첩 구간 재조회는 ID upsert로 처리합니다.
5. 사람인 호출 예산은 예를 들어 정상 작업 400회와 나머지 복구 여유로 배분할 수 있습니다. 실제 요청·재시도를 함께 집계하며 API별 한도를 적용합니다.
6. 본문 수정 시 새 버전을 추출하고 집계에는 최신 성공 버전 한 개를 반영합니다. 마감·삭제·조회 불가 상태를 구분합니다.
7. 원본 재처리와 집계 재생성은 외부 요청 없이 가능하게 구성하되 출처별 보존·삭제 정책을 따릅니다.

## 5. 데이터량·분석 가능성 검증

네 사이트의 데이터 엔지니어 하루 신규 공고 합계와 중복 제거 후 유입량은 **미측정**입니다. 검색 총건수·모집 인원·전체 IT 공고량을 일일 신규 데이터 엔지니어 공고량으로 환산하지 않습니다.

2~4주간 출처별 신규 ID 수, 실제 신규 등록 수, 수정 수, 중복 제거 후 포지션 수, 고유 기업 수, 본문 확보율, 직무 적합률, 기술 추출 성공률을 기록합니다. API 한도 내 수집 가능 여부는 `페이지 수 + 수정 조회 + 재확인 + 재시도`로 산정합니다.

초기 수집 공고와 이후 신규 공고를 구분하고 등록일 기준 실제 확보 기간을 표시합니다. 본문 분석 공고와 키워드 전용 공고는 같은 분모로 합치지 않습니다. 수집량이 적으면 경력 필터를 완화하거나 기간을 넓히되 사용자에게 표본 수를 알립니다.

## 6. 구현 착수 전 확인 항목

- [ ] 최소 한 소스의 신청·승인 또는 웹 자동 수집 이용 가능 범위 확인
- [ ] 저장·보존·통계 생성·추천 결과 공개·출처 표시·삭제 조건 기록
- [ ] 실제 공고 수십 건의 본문·등록일·직무 필드 확보 및 매핑
- [ ] 출처별 페이지·호출 한도·날짜 필터·오류 응답 확인
- [ ] 공고 ID·중복 그룹·변경 버전 정책 정의
- [ ] 본문 소스가 확보되지 않을 경우 핵심 분석 착수 보류 및 대체 소스 검증

## 7. 검증 기록 템플릿

```markdown
### YYYY-MM-DD / 출처
- 검증 종류: 문서 확인 / 사용자 제공 자료 / 실제 요청 / 제공처 답변
- 공식 문서·약관·robots.txt URL 및 확인 시각:
- 신청·승인 상태와 이용 목적:
- 저장·보존·분석·공개 범위 및 출처 표시 조건:
- 요청 조건과 HTTP 상태 (키 제외):
- 페이지네이션·날짜 필터·호출 예산:
- 실제 응답 필드와 본문 확보율:
- 샘플 보관 위치 (공개 가능 여부 포함):
- 신규·수정·마감·중복 처리 확인:
- 판정 및 남은 질문:
```

## 8. MVP 기술 추출 방법

첫 버전은 문장 전체를 AI로 해석하지 않습니다. 자격요건/우대사항 문장을 분리하고, 사람이 관리하는 기술명·별칭 사전으로 문자열을 찾아 공고별 매칭 결과를 만듭니다.

예시 사전:

| 표준명 | 별칭 |
|---|---|
| PostgreSQL | Postgres |
| Google Cloud | GCP, Google Cloud Platform |
| Amazon Web Services | AWS |

각 결과는 `(공고 ID, 영역, 표준 기술명, 근거 문장, 사전 버전)`으로 저장합니다. 공고 안에서 같은 기술이 반복되어도 빈도는 1회입니다. 경력연수·도메인·협업 능력은 기술 사전에 넣지 않습니다.

조합은 같은 공고의 같은 항목에 명확하게 함께 요구된 기술만 기록합니다. `Python과 SQL` 같은 명시적 결합은 조합 후보입니다. `Airflow, Dagster 등`, `GCP/AWS`, `Oracle, MySQL, PostgreSQL 등`은 대안 목록일 수 있으므로 기술 각각의 빈도만 세고 조합을 만들지 않습니다. 불명확한 경우 조합에서 제외합니다.

첫 기술 사전과 샘플 추출 결과는 저장소의 비밀값 없는 fixture로 관리할 수 있지만, 출처 정책이 허용하는 텍스트만 사용합니다. 추출 품질은 공고 표본을 직접 검토해 누락·오탐을 기록하고 사전을 갱신하는 방식으로 개선합니다.
