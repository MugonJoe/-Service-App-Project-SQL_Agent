# SQL Database Agent (LangChain + LangGraph)

본 프로젝트는 SQLite 기반의 Chinook 데이터베이스를 대상으로,  
자연어 질의를 자동으로 SQL 쿼리로 전환하고 실행하는 **데이터베이스 질의 자동화 에이전트**를 구현함.

LangChain과 LangGraph의 메시지 기반 상태 관리 구조를 활용하여  
다음과 같은 워크플로우를 자동화함.

- 테이블과 스키마 구조 파악
- 자연어 → SQL 쿼리 생성
- 쿼리 문법 검증 및 오류 수정
- 데이터베이스 실행 결과 분석
- 자연어 형태로 최종 응답 생성
- LangSmith 기반 응답 정확도 평가

---

## 1. 프로젝트 개요

데이터베이스 사용자들은 종종 정확한 스키마 구조를 모르거나,  
SQL 작성 경험이 부족해 원하는 정보를 바로 조회하지 못하는 경우가 많다.  
이 프로젝트는 이러한 문제를 해결하기 위해:

- **질문 기반 쿼리 자동 생성**
- **쿼리 오류 자동 복구**
- **결과 해석 및 자연어 응답 생성**

과정을 하나의 에이전트로 통합함.

---

## 2. 주요 기능

### ● 자연어 기반 SQL 생성
사용자의 질문을 받아 데이터베이스 구조를 분석한 뒤,  
질문에 해당하는 SQL 쿼리를 자동 작성함.

### ● SQL Query Checker
LLM을 활용한 정적 쿼리 검증 도구로,  
아래 항목에 대한 자동 불일치 탐지를 수행함:

- BETWEEN / NULL / NOT IN 오류
- UNION vs UNION ALL
- JOIN 컬럼 불일치
- 타입 캐스팅 문제
- 함수 인자 개수 오류

문제가 있을 경우 자동으로 쿼리를 재작성함.

### ● 쿼리 실행 및 결과 해석
SQLite 엔진에서 쿼리를 실행하고,  
결과를 자연어 형태로 정리해 최종 답변을 반환함.

### ● LangGraph 기반 상태 머신
쿼리 생성 → 검증 → 실행 → 재시도 등  
모든 단계를 명확히 분리하여 그래프 형태로 구성함.

### ● LangSmith 자동 평가
정답 데이터셋과 실제 Agent 결과를 비교해  
LLM 평가자를 통해 자동 채점함.

---

## 3. 시스템 흐름 (Workflow)

[User Question]

↓
[first_tool_call] — 테이블 목록 조회

↓
[list_tables_tool]

↓
[model_get_schema] — 관련 테이블 추론
↓
[get_schema_tool] — DDL 조회
↓
[query_gen] — SQL 생성/해석
├─ Error → query_gen (재생성)
├─ SQL → correct_query
└─ Answer → END
↓
[correct_query] — Query Checker
↓
[execute_query] — 실제 DB 실행
↓
(결과를 다시 query_gen으로 전달)

<img width="418" height="744" alt="image" src="https://github.com/user-attachments/assets/fa9400cf-d204-43bc-8cc6-424d2e4b006a" />

