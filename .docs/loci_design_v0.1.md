# Loci 설계안 v0.1

## 0) 한 줄 정의

**Loci**는 로컬/깃/GitHub(이슈·PR·위키)에서 Markdown 기반 지식을
수집·정제·청킹하고,\
**SQLite FTS5(BM25) + FAISS(임베딩 유사도)** 하이브리드 검색을
제공하며,\
Agent가 질문에 맞게 **동적으로 검색→선별→컨텍스트 패킹**을 수행하는 로컬
지식베이스다.

------------------------------------------------------------------------

## 1) 목표와 비목표

### 목표(Goals)

-   다양한 출처의 Markdown을 통합 지식베이스로 관리(수집/정제/색인/검색)
-   키워드 검색(정확한 용어) + 의미 검색(표현 변형) + 하이브리드 랭킹
-   증분 동기화(변경된 문서만 재처리)
-   임베딩은 로컬 기본(sentence-transformers) + 추후 ollama로 쉽게 전환
-   Agent가 질문을 분해해 여러 번 검색하고 최적 컨텍스트를 구성

### 비목표(Non-goals)

-   웹 UI
-   멀티유저 서버/권한/SSO
-   답변 생성 LLM 강제 포함

------------------------------------------------------------------------

## 2) 사용자 플로우

1.  `loci init`
2.  `loci source add local --path ./docs --glob "**/*.md"`
3.  `loci source add github --repo owner/repo --issues --prs --wiki`
4.  `loci sync`
5.  `loci search "쿼리" --mode hybrid`
6.  `loci ask "질문"`

------------------------------------------------------------------------

## 3) 저장 구조

    ~/.loci/
      config.toml
      loci.db
      cache/
      indexes/
      logs/

------------------------------------------------------------------------

## 4) DB 스키마 개요

### sources

-   id
-   type
-   name
-   config_json
-   enabled
-   created_at
-   updated_at

### documents

-   id
-   source_id
-   source_uid
-   title
-   url
-   updated_at
-   fetched_at
-   content_hash
-   raw_md
-   plain_text
-   meta_json
-   status
-   error_msg

### chunks

-   id
-   doc_id
-   chunk_index
-   heading_path
-   text
-   embed_text
-   token_count
-   chunk_hash
-   created_at

------------------------------------------------------------------------

## 5) 임베딩 설계

### 기본

-   sentence-transformers (멀티링구얼 모델)
-   normalize_embeddings = True
-   FAISS IndexFlatIP

### 확장

-   EmbeddingProvider 추상화
-   ollama 기반 embedding provider 추가 가능

------------------------------------------------------------------------

## 6) 검색 설계

-   Keyword: SQLite FTS5 (BM25)
-   Semantic: FAISS cosine/IP
-   Hybrid: RRF(Reciprocal Rank Fusion)
-   Optional: recency boost, source boost

------------------------------------------------------------------------

## 7) Agent ask 루프

1.  Query 분석
2.  Query 확장
3.  Hybrid 검색 반복
4.  결과 병합 및 다양성 확보
5.  Context pack 구성
6.  (옵션) LLM 기반 답변 생성

------------------------------------------------------------------------

## 8) CLI 명세

-   `loci init`
-   `loci source add`
-   `loci sync`
-   `loci search`
-   `loci ask`
-   `loci stats`
-   `loci doctor`

------------------------------------------------------------------------

## 9) 구현 마일스톤

1.  DB/CLI 골격
2.  Local MD ingest
3.  FTS 검색
4.  임베딩 + FAISS
5.  Hybrid 랭킹
6.  GitHub 연동
7.  Agent ask 구현

------------------------------------------------------------------------

## 10) 확장 포인트

-   추가 Connector (Notion/Confluence 등)
-   UI 확장 (TUI/Web)
-   학습 기반 랭커
-   고급 증분 전략
