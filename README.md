# Dayflow-AI-Lifestyle-Concierge-Chatbot-with-RAG-
Dayflow — AI Lifestyle Concierge Chatbot (with RAG)


# Dayflow — AI Lifestyle Concierge Chatbot (with RAG)

**"Go with your own flow"**

관계 유형, 취향, 분위기, 시간, 예산, 목적을 입력하면 감정 흐름과 이동 동선까지 고려한 하루 코스를 설계해주는 AI 라이프스타일 컨시어지 챗봇입니다. 폼 입력 → Chain-of-Thought 추론 → JSON → 카드형 HTML UI로 이어지는 구조에, 사용자가 업로드한 문서를 검색해 답변에 반영하는 RAG(검색 증강 생성)를 추가했습니다.

## Overview

- 지역·관계·예산·시간·목적·분위기·취향 등 모임 정보를 폼으로 입력받아 하루 일정을 추천
- 단순 장소 나열이 아니라 **감정 흐름 · 이동 연결성 · 현실적 비용 배분**을 고려해 하나의 자연스러운 코스로 설계
- 추천 결과는 실제 존재하는 장소명, 평점, 대표 메뉴/특징과 함께 Google Maps 링크를 포함한 카드형 HTML로 렌더링
- Gradio 기반 웹 UI로 실행, 공유 링크(`share=True`) 생성

## RAG 통합 설계

문서를 업로드하지 않아도, 업로드해도 동일한 흐름으로 자연스럽게 동작하도록 설계했습니다.

1. **문서 미업로드** → 기존 동작과 100% 동일 (일반 지식 기반 추천)
2. **문서 업로드** → 청킹(`RecursiveCharacterTextSplitter`) → 임베딩(`OpenAIEmbeddings`) → **Chroma** 벡터스토어에 저장
3. 사용자의 취향(`taste`) + 목적(`purpose`) 텍스트로 벡터스토어를 검색해 관련 chunk를 추출
4. 검색된 내용을 `[참고 자료]` 섹션으로 프롬프트에 삽입, 시스템 프롬프트에 "참고 자료를 일반 지식보다 우선 반영"하도록 명시
5. 참고자료가 반영된 장소에는 HTML 카드에 **"참고자료 기반"** 배지 표시

문서 종류는 가리지 않아 맛집 리스트, 회사 복지 문서, 동아리 자료 등 어떤 PDF/TXT/CSV를 올려도 동일하게 동작합니다.

## Prompt Design (Chain-of-Thought)

단순히 장소를 나열하지 않도록, 6단계 추론 과정을 프롬프트에 명시했습니다.

1. 관계 유형과 분위기를 분석해 전체 감정 흐름 방향을 정의
2. 초반 장소 선정 — 긴장 완화/자연스러운 시작에 적합한 장소와 이유
3. 중반 장소 선정 — 편안한 대화와 분위기 유지에 적합한 장소와 이유
4. 후반 장소 선정 — 분위기 상승/마무리에 적합한 장소와 이유
5. 각 장소 간 이동 시간과 연결성 검토
6. 코스별 예상 비용 배분

RAG로 검색된 참고자료가 있을 경우 이 추론 과정 이전에 `[참고 자료]` 섹션이 삽입되어, 모델이 일반 지식보다 참고자료를 우선 활용하도록 유도합니다.

## Tech Stack

- **LLM**: GPT-5.2 (Chat Completions API)
- **RAG**: LangChain (`RecursiveCharacterTextSplitter`, `PyPDFLoader`/`CSVLoader`/`TextLoader`), Chroma 벡터스토어, OpenAI Embeddings
- **UI**: Gradio (`gr.Blocks`, 파일 업로드 컴포넌트 포함)
- **Output**: JSON 스키마 기반 구조화 출력 → 카드형 HTML 렌더링 (Google Maps 링크 자동 생성)

## How It Works

```
사용자 입력 (지역/관계/예산/시간/목적/분위기/취향 ...)
        │
        ├─ (선택) 문서 업로드 → 청킹 → 임베딩 → Chroma 저장
        │
        ▼
  RAG 검색 (taste + purpose 쿼리)
        │
        ▼
  프롬프트 빌드 (Chain-of-Thought 6단계 + [참고자료] 삽입)
        │
        ▼
   GPT-5.2 호출 → JSON 응답
        │
        ▼
   HTML 카드 렌더링 (Gradio UI 출력)
```

## Run

```bash
pip install openai gradio langchain langchain-openai langchain-community chromadb pypdf
```

노트북 셀을 순서대로 실행하면 API key 입력 프롬프트가 뜨고, 마지막 셀 실행 시 Gradio 공유 링크가 생성됩니다.

## Author

이지수 — 국제경영학과 · 정보통계학과 복수전공
