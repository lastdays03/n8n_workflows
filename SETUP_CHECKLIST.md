# n8n YouTube 기술 트렌드 수집 시스템 (v2) 설정 체크리스트

무료 AI 모델(Groq, OpenRouter)과 Fallback 로직이 적용된 v2 워크플로우를 성공적으로 실행하기 위한 단계별 체크리스트입니다.

## ✅ 1단계: 사전 준비

- [ ] n8n 설치 완료
  - [ ] 클라우드 버전 (https://n8n.io) 또는
  - [ ] 셀프 호스팅 (`npx n8n` 또는 Docker)
- [ ] Notion 계정 생성
- [ ] Google 계정 (YouTube API용)
- [ ] **무료 AI API 계정** (Groq 및 OpenRouter)

## ✅ 2단계: API 키 발급

### YouTube Data API v3
- [ ] [Google Cloud Console](https://console.cloud.google.com/) 접속
- [ ] 새 프로젝트 생성 또는 기존 프로젝트 선택
- [ ] "API 및 서비스" → "라이브러리" 이동
- [ ] "YouTube Data API v3" 검색 및 활성화
- [ ] "사용자 인증 정보" → "OAuth 2.0 클라이언트 ID" 생성
  - [ ] 애플리케이션 유형: "웹 애플리케이션"
  - [ ] 승인된 리디렉션 URI 추가: n8n의 OAuth 콜백 URL (예: `http://localhost:5678/rest/oauth2-credential/callback`)
- [ ] Client ID와 Client Secret 복사하여 저장

### 🌟 Groq API (Primary)
- [ ] [Groq Console](https://console.groq.com/keys) 접속
- [ ] "Create API Key" 클릭
- [ ] 생성된 키(`gsk_...`) 복사하여 저장
- [ ] (참고) 하루 1,000건, 분당 30건 무료 (Llama 3.3 70B 기준)

### 🛡️ OpenRouter API (Fallback)
- [ ] [OpenRouter Keys](https://openrouter.ai/keys) 접속
- [ ] "Create Key" 클릭
- [ ] 생성된 키(`sk-or-...`) 복사하여 저장
- [ ] (참고) DeepSeek R1 등 무료 모델 사용 가능

### Notion Integration
- [ ] [Notion Integrations](https://www.notion.so/my-integrations) 접속
- [ ] "New integration" 클릭
- [ ] Integration 이름 입력 (예: "n8n YouTube Collector")
- [ ] "Submit" 클릭
- [ ] "Internal Integration Token" 복사하여 저장

## ✅ 3단계: Notion 데이터베이스 설정

- [ ] Notion에서 새 페이지 생성
- [ ] 데이터베이스 생성 (Table 선택)
- [ ] 다음 속성 추가:

| 완료 | 속성명   | 타입         | 설정                                 |
| ---- | -------- | ------------ | ------------------------------------ |
| [ ]  | 제목     | Title        | 기본 속성                            |
| [ ]  | URL      | URL          | -                                    |
| [ ]  | 채널     | Text         | -                                    |
| [ ]  | 등록일   | Date         | -                                    |
| [ ]  | 수집일   | Date         | -                                    |
| [ ]  | 기술태그 | Multi-select | -                                    |
| [ ]  | AI요약   | Text         | -                                    |
| [ ]  | 상태     | Select       | 옵션: 미확인, 확인완료, 중요, 나중에 |
| [ ]  | 자막여부 | Checkbox     | -                                    |
| [ ]  | 수집방식 | Select       | 옵션: 키워드검색, 채널추적           |

- [ ] 데이터베이스 우측 상단 "..." → "Add connections"
- [ ] 생성한 Integration 선택하여 연결
- [ ] 데이터베이스 URL에서 DATABASE_ID 복사
  ```
  https://www.notion.so/workspace/DATABASE_ID?v=...
  ```

## ✅ 4단계: n8n 워크플로우 Import

- [ ] n8n 대시보드 접속
- [ ] 좌측 메뉴에서 "Workflows" 선택
- [ ] "Import from File" 클릭
- [ ] `youtube-tech-trends-keywords_v2.json` 파일 선택하여 import
- [ ] "Import from File" 다시 클릭
- [ ] `youtube-channel-tracking_v2.json` 파일 선택하여 import

## ✅ 5단계: n8n Credentials 설정

### YouTube OAuth2
- [ ] n8n에서 "Credentials" 메뉴 이동
- [ ] "Add Credential" 클릭
- [ ] "YouTube OAuth2 API" 검색 및 선택
- [ ] Client ID, Client Secret 입력
- [ ] "Sign in with Google" 클릭하여 OAuth 인증 완료
- [ ] Credential 이름 저장 (예: "YouTube OAuth2 account")

### Groq API (Header Auth)
- [ ] "Add Credential" → "Header Auth" 선택 (없으면 Generic Header Auth)
- [ ] Name: `Authorization`
- [ ] Value: `Bearer gsk_YOUR_GROQ_KEY` (앞에 Bearer 필수)
- [ ] Credential 이름 저장 (예: "Groq API Key")

### OpenRouter API (Header Auth)
- [ ] "Add Credential" → "Header Auth" 선택
- [ ] Name: `Authorization`
- [ ] Value: `Bearer sk-or-YOUR_OPENROUTER_KEY`
- [ ] Credential 이름 저장 (예: "OpenRouter API Key")

### Notion API
- [ ] "Add Credential" → "Notion API" 선택
- [ ] Internal Integration Token 입력
- [ ] Credential 이름 저장 (예: "Notion account")

### Slack (선택사항)
- [ ] Slack App 생성 및 OAuth Token 발급 (`chat:write` 권한)
- [ ] "Slack API" Credential 추가

## ✅ 6단계: 워크플로우 설정 수정

### `youtube-tech-trends-keywords_v2.json`

- [ ] 워크플로우 에디터 열기
- [ ] **"Configuration"** 노드 클릭
  - [ ] `keywords` 배열 수정 (원하는 기술 키워드 입력)
- [ ] **"YouTube Search"** 노드 클릭
  - [ ] Credential 선택: YouTube OAuth2 account
- [ ] **"Check Notion for Duplicates"** 노드 클릭
  - [ ] `databaseId` 입력, Credential 선택
- [ ] **"Create Notion Page"** 노드 클릭
  - [ ] `databaseId` 입력, Credential 선택
- [ ] **"Primary: Groq (Llama 3.3)"** 노드 클릭
  - [ ] Credential 선택: Groq API Key
- [ ] **"Fallback: OpenRouter (DeepSeek)"** 노드 클릭
  - [ ] Credential 선택: OpenRouter API Key

### `youtube-channel-tracking_v2.json`

- [ ] 워크플로우 에디터 열기
- [ ] **"Channel Configuration"** 노드 클릭
  - [ ] `channels` 배열 수정 (추적할 채널 ID 및 이름)
    ```javascript
    [
      { "id": "UC...", "name": "채널명", "tags": ["태그"] }
    ]
    ```
- [ ] 모든 노드의 Credentials 및 Database ID 설정 (위와 동일)

## ✅ 7단계: 테스트 실행

### 키워드 검색 워크플로우
- [ ] `youtube-tech-trends-keywords_v2` 워크플로우 열기
- [ ] "Execute Workflow" 클릭
- [ ] 실행 완료 대기 및 로그 확인
- [ ] Notion 데이터베이스에 데이터 추가 확인

### 채널 추적 워크플로우
- [ ] `youtube-channel-tracking_v2` 워크플로우 열기
- [ ] "Execute Workflow" 클릭
- [ ] Notion 데이터 추가 확인

## ✅ 8단계: 스케줄 활성화

- [ ] 두 워크플로우 모두 테스트 성공 확인
- [ ] 각 워크플로우 우측 상단 토글을 "Active"로 설정

## 🐛 문제 해결 체크리스트

### AI 모델 오류 (Groq/OpenRouter)
- [ ] Header Auth 설정 시 `Bearer ` 접두사가 포함되었는지 확인
- [ ] Groq API 키가 유효한지 Console에서 확인
- [ ] OpenRouter 키가 유효한지 확인
- [ ] "Try (Fallback)" 노드가 에러를 제대로 잡아내는지 테스트

### YouTube/Notion 오류
- [ ] 기존 v1 체크리스트와 동일하게 쿼터, 토큰, ID 확인

## 📚 참고 자료
- [ ] `AI_MODEL_SWITCHING_GUIDE.md` (자세한 모델 전환 가이드)
- [ ] `README.md`
