# Improved RAG for Korean Copyright Case Law

이 저장소는 `Improved_RAG_Copyright_CaseLaw.ipynb`를 중심으로 구성된 포트폴리오 프로젝트입니다. 한국어 저작권 판례 데이터와 질의-답변 GroundTruth 데이터를 사용해, Naive RAG 대비 정확도/신뢰도 개선을 목표로 RAG 파이프라인을 구축하고 평가한 결과를 정리했습니다.

## 프로젝트 목표

- 법률 질문에 대해 검색-생성 기반 답변에서 근거성(retrieval quality)과 사실성(factuality)을 높이는 것
- 실험 비교 가능하도록 데이터, 코드, 결과를 재현 가능한 형태로 정리
- GitHub 공개용으로 실행 흐름과 산출물 의존성을 최소화

최종 산출물은 단일 노트북(`Improved_RAG_Copyright_CaseLaw.ipynb`)에서 확인 가능합니다.

## 핵심 흐름

1. 환경 및 라이브러리 구성
2. 데이터 로드(`data/raw/`)
3. 임베딩용 문서 전처리 및 청크 분할
4. 하이브리드 검색기 구성(Chroma + BM25)
5. 1차 후보 추출 + 재순위(reranker) 적용
6. 답변 생성 및 RAGAS 평가
7. 성능 비교 지표 저장

## 파일 구조

```text
.
├─ Improved_RAG_Copyright_CaseLaw.ipynb   # 최종 산출물(메인 노트북)
├─ data/
│  └─ raw/
│     ├─ Copyright_case_law_Supreme.xlsx
│     ├─ Copyright_QA_GroundTrue.xlsx
│     └─ korea_tour_spots_wiki.csv
├─ docs/
│  ├─ improved_rag_report.pdf
│  └─ improved_rag_summary_report.docx
├─ notebooks/
│  └─ deprecated/
│     ├─ naive_rag_practice.ipynb
│     ├─ naive_rag_chaehyunju.ipynb
│     └─ naive_rag_comparison_chaehyunju.ipynb
│
├─ results/
│  ├─ improved_rag_metric_summary.csv
│  └─ improved_rag_full_results.csv
├─ vector_db/
│  └─ chroma_db/
├─ .gitignore
└─ README.md
```

## 데이터 설명

### data/raw

- `Copyright_case_law_Supreme.xlsx`
  - 판례 본문/메타 정보를 포함한 저작권 판례 데이터셋
- `Copyright_QA_GroundTrue.xlsx`
  - 질의-정답 GroundTruth로 평가용 샘플 구성
- `korea_tour_spots_wiki.csv`
  - 샘플/디버깅에 사용한 보조 데이터(실행 환경 점검용)

### 참고: 공개용 주의사항

- 공개 저장소에 업로드 시 원 데이터 접근 권한이 필요한 파일이 있거나 개인정보가 있는지 별도 확인 필요
- 대용량 벡터 인덱스는 `.gitignore`로 제외되어 있어 재생성이 필요

## 설치 및 실행

### 1) Python 환경

- Python 3.10+ 권장
- 가급적 가상환경에서 실행

```bash
python -m venv .venv
. .venv/Scripts/Activate.ps1
pip install -U pip
pip install langchain langchain-community langchain-openai langchain-chroma rank_bm25 ragas pandas openpyxl pypdf numpy
```

### 2) 핵심 환경변수

노트북 상단에서 API 키 입력을 요구하면 직접 입력하는 방식으로 처리했습니다.

필요 라이브러리:

- `pandas`, `openpyxl`
- `langchain`, `langchain-community`, `langchain-openai`, `langchain-chroma`
- `rank_bm25`
- `ragas`
- `chromadb`

### 3) 경로 체크 (노트북 내부 변수)

노트북은 실행 환경에 맞게 아래 경로가 로컬 기준으로 동작하도록 정리되어 있습니다.

- `CASE_DATA_PATH`: `data/raw/Copyright_case_law_Supreme.xlsx`
- `QA_DATA_PATH`: `data/raw/Copyright_QA_GroundTrue.xlsx`
- `CHROMA_DIR`: `vector_db/chroma_db`
- 결과 CSV: `results/`

### 4) 실행 순서

노트북의 셀을 위에서 아래로 순차 실행합니다.

- 1단계: 데이터 로딩 및 전처리
- 2단계: 문서 분할 및 인덱싱
- 3단계: Chroma/BM25 검색기 생성
- 4단계: 하이브리드 검색 + reranker 적용
- 5단계: 질의응답 생성
- 6단계: RAGAS 평가

## 재현 가능성 및 산출물

- 평가 지표 요약: `results/improved_rag_metric_summary.csv`
- 전체 평가 로그: `results/improved_rag_full_results.csv`
- 각 실행은 같은 환경/라이브러리 버전에서 재현성을 높이기 위해 동일 파이프라인을 기준으로 구성되어 있습니다.

## 비교 실험 및 보관본

본 저장소는 최종 산출물 외에 실험 단계 파일을 구분해 보관합니다.

- 실습/초기 실험: `notebooks/deprecated/`

## 성능 개선 포인트

현재 실험 파이프라인은 Naive RAG의 단점을 보완하기 위해 다음을 적용했습니다.

- 문서 분할 전략 최적화로 문맥 단절 완화
- Chroma(Dense) + BM25의 하이브리드 검색으로 recall 기반 후보 수 증가
- 2단계 reranking으로 사실성/근거 일치도 향상
- RAGAS 지표 기반 정량 평가로 반복 튜닝
