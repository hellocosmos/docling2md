Ultimate Hybrid RAG on n8n

n8n에서 semantic rag와 sparse rag를 결합한 rag를 소개합니다. 
제가 업무적으로 사용하고 있는데, 중소기업 정도에서 업무적으로 충분히 활용가능한 수준입니다. 
n8n에서 가능한 rag기법중 가장 효과적으로 구축된 워크플로우일겁니다.!

아시다시피, 임베딩시에 데이터 전처리를 하는게 가장 중요한 일중에 하나인데.. 이건 docling을 사용했습니다. 
docling은 문맥 구조를 이해하게 해서 어느정도 데이터 파편화 문제를 줄여줍니다.  제 레포(https://github.com/hellocosmos/docling2md) 를 참고하시면 됩니다. 

## 주요 기능
1. 자동 문서 처리 파이프라인**
 - Google Drive 파일 생성/수정 시 자동 감지
 - 파일 해시 비교로 중복 처리 방지
 - Docling 기반 청킹으로 문맥 유지
 - PDF/Excel/DOCX 메타데이터 자동 추출
2. Hybrid Search (RRF 알고리즘)
 - Supabase 벡터 검색 + PostgreSQL FTS 결합
 - OpenAI text-embedding-3-small 활용
 - 섹션 번호 기반 정확한 출처 표기
 - 가중치 조합으로 검색 정확도 향상
3. 다국어 QnA 챗봇**
 - GPT-4.1-mini 기반 AI Agent
 - 질문 언어 자동 감지 및 동일 언어 응답
 - 구조화된 답변 템플릿 (설치/설정/문제해결)
 - 웹훅/워크플로우 트리거 지원

------
기업 매뉴얼, FAQ, 기술문서 검색에 즉시 적용 가능한 실전 워크플로우입니다! 
모쪼록 관계자분께 도움이 되었으면 합니다.!
#n8n #RAG #AI #Automation #VectorDB #Supabase #ChatGPT #워크플로우자동화
