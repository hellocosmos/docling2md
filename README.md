# Docling Document Converter API

Docling HybridChunker와 Contextualize를 활용한 다양한 파일 형식을 Markdown으로 변환하는 FastAPI 기반 REST API입니다.

## 📋 개요

이 API는 Docling 라이브러리를 사용하여 다양한 문서 형식을 Markdown으로 변환하고, HybridChunker를 통해 지능적인 문서 청킹(chunking) 및 컨텍스트 강화 기능을 제공합니다.

## ✨ 주요 기능

- **다양한 파일 형식 지원**: PDF, Word, Excel, PowerPoint, HTML, 이미지 등
- **HybridChunker**: 문서를 의미있는 청크로 지능적으로 분할
- **Contextualize**: 각 청크의 컨텍스트를 메타데이터로 강화
- **비동기 작업**: 대용량 파일 처리 시 타임아웃 방지
- **다국어 지원**: 한글 처리에 최적화된 토크나이저 사용
- **일괄 처리**: 여러 파일을 한 번에 변환
- **유연한 출력**: Markdown, JSON, ZIP 형식 지원
- **n8n 통합**: 워크플로우 자동화 도구와 완벽한 호환

## 🚀 시작하기

### 방법 1: 로컬 Python 환경

#### 필수 요구사항

- Python 3.11+
- pip 패키지 관리자

#### 설치 및 실행

```bash
# 의존성 설치
pip install -r requirements.txt

# 서버 실행
python docling-rag-server.py
```

### 접속 정보

- **서버 주소**: `http://localhost:10002`
- **API 문서**: `http://localhost:10002/docs`
- **OpenAPI 스펙**: `http://localhost:10002/openapi.json`

## 📚 지원되는 파일 형식

### 문서

- `.pdf` - PDF 문서
- `.docx` - Word 문서
- `.xlsx` - Excel 스프레드시트
- `.pptx` - PowerPoint 프레젠테이션

### 웹 및 데이터

- `.html`, `.htm` - HTML 파일
- `.md` - Markdown 파일
- `.csv` - CSV 파일
- `.json` - JSON 파일
- `.xml` - XML 파일

### 이미지

- `.jpg`, `.jpeg` - JPEG 이미지
- `.png` - PNG 이미지
- `.gif` - GIF 이미지
- `.bmp` - BMP 이미지

### 텍스트

- `.txt` - 텍스트 파일

## 🔧 API 엔드포인트

### 1. 루트 엔드포인트

```http
GET /
```

API 상태 및 지원 파일 형식 확인

**응답 예시:**

```json
{
  "message": "Docling Document Converter API",
  "status": "running",
  "supported_formats": {...},
  "chunking": {
    "engine": "HybridChunker",
    "tokenizer": "sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2",
    "max_tokens": 512,
    "contextualize": true
  }
}
```

### 2. 상태 확인

```http
GET /health
```

서버 상태 확인

### 3. 단일 파일 변환

```http
POST /convert
```

**파라미터:**

- `file` (required): 변환할 파일
- `output_filename` (optional): 출력 파일명 (확장자 제외)
- `include_metadata` (optional): 메타데이터 포함 여부 (기본: false)
- `use_chunking` (optional): HybridChunker 청킹 적용 여부 (기본: false)
- `contextualize` (optional): 청크 컨텍스트 강화 적용 (기본: true)

**응답:** Markdown 파일 다운로드

**cURL 예시:**

```bash
curl -X POST "http://localhost:10002/convert?use_chunking=true&contextualize=true" \
  -F "file=@document.pdf" \
  -o output.md
```

### 4. 청크별 변환 (JSON 응답)

```http
POST /convert-chunked
```

**파라미터:**

- `file` (required): 변환할 파일
- `include_metadata` (optional): 청크 메타데이터 포함 (기본: false)
- `contextualize` (optional): 청크 컨텍스트 강화 적용 (기본: true)
- `max_tokens` (optional): 최대 토큰 수 (기본: 512)

**응답 예시:**

```json
{
  "success": true,
  "filename": "document.pdf",
  "file_type": "PDF 문서",
  "total_chunks": 15,
  "chunking_config": {
    "tokenizer": "sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2",
    "max_tokens": 512,
    "merge_peers": true,
    "contextualize": true
  },
  "chunks": [
    {
      "chunk_id": 1,
      "text": "원본 텍스트...",
      "contextualized_text": "컨텍스트가 강화된 텍스트...",
      "text_length": 450,
      "contextualized_length": 520,
      "page_info": [1, 2],
      "bbox_info": [...]
    }
  ]
}
```

### 5. 비동기 청킹 (대용량 파일용) ⭐ 신규

```http
POST /convert-chunked-async
```

**파라미터:**

- `file` (required): 변환할 파일
- `include_metadata` (optional): 청크 메타데이터 포함 (기본: false)
- `contextualize` (optional): 청크 컨텍스트 강화 적용 (기본: true)
- `max_tokens` (optional): 최대 토큰 수 (기본: 512)

**응답 예시:**

```json
{
  "job_id": "550e8400-e29b-41d4-a716-446655440000",
  "status": "queued",
  "message": "작업이 시작되었습니다. /job/{job_id} 엔드포인트로 상태를 확인하세요.",
  "filename": "large_document.pdf",
  "file_size": 10485760,
  "created_at": "2025-10-04T12:00:00"
}
```

**사용 이유:**

- 대용량 파일 처리 시 HTTP 타임아웃 방지
- n8n, Zapier 등 워크플로우 도구와 통합
- 긴 작업의 진행 상황 추적

### 6. 작업 상태 조회 ⭐ 신규

```http
GET /job/{job_id}
```

**응답 예시 (처리 중):**

```json
{
  "job_id": "550e8400-e29b-41d4-a716-446655440000",
  "status": "processing",
  "progress": 60,
  "message": "청킹 중...",
  "filename": "document.pdf",
  "created_at": "2025-10-04T12:00:00"
}
```

**응답 예시 (완료):**

```json
{
  "job_id": "550e8400-e29b-41d4-a716-446655440000",
  "status": "completed",
  "progress": 100,
  "message": "처리 완료",
  "filename": "document.pdf",
  "created_at": "2025-10-04T12:00:00",
  "completed_at": "2025-10-04T12:02:30",
  "result": {
    "success": true,
    "total_chunks": 15,
    "chunks": [...]
  }
}
```

**상태 값:**

- `queued`: 대기 중
- `processing`: 처리 중
- `completed`: 완료
- `failed`: 실패

### 7. 모든 작업 조회 ⭐ 신규

```http
GET /jobs
```

모든 진행 중/완료된 작업 목록 반환

### 8. 작업 삭제 ⭐ 신규

```http
DELETE /job/{job_id}
```

완료된 작업을 삭제하고 임시 파일 정리

### 9. 여러 파일 일괄 변환

```http
POST /convert-multiple
```

**파라미터:**

- `files` (required): 변환할 파일들 (최대 10개)
- `output_format` (optional): 출력 형식 - "zip" 또는 "json" (기본: zip)
- `use_chunking` (optional): HybridChunker 청킹 적용 (기본: false)
- `contextualize` (optional): 청크 컨텍스트 강화 적용 (기본: true)

**응답:** ZIP 파일 또는 JSON

**cURL 예시:**

```bash
curl -X POST "http://localhost:10002/convert-multiple?output_format=zip" \
  -F "files=@doc1.pdf" \
  -F "files=@doc2.docx" \
  -F "files=@doc3.xlsx" \
  -o converted_files.zip
```

### 10. 지원 형식 조회

```http
GET /supported-formats
```

지원되는 모든 파일 형식 및 청킹 정보 반환

### 11. 청킹 정보 조회

```http
GET /chunking-info
```

HybridChunker 설정 및 기능 정보 반환

## 🔄 비동기 워크플로우 (n8n, Zapier 등)

대용량 파일 처리 시 다음과 같은 워크플로우를 사용하세요:

### 기본 워크플로우

```
1. POST /convert-chunked-async
   ↓ (job_id 수신)
2. GET /job/{job_id} (5초마다 polling)
   ↓ (status 확인)
3. status === 'completed'
   ↓
4. result에서 청크 데이터 획득
   ↓
5. DELETE /job/{job_id} (정리)
```

### n8n 워크플로우 예시

자세한 n8n 통합 가이드는 [N8N_WORKFLOW_GUIDE.md](./N8N_WORKFLOW_GUIDE.md)를 참고하세요.

**간단 예시:**

```javascript
// 1. 작업 시작
const startResponse = await $http.post('http://localhost:10002/convert-chunked-async', {
  file: $binary.data
});
const jobId = startResponse.job_id;

// 2. 상태 polling (Loop 노드 사용)
let status = 'processing';
while (status === 'processing' || status === 'queued') {
  await new Promise(r => setTimeout(r, 5000)); // 5초 대기
  const statusResponse = await $http.get(`http://localhost:10002/job/${jobId}`);
  status = statusResponse.status;
  
  if (status === 'completed') {
    return statusResponse.result; // 완료된 청크 데이터
  } else if (status === 'failed') {
    throw new Error(statusResponse.error);
  }
}
```

### 6. 지원 형식 조회

```http
POST /convert-multiple
```

**파라미터:**

- `files` (required): 변환할 파일들 (최대 10개)
- `output_format` (optional): 출력 형식 - "zip" 또는 "json" (기본: zip)
- `use_chunking` (optional): HybridChunker 청킹 적용 (기본: false)
- `contextualize` (optional): 청크 컨텍스트 강화 적용 (기본: true)

**응답:** ZIP 파일 또는 JSON

**cURL 예시:**

```bash
curl -X POST "http://localhost:10002/convert-multiple?output_format=zip" \
  -F "files=@doc1.pdf" \
  -F "files=@doc2.docx" \
  -F "files=@doc3.xlsx" \
  -o converted_files.zip
```

### 6. 지원 형식 조회

```http
GET /supported-formats
```

지원되는 모든 파일 형식 및 청킹 정보 반환

### 7. 청킹 정보 조회

```http
GET /chunking-info
```

HybridChunker 설정 및 기능 정보 반환

## 🧩 HybridChunker 기능

### Contextualize (컨텍스트 강화)

각 청크에 문서의 전체 컨텍스트를 포함하여 독립적으로 이해 가능한 텍스트를 생성합니다.

**활용 예시:**

- RAG (Retrieval-Augmented Generation) 시스템
- 문서 검색 및 인덱싱
- 문맥 기반 질의응답 시스템

### Merge Peers (인접 청크 병합)

유사한 내용을 가진 인접 청크를 자동으로 병합하여 의미있는 단위로 관리합니다.

### Hierarchical Chunking (계층적 청킹)

문서의 구조(제목, 섹션, 단락)를 고려하여 계층적으로 청킹합니다.

## 🎯 사용 예시

### Python에서 API 호출

```python
import requests

# 단일 파일 변환
with open('document.pdf', 'rb') as f:
    files = {'file': f}
    params = {
        'use_chunking': True,
        'contextualize': True
    }
    response = requests.post('http://localhost:10002/convert', 
                           files=files, 
                           params=params)
    
    with open('output.md', 'wb') as out:
        out.write(response.content)

# 청크별 JSON 응답 받기
with open('document.pdf', 'rb') as f:
    files = {'file': f}
    response = requests.post('http://localhost:10002/convert-chunked', files=files)
    data = response.json()
    
    print(f"총 청크 수: {data['total_chunks']}")
    for chunk in data['chunks']:
        print(f"청크 {chunk['chunk_id']}: {chunk['text_length']} 문자")
        if 'page_info' in chunk:
            print(f"  페이지: {chunk['page_info']}")
```

### JavaScript/TypeScript에서 API 호출

```javascript
// 파일 업로드 및 변환
const formData = new FormData();
formData.append('file', fileInput.files[0]);

const response = await fetch('http://localhost:10002/convert?use_chunking=true', {
  method: 'POST',
  body: formData
});

const markdown = await response.text();
console.log(markdown);

// 청크별 JSON 데이터 받기
const chunkedResponse = await fetch('http://localhost:10002/convert-chunked', {
  method: 'POST',
  body: formData
});

const data = await chunkedResponse.json();
console.log(`총 ${data.total_chunks}개의 청크가 생성되었습니다.`);
```

## 🛠️ 기술 스택

- **FastAPI**: 고성능 비동기 웹 프레임워크
- **Docling**: 문서 변환 및 처리 라이브러리
- **HybridChunker**: 지능형 문서 청킹 엔진
- **Transformers**: HuggingFace 토크나이저
- **sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2**: 다국어 지원 임베딩 모델 (한글 최적화)

## ⚙️ 설정

### 토크나이저 설정

```python
EMBED_MODEL_ID = "sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2"
MAX_TOKENS = 512  # 청크당 최대 토큰 수
```

### CORS 설정

모든 출처에서의 요청을 허용하도록 설정되어 있습니다. 프로덕션 환경에서는 특정 도메인으로 제한하는 것을 권장합니다.

```python
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],  # 프로덕션에서는 특정 도메인으로 변경
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

## 📊 청크 메타데이터

각 청크는 다음과 같은 메타데이터를 포함합니다:

- **chunk_id**: 청크 번호
- **text**: 원본 텍스트
- **text_length**: 텍스트 길이
- **token_count**: 토큰 수
- **contextualized_text**: 컨텍스트가 강화된 텍스트
- **page_info**: 페이지 정보 (PDF 등)
- **sheet_names**: 시트 이름 (Excel)
- **bbox_info**: 바운딩 박스 정보 (위치 정보)
- **metadata**: 추가 메타데이터

## 🔍 특수 기능

### Excel 파일 처리

Excel 파일의 경우 시트별로 구분하여 처리하며, 각 청크에 시트 이름과 인덱스 정보를 포함합니다.

### 이미지 처리

이미지 파일은 OCR을 통해 텍스트를 추출하여 Markdown으로 변환합니다.

### 페이지 정보 추적

PDF 및 페이지 기반 문서의 경우 각 청크가 어느 페이지에서 추출되었는지 추적합니다.

## 🐛 문제 해결

### 파일 업로드 크기 제한

대용량 파일 처리 시 FastAPI의 업로드 크기 제한을 늘려야 할 수 있습니다.

### 메모리 부족

대량의 파일을 처리할 때 메모리 부족이 발생할 수 있습니다. 이 경우 파일 수를 제한하거나 서버의 메모리를 증가시키세요.

### 한글 처리

현재 `paraphrase-multilingual-MiniLM-L12-v2` 모델을 사용하여 한글 처리에 최적화되어 있습니다.

## 📝 라이선스

이 프로젝트는 관련 라이브러리의 라이선스를 따릅니다.

## 🤝 기여

버그 리포트나 기능 제안은 이슈를 통해 제출해주세요.

## 📞 지원

문의사항이 있으시면 이슈를 등록해주세요.
hellocosmos@gmail.com
---

**Version**: 1.0.0  
**Port**: 10002  
**Host**: 0.0.0.0
