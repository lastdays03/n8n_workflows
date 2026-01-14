# n8n YouTube 기술 트렌드 수집 시스템 설정 체크리스트

워크플로우를 성공적으로 실행하기 위한 단계별 체크리스트입니다.

## ✅ 1단계: 사전 준비

- [ ] n8n 설치 완료
  - [ ] 클라우드 버전 (https://n8n.io) 또는
  - [ ] 셀프 호스팅 (`npx n8n` 또는 Docker)
- [ ] Notion 계정 생성
- [ ] Google 계정 (YouTube API용)
- [ ] OpenAI 계정 (또는 Claude 계정)

## ✅ 2단계: API 키 발급

### YouTube Data API v3
- [ ] [Google Cloud Console](https://console.cloud.google.com/) 접속
- [ ] 새 프로젝트 생성 또는 기존 프로젝트 선택
- [ ] "API 및 서비스" → "라이브러리" 이동
- [ ] "YouTube Data API v3" 검색 및 활성화
- [ ] "사용자 인증 정보" → "OAuth 2.0 클라이언트 ID" 생성
  - [ ] 애플리케이션 유형: "웹 애플리케이션"
  - [ ] 승인된 리디렉션 URI 추가: n8n의 OAuth 콜백 URL
- [ ] Client ID와 Client Secret 복사하여 저장

### OpenAI API
- [ ] [OpenAI Platform](https://platform.openai.com/) 접속
- [ ] "API keys" 메뉴 이동
- [ ] "Create new secret key" 클릭
- [ ] API 키 복사하여 안전하게 저장
- [ ] (선택) 사용량 제한 설정 (Settings → Billing → Usage limits)

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

| 완료 | 속성명 | 타입 | 설정 |
|------|--------|------|------|
| [ ] | 제목 | Title | 기본 속성 |
| [ ] | URL | URL | - |
| [ ] | 채널 | Text | - |
| [ ] | 등록일 | Date | - |
| [ ] | 수집일 | Date | - |
| [ ] | 기술태그 | Multi-select | - |
| [ ] | AI요약 | Text | - |
| [ ] | 상태 | Select | 옵션: 미확인, 확인완료, 중요, 나중에 |
| [ ] | 자막여부 | Checkbox | - |
| [ ] | 수집방식 | Select | 옵션: 키워드검색, 채널추적 |

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
- [ ] `youtube-tech-trends-keywords.json` 파일 선택하여 import
- [ ] "Import from File" 다시 클릭
- [ ] `youtube-channel-tracking.json` 파일 선택하여 import

## ✅ 5단계: n8n Credentials 설정

### YouTube OAuth2
- [ ] n8n에서 "Credentials" 메뉴 이동
- [ ] "Add Credential" 클릭
- [ ] "YouTube OAuth2 API" 검색 및 선택
- [ ] Client ID 입력
- [ ] Client Secret 입력
- [ ] "Sign in with Google" 클릭하여 OAuth 인증 완료
- [ ] Credential 이름 저장 (예: "YouTube OAuth2 account")

### OpenAI API
- [ ] "Add Credential" → "OpenAI API" 선택
- [ ] API Key 입력
- [ ] Credential 이름 저장 (예: "OpenAI account")

### Notion API
- [ ] "Add Credential" → "Notion API" 선택
- [ ] Internal Integration Token 입력
- [ ] Credential 이름 저장 (예: "Notion account")

### Slack (선택사항)
- [ ] [Slack API](https://api.slack.com/apps) 접속
- [ ] "Create New App" → "From scratch"
- [ ] App 이름 및 Workspace 선택
- [ ] "OAuth & Permissions" → "Bot Token Scopes" 추가:
  - [ ] `chat:write`
  - [ ] `chat:write.public` (선택)
- [ ] "Install to Workspace" 클릭
- [ ] "Bot User OAuth Token" 복사
- [ ] n8n에서 "Add Credential" → "Slack API"
- [ ] OAuth Token 입력

## ✅ 6단계: 워크플로우 설정 수정

### `youtube-tech-trends-keywords.json`

- [ ] 워크플로우 에디터 열기
- [ ] **"Configuration"** 노드 클릭
- [ ] `keywords` 배열에 원하는 기술 키워드 추가/수정
  ```javascript
  ["kubernetes", "docker", "terraform", ...]
  ```
- [ ] **"YouTube Search"** 노드 클릭
- [ ] Credential 선택: 앞서 생성한 YouTube OAuth2 credential
- [ ] **"Check Notion for Duplicates"** 노드 클릭
- [ ] `databaseId` 필드에 Notion DATABASE_ID 입력
- [ ] Credential 선택: 앞서 생성한 Notion credential
- [ ] **"AI Summary (OpenAI)"** 노드 클릭
- [ ] Credential 선택: 앞서 생성한 OpenAI credential
- [ ] **"Create Notion Page"** 노드 클릭
- [ ] `databaseId` 필드에 Notion DATABASE_ID 입력
- [ ] Credential 선택: Notion credential

### `youtube-channel-tracking.json`

- [ ] 워크플로우 에디터 열기
- [ ] **"Channel Configuration"** 노드 클릭
- [ ] `channels` 배열에 추적할 채널 정보 입력
  ```javascript
  [
    {
      "id": "채널_ID",
      "name": "채널명",
      "tags": ["태그1", "태그2"]
    }
  ]
  ```
- [ ] YouTube 채널 ID 찾기:
  - [ ] 채널 페이지 접속
  - [ ] 페이지 소스 보기 (Ctrl+U / Cmd+Option+U)
  - [ ] `"channelId":"` 검색하여 ID 복사
- [ ] 모든 노드의 Credentials 설정 (위와 동일)

## ✅ 7단계: 테스트 실행

### 키워드 검색 워크플로우
- [ ] `youtube-tech-trends-keywords` 워크플로우 열기
- [ ] 우측 상단 "Execute Workflow" 클릭
- [ ] 실행 완료까지 대기 (1-3분 소요)
- [ ] 실행 로그에서 오류 확인
- [ ] Notion 데이터베이스에 데이터 추가되었는지 확인

### 채널 추적 워크플로우
- [ ] `youtube-channel-tracking` 워크플로우 열기
- [ ] "Execute Workflow" 클릭
- [ ] 실행 결과 확인
- [ ] Notion에서 "수집방식 = 채널추적" 데이터 확인

## ✅ 8단계: 스케줄 활성화

- [ ] 두 워크플로우 모두 테스트 성공 확인
- [ ] 각 워크플로우에서 우측 상단 토글을 "Active"로 설정
- [ ] "Executions" 탭에서 스케줄 실행 확인

## ✅ 9단계: 모니터링 설정 (선택사항)

### Slack 알림 활성화
- [ ] 워크플로우에서 "Slack Notification" 노드 찾기
- [ ] `disabled: true` 제거 또는 노드 활성화
- [ ] Slack Credential 설정
- [ ] `channel` 필드를 원하는 채널명으로 변경
- [ ] 테스트 실행으로 알림 확인

### 오류 알림 워크플로우
- [ ] n8n Settings → "Error Workflow" 설정
- [ ] 오류 발생 시 Slack/이메일로 알림 받기

## ✅ 10단계: 최적화 및 유지보수

### 비용 최적화
- [ ] OpenAI 사용량 모니터링
- [ ] YouTube API 쿼터 사용량 확인
- [ ] 필요시 검색 키워드/채널 수 조정

### 데이터 관리
- [ ] Notion에서 정기적으로 데이터 리뷰
- [ ] 불필요한 데이터 정리
- [ ] 중요한 콘텐츠 별도 표시

### 워크플로우 개선
- [ ] AI 요약 품질 확인 및 프롬프트 조정
- [ ] 새로운 기술 키워드 추가
- [ ] 관심 채널 업데이트

## 🐛 문제 해결 체크리스트

### YouTube API 오류
- [ ] Google Cloud Console에서 API 활성화 확인
- [ ] OAuth 인증 토큰 유효성 확인
- [ ] 쿼터 초과 여부 확인 (일일 10,000)

### OpenAI API 오류
- [ ] API 키 유효성 확인
- [ ] 계정 잔액 확인
- [ ] 요청 속도 제한 확인

### Notion 연동 오류
- [ ] Integration이 데이터베이스에 연결되었는지 확인
- [ ] DATABASE_ID가 정확한지 확인
- [ ] 속성명이 정확히 일치하는지 확인

### 데이터 중복
- [ ] Notion "Check for Duplicates" 노드의 필터 확인
- [ ] URL 속성이 제대로 설정되었는지 확인

## 📚 참고 자료

- [ ] [n8n 공식 문서](https://docs.n8n.io/)
- [ ] [YouTube Data API 문서](https://developers.google.com/youtube/v3)
- [ ] [OpenAI API 문서](https://platform.openai.com/docs)
- [ ] [Notion API 문서](https://developers.notion.com/)
- [ ] 이 프로젝트의 `README.md` 파일

---

모든 체크리스트를 완료하셨다면 시스템이 정상적으로 작동할 것입니다! 🎉
