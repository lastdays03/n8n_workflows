# YouTube 기술 트렌드 자동 수집 시스템

n8n을 사용하여 YouTube의 기술 트렌드를 자동으로 수집, 분류하고 AI로 요약하여 Notion에 저장하는 워크플로우입니다.

## 📋 개요

### 주요 기능
- **기술 키워드 기반 검색**: 관심 있는 기술 키워드로 YouTube 동영상 자동 검색
- **특정 채널 추적**: 팔로우하는 기술 채널의 신규 동영상 자동 수집
- **AI 요약**: OpenAI API를 활용한 동영상 핵심 내용 자동 요약
- **Notion 통합**: 수집된 데이터를 Notion 데이터베이스에 자동 저장
- **중복 방지**: URL 기반 중복 체크로 효율적인 데이터 관리
- **스케줄 실행**: 매일 자동으로 실행되어 최신 콘텐츠 수집

## 🗂️ 워크플로우 구성

### 1. `youtube-tech-trends-keywords_v2.json` (New! ✨)
**기술 키워드 기반 검색 + 무료 AI Fallback 워크플로우**

- **실행 시간**: 매일 오전 9시 (크론: `0 9 * * *`)
- **AI 엔진**: **Groq (Llama 3.3)** → 실패 시 **OpenRouter (DeepSeek)** 자동 전환
- **수집 방식**: 설정된 기술 키워드로 YouTube 검색
- **비용**: **완전 무료** ($0)

**처리 플로우**:
```
스케줄 트리거 → 설정 로드 → 키워드 분할 → YouTube 검색
→ 중복 제거 → Notion 중복 체크 → 신규 필터링
→ 자막 추출 → [Try: Groq] -(에러)-> [Catch: OpenRouter] 
→ Notion 저장 → 통계 생성 → Slack 알림
```

### 2. `youtube-channel-tracking_v2.json` (New! ✨)
**특정 채널 추적 + 무료 AI Fallback 워크플로우**

- **실행 시간**: 매일 오후 6시 (크론: `0 18 * * *`)
- **AI 엔진**: **Groq** (속도) / **OpenRouter** (안정성)
- **수집 방식**: 지정된 채널의 최신 동영상 조회
- **비용**: **완전 무료** ($0)

**처리 플로우**:
```
스케줄 트리거 → 채널 설정 로드 → 채널 분할 → 채널 동영상 조회
→ 데이터 포맷 → Notion 중복 체크 → 신규 필터링
→ 자막 추출 → [Try: Groq] -(에러)-> [Catch: OpenRouter]
→ Notion 저장 → 통계 생성 → Slack 알림
```

## 🛠️ 설치 및 설정

### 1. 사전 요구사항
 
 - **n8n 설치**
   - 클라우드 또는 셀프 호스팅 (Docker 등)
 - **YouTube Data API v3 키**
 - **Notion 계정 및 Integration**
 - **무료 AI API 키** (다음 중 1개 이상 권장):
   - **Groq API 키** (1순위 추천)
   - **OpenRouter API 키** (DeepSeek, Gemma 등 사용 시)
   - Google Gemini API 키
 
 ### 2. API 키 발급
 
 #### YouTube API
 1. [Google Cloud Console](https://console.cloud.google.com/) 접속
 2. 프로젝트 생성 또는 선택
 3. "API 및 서비스" → "라이브러리" → "YouTube Data API v3" 활성화
 4. "사용자 인증 정보" → "OAuth 2.0 클라이언트 ID" 생성
 5. n8n에서 YouTube OAuth2 연결 설정
 
 **쿼터 정보**:
 - 무료: 하루 10,000 쿼터
 - 검색 1회 = 100 쿼터
 - 하루 약 100회 검색 가능
 
 #### 🌟 Groq API (추천)
 1. [Groq Console](https://console.groq.com/keys) 접속
 2. `Create API Key` 클릭 후 키 복사
 
 #### OpenRouter API
 1. [OpenRouter](https://openrouter.ai/keys) 접속
 2. 키 생성 (Credit 없어도 Free 모델 사용 가능)
 
 #### Notion Integration
 1. [Notion Integrations](https://www.notion.so/my-integrations) 접속
 2. "New integration" 생성
 3. Integration Token 복사
 4. Notion 데이터베이스에서 "연결 추가" → Integration 연결

### 3. Notion 데이터베이스 생성

다음 속성을 가진 Notion 데이터베이스를 생성하세요:

| 속성명   | 타입         | 설명                        |
| -------- | ------------ | --------------------------- |
| 제목     | Title        | 동영상 제목                 |
| URL      | URL          | 동영상 링크                 |
| 채널     | Text         | 채널명                      |
| 등록일   | Date         | 동영상 게시일               |
| 수집일   | Date         | 수집한 날짜                 |
| 기술태그 | Multi-select | 관련 기술 카테고리          |
| AI요약   | Text         | AI가 생성한 요약            |
| 상태     | Select       | 미확인/확인완료/중요/나중에 |
| 자막여부 | Checkbox     | 자막 사용 가능 여부         |
| 수집방식 | Select       | 키워드검색/채널추적         |

**데이터베이스 ID 확인 방법**:
```
https://www.notion.so/workspace/DATABASE_ID?v=...
                              ↑ 이 부분이 DATABASE_ID
```

### 4. n8n 워크플로우 Import
 
 1. n8n 대시보드 접속
 2. "Import from File" 클릭
 3. `youtube-tech-trends-keywords_v2.json` 선택하여 import
 4. `youtube-channel-tracking_v2.json` 선택하여 import
 
 ### 5. Credentials 설정
 
 각 워크플로우에서 다음 credentials를 설정하세요:
 
 #### YouTube OAuth2
 - n8n에서 "Credentials" → "Add Credential" → "YouTube OAuth2 API"
 - Google Cloud Console에서 발급받은 Client ID/Secret 입력
 - OAuth 인증 완료
 
 #### Groq API (Header Auth)
 - n8n에서 "Credentials" → "Add Credential" → "Header Auth" (또는 "Groq API")
 - Name: `Authorization`
 - Value: `Bearer gsk_...` (키 앞에 Bearer 붙여야 함)
 
 #### OpenRouter API (Header Auth)
 - n8n에서 "Credentials" → "Add Credential" → "Header Auth"
 - Name: `Authorization`
 - Value: `Bearer sk-or-...`
 
 #### Notion API
 - n8n에서 "Credentials" → "Add Credential" → "Notion API"
 - Integration Token 입력

### 6. 워크플로우 설정 수정

#### `youtube-tech-trends-keywords.json`

**Configuration 노드**에서 검색 키워드 수정:
```javascript
{
  "keywords": [
    "kubernetes",
    "docker",
    "terraform",
    "AWS",
    "React",
    // 원하는 기술 키워드 추가
  ],
  "daysToSearch": 7,        // 검색 범위 (일)
  "maxResultsPerKeyword": 5  // 키워드당 최대 결과 수
}
```

**Notion 관련 노드**에서 DATABASE_ID 수정:
- "Check Notion for Duplicates" 노드
- "Create Notion Page" 노드
- `databaseId` 필드에 자신의 Notion 데이터베이스 ID 입력

#### `youtube-channel-tracking.json`

**Channel Configuration 노드**에서 추적할 채널 설정:
```javascript
{
  "channels": [
    {
      "id": "UCUpJs89fSBXNolQGOYKn0YQ",    // YouTube 채널 ID
      "name": "Traversy Media",           // 채널명
      "tags": ["Web Dev", "JavaScript"]   // 태그
    },
    {
      "id": "UC8butISFwT-Wl7EV0hUK0BQ",
      "name": "freeCodeCamp",
      "tags": ["Programming", "Tutorial"]
    }
    // 추가 채널
  ],
  "daysToCheck": 3,           // 확인 범위 (일)
  "maxVideosPerChannel": 10   // 채널당 최대 동영상 수
}
```

**YouTube 채널 ID 찾는 방법**:
1. 채널 페이지 접속
2. URL 확인: `youtube.com/channel/CHANNEL_ID` 또는 `youtube.com/@username`
3. `@username` 형식인 경우:
   - 페이지 소스 보기 (Ctrl+U)
   - `"channelId":"` 검색
   - 따옴표 안의 ID 복사

### 7. 스케줄 조정 (선택사항)

크론 표현식을 수정하여 실행 시간을 변경할 수 있습니다:

```
0 9 * * *   # 매일 오전 9시
0 */6 * * * # 6시간마다
0 18 * * 1  # 매주 월요일 오후 6시
```

## 🚀 실행 및 테스트

### 수동 테스트
1. 워크플로우 에디터에서 "Execute Workflow" 클릭
2. 실행 로그 확인
3. Notion 데이터베이스에 데이터가 추가되었는지 확인

### 자동 실행 활성화
1. 워크플로우 우측 상단 토글을 "Active"로 설정
2. 설정된 스케줄에 따라 자동 실행됨

### 실행 로그 확인
- n8n 대시보드 → "Executions" 탭
- 성공/실패 여부 및 상세 로그 확인

## 🎛️ 커스터마이징

### AI 요약 프롬프트 변경

**AI Summary (OpenAI)** 노드의 프롬프트를 수정:

```javascript
{
  "role": "user",
  "content": `아래 유튜브 동영상을 분석해주세요:

제목: {{ $json.title }}
채널: {{ $json.channelTitle }}
설명: {{ $json.description }}
{{ $json.hasTranscript ? '자막: ' + $json.transcriptText : '' }}

다음 형식으로 한국어로 요약해주세요:

1. 핵심 내용 (3-5문장)
2. 주요 기술/키워드
3. 난이도 (초급/중급/고급)
4. 실무 적용도 (상/중/하)
5. 추천 대상`
}
```

### Claude API 사용

OpenAI 대신 Claude API를 사용하려면:

1. **AI Summary (OpenAI)** 노드 삭제
2. **HTTP Request** 노드 추가
3. Claude API 엔드포인트 설정:
   - Method: POST
   - URL: `https://api.anthropic.com/v1/messages`
   - Headers:
     - `x-api-key`: YOUR_CLAUDE_API_KEY
     - `anthropic-version`: `2023-06-01`
     - `content-type`: `application/json`
   - Body (JSON):
     ```json
     {
       "model": "claude-3-5-sonnet-20241022",
       "max_tokens": 500,
       "messages": [{
         "role": "user",
         "content": "={{ /* 프롬프트 */ }}"
       }]
     }
     ```

### Slack 알림 활성화

1. **Slack Notification** 노드의 `disabled` 속성 제거 또는 `false`로 변경
2. Slack credentials 설정:
   - Slack App 생성: https://api.slack.com/apps
   - OAuth & Permissions → Bot Token Scopes 추가: `chat:write`
   - Install App to Workspace
   - Bot User OAuth Token을 n8n에 입력
3. `channel` 필드를 원하는 채널명으로 변경

## 🔧 문제 해결

### YouTube API 쿼터 초과
- 검색 키워드 수 줄이기
- `maxResultsPerKeyword` 값 낮추기
- 실행 주기 늘리기 (매일 → 이틀에 한 번)

### AI API 비용 절감
- 자막이 없는 경우 요약 건너뛰기
- 프롬프트 길이 줄이기
- 더 저렴한 모델 사용 (GPT-4o-mini, Claude Haiku)

### 중복 데이터 발생
- Notion 필터링이 제대로 작동하는지 확인
- URL 속성이 Notion DB에 제대로 설정되었는지 확인

### 자막 추출 실패
- 많은 동영상이 자막을 제공하지 않음 (정상 동작)
- `continueOnFail: true` 설정으로 자막 없어도 계속 진행
- 제목/설명만으로도 AI 요약 가능

## 📊 데이터 활용 팁

### Notion 뷰 추천
1. **미확인 뷰**: 상태 = "미확인", 수집일 내림차순
2. **중요 콘텐츠**: 상태 = "중요", 등록일 내림차순
3. **기술별 뷰**: 기술태그로 그룹화
4. **채널별 뷰**: 채널로 그룹화

### 정기 리뷰 프로세스
1. 매일 아침 "미확인" 뷰 확인
2. 중요한 콘텐츠는 상태를 "중요"로 변경
3. 나중에 볼 것은 "나중에"로 표시
4. 불필요한 항목은 삭제 또는 "확인완료"로 표시

## 💡 향후 개선 아이디어

- **Reddit/HackerNews 통합**: 기술 커뮤니티 트렌드도 수집
- **트렌드 분석 대시보드**: 가장 많이 언급되는 기술 시각화
- **이메일 다이제스트**: 주간 요약 이메일 자동 발송
- **태그 자동 분류**: AI로 기술 카테고리 자동 분류
- **중복 콘텐츠 통합**: 같은 주제를 다룬 여러 동영상 그룹화

## 📝 라이센스

이 프로젝트는 자유롭게 사용, 수정, 배포할 수 있습니다.

## 🤝 기여

개선 사항이나 버그 리포트는 언제든 환영합니다!

---

**작성일**: 2026-01-14
**버전**: 1.0.0
