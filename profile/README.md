<p align="center">
  <img src="https://github.com/team-926/WikiPulse/raw/main/.impeccable/performance/optimized-1440-default.png" alt="WikiPulse 펄스맵 화면" width="720" />
</p>

<h1 align="center">WikiPulse</h1>

<p align="center"><b>위키피디아 편집 급증에서 실시간 이슈를 포착하고, 관련 미국 상장 종목을 근거와 함께 연결하는 이슈 피드 서비스</b></p>

세상에 무슨 일이 생기면 사람들은 위키피디아 문서를 고치고 찾아봅니다.
WikiPulse는 사람이 직접 편집한 문서 가운데 조회수까지 함께 급등한 문서를 찾아 이슈 단위로 묶고,
GDELT 뉴스와 벡터 유사도로 고른 후보 종목을 LLM이 근거 경로로 검증해 이슈 피드로 보여 줍니다.

## 주요 기능

- **이슈 감지·클러스터링**: 봇이 아닌 편집과 조회수 급등을 모두 통과한 문서를 Clickstream·Wikidata 관계로 묶음
- **펄스맵**: 클러스터를 버블로, 급등도를 크기로 표시하고 시간 슬라이더로 과거 시점을 재생
- **이슈 피드와 상세**: 카드/리스트 피드, 대표 문서·타임라인·요약 리포트
- **이슈별 관련 종목**: 임베딩 Top-K 후보 → GDELT 근거 → LLM 검증을 통과한 종목만 노출
- **종목 상세**: 주가 그래프 위에 이슈 발생 시점을 표시하고 관련 이슈를 모아 보기

## 저장소 구성

모든 코드는 [`WikiPulse`](https://github.com/team-926/WikiPulse) 저장소 하나에 있습니다.

| 경로 | 구성 요소 | 기술 |
| --- | --- | --- |
| [`data-pipeline/`](https://github.com/team-926/WikiPulse/tree/main/data-pipeline) | 편집 수집, 스트리밍 1·2차 관문, 클러스터링, GDELT·종목 적재 | Python, Kafka, Spark Structured Streaming, HDFS |
| [`backend/`](https://github.com/team-926/WikiPulse/tree/main/backend) | 이슈·종목 REST API, 후보 생성·LLM 검증 워커, 계정 | Java 17, Spring Boot, JPA, Spring Security |
| [`frontend/`](https://github.com/team-926/WikiPulse/tree/main/frontend) | 펄스맵, 이슈 피드·상세, 종목 상세 | React, Vite, Playwright |
| [`db/`](https://github.com/team-926/WikiPulse/tree/main/db) | 스키마 마이그레이션과 테스트 | PostgreSQL, pgvector |
| [`ai/`](https://github.com/team-926/WikiPulse/tree/main/ai) | 매칭·이슈 텍스트·LLM 검증 실험 | OpenAI 임베딩, Claude |
| [`infra/`](https://github.com/team-926/WikiPulse/tree/main/infra) | 서버별 Compose, Spark·HDFS·Nginx 설정 | Docker Compose, Nginx |
| [`docs/`](https://github.com/team-926/WikiPulse/tree/main/docs) | 요구사항·기술 명세, API·ERD, 검증 기록 | |

## 시스템 구성

```
Wikipedia EventStreams ─▶ Kafka ─▶ Spark Streaming (편집 관문 → 조회수 관문 → 클러스터링) ─┐
pageviews · Clickstream · GDELT ─▶ HDFS (2노드) ─▶ Spark 배치 (기준선 · 근거 추출 · 리플레이) ─┼─▶ PostgreSQL + pgvector ─▶ Spring Boot ─▶ React
종목 마스터 · 사업 설명 ─▶ 임베딩 ─▶ 후보 Top-K ─▶ LLM 근거 검증 ──────────────────────────┘
```

서버 두 대가 각각 HDFS DataNode와 Spark Worker를 맡습니다. 데이터 서버에는 Kafka·Spark Master·NameNode를, 서비스 서버에는 Nginx·Spring Boot·PostgreSQL을 둡니다.

> 종목 연결은 자동 매핑 결과이며 투자 권유가 아닙니다.
